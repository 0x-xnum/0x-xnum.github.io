# PRNG Failures and Predictable Secrets

Weak randomness vulnerabilities occur when applications use insufficient entropy, predictable random number generators (PRNGs), or improper seeding for security-critical operations. This includes:

When randomness is weak, attackers can predict tokens, keys, session IDs, and other security-critical values, completely bypassing encryption and authentication.

***

### Real-World Attack Scenarios

#### Scenario 1: Using random.randint() for Cryptography

Python application generates session tokens using built-in random:

```python
import random

def generate_session_token():
    # VULNERABLE - Not cryptographically secure
    return str(random.randint(0, 999999999999))
```

**The vulnerability:**

`random` uses Mersenne Twister PRNG:

* Deterministic (predictable with 624 observations)
* Not cryptographically secure
* Seed can be recovered from output
* Produces only \~32 bits of entropy (despite larger numbers)

**The attack:**

Attacker observes a few session tokens:

```
Token 1: 382716
Token 2: 384921
Token 3: 386105
```

They recognize the Mersenne Twister pattern and predict:

```
Token 4: 387289
Token 5: 388472
Token 6: 389655
```

Using the predicted token, attacker hijacks any user's session.

**Result:**

* Session hijacking without credentials
* Account takeover
* Complete authentication bypass

**The fix:** Use cryptographically secure random:

```python
import secrets

def generate_session_token():
    return secrets.token_hex(32)  # 256 bits of secure randomness
```

**Finding it:** Search for `random.randint()`, `random.choice()`, `Math.random()`. Look for PRNG usage in security context.

***

#### Scenario 2: Predictable Seed from Timestamp

Application seeds PRNG with current time:

```python
import random
import time

def generate_token():
    # VULNERABLE - seed is predictable
    seed = int(time.time())
    random.seed(seed)
    return random.randint(0, 2**32 - 1)
```

**The vulnerability:**

Seeding with timestamp:

* Current time is publicly known
* Attacker knows seed is within a narrow time window
* Can try all possible seeds in seconds
* Brute-force attack against weak PRNG

**The attack:**

Token generated at 2024-01-15 10:23:45.

Attacker knows seed is one of:

```
10:23:40 to 10:23:50 = only 10 possibilities

For each possibility:
  seed = 1705315420 + offset
  recover token = f(seed)
  
Try 10 seeds, get 10 possible tokens
One of them will match user's actual token
```

With timing information, attacker narrows window to 1-2 seconds = predictable.

**Result:**

* Session token prediction
* Account takeover
* Authentication bypass

**The fix:** Use cryptographically secure random (doesn't need explicit seeding):

```python
import secrets

def generate_token():
    # Auto-seeded from system entropy
    return secrets.randbits(256)
```

**Finding it:** Search for `time.time()`, `datetime.now()`, `System.nanoTime()` used for seeding. Look for `seed()` calls with predictable values.

***

#### Scenario 3: Reusing the Same PRNG Seed

Application initializes PRNG once at startup:

```python
import random

# One-time initialization
random.seed(12345)

# All sessions use same PRNG instance
def generate_token():
    return random.randint(0, 2**32 - 1)

# Session 1: 383729
# Session 2: 384105
# Session 3: 385192
```

**The vulnerability:**

Same seed = same PRNG sequence:

* All tokens are deterministic
* Attacker predicts entire sequence
* Every session token can be forged
* Affects all users, all time periods

**The attack:**

Attacker:

1. Learns the seed (hardcoded, discoverable in code)
2. Recreates the PRNG sequence
3. Knows all past and future tokens
4. Hijacks all sessions ever created

**Result:**

* Complete authentication bypass
* All users compromised
* Historical sessions exploitable

**The fix:** Use cryptographically secure random:

```python
import secrets

def generate_token():
    return secrets.token_hex(32)  # New entropy each time
```

**Finding it:** Look for `seed()` calls. Check if PRNG initialized once globally. Look for hardcoded seeds.

***

#### Scenario 4: Small Randomness Space

Password reset token generated from small space:

```python
import random

def generate_reset_token():
    # Only 4 digits = 10,000 possibilities
    return str(random.randint(0, 9999))
```

**The vulnerability:**

Only 10,000 possible tokens:

* Attacker can brute-force all tokens in seconds
* Reset password for any account
* No prediction needed, pure brute force

**The attack:**

```bash
# Try all 10,000 possible tokens
for token in range(0, 10000):
    response = requests.post(
        'http://target.com/reset-password',
        data={'token': f'{token:04d}', 'new_password': 'hacked123'}
    )
    if response.status_code == 200:
        print(f"Success! Token: {token}")
        break
```

One of 10,000 attempts will work (average: 5,000 attempts).

**Result:**

* Complete password reset
* Account takeover
* Email verification bypassed

**The fix:** Use large randomness space:

```python
import secrets

def generate_reset_token():
    return secrets.token_urlsafe(32)  # 256+ bits
```

Now \~2^256 possibilities (infeasible to brute-force).

**Finding it:** Look for small random ranges. Check password reset/2FA code generation. Look for 4-6 digit tokens.

***

#### Scenario 5: Sequential or Predictable IDs

Application generates user IDs sequentially:

```python
class User:
    next_id = 1000
    
    def __init__(self, name):
        self.id = User.next_id
        User.next_id += 1
```

**The vulnerability:**

Sequential IDs are trivial to predict:

* User 1001, 1002, 1003...
* Attacker enumerates all user IDs
* Can access data of any user by changing ID in request
* No authentication needed to discover users

**The attack:**

```bash
# Enumerate all users
for id in range(1000, 2000):
    response = requests.get(f'http://target.com/api/user/{id}')
    if response.status_code == 200:
        print(response.json())  # User data leaked
```

Attacker discovers:

* All user accounts and emails
* User data
* Sensitive information

**Result:**

* User enumeration
* Data disclosure
* Horizontal privilege escalation

**The fix:** Use cryptographically random IDs:

```python
import secrets

def generate_user_id():
    return secrets.token_hex(16)  # Unpredictable
```

Or use UUID:

```python
import uuid

def generate_user_id():
    return str(uuid.uuid4())
```

**Finding it:** Look for sequential IDs, auto-increment in databases. Check if IDs follow patterns. Test ID enumeration.

***

#### Scenario 6: IV Reuse in CBC Mode

Encryption uses static IV:

```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes

iv = get_random_bytes(16)  # Generated once at startup

def encrypt_message(data):
    cipher = AES.new(key, AES.MODE_CBC, iv)
    return cipher.encrypt(data)

# All messages encrypted with same IV!
```

**The vulnerability:**

Reusing IV with CBC mode:

* Identical plaintext produces identical ciphertext
* Attacker learns when same message is sent
* Can mount known-plaintext attacks
* IV should be random for each encryption

**The attack:**

Attacker intercepts encrypted messages and recognizes patterns:

```
Message 1: [encrypted_bytes_1]
Message 2: [different_bytes]
Message 3: [encrypted_bytes_1]  # Same as message 1!

Attacker knows messages 1 and 3 are identical
```

With known-plaintext attack:

* Attacker guesses message 1 content
* Confirms by matching ciphertext of message 3

**Result:**

* Information disclosure
* Pattern analysis
* Reduced encryption effectiveness

**The fix:** Use random IV for each encryption:

```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes

def encrypt_message(data):
    iv = get_random_bytes(16)  # NEW random IV each time
    cipher = AES.new(key, AES.MODE_CBC, iv)
    encrypted = cipher.encrypt(data)
    return iv + encrypted  # Send IV with ciphertext
```

**Finding it:** Look for static IVs. Check if IV is regenerated per encryption. Test if identical plaintexts produce identical ciphertexts.

***

#### Scenario 7: Weak PRNG in Cryptographic Context

C application uses `rand()` for encryption:

```c
#include <stdlib.h>

void encrypt_data(unsigned char *data) {
    unsigned char key[32];
    
    // VULNERABLE - rand() not cryptographic
    for (int i = 0; i < 32; i++) {
        key[i] = rand() % 256;
    }
    
    // key is predictable!
    AES_encrypt(data, key);
}
```

**The vulnerability:**

`rand()` in C:

* Weak PRNG (linear congruential generator)
* Only \~32 bits of entropy despite 32-byte key
* Predictable and biased
* Not cryptographically secure

**The attack:**

Attacker recovers the weak key through:

1. Observing output
2. Predicting PRNG state
3. Deriving the actual key
4. Decrypting all encrypted data

**Result:**

* Encryption completely broken
* All encrypted data exposed
* Key derivation fails

**The fix:** Use cryptographically secure random:

```c
#include <openssl/rand.h>

void encrypt_data(unsigned char *data) {
    unsigned char key[32];
    
    // Secure random
    RAND_bytes(key, 32);
    
    AES_encrypt(data, key);
}
```

**Finding it:** Search for `rand()`, `Math.random()`, `random.random()`. Look for PRNG usage in cryptographic functions.

***

### How to Identify Weak Randomness During Testing

**1. Check randomness sources**

```bash
# Search for weak RNGs
grep -r "random\|rand\|Math.random" *.py *.java *.js *.c

# Look for PRNG instead of cryptographic random
grep -r "seed(" *.py *.java
```

**2. Test for predictability**

Generate multiple tokens/IDs and analyze:

```bash
# Collect tokens
for i in {1..100}; do
  curl http://target.com/generate-token >> tokens.txt
done

# Analyze pattern
cat tokens.txt | sort | uniq -c
# Check if sequential, patterns, or repeating
```

**3. Check seed sources**

Look for:

* `time.time()`, `System.currentTimeMillis()` (predictable)
* Hardcoded seeds
* Global seed initialization
* User-controlled seeds

**4. Test IDs for enumeration**

```bash
# Try sequential IDs
for id in {1..100}; do
  curl http://target.com/api/user/$id -H "Authorization: Bearer token"
done

# If accessible, IDs are predictable
```

**5. Analyze randomness quality**

```bash
# Collect large sample of "random" values
# Use statistical analysis tools
ent -c tokens.txt  # Entropy analysis

# Look for patterns, non-uniform distribution
```

**6. Check IV reuse**

Encrypt same plaintext multiple times:

```bash
for i in {1..10}; do
  curl -X POST http://target.com/encrypt -d "data=test"
done

# If ciphertext identical, IV is reused
```

***

### Mitigation Strategies

**Always use cryptographically secure randomness**

```python
# Python
import secrets
token = secrets.token_hex(32)

# Java
import java.security.SecureRandom;
SecureRandom sr = new SecureRandom();
byte[] token = new byte[32];
sr.nextBytes(token);

# C
#include <openssl/rand.h>
unsigned char buf[32];
RAND_bytes(buf, 32);

# JavaScript
const token = require('crypto').randomBytes(32).toString('hex');
```

**Never use for cryptography:**

* `random.randint()`, `random.choice()`
* `Math.random()`
* `rand()`, `srand()`
* Time-based seeds
* Hardcoded seeds

**Use random values for each operation**

```python
# Wrong
seed_once = 12345
for i in range(100):
    token = generate_from_seed(seed_once)  # Predictable

# Right
for i in range(100):
    token = secrets.token_hex(32)  # Fresh entropy
```

**Generate cryptographic IDs**

```python
import uuid
import secrets

# Option 1: UUID
user_id = uuid.uuid4()

# Option 2: Random hex
user_id = secrets.token_hex(16)
```

**Use random IV/nonce each time**

```python
from Crypto.Random import get_random_bytes
from Crypto.Cipher import AES

for each_message:
    iv = get_random_bytes(16)  # NEW random IV
    cipher = AES.new(key, AES.MODE_CBC, iv)
    encrypted = cipher.encrypt(message)
    send(iv + encrypted)
```

**Sufficient entropy for purpose**

* Session tokens: 128+ bits
* Cryptographic keys: 256+ bits
* Password reset tokens: 256+ bits
* API keys: 256+ bits
* IDs: 128+ bits (UUID size)

**Test randomness**

```bash
# Collect sample and analyze
# Use NIST randomness tests
# Verify no patterns, high entropy
```

***

{% embed url="<https://docs.python.org/3/library/secrets.html>" %}

{% embed url="<https://docs.oracle.com/javase/8/docs/api/java/security/SecureRandom.html>" %}

{% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Randomness_Cheat_Sheet.html>" %}

{% embed url="<https://tools.ietf.org/html/rfc4122>" %}

{% embed url="<https://www.openssl.org/docs/man1.1.1/man3/RAND_bytes.html>" %}
