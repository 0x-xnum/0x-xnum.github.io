# Cryptographic Key Management and Implementation

Cryptographic key management failures occur when applications mishandle encryption keys, fail to properly authenticate communications, reuse cryptographic values, or implement encryption incorrectly. This includes:

Even perfect algorithms fail when keys are mismanaged or implementation is flawed.

***

### Real-World Attack Scenarios

#### Scenario 1: Hardcoded Cryptographic Key (CWE-321)

An application has a hardcoded encryption key in source code:

```python
# In source code, version control, and deployments
SECRET_KEY = "MySecretKey2024!@#$%"

def encrypt_password(password):
    cipher = AES.new(SECRET_KEY, AES.MODE_CBC)
    return cipher.encrypt(password)
```

**The vulnerability:**

Hardcoded keys are discoverable:

* Visible in source code
* Exposed in git history
* Visible in compiled binaries
* Visible in decompiled applications
* Same key used for all deployments

**The attack:**

Attacker:

1. Clones git repository
2. Searches for strings: `key =`, `secret =`, `password =`
3. Finds: `SECRET_KEY = "MySecretKey2024!@#$%"`
4. Uses this key to decrypt all encrypted data
5. All user passwords, API keys exposed

**Result:**

* Complete encryption bypass
* All encrypted data decrypted
* Single source for compromising entire application

**The fix:** Never hardcode keys:

```python
import os
from dotenv import load_dotenv

load_dotenv()
SECRET_KEY = os.getenv('SECRET_KEY')

# Or use Key Management System
from aws_secretsmanager import client
secret = client.get_secret_value(SecretId='encryption-key')
```

**Finding it:** Search source code for `key =`, `secret =`, `password =`. Check git history. Decompile if compiled code. Search environment files.

**Exploit:**

```bash
# Search git history
git log -p | grep -i "key\|secret\|password"

# Or search current code
grep -r "SECRET_KEY\|API_KEY\|DB_PASSWORD" .

# Found! Use key to decrypt all data
```

***

#### Scenario 2: Unprotected Transport of Credentials (CWE-523)

Application sends API credentials and passwords over HTTP:

```python
import requests

def authenticate_user(username, password):
    # VULNERABLE - HTTP, not HTTPS
    response = requests.post(
        'http://api.example.com/login',
        data={'username': username, 'password': password}
    )
    return response.json()
```

**The vulnerability:**

Credentials sent unencrypted:

* Network sniffer captures credentials
* Attacker gains authentication
* No data protection in transit
* All user passwords exposed

**The attack:**

Attacker on same network (coffee shop WiFi, ISP, corporate network):

```bash
# Sniff HTTP traffic
tcpdump -i eth0 'tcp port 80' -A | grep -i "username\|password"

# Captures:
# username=admin&password=SuperSecret123

# Use credentials to access accounts
curl -u admin:SuperSecret123 http://api.example.com/data
```

**Result:**

* Credential theft
* Authentication bypass
* Account takeover
* Data access

**The fix:** Always use HTTPS:

```python
response = requests.post(
    'https://api.example.com/login',  # HTTPS
    data={'username': username, 'password': password}
)
```

And don't send passwords in requests body - use tokens:

```python
# Better: Send username, get token back
response = requests.post(
    'https://api.example.com/login',
    json={'username': username, 'password': password}
)
token = response.json()['token']

# Use token for subsequent requests
requests.get(
    'https://api.example.com/data',
    headers={'Authorization': f'Bearer {token}'}
)
```

**Finding it:** Check for HTTP endpoints. Monitor network traffic. Intercept requests with Burp Suite. Look for credentials in plaintext transmission.

***

#### Scenario 3: Reusing Nonce/IV (CWE-323)

Stream cipher uses same nonce for multiple messages:

```python
from Crypto.Cipher import ChaCha20

nonce = os.urandom(12)

def encrypt_message1(msg1):
    cipher = ChaCha20.new(key=key, nonce=nonce)
    return cipher.encrypt(msg1)

def encrypt_message2(msg2):
    cipher = ChaCha20.new(key=key, nonce=nonce)  # REUSED NONCE!
    return cipher.encrypt(msg2)

# Message 1: plaintext1 XOR keystream
# Message 2: plaintext2 XOR keystream

# Attacker: ciphertext1 XOR ciphertext2 = plaintext1 XOR plaintext2
```

**The vulnerability:**

Nonce reuse with stream cipher:

* Keystream is deterministic
* XORing two ciphertexts reveals plaintext relationship
* Attacker can recover both plaintexts

**The attack:**

Attacker intercepts two encrypted messages with same nonce:

```
Ciphertext1 = Plaintext1 XOR Keystream
Ciphertext2 = Plaintext2 XOR Keystream

XOR together:
C1 XOR C2 = (P1 XOR KS) XOR (P2 XOR KS) = P1 XOR P2

If attacker knows P1, they can recover P2:
P2 = P1 XOR (C1 XOR C2)
```

**Result:**

* Plaintext disclosure
* Message recovery without key

**The fix:** Use random nonce each time:

```python
def encrypt_message(msg):
    nonce = os.urandom(12)  # NEW random nonce each time
    cipher = ChaCha20.new(key=key, nonce=nonce)
    ciphertext = cipher.encrypt(msg)
    return nonce + ciphertext  # Send nonce with ciphertext
```

**Finding it:** Look for reused nonces/IVs. Check if nonce regenerated for each encryption. Analyze traffic for patterns indicating nonce reuse.

***

#### Scenario 4: Key Exchange Without Entity Authentication (CWE-322)

Two systems exchange keys without verifying identity:

```python
# System A sends public key
# System B sends public key
# They trust whoever claims to be the other party
```

**The vulnerability:**

No authentication during key exchange:

* Attacker intercepts key exchange
* Sends their own public key instead
* Becomes man-in-the-middle
* Can decrypt all communications

**The attack:**

```
System A                  Attacker                 System B
    |                        |                        |
    +------ "Here's my ------>+                        |
    |        public key       |                        |
    |                        (Intercepts, stores)      |
    |                        |------ "Here's my ------>+
    |                        |    public key          |
    |                        |   (Attacker's key)      |
    |                        |                        |
    |<------ "Here's my ------+                        |
    |        public key       |                        |
    |  (Attacker's key)       |                        |
    |                        |<----- "Here's my -------+
    |                        |      public key        |
    |                        (Intercepts, stores)     |
```

Now:

* A encrypts to attacker's key, attacker decrypts, re-encrypts to B
* B encrypts to attacker's key, attacker decrypts, re-encrypts to A
* Attacker reads all communications

**Result:**

* Man-in-the-middle attack
* Complete encryption bypass
* All communications compromised

**The fix:** Authenticate public keys:

```python
# Option 1: Certificate-based PKI
# Trust CA, verify certificate chain
# Ensures public key belongs to claimed identity

# Option 2: Out-of-band verification
# Verify fingerprint through secure channel
# "Did you get public key ABC123...?"

# Option 3: Pre-shared secrets
# Both parties know shared secret
# Use for initial authentication
```

**Finding it:** Check for certificate verification. Test if MITM proxy succeeds. Look for missing signature verification.

***

#### Scenario 5: Using Expired Keys (CWE-324)

Application continues using encryption key after expiration:

```python
import time

KEY_EXPIRATION = 1704067200  # Jan 1, 2024

def encrypt_with_key(data):
    current_time = int(time.time())
    
    # VULNERABLE - No expiration check
    cipher = AES.new(SECRET_KEY, AES.MODE_GCM)
    return cipher.encrypt(data)

# If current_time > KEY_EXPIRATION, key should be rotated
# But code doesn't check!
```

**The vulnerability:**

Using keys past expiration:

* Keys should be rotated periodically
* Expired keys compromise forward secrecy
* Extended exposure window increases breach risk
* Violates security policy

**The attack:**

Attacker:

1. Compromises old encryption key
2. Even though it's expired, application still uses it
3. Can decrypt all data encrypted with that key indefinitely
4. Key rotation never happened

**Result:**

* Extended vulnerability window
* Increased risk of compromise
* All data encrypted with old key vulnerable

**The fix:** Enforce key expiration:

```python
import time

KEY_EXPIRATION = 1704067200  # Jan 1, 2024

def encrypt_with_key(data):
    current_time = int(time.time())
    
    if current_time > KEY_EXPIRATION:
        raise Exception("Key has expired! Rotate to new key.")
    
    cipher = AES.new(CURRENT_KEY, AES.MODE_GCM)
    return cipher.encrypt(data)
```

Better: Automatic key rotation:

```python
def get_current_key():
    # Query key management system
    # Returns current valid key
    # KMS automatically rotates when expired
    return KMS.get_key('encryption-key')
```

**Finding it:** Check key expiration logic. Look for keys without expiration dates. Verify key rotation is enforced.

***

#### Scenario 6: Missing Signature Verification (CWE-347)

Application doesn't verify digital signatures:

```python
import json
from Crypto.PublicKey import RSA
from Crypto.Signature import pkcs1_v1_5

def verify_message(message, signature, public_key):
    # VULNERABLE - No verification!
    # Just assumes message is valid
    return json.loads(message)
```

**The vulnerability:**

No signature verification:

* Attacker can forge messages
* Modify data without detection
* No authenticity guarantee
* Message integrity compromised

**The attack:**

Attacker intercepts message and signature:

```
Message: {"account_id": 12345, "amount": 100}
Signature: [digital_signature]
```

Attacker modifies message:

```
Message: {"account_id": 99999, "amount": 1000000}
Signature: [old_signature_still_attached]
```

Without verification, application accepts modified message.

**Result:**

* Message forgery
* Data tampering
* Integrity violation

**The fix:** Always verify signatures:

```python
def verify_message(message, signature, public_key):
    verifier = pkcs1_v1_5.new(public_key)
    digest = SHA256.new(message.encode())
    
    # Verify signature
    if not verifier.verify(digest, signature):
        raise Exception("Signature verification failed!")
    
    return json.loads(message)
```

**Finding it:** Search for signature verification code. Test with modified signatures. Look for missing `verify()` calls.

***

#### Scenario 7: Algorithm Downgrade Attack (CWE-757)

SSL/TLS negotiation allows downgrading to weak algorithms:

```python
# Server allows both:
# - TLS 1.3 with AES-256-GCM (secure)
# - SSL 3.0 with DES (broken)

# Client requests TLS 1.3
# Attacker intercepts negotiation
# Forces downgrade to SSL 3.0
# Connection uses DES encryption
```

**The vulnerability:**

Accepting weak algorithms during negotiation:

* Attacker forces weak cipher suite
* Perfect encryption downgraded to broken
* Defeats purpose of strong algorithms

**The attack:**

```
Client: "I support TLS 1.3, TLS 1.2, SSL 3.0"
Attacker intercepts, modifies:
Modified: "I support SSL 3.0"
Server: "OK, using SSL 3.0"

Now using 56-bit DES instead of 256-bit AES!
```

**Result:**

* Forced use of weak encryption
* Data encrypted with breakable algorithm
* Complete encryption bypass

**The fix:** Only accept secure algorithms:

```python
# Only allow TLS 1.2+
ssl_context.minimum_version = ssl.TLSVersion.TLSv1_2

# Only allow strong cipher suites
ssl_context.set_ciphers('ECDHE+AESGCM:ECDHE+CHACHA20')

# Disable weak ciphers
ssl_context.options |= ssl.OP_NO_SSLv3
ssl_context.options |= ssl.OP_NO_COMPRESSION
```

**Finding it:** Check TLS configuration. Test with Nessus or testssl.sh. Look for SSLv3, DES, RC4 support.

***

### Mitigation Strategies

**Never hardcode keys**

Use Key Management Systems:

* AWS Secrets Manager
* Azure Key Vault
* HashiCorp Vault
* Google Cloud KMS

**Implement key rotation**

```python
# Rotate keys regularly
# Old keys still used to decrypt, but not to encrypt
# Eventually deprecated and removed
```

**Use HTTPS everywhere**

```
https:// for all endpoints
Redirect HTTP to HTTPS
HSTS headers
Certificate pinning for mobile apps
```

**Verify signatures always**

```python
# Never skip signature verification
if not verify(message, signature):
    reject()
```

**Authenticate key exchange**

* Use certificates (PKI)
* Out-of-band verification
* Pre-shared secrets for bootstrap

**Enforce key expiration**

```python
if key.expired():
    rotate_to_new_key()
```

**Use random, unique nonces/IVs**

```python
for each_encryption:
    nonce = os.urandom(12)  # Fresh for each message
```

**Disable weak algorithms**

* Minimum TLS 1.2
* No DES, RC4, MD5, SHA-1
* Only strong cipher suites

**Audit key usage**

* Log all key operations
* Monitor for anomalies
* Alert on unauthorized access
* Compliance reporting

***

{% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html>" %}

{% embed url="<https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-57p1r4.pdf>" %}

{% embed url="<https://aws.amazon.com/secrets-manager/>" %}

{% embed url="<https://www.vaultproject.io/>" %}

{% embed url="<https://github.com/drwetter/testssl.sh>" %}
