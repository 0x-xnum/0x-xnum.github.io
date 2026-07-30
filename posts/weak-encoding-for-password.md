# Weak Encoding for Password

CWE-261 occurs when an application **stores, transmits, or processes passwords using weak or reversible encoding** instead of **strong cryptographic hashing**. This makes it easier for attackers to retrieve plaintext passwords through **decryption, encoding flaws, or weak obfuscation methods**.

**Common Weak Encoding Methods:**

\- **Base64 Encoding** → Easily reversible with `base64 -d` or `atob()`.

\- **XOR Encoding** → Weak if the key is short or static.

&#x20;\- **Custom Obfuscation** → Can often be reversed with pattern analysis.

***

#### **How Attackers Exploit CWE-261**

**1. Reversing Base64-Encoded Passwords**

Many applications mistakenly store passwords as Base64-encoded strings:

```python
import base64
password = "hunter2"
encoded = base64.b64encode(password.encode()).decode()
print(encoded)  # aHVudGVyMg==
```

**Attack:**

```bash
echo "aHVudGVyMg==" | base64 -d
```

**Base64 is NOT encryption**—it’s just encoding!

***

**2. XOR Encoding Can Be Easily Reversed**

A weak XOR-based encoding method:

```python
password = "supersecret"
key = 5  # Static key
encoded = ''.join(chr(ord(c) ^ key) for c in password)
print(encoded)
```

**Attack:** If the key is small (like `5`), brute-force is trivial.

***

**3. Reversing Custom Obfuscation**

Some apps use **rot-based encoding** (e.g., ROT13):

```python
import codecs
print(codecs.encode("mypassword", "rot_13"))
```

**Attack:** Decode with the same function.

***

#### **How to Identify CWE-261?**

\- **Look for Base64/XOR in password handling functions.**\
\- **Check JavaScript files for atob() or weak obfuscation.**\
\- **Dump the database and analyze password storage format.**\
\- **Test API responses for encoded credentials.**
