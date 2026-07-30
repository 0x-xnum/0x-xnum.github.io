# Certificate Validation Failures

Vulnerabilities in how applications validate SSL/TLS certificates:

Together, they allow man-in-the-middle attacks despite HTTPS.

***

### Real-World Attack Scenarios

#### Scenario 1: Accepting Any Certificate

Application doesn't validate certificate at all:

```python
import requests

# VULNERABLE - verify=False
response = requests.get('https://api.example.com', verify=False)

# Or in urllib
import urllib.request
import ssl

context = ssl._create_unverified_context()
urllib.request.urlopen('https://api.example.com', context=context)
```

**The attack:**

Attacker performs MITM:

1. User connects to attacker's fake server
2. Fake server uses self-signed certificate
3. Application accepts ANY certificate (even self-signed)
4. Attacker intercepts all HTTPS traffic
5. All data encrypted to attacker, not real server

```bash
# Attacker sets up fake server with self-signed cert
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365

# Start fake server
python -m http.server 443 --bind 0.0.0.0

# Application connects to attacker's server thinking it's real
# Traffic decrypted by attacker
```

**Result:**

* Complete MITM attack
* All HTTPS data compromised
* Credentials stolen
* Complete encryption bypass

**Finding it:** Check certificate validation code. Look for `verify=False`. Test with self-signed certificate.

***

#### Scenario 2: Hostname Mismatch

Certificate for wrong hostname accepted:

```
Certificate issued for: evil.com
Request to: example.com
Check: Only verifies certificate validity, not hostname
Result: Connection accepted even though hostname doesn't match!
```

**The attack:**

```
Attacker obtains certificate for attacker.com
User tries to connect to example.com
Attacker intercepts connection
Sends certificate for attacker.com
Application checks: "Is certificate valid?" → Yes
Application doesn't check: "Is this the right domain?" → No
Connection established to attacker's server
```

```bash
# Certificate is valid but for wrong domain
openssl x509 -in cert.pem -text -noout | grep "Subject:"
# Subject: CN = attacker.com

# But application connects to example.com
# Validation only checks expiration/signature, not CN
```

**Result:**

* MITM attack succeeds
* Encryption provides false sense of security
* All traffic compromised

**Finding it:** Intercept with proxy using wrong certificate. Check if accepted.

***

#### Scenario 3: Wildcard Certificate Mismatch

Wildcard certificate abused:

```
Certificate: *.evil.com (matches any subdomain of evil.com)
User connects to: example.com
Attacker claims to be: evil.com.example.com (wrong match)
Result: Connection fails correctly
But if: example.com.evil.com, connection might succeed
```

**Finding it:** Test with wildcard certificates. Try subdomain variations.

***

#### Scenario 4: Self-Signed Certificate Accepted

Application trusts self-signed certificates:

```python
# Vulnerable code
context = ssl.create_default_context()
context.check_hostname = False  # WRONG!
context.verify_mode = ssl.CERT_NONE  # WRONG!

# Now accepts any certificate, self-signed or invalid
```

**The attack:**

Attacker uses self-signed certificate for MITM:

```bash
# Attacker generates self-signed cert
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem

# Application accepts it without validation
# MITM successful
```

**Result:**

* MITM attack
* All traffic compromised

**Finding it:** Look for `CERT_NONE`, `check_hostname = False`. Test with self-signed cert.

***

#### Scenario 5: Origin Validation Error

Application doesn't validate request origin:

```python
@app.route('/api/transfer', methods=['POST'])
def transfer_funds():
    # No check of request origin!
    amount = request.json['amount']
    destination = request.json['destination']
    
    # Process transfer
    transfer_money(amount, destination)
    return "Success"
```

Attacker from different origin can make request:

```bash
# From attacker's domain
curl -X POST https://bank.com/api/transfer \
  -H "Content-Type: application/json" \
  -H "Origin: https://attacker.com" \
  -d '{"amount": 1000, "destination": "attacker"}'

# If no CORS check, request succeeds
# Funds transferred to attacker
```

**Result:**

* CSRF attack
* Unauthorized actions
* Fund theft

**Finding it:** Check CORS headers. Try requests from different origins. Check for Origin validation.

***

### Mitigation Strategies

**Always verify certificates**

```python
# Good
response = requests.get('https://api.example.com')
# Automatically verifies certificate

# Or explicit
response = requests.get('https://api.example.com', verify=True)
```

**Use proper SSL context**

```python
import ssl
import certifi

context = ssl.create_default_context(cafile=certifi.where())
# Verifies against trusted CA bundle
```

**Never disable verification**

```python
# NEVER do this
context = ssl.create_default_context()
context.check_hostname = False
context.verify_mode = ssl.CERT_NONE

# Always verify!
```

**Use certificate pinning**

For mobile apps, pin expected certificates:

```swift
// iOS - Certificate Pinning
let publicKeyPinning = ["example.com": ["sha256/...certificate hash..."]]

// If certificate doesn't match pinned hash, reject connection
```

**Implement CORS properly**

```python
from flask_cors import CORS

# Only allow specific origins
CORS(app, origins=["https://trusted-domain.com"])

# Not: CORS(app)  which allows all origins!
```

**Validate origin header**

```python
@app.route('/api/transfer', methods=['POST'])
def transfer_funds():
    # Check origin
    if request.origin != 'https://trustedbank.com':
        return "Forbidden", 403
    
    # Process request
    process_transfer()
```

**Use HSTS**

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

Tells browsers to always use HTTPS, never HTTP.

***

### Related CWE Entries

&#123;% embed url="<https://cwe.mitre.org/data/definitions/295.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/297.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/346.html>" %}

&#123;% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/295.html>" %}

&#123;% embed url="<https://github.com/drwetter/testssl.sh>" %}
