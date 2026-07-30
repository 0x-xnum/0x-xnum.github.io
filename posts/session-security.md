# Session Security

### Real-World Attack Scenarios

#### Scenario 1: Session Fixation

Attacker sets victim's session ID to a known value:

```
Attacker obtains session ID: abc123def456
Sends victim URL with this SID: http://example.com/?sid=abc123def456
Victim clicks link, uses attacker's SID
Attacker uses same SID: cookie: sessionid=abc123def456
Both user and attacker have same session!
Attacker can see what victim sees, perform actions as victim
```

**The attack:**

```bash
# Attacker sets own session
curl http://example.com -c cookies.txt
# Receives: Set-Cookie: sessionid=abc123def456

# Attacker sends victim this link
http://example.com/?sid=abc123def456

# Victim clicks, uses attacker's session
# Victim logs in with this session
# Now attacker is logged in as victim!

# Attacker uses same sessionid
curl http://example.com/account -b "sessionid=abc123def456"
# Victim's account!
```

**Result:**

* Account takeover without credentials
* No password needed
* Attacker access to victim's account

**Finding it:** Try setting session ID. Check if server accepts attacker-supplied SID.

***

#### Scenario 2: Insufficient Session Expiration

Session valid for too long (1 year expiration):

```
User logs in
Session: sessionid=abc123; max_age=31536000 (1 year)
User logs out
Session still valid for 1 year!
Attacker steals session cookie
Can use it for entire year
```

**The attack:**

```bash
# User logs out but session still valid
# Attacker finds session cookie in:
# - Browser cache
# - Proxies
# - Network traffic
# - Backups

# One year later, uses session
curl http://example.com/account -b "sessionid=abc123"
# Still logged in!
```

**Result:**

* Extended compromise window
* Stolen sessions valid longer
* Increased risk

**Finding it:** Check session expiration. Try using old session IDs. Verify logout clears session.

***

#### Scenario 3: Unverified Password Change

Password can be changed without verifying old password:

```python
@app.route('/change-password', methods=['POST'])
def change_password():
    # NO verification of old password!
    new_password = request.form['new_password']
    
    # Just change it
    user.password = hash_password(new_password)
    db.commit()
```

**The attack:**

Attacker accesses victim's session (XSS, CSRF, session theft):

```bash
# Attacker has victim's session
# Changes victim's password
curl -X POST http://example.com/change-password \
  -H "Cookie: sessionid=victim_session" \
  -d "new_password=attacker_password"

# Victim's password changed!
# Victim locked out
# Attacker takes over account
```

**Result:**

* Account takeover
* Victim locked out
* No verification required

**Finding it:** Test password change without old password verification. Use session hijacking + password change.

***

#### Scenario 4: Missing Authentication Step

Critical operation skips authentication check:

```python
@app.route('/delete-account', methods=['POST'])
def delete_account():
    # NO authentication check!
    # Just reads user_id from request
    user_id = request.form['user_id']
    
    User.query.filter_by(id=user_id).delete()
    db.commit()
```

**The attack:**

```bash
# Delete any user's account
curl -X POST http://example.com/delete-account \
  -d "user_id=999"

# User 999's account deleted!
# No auth required
```

**Result:**

* Delete any account
* Denial of service
* Data destruction

***

#### Scenario 5: Insufficient Multi-Factor Authentication

2FA implemented but with weaknesses:

```python
# Weakness 1: Bypass if no MFA set up
if not user.mfa_enabled:
    allow_login()  # No 2FA check

# Weakness 2: Code sent via SMS (interceptable)
send_sms_code(user.phone)  # Easy to intercept

# Weakness 3: No rate limiting on code attempts
# Weakness 4: Code doesn't expire
```

**The attack:**

User without MFA set up:

```
Attacker has credentials
Logs in
MFA not enabled
Access granted!
```

Or with weak MFA:

```
SMS code intercepted or guessed
Multiple code attempts allowed
Code doesn't expire
Attacker enters code, gains access
```

**Result:**

* Complete account compromise
* MFA bypass

**Finding it:** Test with credentials + no MFA. Try MFA bypass. Intercept SMS codes.

***

### Mitigation Strategies

**Regenerate session ID on login**

```python
@app.route('/login', methods=['POST'])
def login():
    user = authenticate(username, password)
    
    # Regenerate session (new SID)
    session.regenerate()
    session['user_id'] = user.id
```

**Set appropriate session expiration**

```
Session lifetime: 30 minutes for sensitive operations
Idle timeout: 15 minutes of inactivity
Remember-me: 30 days max (with re-authentication on sensitive operations)
```

```python
response.set_cookie(
    'sessionid',
    value=session_id,
    max_age=1800,  # 30 minutes
    secure=True,
    httponly=True,
    samesite='Strict'
)
```

**Require password verification for sensitive changes**

```python
@app.route('/change-password', methods=['POST'])
def change_password():
    # Verify current password
    if not verify_password(request.form['current_password'], user.password):
        return "Incorrect password", 403
    
    # Then allow change
    user.password = hash_password(request.form['new_password'])
```

**Require authentication for all critical operations**

```python
@app.route('/delete-account', methods=['POST'])
@require_auth  # Decorator
def delete_account():
    # Requires authentication
    user_id = session['user_id']  # From session, not request
    
    User.query.filter_by(id=user_id).delete()
```

**Implement proper 2FA**

* Use TOTP (time-based, not SMS)
* Backup codes for recovery
* Rate limiting on attempts
* Code expiration
* No 2FA bypass

```python
# Require 2FA check
@require_2fa
def protected_operation():
    pass

# Verify TOTP code
import pyotp
totp = pyotp.TOTP(user.mfa_secret)
if not totp.verify(code):
    return "Invalid code", 403
```

**Invalidate sessions on logout**

```python
@app.route('/logout', methods=['POST'])
def logout():
    # Remove session from server
    session.delete()
    
    # Clear client cookie
    response.delete_cookie('sessionid')
    
    return redirect('/')
```

***

### Related CWE Entries

{% embed url="<https://cwe.mitre.org/data/definitions/384.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/613.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/620.html>" %}

{% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html>" %}
