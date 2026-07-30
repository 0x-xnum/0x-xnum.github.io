# Authentication Bypass

Vulnerabilities that allow bypassing authentication entirely

### Real-World Attack Scenarios

#### Scenario 1: Alternate Path Bypass

Attacker accesses protected resource through different URL path:

```
Protected: /admin/dashboard
Bypass: /Admin/dashboard (case variation)
Bypass: /admin//dashboard (double slash)
Bypass: /admin/./dashboard (dot slash)
Bypass: /ADMIN/dashboard (uppercase)
```

Web server treats `/admin/dashboard` as protected and requires authentication. But `/Admin/dashboard` bypasses protection due to case-insensitive filesystem.

```bash
# Denied
curl http://example.com/admin/dashboard
# 401 Unauthorized

# Succeeds!
curl http://example.com/Admin/dashboard
# 200 OK - admin panel loads
```

**Finding it:** Try case variations, double slashes, URL encoding variations.

***

#### Scenario 2: Alternate Name Bypass

Authentication check looks for specific username format:

```python
def check_admin_access(username):
    if username == 'admin':
        return True
    return False
```

Attacker uses alternate format:

```
admin123 (append number)
admin. (append dot)
admin (add space)
admin_backup
root (alternate admin name)
administrator
```

Some might pass:

```bash
curl http://example.com/admin -u "admin123:"
# If admin123 is assigned admin role, access granted
```

**Finding it:** Try username variations. Check if system has multiple admin accounts.

***

#### Scenario 3: Spoofing Bypass

Attacker impersonates trusted entity:

```python
# Vulnerable - trusts X-Forwarded-For header
if request.headers.get('X-Forwarded-For') == 'trusted-ip':
    grant_admin_access()
```

Attacker simply sends the header:

```bash
curl http://example.com/admin \
  -H "X-Forwarded-For: 192.168.1.100"
# Access granted - spoofed IP!
```

Or by IP spoofing:

* Spoof source IP to appear as internal network
* Bypass IP-based authentication

**Finding it:** Try trusted headers (X-Forwarded-For, X-Real-IP). Test IP spoofing.

***

#### Scenario 4: Immutable Data Modification

Application assumes user ID can't change:

```python
# Vulnerable - assumes user_id immutable
@app.route('/profile/<user_id>')
def get_profile(user_id):
    if not authenticated():
        return "Unauthorized"
    return get_user_data(user_id)
```

But attacker modifies their user\_id between authentication and use:

```bash
# Authenticate as user 1
curl -H "Authorization: Bearer token1" http://example.com/profile/1
# Then modify user_id
curl -H "Authorization: Bearer token1" http://example.com/profile/999
# Access user 999's data with user 1's token!
```

**Finding it:** Check if IDs can be modified mid-request. Use race conditions.

***

#### Scenario 5: Weak Primary Authentication

Primary authentication weak, but assumed strong:

```python
# Weak authentication - can be bypassed
if password == stored_hash:
    authenticated = True

# Then trusted for all operations
if authenticated:
    grant_admin_access()  # Assumes primary auth is strong
```

If primary auth (password check) is weak:

* Logic error
* Timing attack
* Default credentials
* SQL injection

Then attacker bypasses entire system.

**Finding it:** Test authentication mechanism thoroughly. Look for weaknesses in primary auth.

***

### Mitigation Strategies

**Normalize URLs**

```python
# Lowercase URLs
path = request.path.lower()

# Remove extra slashes
path = '/'.join(filter(None, path.split('/')))

# Decode URL encoding
from urllib.parse import unquote
path = unquote(path)
```

**Strict authentication checks**

```python
# Check exact username, not similar
allowed_usernames = ['admin', 'administrator']
if username not in allowed_usernames:
    return "Unauthorized"
```

**Never trust client headers for auth**

```python
# Bad
if request.headers.get('X-Forwarded-For') == 'trusted-ip':
    grant_access()

# Good
# Use server-verified IP from connection
client_ip = request.remote_addr
if client_ip in TRUSTED_INTERNAL_IPS:
    grant_access()
```

**Validate immutable data**

```python
# Store user_id in session, don't accept from request
user_id = session['user_id']  # Not from URL parameter
```

**Strong primary authentication**

Ensure primary authentication is:

* Not bypassable
* Tested thoroughly
* Uses strong hashing
* Resistant to timing attacks

***

&#123;% embed url="<https://cwe.mitre.org/data/definitions/288.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/289.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/290.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/302.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/305.html>" %}
