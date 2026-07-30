# Use of Obsolete Function

Use of Obsolete Function occurs when code uses deprecated or obsolete functions that have known security vulnerabilities, performance issues, or design flaws. Developers continue using old APIs even though better, safer alternatives exist. The presence of obsolete functions signals that code hasn't been actively reviewed or maintained, often indicating deeper security and stability problems lurking nearby.

Programming languages and libraries deprecate functions when they become insecure, inefficient, or outdated. Using them anyway is a red flag for poor code quality and potential vulnerabilities.

***

### Real-World Attack Scenarios

#### Scenario 1: Buffer Overflow via Gets()

The `gets()` function reads a line from input without checking buffer bounds:

```c
#include <stdio.h>

int main() {
    char password[20];
    
    printf("Enter password: ");
    gets(password);  // OBSOLETE - No bounds checking!
    
    if (strcmp(password, "secret123") == 0) {
        printf("Access granted\n");
    }
    
    return 0;
}
```

**The vulnerability:**

`gets()` reads input until a newline, with NO size limit. If input exceeds 20 bytes, it overwrites memory beyond the buffer.

**The attack:**

An attacker inputs 50 bytes of data:

```
Enter password: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\x08\x04\x00\x00\x10\x01\x00\x00
```

The overflow overwrites:

* Local variables
* Return address
* Saved frame pointer
* Function pointers

**Result:**

* Application crashes
* Arbitrary code execution (if ROP gadgets available)
* Complete system compromise

**The fix:** Use `fgets()` with explicit size limit:

```c
fgets(password, sizeof(password), stdin);
```

**Finding it:** Search code for `gets()`. Test with inputs exceeding buffer size. Check for buffer overflows. Try fuzzing with long inputs.

**Exploit:**

```bash
# Create input that overflows buffer
python -c "print('A' * 100)" | ./vulnerable_program

# Program crashes or executes injected code
```

***

#### Scenario 2: Password Verification Using getpw()

The deprecated `getpw()` function looks up user passwords with a buffer overflow vulnerability:

```c
#include <pwd.h>
#include <crypt.h>
#include <string.h>

int verify_password(int uid, char *plainpw) {
    char pwdline[1024];
    char *cryptpw;
    int result = 0;
    int i;
    
    // OBSOLETE - getpw() has buffer overflow vulnerability
    getpw(uid, pwdline);
    
    for (i = 0; i < 3; i++) {
        cryptpw = strtok(pwdline, ":");
        pwdline = 0;
    }
    
    result = strcmp(crypt(plainpw, cryptpw), cryptpw) == 0;
    return result;
}
```

**The vulnerability:**

The `getpw()` function has multiple problems:

1. No bounds checking on the `pwdline` buffer
2. If the password file entry is longer than buffer, it overflows
3. The function is deprecated since the 1990s
4. Has been removed from modern systems

**The attack:**

An attacker creates a user account with an extremely long home directory or shell path in /etc/passwd. When `getpw()` reads this entry, it overflows the buffer.

**Result:**

* Buffer overflow
* Potential code execution with the privileges of the process running password verification
* Authentication bypass

**The fix:** Use `getpwuid()` which is safe:

```c
struct passwd *pwd = getpwuid(uid);
if (pwd != NULL) {
    // Use pwd->pw_passwd safely
}
```

**Finding it:** Search for `getpw(`. Check password verification functions. Look for manual password file parsing.

***

#### Scenario 3: Weak Cryptography via DES

Developers use obsolete `crypt()` function with DES encryption (deprecated since 1970s):

```c
#include <crypt.h>

void hash_password(char *password) {
    char salt[2] = "ab";
    char *hash = crypt(password, salt);  // OBSOLETE - Uses weak DES
    printf("Password hash: %s\n", hash);
}
```

**The vulnerability:**

DES is obsolete and broken:

* Only 56-bit key (brute-forceable with modern hardware in seconds)
* Can be cracked in < 1 day with GPUs
* No salt randomization (salt only 2 characters = 4,096 possibilities)
* Deterministic hashing (same password always produces same hash)

**The attack:**

An attacker:

1. Obtains password hashes from the system
2. Runs dictionary attack against DES hashes
3. Cracks passwords in hours (not years like with bcrypt)
4. Gains access to user accounts

**Result:**

* All user passwords compromised
* Complete system access
* No protection against rainbow tables

**The fix:** Use modern hashing like bcrypt or Argon2:

```python
import bcrypt

hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
```

**Finding it:** Search for `crypt()`. Look for DES usage. Check password hashing functions. Test if hashes are weak.

***

#### Scenario 4: Insecure Random Number Generation

Code uses obsolete `rand()` function for generating security tokens:

```c
#include <stdlib.h>

char* generate_csrf_token() {
    static char token[64];
    
    // OBSOLETE - rand() is predictable!
    for (int i = 0; i < 63; i++) {
        token[i] = 'A' + (rand() % 26);
    }
    token[63] = '\0';
    
    return token;
}
```

**The vulnerability:**

`rand()` is not cryptographically secure:

* Uses a predictable PRNG
* Seed value often predictable (time-based)
* Insufficient entropy for security
* Linear congruential generator is broken

**The attack:**

An attacker:

1. Discovers the seed (usually current time)
2. Predicts all future `rand()` values
3. Forges CSRF tokens
4. Impersonates users

**Result:**

* CSRF tokens can be forged
* Account takeover
* Unauthorized actions performed on behalf of users

**The fix:** Use cryptographically secure random:

```c
// Use /dev/urandom or crypto libraries
int secure_random = arc4random();  // BSD/macOS
// Or: getrandom() on Linux
```

**Finding it:** Search for `rand()`. Look for cryptographic operations using weak randomness. Check token generation.

***

#### Scenario 5: Character Encoding Issues with String Constructor

Java code uses obsolete String constructor that doesn't specify charset:

```java
// OBSOLETE - Charset not specified
String str = new String(byteArray);

// Modern replacement
String str = new String(byteArray, StandardCharsets.UTF_8);
```

**The vulnerability:**

Without explicit charset:

1. Behavior depends on system default charset
2. May differ between Windows, Linux, macOS
3. May change between Java versions
4. Can cause encoding attacks

**Example attack:**

Attacker sends bytes that:

* Decode differently depending on charset
* Bypass validation on one charset but pass on another
* Result in different data after charset conversion

**Result:**

* Validation bypass
* XSS due to encoding confusion
* Data corruption

**Finding it:** Search for `new String(byte[])`. Look for character encoding operations. Test with non-ASCII characters.

***

#### Scenario 6: Deprecated OpenSSL Functions

Code uses deprecated OpenSSL functions for encryption:

```c
#include <openssl/des.h>

void encrypt_data(unsigned char *plaintext, unsigned char *key) {
    unsigned char ciphertext[DES_CBLOCK];
    DES_cblock cblock_key;
    
    // OBSOLETE - DES is weak
    DES_set_odd_parity(&cblock_key);
    DES_ecb_encrypt(&plaintext, &ciphertext, &cblock_key, 1);
}
```

**The vulnerability:**

Multiple issues:

1. DES is cryptographically broken
2. ECB mode reveals patterns in plaintext
3. Function deprecated in OpenSSL 1.1.0, removed in 3.0.0
4. Code won't compile on modern systems

**The attack:**

An attacker:

1. Captures encrypted data
2. Identifies patterns (ECB reveals repeated blocks)
3. Launches known-plaintext attacks
4. Decrypts sensitive information

**Result:**

* Encryption provides no actual security
* Data easily decrypted
* System incompatible with modern OpenSSL

**Finding it:** Search for deprecated OpenSSL functions. Check OpenSSL version. Look for DES, MD5, RC4, SHA-1.

**Exploit:**

```bash
# System tries to build code using obsolete OpenSSL API
gcc vulnerable.c -lssl -lcrypto

# Error: 'DES_ecb_encrypt' is unavailable
# Function removed in OpenSSL 3.0.0
```

***

#### Scenario 7: Deprecated PHP Functions

PHP code uses obsolete `mysql_*` functions (removed in PHP 7.0):

```php
<?php
// OBSOLETE - Removed in PHP 7.0
$conn = mysql_connect("localhost", "user", "password");
mysql_select_db("database");

$query = "SELECT * FROM users WHERE id = " . $_GET['id'];
$result = mysql_query($query);
?>
```

**The vulnerability:**

Multiple critical issues:

1. `mysql_*` functions removed entirely
2. No prepared statements (SQL injection vulnerable)
3. Functions deprecated in 2001, removed 2016
4. Code hasn't been maintained in years

**The attack:**

SQL injection through `id` parameter:

```
GET /page.php?id=1 OR 1=1
```

Query becomes:

```sql
SELECT * FROM users WHERE id = 1 OR 1=1
```

Returns all users instead of one.

**Result:**

* SQL injection vulnerability
* Database compromise
* All user data exposed
* Code incompatible with PHP 7+

**Finding it:** Search for `mysql_connect`, `mysql_query`, `mysql_fetch`. Look for string concatenation in SQL queries. Check PHP version.

**Exploit:**

```bash
# Try to run on PHP 7.0+
php vulnerable.php

# Fatal error: Call to undefined function mysql_connect()
```

***

### Mitigation Strategies

**Replace all deprecated functions immediately**

Replace `gets()` → `fgets()`

```c
// Bad
gets(buffer);

// Good
fgets(buffer, sizeof(buffer), stdin);
```

Replace `strcpy()` → `strncpy()` or `strlcpy()`

```c
// Bad
strcpy(dest, src);

// Good
strncpy(dest, src, sizeof(dest) - 1);
dest[sizeof(dest) - 1] = '\0';
```

Replace `crypt()` → `bcrypt` or `Argon2`

```python
# Bad
hash = crypt(password, salt)

# Good
hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
```

Replace `rand()` → secure random

```c
// Bad
int token = rand();

// Good
int token = arc4random();  // or getrandom()
```

Replace `mysql_*` → `mysqli` or `PDO`

```php
// Bad
$result = mysql_query("SELECT * FROM users WHERE id = " . $_GET['id']);

// Good
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$_GET['id']]);
```

**Regular code audits**

* Schedule quarterly code reviews
* Focus on deprecated function usage
* Create refactoring roadmap
* Track and remove technical debt

**Update dependencies regularly**

* Keep language updated to latest stable version
* Update libraries regularly
* Remove deprecated library versions
* Monitor security advisories

**Enable compiler warnings**

```bash
gcc -Wall -Wdeprecated-declarations code.c
clang -Wdeprecated code.m
javac -Xlint:deprecation Code.java
```

**Automated scanning** Use tools to detect deprecated usage:

* SonarQube
* Checkmarx
* FindBugs
* Clang-Tidy
* ESLint

**Documentation**

* Document why replacements needed
* Track deprecation lifecycle
* Maintain migration guides
* Archive obsolete code safely

***

{% embed url="<https://cwe.mitre.org/data/definitions/78.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/120.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/327.html>" %}

{% embed url="<https://owasp.org/www-community/vulnerabilities/Use_of_Obsolete_Methods>" %}

{% embed url="<https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard>" %}
