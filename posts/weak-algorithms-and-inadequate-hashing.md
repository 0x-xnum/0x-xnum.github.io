# Weak Algorithms and Inadequate Hashing

Broken cryptography occurs when applications use cryptographic algorithms, hashing functions, or encryption methods that are mathematically broken, cryptographically obsolete, or provide insufficient security. This includes:

When cryptography is broken, it provides a false sense of security. Data appears encrypted, but attackers can decrypt it trivially or bypass the protection entirely.

***

### Real-World Attack Scenarios

#### Scenario 1: DES Encryption (56-bit) in Modern Applications

An e-commerce site encrypts credit card numbers using DES:

```python
from Crypto.Cipher import DES

def encrypt_card(card_number):
    key = b'12345678'  # 56-bit key
    cipher = DES.new(key, DES.MODE_ECB)
    encrypted = cipher.encrypt(card_number.encode())
    return encrypted.hex()
```

**The vulnerability:**

DES was broken in 1998. It has only 56 bits of key space:

* Can be brute-forced in **1 second** with modern GPUs
* Entire database of encrypted cards cracked in **hours**
* No computational effort required

**The attack:**

```bash
# Attacker captures encrypted credit cards from database
# Brute force 56-bit key space
hashcat -m 14000 encrypted_cards.txt -a 3 ?a?a?a?a?a?a?a?a

# Within hours, all cards decrypted
# Fraudulent charges across all credit cards
```

**Result:**

* All encrypted credit card numbers exposed
* PCI DSS violation (massive fines)
* Customer data breach
* Lawsuits and reputational damage

**The fix:** Use AES-256:

```python
from Crypto.Cipher import AES

def encrypt_card(card_number):
    key = os.urandom(32)  # 256-bit key
    cipher = AES.new(key, AES.MODE_GCM)
    encrypted = cipher.encrypt(card_number.encode())
    return encrypted.hex()
```

**Finding it:** Search code for DES, RC4, MD5. Check cryptographic imports. Look for 56-bit or smaller keys.

***

#### Scenario 2: MD5 Password Hashing

A user management system hashes passwords with MD5:

```php
<?php
function hash_password($password) {
    return md5($password);  // BROKEN
}

function verify_password($password, $hash) {
    return md5($password) === $hash;
}
?>
```

**The vulnerability:**

MD5 is cryptographically broken:

* Hash collisions discovered (same hash for different inputs)
* Precomputed rainbow tables available for all common passwords
* Can crack 8-character password in **seconds**
* No salt (same password always produces same hash)

**The attack:**

1. Attacker obtains password hash from database: `5f4dcc3b5aa765d61d8327deb882cf99`
2. Looks up hash in rainbow tables
3. Instantly finds password: `password123`
4. Accesses user account

Or:

```bash
# Generate MD5 rainbow table for all passwords
# hashcat -m 0 -a 0 passwords.txt -o rainbowtable.txt

# Look up captured hash
grep "5f4dcc3b5aa765d61d8327deb882cf99" rainbowtable.txt
# Output: password123
```

**Result:**

* All passwords exposed
* Complete account compromise
* Attacker can reset other users' passwords
* Access admin accounts if any use common passwords

**The fix:** Use bcrypt or Argon2:

```python
import bcrypt

def hash_password(password):
    salt = bcrypt.gensalt(rounds=12)
    return bcrypt.hashpw(password.encode(), salt)

def verify_password(password, hash):
    return bcrypt.checkpw(password.encode(), hash)
```

**Finding it:** Search for `md5(`, `MD5`, `hash()`. Look for password hashing without salt or slow hashes.

***

#### Scenario 3: SHA-1 with No Salt

A system hashes passwords with SHA-1:

```javascript
const crypto = require('crypto');

function hashPassword(password) {
    return crypto.createHash('sha1').update(password).digest('hex');
}

// Stores: password123 → f8b86b3d10b4e3f23afdae3228fa93e9f0f
```

**The vulnerability:**

SHA-1 is broken (collisions found in 2017):

* No salt (deterministic hashing)
* Same password always produces same hash
* Rainbow tables available for all common passwords
* Can crack in seconds

**The attack:**

User registers with password "summer2024":

```
SHA1("summer2024") = 3a10a3b5f5e5c8f8e8c8f8e8c8f8e8c8f8e8c8f
```

An attacker:

1. Looks up the hash online or in rainbow tables
2. Instantly finds password: `summer2024`
3. Can now login as that user

**Multiple users with same password:**

```
User A: summer2024 → hash
User B: summer2024 → same hash

Attacker knows two users have identical passwords
```

**Result:**

* Password exposure
* Account takeover
* Pattern recognition (same hash = same password)

**The fix:** Use bcrypt with salt:

```javascript
const bcrypt = require('bcrypt');

async function hashPassword(password) {
    const salt = await bcrypt.genSalt(10);
    return await bcrypt.hash(password, salt);
}

// Even identical passwords produce different hashes
```

**Finding it:** Search for `sha1(`, `SHA1`, `MD5`. Look for password hashing without cryptographic functions.

***

#### Scenario 4: Weak Salt or Predictable Salt

A system uses a weak salt for password hashing:

```python
import hashlib

def hash_password(password):
    salt = "fixed_salt_1234"  # WEAK - same for all passwords
    return hashlib.pbkdf2_hmac('sha256', password.encode(), salt, 100000)
```

**The vulnerability:**

Same salt for all passwords:

* Attacker can precompute hashes for common passwords once
* All users with common passwords instantly compromised
* Defeats purpose of salt

**The attack:**

Attacker precomputes hashes for top 1 million passwords with salt "fixed\_salt\_1234":

```
password123 + fixed_salt_1234 → hash1
summer2024 + fixed_salt_1234 → hash2
admin123 + fixed_salt_1234 → hash3
...
```

When attacker obtains password database:

* Looks up each hash in their precomputed table
* Instantly finds all passwords

**Result:**

* Complete password database compromise
* All common passwords exposed

**The fix:** Use random salt for each password:

```python
import hashlib
import os

def hash_password(password):
    salt = os.urandom(32)  # Random salt for each password
    return hashlib.pbkdf2_hmac('sha256', password.encode(), salt, 100000)
```

Now precomputation is impossible (billions of combinations).

***

#### Scenario 5: Insufficient Hashing Iterations (Fast Hashing)

Password hashing with only 1 iteration:

```python
def hash_password(password):
    # Fast but insecure
    return hashlib.pbkdf2_hmac('sha256', password.encode(), salt, 1)
```

**The vulnerability:**

With only 1 iteration:

* Cracking can try millions of passwords per second on GPU
* 8-character password cracked in seconds
* Billions of password combinations tested in minutes

**The attack:**

Attacker has password database with 1-iteration hashes:

```bash
# Crack with hashcat on GPU
hashcat -m 10000 hashes.txt /usr/share/wordlists/rockyou.txt

# Speed: 1 billion hashes per second
# 8-character password cracked in < 1 second
# Dictionary attack covers most users in minutes
```

**Result:**

* All passwords cracked
* Complete compromise

**The fix:** Use bcrypt with proper rounds:

```python
import bcrypt

def hash_password(password):
    # Minimum 10 rounds, preferably 12+
    salt = bcrypt.gensalt(rounds=12)
    return bcrypt.hashpw(password.encode(), salt)
    
# Each round doubles computation time
# Makes GPU attacks impractical
```

**Finding it:** Look for hash functions with < 100,000 iterations. Search for PBKDF2 with low iteration count. Check password hashing code for iteration count.

***

#### Scenario 6: RSA Encryption Without OAEP Padding (CWE-780)

An application encrypts data with RSA but doesn't use proper padding:

```python
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_v1_5

def encrypt_data(data, public_key):
    # VULNERABLE - using raw RSA without OAEP
    cipher = PKCS1_v1_5.new(public_key)
    encrypted = cipher.encrypt(data)
    return encrypted
```

**The vulnerability:**

RSA without OAEP padding is vulnerable to:

* Chosen ciphertext attacks
* Padding oracle attacks
* Malleable encryption (attackers modify ciphertext)
* Decryption failures that leak information

**The attack:**

Attacker intercepts encrypted message. Through repeated encryption/decryption attempts, they can:

1. Determine if decryption succeeds or fails
2. Learn information about plaintext
3. Gradually recover the original message
4. Forge signatures

Example padding oracle attack:

```bash
# Attacker modifies ciphertext slightly
# Sends to server
# Observes if decryption succeeds or fails
# Each success/failure reveals 1 bit of information
# Repeat millions of times to recover plaintext
```

**Result:**

* Message decryption without key
* Signature forgery
* Complete encryption bypass

**The fix:** Use OAEP padding:

```python
from Crypto.Cipher import PKCS1_OAEP

def encrypt_data(data, public_key):
    cipher = PKCS1_OAEP.new(public_key)
    encrypted = cipher.encrypt(data)
    return encrypted
```

OAEP makes padding oracle attacks infeasible.

***

#### Scenario 7: Reversible One-Way Hash (CWE-328)

Developers think they're hashing but actually use encryption:

```python
import base64

def "hash" password(password):
    # NOT A HASH - This is just encoding!
    return base64.b64encode(password.encode()).decode()

# password123 → cGFzc3dvcmQxMjM=
```

**The vulnerability:**

Base64 is not a hash:

* Reversible (easily decoded)
* Not cryptographic
* Anyone can decrypt it

**The attack:**

Attacker obtains "hashed" password: `cGFzc3dvcmQxMjM=`

```bash
# Decode it
echo "cGFzc3dvcmQxMjM=" | base64 -d
# Output: password123
```

Password instantly recovered.

**Result:**

* Passwords in plaintext
* Complete account compromise

**The fix:** Use actual cryptographic hash:

```python
import bcrypt

def hash_password(password):
    salt = bcrypt.gensalt()
    return bcrypt.hashpw(password.encode(), salt)
```

One-way hash cannot be reversed.

***

#### Scenario 8: ECB Mode (Electronic Codebook)

Encryption in ECB mode reveals patterns:

```python
from Crypto.Cipher import AES

def encrypt_credit_card(card_number):
    cipher = AES.new(key, AES.MODE_ECB)  # VULNERABLE
    encrypted = cipher.encrypt(card_number)
    return encrypted
```

**The vulnerability:**

ECB mode encrypts each block independently:

* Same plaintext block → same ciphertext block
* Patterns in plaintext visible in ciphertext
* Attacker can see which cards are identical
* Reveals structure of data

**The attack:**

Attacker intercepts encrypted credit cards:

```
Card A encrypted: 4f3b2a1c... 4f3b2a1c... (repeating blocks)
Card B encrypted: 7f8c9d2e... 7f8c9d2e... (different repeating)
Card C encrypted: 4f3b2a1c... 4f3b2a1c... (same as Card A)

Attacker knows Cards A and C are identical!
```

**Result:**

* Pattern analysis reveals data structure
* Identical records identified
* Reduced encryption effectiveness

**The fix:** Use GCM or CBC mode with random IV:

```python
from Crypto.Cipher import AES
import os

def encrypt_credit_card(card_number):
    iv = os.urandom(16)
    cipher = AES.new(key, AES.MODE_GCM, nonce=iv)
    encrypted = cipher.encrypt(card_number)
    return iv + encrypted  # Send IV with ciphertext
```

Each encryption produces different ciphertext.

***

### Mitigation Strategies

**Use strong symmetric encryption**

```python
from cryptography.fernet import Fernet
# or
from Crypto.Cipher import AES

# AES-256-GCM
cipher = AES.new(key, AES.MODE_GCM)
```

**Never use**

* DES, 3DES
* MD5, SHA-1
* RC4, ECB mode
* Reversible "hashes"
* Custom cryptography

**Hash passwords properly**

```python
import bcrypt

def hash_password(password):
    salt = bcrypt.gensalt(rounds=12)  # 12+ rounds
    return bcrypt.hashpw(password.encode(), salt)

# Or Argon2
import argon2

ph = argon2.PasswordHasher()
hash = ph.hash(password)
```

**Use random, unique salt**

```python
import os

salt = os.urandom(32)  # Random for each password
```

**Use secure random number generation**

```python
import secrets
import os

# For cryptographic randomness
token = secrets.token_hex(32)

# or
key = os.urandom(32)

# NOT: random.randint() or hashlib.md5(time.time())
```

**Use authenticated encryption**

Always use GCM, ChaCha20-Poly1305, or similar:

```python
cipher = AES.new(key, AES.MODE_GCM)
```

**Key rotation**

* Rotate encryption keys regularly
* Never hardcode keys
* Use key management systems (KMS)
* Store keys in secure vaults

**Enforce in code review**

* Review all cryptographic code
* Verify algorithm choices
* Check key length and iterations
* Prevent weak algorithms from merging

***

{% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html>" %}

{% embed url="<https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-132.pdf>" %}

{% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html>" %}

{% embed url="<https://cryptography.io/>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/327.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/326.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/328.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/759.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/760.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/780.html>" %}
