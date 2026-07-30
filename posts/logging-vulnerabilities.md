# Logging Vulnerabilities

Logging is a concept that most developers already use for debugging and diagnostic purposes. Security logging is an equally basic concept: to log security information during the runtime operation of an application.

Monitoring is the live review of application and security logs using various forms of automation. The same tools and patterns can be used for operations, debugging and security purposes.

The goal of security logging is to detect and respond to potential security incidents.

***

### Real-World Attack Scenarios

#### Scenario 1: Log Injection via Unvalidated Input (CWE-117)

An application logs user login attempts without sanitization:

```python
import logging

def log_login_attempt(username, success):
    # VULNERABLE - User input not sanitized
    if success:
        logging.info(f"User {username} logged in successfully")
    else:
        logging.info(f"Failed login attempt for user {username}")
```

**The vulnerability:**

User-controlled input written directly to logs:

* Newline characters (`\n`) not filtered
* Attacker can inject fake log entries
* Can hide malicious activity
* Can frame other users
* Can corrupt log files

**The attack:**

Attacker enters username with embedded newlines:

```
Username: admin\nUser admin logged in successfully\nUser attacker
```

Resulting log entry:

```
2024-01-15 10:23:45 Failed login attempt for user admin
2024-01-15 10:23:45 User admin logged in successfully
2024-01-15 10:23:45 User attacker
```

Looks like admin logged in successfully, hiding the failed attempt.

Or inject malicious entries:

```
Username: admin\n[2024-01-15 10:25:00] CRITICAL Security scan completed - no issues found\n
```

Resulting log:

```
2024-01-15 10:23:45 Failed login attempt for user admin
2024-01-15 10:25:00 CRITICAL Security scan completed - no issues found
```

Attacker hides real attack by injecting fake "all clear" message.

**Advanced attack - Log forging:**

```
Username: admin\n[2024-01-15 10:30:00] User victim performed unauthorized action\n[2024-01-15 10:30:01] Suspicious activity detected from IP 192.168.1.100\n
```

Attacker frames another user for malicious activity.

**Result:**

* Forged audit trail
* Hidden attack evidence
* Framing innocent users
* Corrupted forensic data
* Investigation misdirection

**Finding it:** Test login forms and any logged input. Try `\n`, `\r\n`, null bytes. Check if log entries can be injected.

**Exploit:**

```bash
# Test log injection
curl -X POST http://target.com/login \
  -d "username=admin%0AUser admin logged in successfully%0A" \
  -d "password=test"

# Check logs for injected entry
```

**The fix:**

Sanitize all user input before logging:

```python
import logging
import re

def sanitize_log_input(text):
    # Remove newlines and control characters
    return re.sub(r'[\n\r\t\x00-\x1f]', '', text)

def log_login_attempt(username, success):
    safe_username = sanitize_log_input(username)
    if success:
        logging.info(f"User {safe_username} logged in successfully")
    else:
        logging.info(f"Failed login attempt for user {safe_username}")
```

Or use structured logging (JSON):

```python
import logging
import json

def log_login_attempt(username, success):
    log_entry = {
        "event": "login_attempt",
        "username": username,  # JSON encoding handles escaping
        "success": success,
        "timestamp": datetime.now().isoformat()
    }
    logging.info(json.dumps(log_entry))
```

***

#### Scenario 2: Passwords in Log Files (CWE-532)

Application logs all HTTP requests including authentication:

```python
import logging

@app.before_request
def log_request():
    # VULNERABLE - Logs entire request including passwords
    logging.info(f"Request: {request.method} {request.url}")
    logging.info(f"Headers: {request.headers}")
    logging.info(f"Body: {request.get_data()}")
```

**The vulnerability:**

Sensitive data written to logs:

* Passwords in POST bodies
* API keys in headers
* Session tokens in cookies
* Credit cards in request data
* PII (SSN, email, phone) in parameters

**The attack:**

User logs in with credentials:

```
POST /login HTTP/1.1
Content-Type: application/json

{"username": "john", "password": "SecretPass123!"}
```

Log file contains:

```
2024-01-15 10:23:45 Request: POST /login
2024-01-15 10:23:45 Headers: {'Authorization': 'Bearer sk_REDACTED_live_4eC39HqLyjWDarht8Zlt5Kda'}
2024-01-15 10:23:45 Body: {"username": "john", "password": "SecretPass123!"}
```

Attacker gains access to log files (misconfiguration, backup exposure, insider threat):

```bash
# Search logs for passwords
grep -r "password" /var/log/app/*.log

# Output: Hundreds of plaintext passwords
john:SecretPass123!
admin:AdminPass456!
alice:MyP@ssw0rd!
```

Or API keys:

```bash
grep -r "Bearer" /var/log/app/*.log

# Output: All API keys ever used
Bearer sk_REDACTED_live_4eC39HqLyjWDarht8Kda
Bearer pk_test_51JxK9pE4cNwD2f
```

**Result:**

* Mass credential theft
* Complete account compromise
* API key leakage
* PCI DSS violation (logging credit cards)
* GDPR violation (logging PII)

**Finding it:** Access log files. Search for passwords, tokens, API keys, credit cards. Check if sensitive data logged.

**Exploit:**

```bash
# Common log locations
/var/log/application.log
/var/log/apache2/access.log
/var/log/nginx/access.log
~/.pm2/logs/
/opt/app/logs/

# Search for credentials
grep -rE "password|apikey|token|bearer|authorization" /var/log/

# Search for credit cards
grep -rE "\b[0-9]{13,16}\b" /var/log/
```

**The fix:**

Never log sensitive data:

```python
import logging
import re

SENSITIVE_FIELDS = ['password', 'api_key', 'token', 'credit_card', 'ssn']

def sanitize_request_data(data):
    sanitized = data.copy()
    for field in SENSITIVE_FIELDS:
        if field in sanitized:
            sanitized[field] = '[REDACTED]'
    return sanitized

@app.before_request
def log_request():
    # Sanitize before logging
    safe_data = sanitize_request_data(request.get_json() or {})
    logging.info(f"Request: {request.method} {request.path}")
    logging.info(f"Body: {safe_data}")
```

Or use allowlist approach:

```python
def log_request():
    # Only log safe fields
    safe_fields = ['username', 'email', 'user_agent', 'ip_address']
    log_data = {k: v for k, v in request.get_json().items() if k in safe_fields}
    logging.info(f"Request data: {log_data}")
```

***

#### Scenario 3: No Logging of Security Events (CWE-778, CWE-221)

Application doesn't log critical security events:

```python
@app.route('/admin/delete-user/<user_id>', methods=['POST'])
def delete_user(user_id):
    # VULNERABLE - No logging!
    user = User.query.get(user_id)
    db.session.delete(user)
    db.session.commit()
    return "User deleted"
```

**The vulnerability:**

No audit trail for:

* Failed login attempts
* Privilege escalation
* Access to sensitive data
* Configuration changes
* User deletions
* Password changes
* Admin actions

**The attack:**

Attacker gains admin access and deletes users:

```bash
curl -X POST http://target.com/admin/delete-user/1 \
  -H "Authorization: Bearer stolen_token"

# User 1 deleted - no trace in logs
# No record of who deleted it
# No timestamp of deletion
# No evidence of compromise
```

Investigation reveals nothing:

```bash
# Search logs for deletion
grep "delete" /var/log/app.log
# No results

# Search for user access
grep "user/1" /var/log/app.log
# No results

# Completely blind to the attack
```

**Result:**

* No incident detection
* No forensic evidence
* Cannot identify attacker
* Cannot determine scope of breach
* Compliance failure (PCI DSS requires logging)

**Finding it:** Review code for sensitive operations. Check if logging present. Test security events and verify they appear in logs.

**The fix:**

Log all security-relevant events:

```python
import logging

@app.route('/admin/delete-user/<user_id>', methods=['POST'])
def delete_user(user_id):
    user = User.query.get(user_id)
    
    # Log the security event
    logging.warning(
        f"SECURITY: User deletion - "
        f"User ID: {user_id}, "
        f"Deleted by: {session['username']}, "
        f"IP: {request.remote_addr}, "
        f"Timestamp: {datetime.now()}"
    )
    
    db.session.delete(user)
    db.session.commit()
    return "User deleted"
```

Or use structured logging:

```python
def log_security_event(event_type, details):
    log_entry = {
        "event": event_type,
        "actor": session.get('username'),
        "ip": request.remote_addr,
        "timestamp": datetime.now().isoformat(),
        "details": details
    }
    logging.warning(json.dumps(log_entry))

@app.route('/admin/delete-user/<user_id>', methods=['POST'])
def delete_user(user_id):
    user = User.query.get(user_id)
    
    log_security_event("user_deletion", {
        "user_id": user_id,
        "username": user.username
    })
    
    db.session.delete(user)
    db.session.commit()
    return "User deleted"
```

***

#### Scenario 4: Missing Context in Logs (CWE-223)

Application logs events but omits critical information:

```python
@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']
    
    if verify_password(username, password):
        # VULNERABLE - Missing context
        logging.info("Login successful")
        return "Welcome"
    else:
        # VULNERABLE - Missing context
        logging.info("Login failed")
        return "Invalid credentials"
```

**The vulnerability:**

Logs missing:

* Username (who logged in?)
* IP address (from where?)
* Timestamp (exact time?)
* User agent (what client?)
* Session ID (which session?)

**The attack:**

Attacker performs brute-force attack:

```bash
for password in $(cat passwords.txt); do
  curl -X POST http://target.com/login \
    -d "username=admin&password=$password"
done
```

Logs show:

```
2024-01-15 10:23:45 Login failed
2024-01-15 10:23:46 Login failed
2024-01-15 10:23:47 Login failed
...
2024-01-15 10:25:32 Login successful
```

But administrators cannot determine:

* Which account was attacked
* Source IP of attack
* If attack is ongoing
* If attack targeted multiple accounts

**Result:**

* Cannot detect brute-force attacks
* Cannot block attacker IP
* Cannot identify compromised accounts
* Incomplete forensic evidence

**Finding it:** Review log entries. Check if they contain sufficient context (who, what, when, where, how).

**The fix:**

Include comprehensive context:

```python
@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']
    
    if verify_password(username, password):
        logging.info(
            f"Login successful - "
            f"User: {username}, "
            f"IP: {request.remote_addr}, "
            f"User-Agent: {request.headers.get('User-Agent')}, "
            f"Timestamp: {datetime.now()}"
        )
        return "Welcome"
    else:
        logging.warning(
            f"Login failed - "
            f"User: {username}, "
            f"IP: {request.remote_addr}, "
            f"User-Agent: {request.headers.get('User-Agent')}, "
            f"Timestamp: {datetime.now()}"
        )
        return "Invalid credentials"
```

Or structured format:

```python
def log_login_event(username, success):
    log_entry = {
        "event": "login_attempt",
        "username": username,
        "success": success,
        "ip": request.remote_addr,
        "user_agent": request.headers.get('User-Agent'),
        "timestamp": datetime.now().isoformat(),
        "session_id": session.get('session_id')
    }
    
    if success:
        logging.info(json.dumps(log_entry))
    else:
        logging.warning(json.dumps(log_entry))
```

***

#### Scenario 5: Log Files Publicly Accessible

Application stores logs in web-accessible directory:

```
/var/www/html/logs/application.log
/var/www/html/logs/access.log
/var/www/html/logs/error.log
```

**The vulnerability:**

Log files accessible via web browser:

* No authentication required
* Directory listing enabled
* Logs contain sensitive data
* Anyone can download them

**The attack:**

```bash
# Discover logs
curl http://target.com/logs/
# Directory listing shows: application.log, error.log

# Download logs
curl http://target.com/logs/application.log > stolen_logs.txt

# Extract credentials
grep -E "password|token|api_key" stolen_logs.txt

# Output: All passwords and keys ever logged
```

Or automated scanning:

```bash
# Common log paths
for path in logs/ log/ debug/ application.log error.log access.log; do
  curl -s http://target.com/$path -o /dev/null -w "%{http_code} $path\n"
done

# 200 logs/application.log - FOUND!
```

**Result:**

* Complete log exposure
* Credential theft
* Attack pattern visibility
* Business logic disclosure
* User behavior tracking

**Finding it:** Check if `/logs/`, `/log/`, `/debug/` accessible. Try common log filenames. Look for directory listing.

**Exploit:**

```bash
# Test common paths
curl http://target.com/logs/
curl http://target.com/application.log
curl http://target.com/debug.log
curl http://target.com/error.log

# Automated scanner
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt -x log,txt
```

**The fix:**

Store logs outside web root:

```
# Wrong
/var/www/html/logs/

# Right
/var/log/application/
/opt/logs/
~/.local/logs/
```

Restrict permissions:

```bash
chmod 600 /var/log/application/*.log
chown app:app /var/log/application/*.log
```

Disable directory listing:

```apache
# Apache
<Directory /var/www/html>
    Options -Indexes
</Directory>
```

```nginx
# Nginx
autoindex off;
```

Block log access:

```apache
# .htaccess
<FilesMatch "\.(log|txt)$">
    Deny from all
</FilesMatch>
```

***

#### Scenario 6: Insufficient Failed Login Logging

Application only logs successful logins:

```python
@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']
    
    if verify_password(username, password):
        logging.info(f"User {username} logged in")
        return "Welcome"
    else:
        # VULNERABLE - Failed login not logged!
        return "Invalid credentials"
```

**The vulnerability:**

No logging of:

* Failed login attempts
* Password guessing attacks
* Brute-force attempts
* Account enumeration

**The attack:**

Attacker performs credential stuffing:

```bash
# Try 10,000 stolen passwords against admin account
for password in $(cat leaked_passwords.txt); do
  curl -X POST http://target.com/login \
    -d "username=admin&password=$password"
done
```

After 10,000 attempts, password found. But logs show:

```
2024-01-15 10:45:32 User admin logged in
```

No trace of:

* 10,000 failed attempts
* Source IP
* Attack duration
* Attack pattern

**Result:**

* Brute-force attacks undetected
* No alerting or blocking
* Successful compromise invisible until damage done

**Finding it:** Review authentication code. Check if failed attempts logged. Test with wrong passwords.

**The fix:**

Log all authentication attempts:

```python
@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']
    
    if verify_password(username, password):
        logging.info(
            f"LOGIN_SUCCESS - User: {username}, IP: {request.remote_addr}"
        )
        return "Welcome"
    else:
        # Log failed attempts
        logging.warning(
            f"LOGIN_FAILED - User: {username}, IP: {request.remote_addr}"
        )
        return "Invalid credentials"
```

With rate limiting:

```python
from collections import defaultdict
from datetime import datetime, timedelta

failed_attempts = defaultdict(list)

@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']
    ip = request.remote_addr
    
    # Check failed attempts
    recent_failures = [
        t for t in failed_attempts[ip]
        if t > datetime.now() - timedelta(minutes=15)
    ]
    
    if len(recent_failures) >= 5:
        logging.critical(
            f"BRUTE_FORCE_DETECTED - IP: {ip}, User: {username}, "
            f"Attempts: {len(recent_failures)}"
        )
        return "Too many failed attempts", 429
    
    if verify_password(username, password):
        # Clear failed attempts
        failed_attempts[ip] = []
        logging.info(f"LOGIN_SUCCESS - User: {username}, IP: {ip}")
        return "Welcome"
    else:
        # Record failed attempt
        failed_attempts[ip].append(datetime.now())
        logging.warning(
            f"LOGIN_FAILED - User: {username}, IP: {ip}, "
            f"Total failures: {len(recent_failures) + 1}"
        )
        return "Invalid credentials"
```

***

#### Scenario 7: Log Tampering (Inadequate Protection)

Logs stored with weak permissions:

```bash
-rw-rw-rw- 1 root root 1.2G Jan 15 10:23 application.log
```

**The vulnerability:**

Anyone can modify logs:

* Attacker removes evidence
* Attacker injects false entries
* Forensic evidence destroyed
* Incident investigation impossible

**The attack:**

Attacker compromises system, deletes traces:

```bash
# Remove all login failures
sed -i '/LOGIN_FAILED/d' /var/log/application.log

# Remove specific IP
sed -i '/192.168.1.100/d' /var/log/application.log

# Remove time range
sed -i '/2024-01-15 10:2[0-9]:/d' /var/log/application.log

# Or delete entire log
rm /var/log/application.log
```

Investigation finds nothing:

```bash
grep "LOGIN_FAILED" /var/log/application.log
# No results - attacker deleted them
```

**Result:**

* Evidence destruction
* Cannot prove breach occurred
* Cannot identify attacker
* Legal proceedings impossible

**Finding it:** Check file permissions on logs. Test if logs can be modified or deleted.

**The fix:**

Restrict log permissions:

```bash
chmod 640 /var/log/application/*.log
chown root:adm /var/log/application/*.log
```

Use append-only flag:

```bash
# Linux
chattr +a /var/log/application.log
# Now file can only be appended to, not modified or deleted

# Remove flag (requires root)
chattr -a /var/log/application.log
```

Send logs to central server:

```python
import logging
from logging.handlers import SysLogHandler

# Send to remote syslog
handler = SysLogHandler(address=('logs.company.com', 514))
logging.getLogger().addHandler(handler)

# Now logs stored remotely where attacker can't delete them
```

Use write-once storage:

```bash
# Send logs to AWS S3 with object lock
aws s3 cp /var/log/application.log s3://logs-bucket/ \
  --storage-class GLACIER \
  --metadata "retention=7years"
```

***

### Mitigation Strategies

**Sanitize all logged input**

```python
import re

def sanitize_for_log(text):
    # Remove control characters
    sanitized = re.sub(r'[\x00-\x1f\x7f-\x9f]', '', text)
    # Remove newlines
    sanitized = sanitized.replace('\n', '').replace('\r', '')
    return sanitized

logging.info(f"User {sanitize_for_log(username)} logged in")
```

**Never log sensitive data**

```python
SENSITIVE_FIELDS = [
    'password', 'passwd', 'pwd',
    'api_key', 'apikey', 'secret',
    'token', 'auth', 'authorization',
    'credit_card', 'cc', 'cvv',
    'ssn', 'social_security'
]

def redact_sensitive(data):
    if isinstance(data, dict):
        return {
            k: '[REDACTED]' if k.lower() in SENSITIVE_FIELDS else v
            for k, v in data.items()
        }
    return data

logging.info(f"Request: {redact_sensitive(request_data)}")
```

**Log all security events**

```python
# Always log:
# - Authentication (success/failure)
# - Authorization failures
# - Input validation failures
# - Privilege escalation
# - Admin actions
# - Configuration changes
# - Data access/modification/deletion

def log_security_event(event_type, **kwargs):
    log_data = {
        'event': event_type,
        'timestamp': datetime.now().isoformat(),
        'user': session.get('username'),
        'ip': request.remote_addr,
        **kwargs
    }
    logging.warning(json.dumps(log_data))
```

**Include comprehensive context**

```python
# Always include:
# - Who (user, session ID)
# - What (action performed)
# - When (timestamp)
# - Where (IP, location, endpoint)
# - How (method, user agent)
# - Result (success/failure)

logging.info(
    f"Event: {event}, "
    f"User: {user}, "
    f"IP: {ip}, "
    f"Time: {timestamp}, "
    f"Endpoint: {endpoint}, "
    f"Result: {result}"
)
```

**Protect log files**

```bash
# Restrict permissions
chmod 640 /var/log/application.log
chown root:adm /var/log/application.log

# Use append-only
chattr +a /var/log/application.log

# Store outside web root
# /var/log/ NOT /var/www/html/logs/
```

**Use structured logging**

```python
import logging
import json

def structured_log(level, event_type, **kwargs):
    log_entry = {
        'timestamp': datetime.now().isoformat(),
        'level': level,
        'event': event_type,
        **kwargs
    }
    logging.log(getattr(logging, level), json.dumps(log_entry))

# Usage
structured_log('INFO', 'login_success', user='admin', ip='192.168.1.1')
```

**Centralize logs**

```python
# Send to central logging server
from logging.handlers import SysLogHandler

handler = SysLogHandler(address=('logs.company.com', 514))
logging.getLogger().addHandler(handler)

# Or use ELK stack, Splunk, Datadog, etc.
```

**Monitor and alert**

```python
# Set up automated monitoring
# Alert on:
# - Multiple failed logins
# - Privilege escalation attempts
# - Unusual access patterns
# - Missing logs (gaps in timeline)

# Example: Alert on 5 failed logins in 5 minutes
if failed_login_count > 5:
    send_alert(f"Brute force detected from {ip}")
```

***

<https://cwe.mitre.org/data/definitions/117.html>

&#123;% embed url="<https://cwe.mitre.org/data/definitions/221.html>" %}

&#123;% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html>" %}

&#123;% embed url="<https://www.pcisecuritystandards.org/>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/223.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/532.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/778.html>" %}

&#123;% embed url="<https://csrc.nist.gov/publications/detail/sp/800-92/final>" %}
