# Fail-Open Vulnerabilities

Fail-open vulnerabilities occur when applications default to allowing access or trusting input when security checks fail or logic errors occur.&#x20;

The opposite of proper security is "fail-open" — when something goes wrong, the system defaults to allowing everything instead of denying everything. This is the most dangerous category of logic errors because it actively weakens security.

***

### Real-World Attack Scenarios

#### Scenario 1: Failing Open on Authorization Check (CWE-636)

An application checks if user is admin:

```python
def delete_user(user_id):
    # Check if current user is admin
    try:
        is_admin = check_admin_status()
    except:
        # FAIL OPEN - Assume admin on error!
        pass
    
    # No check if is_admin is True!
    # If exception, is_admin is undefined!
    
    # Proceed with deletion
    db.delete_user(user_id)
    return {"status": "User deleted"}
```

**The vulnerability:**

If `check_admin_status()` throws exception:

* Exception caught but not handled properly
* `is_admin` undefined
* Code proceeds anyway
* User deleted even though authorization failed

**The attack:**

Attacker sends request while admin check service is down:

```bash
curl -X POST http://target.com/admin/delete-user \
  -d '{"user_id": 42}'
```

Admin check service fails with connection error:

* Exception thrown
* Caught and ignored (pass statement)
* Code proceeds anyway
* User 42 deleted by non-admin attacker

**Result:**

* Complete authorization bypass
* Unauthorized admin actions
* Data deletion
* Service unavailable = security disabled

**The fix:**

Fail securely (default deny):

```python
def delete_user(user_id):
    try:
        is_admin = check_admin_status()
    except Exception as e:
        logger.error(f"Admin check failed: {e}")
        # FAIL SECURE - Deny access
        return {"error": "Authorization check failed"}, 403
    
    if not is_admin:
        return {"error": "Unauthorized"}, 403
    
    # Only proceed if check succeeded AND user is admin
    db.delete_user(user_id)
    return {"status": "User deleted"}
```

**Finding it:** Look for try/except blocks that catch security checks. Check if exception handling leads to access grant rather than denial.

***

#### Scenario 2: Missing Break Statement in Switch (CWE-484)

User role authorization check missing break:

```java
public void checkAccess(String role) {
    switch(role) {
        case "user":
            // User permissions
            accessLevel = 1;
            // MISSING break!
        
        case "admin":
            // Admin permissions
            accessLevel = 10;
            break;
        
        case "guest":
            accessLevel = 0;
            break;
    }
    
    return accessLevel;
}
```

**The vulnerability:**

Missing `break` causes fall-through:

* `role = "user"` sets `accessLevel = 1`
* **Falls through to `admin` case**
* Overrides with `accessLevel = 10`
* User becomes admin!

**The attack:**

Attacker registers as regular user, then somehow triggers the "user" case followed by admin case:

```java
checkAccess("user")  // Sets accessLevel = 1, then falls through
// accessLevel = 10 (admin!)
```

**Result:**

* Privilege escalation
* Regular user becomes admin
* Unauthorized access

**The fix:**

Always include break statements:

```java
public void checkAccess(String role) {
    switch(role) {
        case "user":
            accessLevel = 1;
            break;  // Prevent fall-through
        
        case "admin":
            accessLevel = 10;
            break;
        
        case "guest":
            accessLevel = 0;
            break;
        
        default:
            accessLevel = 0;  // Default deny
            break;
    }
    
    return accessLevel;
}
```

**Finding it:** Search for switch statements without break. Use compiler warnings. Test for fall-through behavior.

***

#### Scenario 3: Missing Default Case (CWE-478)

Payment status check missing default case:

```python
def process_payment(status):
    if status == "approved":
        deliver_product()
    elif status == "declined":
        send_decline_message()
    # MISSING else/default!
    
    # If status is anything else, what happens?
    # Falls through without doing anything
```

**The vulnerability:**

No default case for unexpected values:

* `status = "approved"` → Delivers (correct)
* `status = "declined"` → Declines (correct)
* `status = "pending"` → **Falls through, does nothing**
* `status = "refund"` → **Falls through, does nothing**
* `status = "invalid"` → **Falls through, does nothing**

Attacker sends `status = "pending"`:

* No delivery
* No payment charge
* Free product

**The attack:**

Attacker modifies API call:

```bash
curl -X POST http://target.com/process-payment \
  -d '{"status": "pending", "product": "iPhone"}'

# Payment process doesn't handle "pending"
# Falls through default case (missing)
# Product delivered without payment!
```

**Result:**

* Free products
* Revenue loss
* Fraud

**The fix:**

Always have default case:

```python
def process_payment(status):
    if status == "approved":
        charge_card()
        deliver_product()
    elif status == "declined":
        send_decline_message()
    else:
        # Default: deny everything
        logger.warning(f"Unknown payment status: {status}")
        send_error_message()
        return False
    
    return True
```

Or use explicit state machine:

```python
VALID_STATUSES = {"approved", "declined", "pending"}

def process_payment(status):
    if status not in VALID_STATUSES:
        raise InvalidStatusError(f"Invalid status: {status}")
    
    if status == "approved":
        deliver_product()
    elif status == "declined":
        send_decline_message()
    # All valid cases handled
```

**Finding it:** Look for if/elif chains without else. Look for switch statements without default. Check for unhandled enum values.

***

#### Scenario 4: NULL Pointer Dereference (CWE-476)

User authorization check returns NULL:

```java
public boolean isAdmin(int userId) {
    User user = findUserById(userId);
    
    // VULNERABLE: No null check!
    // If user is null, next line crashes
    
    return user.isAdmin();  // NullPointerException if user is null
}

public void deleteUser(int targetId) {
    try {
        if (isAdmin(getCurrentUserId())) {
            // Delete the user
            db.deleteUser(targetId);
        }
    } catch (NullPointerException e) {
        // FAIL OPEN - Exception caught, ignored
        // No error handling
    }
    
    return;
}
```

**The vulnerability:**

If `getCurrentUserId()` returns invalid ID:

* `findUserById()` returns null
* `user.isAdmin()` throws NullPointerException
* Exception caught and ignored
* Authorization check fails silently
* Code continues anyway
* User deletion proceeds without authorization check

**The attack:**

Attacker sends request with invalid user ID:

```bash
curl -X POST http://target.com/delete-user \
  -H "User-ID: invalid"
```

* `findUserById("invalid")` returns null
* `isAdmin()` throws NullPointerException
* Exception silently caught
* `deleteUser()` proceeds
* User deleted without proper authorization

**Result:**

* Authorization bypass
* Unauthorized deletion
* Data destruction

**The fix:**

Always check for null:

```java
public boolean isAdmin(int userId) {
    User user = findUserById(userId);
    
    // Check for null
    if (user == null) {
        throw new UserNotFoundException("User not found");
    }
    
    return user.isAdmin();
}

public void deleteUser(int targetId) {
    try {
        if (isAdmin(getCurrentUserId())) {
            db.deleteUser(targetId);
        } else {
            throw new UnauthorizedException("Not authorized");
        }
    } catch (UserNotFoundException | UnauthorizedException e) {
        // Handle specific exceptions
        return {"error": "Authorization failed"}, 403;
    }
}
```

**Finding it:** Search for method calls without null checks. Use static analysis tools (FindBugs, Checkstyle). Enable compiler warnings for null dereferences.

***

#### Scenario 5: Divide By Zero (CWE-369)

Price calculation with user-provided denominator:

```python
def calculate_discount(original_price, discount_percent):
    # User provides discount_percent
    # No validation!
    
    discount_amount = original_price * discount_percent / 100
    final_price = original_price - discount_amount
    
    return final_price
```

**Normal case:**

```python
calculate_discount(100, 10)  # $100 with 10% discount = $90
```

**The attack:**

Attacker provides 0 discount:

```python
calculate_discount(100, 0)
# discount_amount = 100 * 0 / 100 = 0 (OK)
```

Actually, divide by zero needs different scenario. Better example:

```python
def calculate_price_per_item(total_price, num_items):
    # No validation on num_items!
    price_per_item = total_price / num_items
    return price_per_item
```

**The attack:**

Attacker provides 0 items:

```bash
curl http://target.com/api/price?total=100&num_items=0

# Calculation: 100 / 0
# ZeroDivisionError!
# Exception propagates
# Application crashes (DoS)
```

**Result:**

* Divide by zero exception
* Application crash
* Denial of service

**The fix:**

Validate input:

```python
def calculate_price_per_item(total_price, num_items):
    # Validate input
    if num_items <= 0:
        raise ValueError("num_items must be > 0")
    
    if total_price < 0:
        raise ValueError("total_price must be >= 0")
    
    price_per_item = total_price / num_items
    return price_per_item
```

**Finding it:** Look for division operations. Check if denominator is validated. Test with zero values.

***

#### Scenario 6: Inverted Authorization Logic

Authorization check logic accidentally inverted:

```python
def edit_profile(user_id, user_data):
    # Get current user
    current_user = get_current_user()
    
    # VULNERABLE: Logic inverted!
    if user_id == current_user.id:
        # Only owner should be able to edit
        # But if NOT owner, proceed!
        return {"error": "Cannot edit your own profile"}
    else:
        # If editing someone else's profile, allow!
        update_user(user_id, user_data)
        return {"status": "Profile updated"}
```

**The vulnerability:**

Logic is backwards:

* If `user_id == current_user.id` → Deny (your own profile)
* If `user_id != current_user.id` → Allow (someone else's profile)

**The attack:**

Attacker gets other user's ID (123) and modifies their profile:

```bash
curl -X POST http://target.com/profile/123 \
  -H "Authorization: Bearer token_for_attacker" \
  -d '{"name": "Hacked", "email": "attacker@evil.com"}'
```

Authorization check:

* `user_id = 123` (victim)
* `current_user.id = 456` (attacker)
* `123 != 456` → Allow!
* Attacker modifies victim's profile

**Result:**

* Account takeover
* Profile defacement
* Email change
* Password reset

**The fix:**

Correct the logic:

```python
def edit_profile(user_id, user_data):
    current_user = get_current_user()
    
    # Only owner can edit
    if user_id != current_user.id:
        return {"error": "Cannot edit other's profile"}, 403
    
    # Allow edit
    update_user(user_id, user_data)
    return {"status": "Profile updated"}
```

**Finding it:** Review authorization logic carefully. Test with unauthorized users. Look for inverted conditions (not equals instead of equals).

***

#### Scenario 7: Default Allow in Access Control

Access control matrix with missing entries defaults to allow:

```python
ACCESS_MATRIX = {
    "admin": {
        "view_users": True,
        "edit_users": True,
        "delete_users": True,
    },
    "user": {
        "view_users": False,
        "edit_users": False,
    },
    # "guest" role completely missing!
}

def can_access(role, action):
    # If role not in matrix, or action not in role:
    # Default returns None, which is falsy
    # So access denied? Let's check...
    
    if role not in ACCESS_MATRIX:
        return False  # Actually denies
    
    # But what if action missing?
    return ACCESS_MATRIX[role].get(action)  # Returns None (falsy)
```

**The vulnerability:**

Actually, this one defaults to deny. Better example:

```python
def can_access(role, action):
    # Missing check for empty/null role
    
    if not role:
        # FAIL OPEN - Empty role defaults to admin?
        return True  # BUG!
    
    return ACCESS_MATRIX[role].get(action, False)
```

**The attack:**

Attacker sends empty or null role:

```bash
curl -X POST http://target.com/admin/action \
  -d '{"role": "", "action": "delete"}'

# can_access("", "delete")
# Empty role defaults to True
# Authorization check passes!
# Attacker gains admin access
```

**Result:**

* Complete authorization bypass
* Unauthorized access
* Admin actions as non-admin

**The fix:**

Always validate and fail securely:

```python
def can_access(role, action):
    # Validate role
    if not role or role not in ACCESS_MATRIX:
        return False  # Default deny
    
    # Validate action exists in matrix
    if action not in ACCESS_MATRIX[role]:
        return False  # Default deny
    
    # Check permission
    return ACCESS_MATRIX[role][action]
```

**Finding it:** Review access control logic. Check for default allow. Look for missing validation. Test with null/empty values.

***

### Mitigation Strategies

**Fail secure (default deny)**

```python
# Bad: Default allow
if user_is_admin:
    allow_access()
else:
    allow_access()  # Always allows!

# Good: Default deny
if user_is_admin:
    allow_access()
else:
    deny_access()  # Default deny
```

**Validate all input**

```python
if not role or role not in VALID_ROLES:
    deny_access()
```

**Check for null/empty**

```python
if not user or not user.permissions:
    deny_access()
```

**Use explicit state machines**

```python
VALID_STATUSES = {"approved", "declined"}

def process(status):
    if status not in VALID_STATUSES:
        raise InvalidStatus()
```

**Comprehensive exception handling**

```python
try:
    check_auth()
except:
    # Deny, don't allow
    return 403
```

**Code review**

* Review all authorization logic
* Check switch statements for break
* Verify all cases handled
* Test error paths

**Static analysis**

* Use FindBugs, Checkstyle, SonarQube
* Check for null dereferences
* Check for missing cases
* Check for fall-through

***

&#123;% embed url="<https://cwe.mitre.org/data/definitions/636.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/476.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/484.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/478.html>" %}

&#123;% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/636.html>" %}

&#123;% embed url="<http://findbugs.sourceforge.net/>" %}

&#123;% embed url="<https://www.sonarqube.org/>" %}
