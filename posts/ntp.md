# NTP

**`Default Port: 123 (UDP)`**

**Network Time Protocol (NTP)** is a networking protocol for clock synchronization between computer systems over packet-switched, variable-latency data networks. NTP is one of the oldest internet protocols still in use and is critical for maintaining accurate time across networks. Precise time synchronization is essential for security protocols like Kerberos, logging systems, and distributed applications. NTP servers can leak system information and, in some cases, be exploited for amplification attacks.

### Connect <a href="#connect" id="connect"></a>

#### Using ntpq <a href="#using-ntpq" id="using-ntpq"></a>

```
# Query NTP server
ntpq -c readvar target.com

# Get peer information
ntpq -p target.com

# Interactive mode
ntpq target.com

# Show system variables
ntpq -c sysinfo target.com
```

#### Using ntpdate <a href="#using-ntpdate" id="using-ntpdate"></a>

```
# Check time from NTP server
ntpdate -q target.com

# Synchronize time (requires root)
ntpdate target.com

# Debug mode
ntpdate -d target.com
```

#### Using ntpdc <a href="#using-ntpdc" id="using-ntpdc"></a>

```
# Connect to NTP server
ntpdc -c sysinfo target.com

# Get peer stats
ntpdc -c peers target.com

# Monitor queries
ntpdc -c monlist target.com
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use Nmap to detect NTP services and identify server capabilities.

```
nmap -sU -p 123 target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Query NTP servers to gather version and configuration information.

**Using ntpq**

```
# Using ntpq
ntpq -c version target.com
ntpq -c readvar target.com
```

**Using ntpdc**

```
# Using ntpdc
ntpdc -c sysinfo target.com
```

**Using nmap**

```
# Using nmap
nmap -sU -p 123 --script ntp-info target.com
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### System Information <a href="#system-information" id="system-information"></a>

NTP servers expose system details including processor type, operating system, and software versions through query responses.

```
# Get system information
ntpq -c sysinfo target.com

# Read variables
ntpq -c readvar target.com

# Output includes:
# - System time
# - Processor type
# - System name
# - NTP version
# - Stratum (distance from reference clock)

# Get peer information
ntpq -c peers target.com
ntpq -c associations target.com
```

#### Monlist Command (CVE-2013-5211) <a href="#monlist-command-cve-2013-5211" id="monlist-command-cve-2013-5211"></a>

The monlist command can expose up to 600 recent NTP client IP addresses and is also a major DDoS amplification vector.

**Using ntpdc**

```
# Get monitoring list
ntpdc -c monlist target.com

# Can reveal:
# - Internal IP addresses
# - Network topology
# - Connected clients
# - Traffic patterns
```

**Using nmap**

```
# Using Nmap
nmap -sU -p 123 --script ntp-monlist target.com
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### NTP Amplification (DDoS) <a href="#ntp-amplification-ddos" id="ntp-amplification-ddos"></a>

NTP can be abused for reflection/amplification attacks.

```
# Check if monlist is enabled (amplification factor: 556x)
nmap -sU -p 123 --script ntp-monlist target.com

# If monlist responds, server can be abused
# Small request -> Large response
# (Don't perform without authorization)
```

#### Mode 6/7 Query Exploitation <a href="#mode-67-query-exploitation" id="mode-67-query-exploitation"></a>

Mode 6 and 7 queries can reveal sensitive information.

```
# Mode 6 query (control messages)
# Can execute certain commands on vulnerable servers

# Mode 7 query (private/restricted)
# May reveal additional information

# Using ntpq with mode 6
ntpq -c "rv 0 processor,system,leap" target.com
```

#### Time Manipulation <a href="#time-manipulation" id="time-manipulation"></a>

Manipulating NTP can affect time-sensitive protocols.

```
# If you control an NTP server that target uses
# You can manipulate time

# Affects:
# - Kerberos tickets (time-based)
# - SSL/TLS certificates (expiry)
# - Log timestamps (forensics)
# - Scheduled tasks (cron)
# - Session timeouts

# Using ntpd config (if you compromise NTP server)
# Edit /etc/ntp.conf
# Add malicious time source
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Information Gathering <a href="#information-gathering" id="information-gathering"></a>

Extract comprehensive information from NTP servers for analysis.

```
# Extract all available information
ntpq -c readvar target.com > ntp_info.txt
ntpq -c sysinfo target.com >> ntp_info.txt
ntpq -c peers target.com >> ntp_info.txt

# Analyze for:
# - OS version hints
# - Internal IP addresses
# - Network architecture
# - Connected systems
```

#### Network Mapping <a href="#network-mapping" id="network-mapping"></a>

Use NTP information to map network topology and discover additional targets.

```
# Monlist reveals client IPs
ntpdc -c monlist target.com | awk '{print $1}' | sort -u > client_ips.txt

# Scan discovered IPs
nmap -sn -iL client_ips.txt

# Build network map from NTP associations
```

### NTP Packet Structure <a href="#ntp-packet-structure" id="ntp-packet-structure"></a>

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|LI | VN  |Mode |    Stratum    |     Poll      |   Precision   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

LI   : Leap Indicator
VN   : Version Number
Mode : Association Mode (0-7)
```

### Common NTP Commands <a href="#common-ntp-commands" id="common-ntp-commands"></a>

| Command   | Description    | Usage                         |
| --------- | -------------- | ----------------------------- |
| `ntpq`    | NTP query      | `ntpq -p target.com`          |
| `ntpdate` | Set/query time | `ntpdate -q target.com`       |
| `ntpdc`   | NTP control    | `ntpdc -c monlist target.com` |
| `sntp`    | Simple NTP     | `sntp target.com`             |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool       | Description            | Primary Use Case  |
| ---------- | ---------------------- | ----------------- |
| ntpq       | NTP query tool         | Server querying   |
| ntpdc      | NTP control            | Administration    |
| ntpdate    | Time sync              | Time querying     |
| Nmap       | Network scanner        | Service detection |
| Metasploit | Exploitation framework | Automated testing |

### Security Misconfigurations <a href="#security-misconfigurations" id="security-misconfigurations"></a>

* ❌ Monlist command enabled (amplification)
* ❌ Mode 6/7 queries allowed
* ❌ No access restrictions
* ❌ Exposed to internet
* ❌ Outdated NTP version
* ❌ No rate limiting
* ❌ Default configuration
* ❌ No authentication
* ❌ Verbose responses
* ❌ No monitoring

<br>
