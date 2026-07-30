# Sensitive Data Storage: Encryption, Caching, and Cookies

Multiple vulnerabilities related to how applications store and cache sensitive data:

Together, they expose sensitive data through multiple storage mechanisms.

***

### Real-World Attack Scenarios

#### Scenario 1: Plaintext Passwords in Database

A user management system stores passwords in plaintext:

```sql
-- Database table
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(255),
    password VARCHAR(255)  -- CLEARTEXT!
);

-- Sample data
INSERT INTO users VALUES (1, 'admin', 'admin123');
INSERT INTO users VALUES (2, 'user1', 'password456');
```

**The vulnerability:**

Passwords stored as plaintext:

* Database breach exposes all passwords
* No effort required to read them
* Attacker has access to all accounts immediately

**The attack:**

Attacker gains database access (SQL injection, misconfiguration, insider threat):

```sql
SELECT * FROM users;
-- Output:
-- id=1, username=admin, password=admin123
-- id=2, username=user1, password=password456
```

Attacker now has:

* All user passwords
* Access to all accounts
* Ability to login as any user
* Ability to reset other passwords
* Complete system compromise

**Result:**

* Complete authentication bypass
* All accounts compromised
* Attacker access to all data
* Reputation damage
* PCI DSS violation ($100K+ fines)

**Finding it:** Gain database access. Query user table. Check if passwords are plaintext or hashed.

***

#### Scenario 2: Unencrypted Payment Information

E-commerce site stores credit card information unencrypted:

```python
# Storing credit cards in database
def save_payment_info(user_id, card_number, expiry):
    db.insert('payments', {
        'user_id': user_id,
        'card_number': card_number,  # Plaintext!
        'expiry': expiry
    })

# Database contains plaintext card numbers
# card_number: 4111111111111111
# card_number: 4222222222222222
```

**The vulnerability:**

Credit cards stored without encryption:

* Database breach exposes all cards
* PCI DSS explicitly prohibits this
* Massive fines and liability

**The attack:**

Attacker gains database access:

```sql
SELECT card_number FROM payments LIMIT 10;
-- Returns:
-- 4111111111111111
-- 4222222222222222
-- 4333333333333333
-- 5555555555554444
```

Attacker uses cards for fraud:

```
Charge thousands to each card
Sell card data on dark web
Customer accounts compromised
```

**Result:**

* Massive financial fraud
* PCI DSS violation ($100K-$1M+ fines)
* Lawsuits from affected customers
* Company reputation destroyed

**Finding it:** Access database. Check if payment data encrypted. Look for plaintext card numbers.

***

#### Scenario 3: API Keys in Browser Cache

Web application caches pages with API keys:

```html
<!-- Page returned with Cache-Control header -->
<html>
<body>
    <h1>API Configuration</h1>
    <p>Your API Key: sk_REDACTED_live_4eC39HqLyjWDarht8Zlt5Kda</p>
</body>
</html>

<!-- Server sends with default caching headers -->
HTTP/1.1 200 OK
Content-Type: text/html
(No Cache-Control header - page cached by default!)
```

**The vulnerability:**

Page cached in browser, disk, and intermediate proxies:

* Browser cache contains the secret
* Anyone with access to computer can retrieve it
* Network proxies cache the page
* Page history contains the URL

**The attack:**

Attacker:

1. Uses victim's computer
2. Opens browser cache: `~/.cache/` (Linux) or `%TEMP%` (Windows)
3. Finds cached HTML with API key
4. Extracts API key
5. Uses it to access APIs

Or:

```bash
# Check browser history and cache
cat ~/.mozilla/firefox/*/cache2/entries/*

# Or use tools to extract
python browser_cache_extractor.py
# Finds cached pages with secrets
```

**Result:**

* API key theft
* Unauthorized API access
* Data exfiltration
* Complete API compromise

**Finding it:** Access cached pages. Check if sensitive data visible in cache. Use browser cache analysis tools.

***

#### Scenario 4: Session Tokens in Persistent Cookies

Application stores session tokens in persistent cookies:

```javascript
// Set cookie that persists for 1 year
document.cookie = "sessionToken=" + token + "; max-age=31536000; path=/";
```

**The vulnerability:**

Session token in persistent cookie:

* Survives browser restart
* Stored on disk in plaintext
* Can be extracted from disk
* Long-lived cookie = longer attack window

**The attack:**

Attacker:

1. Gains access to victim's computer
2. Extracts cookie from disk
3. Uses cookie in their own browser
4. Impersonates victim indefinitely
5. Token valid for entire year

```bash
# Extract cookies from browser
sqlite3 ~/.mozilla/firefox/*/cookies.sqlite "SELECT * FROM moz_cookies;"

# Found:
# sessionToken=abc123def456...

# Use in Burp Suite
# Set Cookie header to stolen token
# Hijack victim's session
```

**Result:**

* Session hijacking
* Long-term account compromise
* Indefinite impersonation

**Finding it:** Check cookie storage. Verify expiration. Look for session tokens in persistent cookies.

***

#### Scenario 5: Sensitive Data in URL Query String

Application passes sensitive data in URLs:

```
GET /search?q=ssn:123-45-6789&credit_card=4111111111111111
GET /checkout?total=99.99&discount_code=SENSITIVE123
GET /profile?user_id=42&email=user@example.com&phone=555-0123
```

**The vulnerability:**

URL parameters are logged and cached:

* Web server access logs contain URLs
* Browser history contains URLs
* Proxy logs contain URLs
* Referrer headers leak URLs
* URLs visible in browser address bar

**The attack:**

Attacker accesses:

1. Web server logs: `tail /var/log/apache2/access.log | grep credit_card`
2. Browser history: victim's browser contains URL with data
3. Proxy logs: network proxies log all traffic
4. Referrer headers: other sites receive referrer with data

All containing sensitive information.

**Result:**

* Credential exposure
* Data leakage
* Privacy violation

**Finding it:** Intercept requests. Monitor for sensitive data in URLs. Check logs for exposed data.

***

#### Scenario 6: Secrets in Memory Without Clearing

Passwords loaded into memory and never cleared:

```csharp
// Load password into memory
char[] password = userInput.ToCharArray();

// Use password for authentication
if (VerifyPassword(password, storedHash)) {
    AuthenticateUser();
}

// VULNERABLE - Password still in memory!
// Garbage collector hasn't run yet
// Memory dump reveals password
```

**The vulnerability:**

Sensitive data left in memory:

* Memory dumps (crash, core dumps) contain password
* Debuggers can inspect memory
* Process monitoring tools see memory
* Garbage collection doesn't immediately clear memory

**The attack:**

Attacker:

1. Crashes application (trigger error)
2. Core dump written to disk containing password
3. Or attaches debugger to running process
4. Inspects memory containing password
5. Extracts plaintext password

```bash
# Trigger crash and create core dump
kill -11 <pid>

# Examine core dump
gdb ./application core

# View memory
(gdb) x/s 0x7fffffff1234
# Output: "myPassword123"
```

**Result:**

* Plaintext password extraction
* Account compromise
* Authentication bypass

**Finding it:** Trigger memory dumps. Use memory analysis. Check if sensitive data cleared from memory.

***

#### Scenario 7: Credentials in Config Files

Application stores credentials in plaintext config files:

```ini
# config.ini
[database]
host=db.internal.company.com
username=db_user
password=SuperSecretPassword123!

[aws]
access_key_id=AKIAIOSFODNN7EXAMPLE
secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

[api]
stripe_secret_key=sk_REDACTED_live_4eC39HqLyjWDarht8Zlt5Kda
```

**The vulnerability:**

Credentials in plaintext files:

* Version control exposes them
* Backups contain them
* File system access reveals them
* Logs might include them

**The attack:**

Attacker:

1. Clones repository: `git clone https://repo.git`
2. Finds config.ini with credentials
3. Uses credentials to access database, AWS, Stripe, etc.

Or:

```bash
# Search for config files
find . -name "*.ini" -o -name "*.conf" -o -name "config.*"

# Extract credentials
grep -r "password\|secret\|key" *.ini
```

**Result:**

* Complete credential exposure
* Access to database, AWS, external services
* Complete system compromise

**Finding it:** Search for config files. Check for plaintext credentials. Review git history for committed secrets.

***

### How to Identify Sensitive Data Storage Issues During Testing

**1. Test password storage**

```sql
-- Check database
SELECT * FROM users LIMIT 5;

-- Look for plaintext passwords
-- Or weak hashes (MD5, SHA1)
```

**2. Test payment data storage**

Check if credit cards stored encrypted:

```sql
SELECT * FROM payments;
-- Should show encrypted data, not plaintext card numbers
```

**3. Check browser cache**

```bash
# Linux
ls -la ~/.cache/

# Windows
%TEMP%
%LOCALAPPDATA%\Cache
```

Look for pages containing sensitive data.

**4. Check cookies**

Developer Tools → Application → Cookies Look for:

* Plaintext tokens
* Sensitive information
* Long expiration dates
* Missing security flags

**5. Check URLs**

Intercept with Burp Suite:

* Look for sensitive data in URLs
* Check if data logged somewhere

**6. Check memory**

Trigger crashes, analyze memory dumps for plaintext passwords.

**7. Check config files**

```bash
find . -name "*.conf" -o -name "*.ini" -o -name "config.*"
grep -r "password\|secret\|key" .
```

***

### Mitigation Strategies

**Never store passwords plaintext**

Always hash with bcrypt or Argon2:

```python
import bcrypt

# Hash at storage time
salt = bcrypt.gensalt(rounds=12)
hashed = bcrypt.hashpw(password.encode(), salt)

# Verify at authentication
if bcrypt.checkpw(password.encode(), hashed):
    authenticate()
```

**Encrypt sensitive data at rest**

```python
from cryptography.fernet import Fernet

cipher = Fernet(key)
encrypted_card = cipher.encrypt(card_number.encode())

# Store encrypted_card
# Decrypt only when needed
```

**Never cache sensitive pages**

```python
# Django
from django.views.decorators.cache import never_cache

@never_cache
def sensitive_page(request):
    return render(request, 'sensitive.html')

# Or set headers
response['Cache-Control'] = 'no-cache, no-store, must-revalidate'
response['Pragma'] = 'no-cache'
response['Expires'] = '0'
```

**Use secure cookies**

```python
# Set security flags
response.set_cookie(
    'sessionid',
    value=token,
    secure=True,        # HTTPS only
    httponly=True,      # No JavaScript access
    samesite='Strict',  # CSRF protection
    max_age=3600        # 1 hour expiration
)
```

**Never put sensitive data in URLs**

Use POST instead:

```python
# Bad
GET /search?ssn=123-45-6789

# Good
POST /search
data={'ssn': '123-45-6789'}
```

**Clear sensitive data from memory**

```csharp
// After using password
Array.Clear(password, 0, password.Length);

// Or use SecureString
SecureString pwd = new SecureString();
pwd.AppendChar('p');
// ... etc
// SecureString automatically clears when disposed
```

**Never commit secrets to git**

```bash
# Use .gitignore
echo ".env" >> .gitignore
echo "config.ini" >> .gitignore
echo "secrets.*" >> .gitignore

# Or use environment variables
export DB_PASSWORD=secret
# Access in code: os.getenv('DB_PASSWORD')
```

**Use secrets management systems**

* AWS Secrets Manager
* Azure Key Vault
* HashiCorp Vault
* Kubernetes Secrets

***

{% embed url="<https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure/>" %}

{% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/256.html>" %}

{% embed url="<https://www.pcisecuritystandards.org/>" %}
