# Hardcoded & Default Credentials

### Why They're Critical

Hardcoded credentials are the easiest attack vector:

* **No exploitation needed:** Just use the credentials
* **Permanently compromised:** Can't be changed without code modification
* **Mass exposure:** Same credentials on all deployments
* **Easy discovery:** Found in source code, binaries, git history
* **Insider threat:** Any developer has production credentials
* **Complete bypass:** Authentication entirely bypassed

A single hardcoded credential can compromise thousands of deployments.

***

### Real-World Attack Scenarios

#### Scenario 1: Hardcoded Database Password in Code

Database credentials hardcoded in application:

```python
# config.py
DB_HOST = 'prod-db.internal.company.com'
DB_USER = 'app_user'
DB_PASSWORD = 'SuperSecret123!@#'  # HARDCODED!
DB_DATABASE = 'production'

# Used everywhere in application
def get_db_connection():
    return psycopg2.connect(
        host=DB_HOST,
        user=DB_USER,
        password=DB_PASSWORD,
        database=DB_DATABASE
    )
```

**The vulnerability:**

Password visible in:

* Source code
* Compiled binaries
* Git history
* Backup files
* Version control systems
* CI/CD logs

**The attack:**

1. Attacker clones repository: `git clone https://repo.git`
2. Finds hardcoded credentials: `grep -r "DB_PASSWORD" .`
3. Uses credentials to connect: `psql -h prod-db.internal... -U app_user`
4. Complete database access achieved
5. Can read/modify all data

Or:

```bash
# Search git history
git log -p | grep -i "password\|secret"
# Found: DB_PASSWORD = 'SuperSecret123!@#'

# Use to access database
psql -h prod-db.internal.company.com -U app_user -d production
# SELECT * FROM users;  # Access all data
```

**Result:**

* Complete database compromise
* All user data exposed
* Ability to modify data
* No audit trail (legitimate credentials used)

**Finding it:** Search source code for hardcoded passwords. Check git history. Decompile binaries. Review configuration files.

**Exploit:**

```bash
# Search for credentials
grep -r "password\|secret\|key\|token" . --include="*.py" --include="*.js" --include="*.java"

# Or in git history
git log -p -S"password=" | head -50
```

***

#### Scenario 2: Default Admin Credentials Never Changed

Router/appliance installed with default credentials:

```
Device: Cisco Router
Default Username: admin
Default Password: admin

Device: HP Printer
Default Username: admin
Default Password: 12345

Device: AWS RDS
Default Master Username: admin
```

**The vulnerability:**

Credentials are:

* Well-known (publicly documented)
* Never changed after installation
* Same across all deployments
* Impossible to distinguish from legitimate access

**The attack:**

Attacker finds device on network:

```bash
# Scan network for devices
nmap -sV -p 22,23,80,443,8080 target.com

# Try default credentials
ssh admin@target.com
# Password: admin
# Login successful!

# Or
curl -u admin:admin http://target.com:8080/admin
# Full access to admin panel
```

Attacker now has:

* Complete device control
* Can modify configuration
* Can access all data
* Can create backdoors

**Result:**

* Device compromise
* Network access
* Potential lateral movement
* Data theft

**Finding it:** Research default credentials for found devices/software. Try common default passwords (admin/admin, admin/12345, etc.).

***

#### Scenario 3: Empty Password in Configuration File

Configuration file with blank password:

```ini
# config.ini
[database]
host=db.internal.company.com
username=app_user
password=  # EMPTY!

[cache]
host=redis.internal
password=  # EMPTY!

[api]
key=
secret=  # EMPTY!
```

**The vulnerability:**

Empty passwords mean:

* No authentication
* Any user can connect
* Configuration stored but never validated
* Applications crash when password required
* Developers might add placeholder and forget to fill it

**The attack:**

Attacker:

1. Finds config file (repository, backup, exposed server)
2. Sees empty password
3. Connects without authentication: `psql -h db.internal -U app_user`
4. No password prompted
5. Full database access

```bash
# Try to connect without password
psql -h db.internal -U app_user -d production
# No password prompt, direct access!

redis-cli -h redis.internal
# No authentication, direct access to cache!
```

**Result:**

* Unauthenticated database access
* Complete data compromise
* No audit trail

**Finding it:** Examine configuration files. Look for empty values. Try connecting without credentials.

***

#### Scenario 4: API Key Hardcoded in Frontend

API key embedded in JavaScript:

```javascript
// app.js
const API_KEY = 'sk_REDACTED_live_4eC39HqLyjWDarht8Zlt5Kda';
const STRIPE_SECRET = 'rk_live_51234567890abcdef';

async function processPayment(amount) {
    const response = await fetch('https://api.stripe.com/v1/charges', {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${API_KEY}`,
            'X-Stripe-Secret': STRIPE_SECRET
        },
        body: JSON.stringify({amount: amount})
    });
    return response.json();
}
```

**The vulnerability:**

API keys in frontend:

* Visible in source code: `View Source` in browser
* Visible in network traffic (unless HTTPS)
* Visible in compiled bundles
* Visible in git history
* Visible in browser cache

**The attack:**

Attacker views page source:

```bash
# View source in browser or:
curl http://example.com/app.js | grep -i "api_key\|secret"

# Found: sk_REDACTED_live_4eC39HqLyjWDarht8Zlt5Kda

# Use API key to access Stripe
curl https://api.stripe.com/v1/charges \
  -H "Authorization: Bearer sk_REDACTED_live_4eC39HqLyjWDarht8Zlt5Kda"

# Can now:
# - Create charges
# - Access customer data
# - Perform refunds
# - Commit fraud
```

**Result:**

* API key compromise
* Fraudulent charges
* Customer data theft
* Complete payment system access

**Finding it:** View page source. Check network tab. Search for API keys. Decompile bundles.

***

#### Scenario 5: SSH Private Key in Repository

Private SSH key committed to git:

```bash
# File: .ssh/id_rsa (accidentally committed)
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA2x3k7v+9x8k7v+9x8k7v+9x8k7v+9x8k7v+9x8k7v+9x
...
-----END RSA PRIVATE KEY-----
```

**The vulnerability:**

SSH key in repository:

* Cloneable by anyone with repo access
* In git history forever
* Allows impersonation of users/services
* Can be used even after removal (still in history)

**The attack:**

```bash
# Clone repository
git clone https://repo.git

# Find SSH key
find . -name "id_rsa" -o -name "*private*"

# Use key to SSH to servers
ssh -i .ssh/id_rsa deploy@production-server.com

# Now has access to production servers
# Can modify code, access data, create backdoors
```

Or:

```bash
# Check git history for deleted files
git log --all --full-history -- ".ssh/id_rsa"

# Recover deleted key from history
git show <commit>:.ssh/id_rsa > recovered_key
chmod 600 recovered_key

# Use recovered key
ssh -i recovered_key deploy@production-server.com
```

**Result:**

* Server compromise
* Code modification
* Data theft
* Complete infrastructure access

**Finding it:** Search repository for private keys. Check git history for deleted files. Look for SSH keys in configs.

***

#### Scenario 6: Default Credentials in Backup Account

Test/backup account with default credentials never removed:

```sql
-- Database accounts
CREATE USER 'test'@'localhost' IDENTIFIED BY 'test';
CREATE USER 'backup'@'localhost' IDENTIFIED BY '123456';
CREATE USER 'admin'@'localhost' IDENTIFIED BY 'admin';

-- Only production account has strong password
-- But test/backup accounts still exist with default passwords
```

**The vulnerability:**

Test/backup accounts:

* Created during development
* Never removed from production
* Have same permissions as main account
* Use default/weak passwords
* Easy to guess and compromise

**The attack:**

Attacker:

1. Discovers database server
2. Tries common test account names: test, backup, admin, demo
3. Tries default passwords: test, 123456, password, admin
4. One of them works: `mysql -u backup -p123456`
5. Full database access

```bash
# Try default test accounts
mysql -h db.internal -u test -ptest

# Or
mysql -h db.internal -u backup -p123456

# Or
mysql -h db.internal -u admin -padmin

# One succeeds!
mysql> GRANT ALL ON *.* TO user@'%';
# Now attacker has admin access
```

**Result:**

* Database compromise
* Privilege escalation
* All data exposed

**Finding it:** List database users. Try default credentials. Look for test/backup accounts.

***

### How to Identify Hardcoded Credentials During Testing

**1. Search source code**

```bash
# Search for password patterns
grep -r "password\|secret\|key\|api_key\|token" . \
  --include="*.py" \
  --include="*.js" \
  --include="*.java" \
  --include="*.php"

# More specific
grep -r "=.*['\"]" . | grep -i "password\|secret"
```

**2. Check git history**

```bash
# Search entire history
git log -p -S "password=" | head -100

# Search for deleted files
git log --all --full-history -- "*private*"
git log --all --full-history -- "*.pem"
git log --all --full-history -- "*.key"
```

**3. Examine configuration files**

```bash
# Find config files
find . -name "*.ini" -o -name "*.conf" -o -name "*.yaml" -o -name "*.env"

# Check contents
cat config.ini | grep -i "password\|secret"
```

**4. Decompile binaries**

```bash
# For Java
jd-cli application.jar | grep -i "password\|secret"

# For .NET
dnSpy application.dll
```

**5. Try default credentials**

Research default credentials for:

* Operating system
* Database
* Web server
* Application
* Hardware

**6. Check environment variables**

```bash
# Look for credentials in docker files
cat Dockerfile | grep -i "env\|password\|secret"

# Or in scripts
grep -r "export.*PASSWORD\|API_KEY" .
```

***

### Mitigation Strategies

**Never hardcode credentials**

Use external configuration:

```python
# Bad
DB_PASSWORD = 'SuperSecret123!@#'

# Good
import os
DB_PASSWORD = os.getenv('DB_PASSWORD')

# Even better - secrets management
import boto3
client = boto3.client('secretsmanager')
secret = client.get_secret_value(SecretId='db-password')
DB_PASSWORD = secret['SecretString']
```

**Always change default credentials**

On first deployment:

```bash
# Remove test accounts
DROP USER 'test'@'localhost';

# Change default passwords
ALTER USER 'admin'@'localhost' IDENTIFIED BY 'strong_random_password';
```

**Use environment variables**

```python
import os

DATABASE_URL = os.getenv('DATABASE_URL')
API_KEY = os.getenv('API_KEY')
STRIPE_SECRET = os.getenv('STRIPE_SECRET')
```

**Use secrets management systems**

* AWS Secrets Manager
* Azure Key Vault
* HashiCorp Vault
* Kubernetes Secrets

```python
from aws_secretsmanager import client

def get_db_password():
    response = client.get_secret_value(SecretId='prod/db/password')
    return response['SecretString']
```

**Prevent credentials in source control**

```bash
# .gitignore
.env
*.key
*.pem
config.ini
secrets.json
private/
```

**Use .gitignore strictly**

```bash
# Verify no secrets committed
git log -p | grep -i "password\|secret\|key" | head -5
# Should return nothing
```

**Rotate credentials regularly**

Even if hardcoded (legacy), rotate every 30-90 days:

```bash
# Generate new credentials
# Update all deployments
# Retire old credentials
```

**Scan for credentials in CI/CD**

```bash
# GitGuardian
detect-secrets scan

# TruffleHog
truffleHog filesystem . --json
```

**Audit and review**

* Regular code reviews
* Scanning tools
* Security audits
* Penetration testing

***

{% embed url="<https://cwe.mitre.org/data/definitions/258.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/259.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/798.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/1392.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/1393.html>" %}

{% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Credentials_Cheat_Sheet.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/798.htm>" %}

{% embed url="<https://github.com/Yelp/detect-secrets>" %}

{% embed url="<https://github.com/trufflesecurity/trufflehog>" %}

{% embed url="<https://www.gitguardian.com/>" %}
