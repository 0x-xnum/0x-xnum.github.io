# Race Conditions

Race conditions (CWE-362) occur when code doesn't properly synchronize access to shared resources, allowing multiple concurrent operations to interfere with each other. The most common variant is **Time-of-Check-Time-of-Use (TOCTOU)**, where a check is performed, then a resource is used, but the resource changes between the check and use.

***

### Real-World Attack Scenarios

#### Scenario 1: TOCTOU in File Access

A file operation checks ownership then accesses the file:

```python
def delete_file(filename):
    # CHECK: Verify user owns file
    if not user_owns_file(filename):
        raise PermissionDenied()
    
    # TIME WINDOW - Attacker can act here!
    
    # USE: Delete the file
    os.remove(filename)
```

**The vulnerability:**

Between check and use, attacker can modify the file (symlink attack):

```
Thread 1 (victim):
1. Check: user_owns_file("/home/user/myfile.txt") → True
2. [Context switch]

Thread 2 (attacker):
3. Replace symlink: ln -sf /etc/passwd /home/user/myfile.txt

Thread 1 continues:
4. Use: os.remove("/home/user/myfile.txt")
5. ACTUALLY DELETES: /etc/passwd!
```

**The attack:**

```bash
# Create initial file
echo "data" > /home/user/myfile.txt

# In a loop, delete the file and replace with symlink to target
while true; do
    rm /home/user/myfile.txt
    ln -sf /etc/shadow /home/user/myfile.txt
done

# Simultaneously, trigger application to delete file
# App checks ownership (passes), deletes file
# But file is now symlink to /etc/shadow
# /etc/shadow gets deleted!
```

**Result:**

* Delete system files
* Modify critical files
* Privilege escalation
* System compromise

**Finding it:** Identify file operations. Try racing with symlinks. Monitor system files for unexpected modifications.

***

#### Scenario 2: TOCTOU in Balance Checks

Bank application checks balance before allowing withdrawal:

```python
def withdraw(user_id, amount):
    # CHECK: Get user's balance
    balance = get_balance(user_id)
    
    if balance < amount:
        raise InsufficientFunds()
    
    # TIME WINDOW - Balance can change here!
    
    # USE: Perform withdrawal
    new_balance = balance - amount
    set_balance(user_id, new_balance)
    
    return new_balance
```

**The vulnerability:**

Between check and use, another withdrawal can occur:

```
User has $100

Thread 1 (Withdrawal A - $100):
1. Check: balance = get_balance() → $100
2. Check: $100 >= $100 ✓
3. [Context switch]

Thread 2 (Withdrawal B - $100):
4. Check: balance = get_balance() → $100
5. Check: $100 >= $100 ✓
6. set_balance($0)
7. [Context switch]

Thread 1 continues:
8. set_balance($0)  # Should be $0, but computed as $100-$100=$0

Result: Both withdrawals succeed!
User withdrew $200 with $100 balance!
```

**The attack:**

1. User has $100
2. Attacker (or accomplice) makes two simultaneous $100 withdrawals
3. Both pass balance check
4. Both execute withdrawal
5. Account balance is now -$100
6. Attacker steals money

```bash
# Trigger two withdrawals simultaneously
curl http://bank.com/withdraw?amount=100 &
curl http://bank.com/withdraw?amount=100 &
wait

# Both succeed despite insufficient balance
```

**Result:**

* Money theft
* Account goes negative
* Bank loses money
* System inconsistency

**Finding it:** Identify balance check operations. Make simultaneous requests. Check if both succeed.

***

#### Scenario 3: TOCTOU in Permission Changes

Admin changes user permissions:

```python
def promote_to_admin(user_id):
    # CHECK: Verify user is moderator
    user = get_user(user_id)
    if user.role != 'moderator':
        raise PermissionDenied()
    
    # TIME WINDOW - User role can change!
    
    # USE: Promote to admin
    user.role = 'admin'
    save_user(user)
```

**The vulnerability:**

Attacker changes their own role between check and promotion:

```
Thread 1 (Admin promotion):
1. Check: user.role = 'moderator' ✓
2. [Context switch]

Thread 2 (Attacker - different session):
3. Somehow change own role to 'moderator' (exploit)
4. Then: admin_promote_to_admin(my_id)

Thread 1 continues:
5. Check still passes (moderator when checked)
6. Promote attacker to admin!
```

**Result:**

* Privilege escalation
* Attacker becomes admin
* Complete system control

***

#### Scenario 4: Check-After-Action Race

Application performs action then checks if it should have:

```python
def delete_account(user_id):
    # ACTION: Delete account
    database.delete_user(user_id)
    
    # CHECK: Verify user had permission (AFTER!)
    user = get_user_from_cache(user_id)  # Outdated cache
    if not user.can_delete_self:
        # Too late, already deleted!
        raise PermissionDenied()
```

**The vulnerability:**

Action happens before permission check!

**Result:**

* Users delete accounts they shouldn't have access to
* Data loss
* System inconsistency

***

#### Scenario 5: Inventory Race Condition

E-commerce checks stock before selling:

```python
def purchase_item(item_id, quantity):
    # CHECK: Get stock
    stock = get_stock(item_id)
    
    if stock < quantity:
        raise OutOfStock()
    
    # TIME WINDOW - Stock can change!
    
    # USE: Sell items
    new_stock = stock - quantity
    set_stock(item_id, new_stock)
    
    return new_stock
```

**The vulnerability:**

Multiple customers buy simultaneously from limited stock:

```
Item has 10 units in stock

Customer A (buy 8):
1. Check: stock = 10 ✓
2. [Context switch]

Customer B (buy 5):
3. Check: stock = 10 ✓
4. set_stock(5)  # 10 - 5

Customer A continues:
5. set_stock(2)  # 10 - 8

RESULT: Sold 13 units from 10 in stock!
```

**Result:**

* Overselling
* Unfulfilled orders
* Revenue loss
* Customer complaints

**Finding it:** Purchase items simultaneously. Monitor inventory. Check if overselling possible.

***

### How to Identify Race Conditions During Testing

**1. Identify potential race windows**

Look for patterns:

* Check then use
* Check then modify
* Multiple step operations
* Database operations without transactions

**2. Test with concurrent requests**

Use threading or multiple processes:

```python
import threading
import requests

def withdraw():
    requests.post('http://bank.com/withdraw', {'amount': 100})

# Create 5 concurrent threads
threads = [threading.Thread(target=withdraw) for _ in range(5)]

for t in threads:
    t.start()
    
for t in threads:
    t.join()
    
# Check result - should fail but might succeed
```

**3. Use timing attacks**

Deliberately delay operations to increase race window:

```python
# Add debugging/logging to increase timing window
# This increases likelihood of hitting race condition
```

**4. Monitor for inconsistent state**

After concurrent operations, check if state is consistent:

* Total balance correct?
* Inventory counts correct?
* Permissions properly assigned?

**5. Use race condition testing tools**

* Apache JMeter for load testing
* Burp Intruder for repeating requests
* Custom scripts for precise timing

**6. Analyze transaction handling**

Check if operations are atomic:

```sql
-- Bad: Multiple separate queries
SELECT balance FROM users WHERE id=1;
-- ... compute ...
UPDATE users SET balance=new_balance WHERE id=1;

-- Good: Atomic transaction
BEGIN TRANSACTION;
UPDATE users SET balance = balance - amount WHERE id=1;
COMMIT;
```

***

### Mitigation Strategies

**Use atomic operations**

Ensure check and use happen atomically:

```python
# Bad
if balance >= amount:
    new_balance = balance - amount
    set_balance(new_balance)

# Good - Atomic operation
def withdraw_safe(user_id, amount):
    # Single atomic operation
    result = database.execute("""
        UPDATE accounts 
        SET balance = balance - ?
        WHERE id = ? AND balance >= ?
        RETURNING balance
    """, [amount, user_id, amount])
    
    if not result:
        raise InsufficientFunds()
    
    return result[0]
```

**Use database transactions**

```python
with database.transaction():
    balance = get_balance(user_id)
    if balance < amount:
        raise InsufficientFunds()
    set_balance(user_id, balance - amount)
```

**Use locks and synchronization**

```python
import threading

balance_lock = threading.Lock()

def withdraw(amount):
    with balance_lock:  # Acquire lock
        balance = get_balance()
        if balance < amount:
            raise InsufficientFunds()
        set_balance(balance - amount)
    # Lock released
```

**Use optimistic locking**

Add version/timestamp to prevent concurrent modifications:

```python
def update_user(user_id, data, version):
    # Only update if version matches
    result = database.execute("""
        UPDATE users
        SET data = ?, version = version + 1
        WHERE id = ? AND version = ?
    """, [data, user_id, version])
    
    if not result:
        raise VersionMismatch()
```

**Avoid symlink attacks**

Use secure file handling:

```python
# Don't follow symlinks
os.remove(filename, follow_symlinks=False)

# Or use secure temp files
import tempfile
with tempfile.NamedTemporaryFile(delete=False) as f:
    f.write(data)
    f.flush()
    # File is secure
```

**Use file locking**

```python
import fcntl

def safe_file_operation(filename):
    with open(filename, 'r') as f:
        fcntl.flock(f.fileno(), fcntl.LOCK_EX)  # Exclusive lock
        # Perform operation
        fcntl.flock(f.fileno(), fcntl.LOCK_UN)  # Unlock
```

***

&#123;% embed url="<https://cwe.mitre.org/data/definitions/362.html>" %}

&#123;% embed url="<https://owasp.org/www-community/attacks/Race_condition>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/362.html>" %}

&#123;% embed url="<https://portswigger.net/web-security/race-conditions>" %}
