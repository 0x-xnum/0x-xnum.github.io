# Uncaught Exceptions and Improper Recovery

Error handling failures occur when applications don't properly catch, handle, or recover from exceptions and error conditions. This includes:

* Uncaught Exception
* Unchecked Return Value
* Detection of Error Condition Without Action
* Unchecked Error Condition
* Unexpected Status Code or Return Value
* Declaration of Catch for Generic Exception
* Declaration of Throws for Generic Exception
* Improper Cleanup on Thrown Exception
* &#x20;Improper Check or Handling of Exceptional Conditions
* Improper Check for Unusual or Exceptional Conditions
* &#x20;Improper Handling of Exceptional Conditions

When exceptions aren't handled, applications crash, leak information, enter inconsistent states, or allow attacks to proceed unchecked.

***

### Real-World Attack Scenarios

#### Scenario 1: Uncaught Exception Leading to Crash (DoS)

An application processes user file uploads without exception handling:

```python
def process_upload(file):
    data = file.read()  # No try/except
    parsed = json.loads(data)  # Could fail
    
    user_id = parsed['user_id']  # Could raise KeyError
    product_id = parsed['product_id']  # Could raise KeyError
    
    db.insert('orders', user_id, product_id)  # Could fail
    return {"status": "success"}
```

**The vulnerability:**

No exception handling. Any error crashes the application:

* Invalid JSON → `json.JSONDecodeError`
* Missing key → `KeyError`
* Database error → `DatabaseException`

**The attack:**

Attacker sends malformed JSON:

```bash
curl -X POST http://target.com/upload \
  -F "file=@malformed.json"

# File contains: {invalid json content}
```

Application crashes:

```
Traceback (most recent call last):
  File "app.py", line 45, in process_upload
    parsed = json.loads(data)
json.JSONDecodeError: Expecting value: line 1 column 1

Application exits
```

**Result:**

* Application crashes
* Service unavailable (DoS)
* User requests fail
* Revenue impact

**The fix:**

Catch and handle exceptions:

```python
def process_upload(file):
    try:
        data = file.read()
        parsed = json.loads(data)
        
        user_id = parsed.get('user_id')
        product_id = parsed.get('product_id')
        
        if not user_id or not product_id:
            return {"error": "Missing required fields"}, 400
        
        db.insert('orders', user_id, product_id)
        return {"status": "success"}
        
    except json.JSONDecodeError:
        return {"error": "Invalid JSON"}, 400
    except DatabaseException as e:
        logger.error(f"Database error: {e}")
        return {"error": "Database error"}, 500
    except Exception as e:
        logger.error(f"Unexpected error: {e}")
        return {"error": "Server error"}, 500
```

**Finding it:** Send malformed input. Try to crash the application. Monitor for exceptions in logs.

***

#### Scenario 2: Unchecked Return Value (Silent Failure)

An application doesn't check if critical operations succeeded:

```python
def save_user(user_data):
    # No error checking on database operation
    db.insert('users', user_data)  # Returns bool, but not checked
    
    # Send welcome email
    email.send(user_data['email'], 'Welcome!')  # Returns bool
    
    # Update cache
    cache.set(f"user:{user_data['id']}", user_data)  # Might fail
    
    # All assumed to succeed!
    return {"status": "User created"}
```

**The vulnerability:**

Return values not checked. Operations might fail silently:

* Database insert fails → User not created, but response says success
* Email fails → User not notified, but response says email sent
* Cache fails → Cache inconsistent with database

**The attack:**

Attacker creates 1000 accounts in rapid succession. Database transaction pool exhausted, insert fails:

* Response: "User created successfully"
* Reality: User not actually created
* Attacker receives success notification but account doesn't exist
* Next login attempt fails (unexpected)

Or worse, if database fails randomly due to load:

* Some users created, some not
* Application thinks all created
* Database state corrupted

**Result:**

* Inconsistent state
* Data corruption
* User confusion
* Security checks bypassed (user created according to app, but not in DB)

**The fix:**

Check all return values:

```python
def save_user(user_data):
    try:
        # Check if insert succeeded
        user_id = db.insert('users', user_data)
        if not user_id:
            raise Exception("Failed to create user")
        
        # Check if email sent
        if not email.send(user_data['email'], 'Welcome!'):
            logger.error(f"Failed to send welcome email for user {user_id}")
            # Decide: fail the whole operation, or just log?
            # Depends on requirements
        
        # Check cache operation
        if not cache.set(f"user:{user_data['id']}", user_data):
            logger.warning(f"Failed to cache user {user_id}")
        
        return {"status": "User created", "user_id": user_id}
        
    except DatabaseException as e:
        logger.error(f"Failed to create user: {e}")
        return {"error": "Failed to create user"}, 500
```

**Finding it:** Check code for function calls where return value is ignored. Look for operations that might fail but assume success. Test with resource exhaustion.

***

#### Scenario 3: Generic Exception Catching (Hiding Real Errors)

Application catches all exceptions generically:

```python
def process_payment(amount, card):
    try:
        # Payment processing
        result = payment_gateway.charge(amount, card)
        return result
        
    except Exception:
        # Generic catch - hides all errors!
        return {"status": "success"}  # Always returns success
```

**The vulnerability:**

Catching `Exception` broadly hides all errors:

* Payment fails → Returns success anyway
* Invalid card → Returns success
* Network error → Returns success
* Timeout → Returns success

**The attack:**

Attacker:

1. Sends payment request with invalid card
2. Payment fails with exception
3. Exception caught generically
4. Application returns "success"
5. Attacker charged? Not charged? Unknown!
6. Customer charged multiple times or not at all

**Result:**

* Financial inconsistency
* Revenue loss
* Customer charge disputes
* Payment system confused

**The fix:**

Catch specific exceptions:

```python
def process_payment(amount, card):
    try:
        result = payment_gateway.charge(amount, card)
        
        if not result['success']:
            return {"error": "Payment declined"}, 400
            
        return {"status": "success", "transaction_id": result['id']}
        
    except InvalidCardException:
        return {"error": "Invalid card"}, 400
    except InsufficientFundsException:
        return {"error": "Insufficient funds"}, 400
    except PaymentTimeoutException:
        # Log for manual review
        logger.error(f"Payment timeout for ${amount}")
        return {"error": "Payment processing timeout"}, 500
    except PaymentGatewayException as e:
        logger.error(f"Payment gateway error: {e}")
        return {"error": "Payment processing error"}, 500
```

**Finding it:** Search for bare `except:` or `except Exception:`. Look for code that catches broad exceptions and doesn't log/handle them.

***

#### Scenario 4: Missing Cleanup on Exception (Resource Leak)

File not closed if exception occurs:

```python
def read_sensitive_file():
    file = open('/etc/passwd', 'r')
    data = file.read()  # If exception here, file never closes
    file.close()  # Unreachable if exception above
    return data
```

**The vulnerability:**

If exception occurs before `file.close()`:

* File descriptor remains open
* Resource leaked
* Repeated calls leak more file descriptors
* Eventually: "Too many open files" error
* Application crashes (DoS)

**The attack:**

Attacker triggers the exception repeatedly:

```bash
# Send requests that trigger exception in file reading
for i in {1..10000}; do
  curl http://target.com/read-file &
done

# File descriptors leak with each request
# Eventually: "Too many open files"
# Application crashes
```

**Result:**

* Resource exhaustion
* Denial of service
* Application crash

**The fix:**

Use try/finally or context manager:

```python
# Option 1: Try/finally
def read_sensitive_file():
    file = open('/etc/passwd', 'r')
    try:
        data = file.read()
        return data
    finally:
        file.close()  # Guaranteed to execute

# Option 2: Context manager (recommended)
def read_sensitive_file():
    with open('/etc/passwd', 'r') as file:
        data = file.read()
        return data
    # File automatically closed
```

**Finding it:** Look for files/resources opened without try/finally or context managers. Check for multiple cleanup statements (not guaranteed if exception occurs).

***

#### Scenario 5: Unchecked Error, Code Continues (Privilege Escalation)

Critical security check fails but code continues:

```python
def admin_action(user_id, action):
    # Check if user is admin
    admin_check = check_if_admin(user_id)  # Returns bool or raises exception
    
    # BUG: Exception not caught, code continues anyway
    
    # Perform admin action
    if action == 'delete_user':
        db.delete_user(...)
    elif action == 'modify_user':
        db.update_user(...)
    
    return {"status": "action completed"}
```

**The vulnerability:**

If `check_if_admin()` raises exception:

* Exception propagates
* Code continues execution
* Admin action performed despite failed check
* Security control bypassed

**The attack:**

Attacker triggers exception in admin check:

```bash
curl -X POST http://target.com/admin/action \
  -d '{"user_id": "invalid_type", "action": "delete_user"}'

# check_if_admin() fails with TypeError
# Exception propagates
# But code continues
# Attacker's delete_user request executed!
```

**Result:**

* Privilege escalation
* Unauthorized admin actions
* Data deletion
* Complete security bypass

**The fix:**

Catch exceptions and fail securely:

```python
def admin_action(user_id, action):
    try:
        # Check if user is admin
        is_admin = check_if_admin(user_id)
        
        if not is_admin:
            return {"error": "Unauthorized"}, 403
        
        # Perform admin action only if admin check passed
        if action == 'delete_user':
            db.delete_user(...)
        elif action == 'modify_user':
            db.update_user(...)
        
        return {"status": "action completed"}
        
    except AuthenticationException:
        return {"error": "Authorization failed"}, 403
    except Exception as e:
        logger.error(f"Admin action error: {e}")
        return {"error": "Action failed"}, 500
```

**Finding it:** Look for security checks that aren't wrapped in try/except. Check if authorization can fail silently. Test with invalid input to authorization functions.

***

#### Scenario 6: Unexpected Status Code Not Checked

API call doesn't check status code:

```python
def get_user_data(user_id):
    # Call external API
    response = requests.get(f"http://api.example.com/users/{user_id}")
    
    # No status code check!
    data = response.json()  # Assumes success
    
    return data['name']  # Assumes 'name' field exists
```

**The vulnerability:**

Status code not checked:

* 404 → User not found, but continues
* 401 → Authentication failed, but continues
* 500 → Server error, but continues
* 403 → Forbidden, but continues

**The attack:**

Attacker requests data for user they shouldn't access:

```
GET /api/users/123  (not their user)
```

External API returns 403 Forbidden:

```json
{"error": "Forbidden"}
```

Application code continues anyway:

```python
data = response.json()  # {"error": "Forbidden"}
return data['name']  # KeyError! Crash or unexpected behavior
```

**Result:**

* Crash or undefined behavior
* Data access validation bypassed
* Authorization checks ignored

**The fix:**

Check status codes:

```python
def get_user_data(user_id):
    response = requests.get(f"http://api.example.com/users/{user_id}")
    
    # Check status code
    if response.status_code == 404:
        raise UserNotFoundException()
    elif response.status_code == 403:
        raise UnauthorizedException()
    elif response.status_code >= 400:
        raise APIException(f"API returned {response.status_code}")
    
    # Only now is it safe to parse
    data = response.json()
    
    if 'name' not in data:
        raise APIException("Invalid API response")
    
    return data['name']
```

**Finding it:** Search for `response.json()` without status code check. Look for API calls not validating response. Test with error responses.

***

### Mitigation Strategies

**Catch specific exceptions**

```python
try:
    # Code
except SpecificException as e:
    # Handle specific case
except AnotherException as e:
    # Handle another case
except Exception as e:
    # Generic fallback
```

**Check all return values**

```python
result = function_call()
if not result or result.status_code>= 400:
    # Handle error
```

**Use finally for cleanup**

```python
file = None
try:
    file = open('data.txt')
    data = file.read()
finally:
    if file:
        file.close()  # Guaranteed execution
```

**Or use context managers**

```python
with open('data.txt') as file:
    data = file.read()  # Auto-closes
```

**Fail securely**

```python
# If security check fails, deny access
try:
    is_authorized = check_authorization()
except:
    # On error, assume unauthorized
    return {"error": "Unauthorized"}, 403
```

**Log all errors**

```python
except Exception as e:
    logger.error(f"Error: {e}", exc_info=True)
    # Don't expose to user
    return {"error": "Server error"}, 500
```

**Test error paths**

* Unit tests for error cases
* Integration tests for API failures
* Fuzz testing with invalid input
* Load testing to find resource leak.

***

&#123;% embed url="<https://cwe.mitre.org/data/definitions/248.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/252.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/390.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/703.html>" %}

&#123;% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/248.html>" %}

&#123;% embed url="<https://docs.python.org/3/tutorial/errors.html>" %}

&#123;% embed url="<https://docs.oracle.com/javase/tutorial/essential/exceptions/>" %}
