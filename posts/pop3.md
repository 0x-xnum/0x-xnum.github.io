# POP3

**`Default Ports: 110 (POP3), 995 (POP3S)`**

**Post Office Protocol version 3 (POP3)** is an email protocol used to retrieve emails from a remote server to a local client. Unlike IMAP, POP3 typically downloads emails to the client and deletes them from the server (though this can be configured). POP3 is simpler than IMAP but less feature-rich, primarily designed for offline email access.

### Connect <a href="#connect" id="connect"></a>

#### Using Telnet <a href="#using-telnet" id="using-telnet"></a>

```
# Connect to POP3 server
telnet target.com 110

# Basic POP3 conversation
USER username
PASS password
LIST
RETR 1
QUIT
```

#### Using openssl (POP3S) <a href="#using-openssl-pop3s" id="using-openssl-pop3s"></a>

```
# Connect with SSL
openssl s_client -connect target.com:995 -crlf -quiet

# POP3 commands
USER username
PASS password
LIST
QUIT
```

#### Using curl <a href="#using-curl" id="using-curl"></a>

```
# List emails
curl -u username:password pop3://target.com/

# Read specific email
curl -u username:password pop3://target.com/1

# POP3S
curl -u username:password pop3s://target.com/ --insecure
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use Nmap to detect POP3 mail servers and identify server capabilities.

```
nmap -p 110,995 target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Connect to POP3 servers to gather version and service information.

**Using netcat**

```
# Using netcat
nc target.com 110
```

**Using telnet**

```
# Using telnet
telnet target.com 110
```

**Using nmap**

```
# Using nmap
nmap -p 110 -sV target.com
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Capability Enumeration <a href="#capability-enumeration" id="capability-enumeration"></a>

POP3 servers advertise their supported features and extensions through the CAPA command.

```
# Get server capabilities
telnet target.com 110
CAPA

# Response shows:
# +OK Capability list follows
# USER
# PIPELINING
# TOP
# UIDL
# STLS
# .
```

#### Mailbox Enumeration <a href="#mailbox-enumeration" id="mailbox-enumeration"></a>

Explore mailbox contents and message information.

```
# After login
USER username
PASS password

# List messages
LIST

# Message count and size
STAT

# Get message UIDs
UIDL
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Brute Force <a href="#brute-force" id="brute-force"></a>

Brute forcing POP3 credentials can reveal weak email account passwords.

**Using Hydra**

```
# POP3 (plaintext)
hydra -l user@target.com -P passwords.txt pop3://target.com

# POP3S (SSL/TLS)
hydra -l user@target.com -P passwords.txt pop3s://target.com:995

# Multiple users
hydra -L users.txt -P passwords.txt pop3://target.com
```

**Using Nmap**

```
nmap -p 110 --script pop3-brute target.com
```

#### User Enumeration <a href="#user-enumeration" id="user-enumeration"></a>

POP3 doesn't have VRFY/EXPN like SMTP, but you can enumerate via login attempts.

```
# POP3 doesn't have VRFY/EXPN like SMTP
# But you can enumerate via login attempts

# Different error messages may reveal valid users
telnet target.com 110
USER admin
# +OK vs -ERR can indicate if user exists

# Timing attacks
# Valid users may take longer to respond
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Email Download <a href="#email-download" id="email-download"></a>

Download emails from compromised POP3 accounts for analysis.

**Automated Email Download**

```
# Download all emails with curl
for i in {1..100}; do
  curl -u username:password "pop3://target.com/$i" > email_$i.eml 2>/dev/null
done
```

**Manual Email Retrieval**

```
# Or using telnet
telnet target.com 110
USER username
PASS password
STAT  # Get message count
RETR 1  # Retrieve first email
RETR 2  # Second email
```

#### Credential Harvesting <a href="#credential-harvesting" id="credential-harvesting"></a>

Extract sensitive information from downloaded emails.

```
# Search downloaded emails for credentials
grep -r "password\|credential\|username" *.eml

# Extract URLs
grep -Eiorh 'https?://[^\s]+' *.eml

# Extract email addresses
grep -Eiorh '\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b' *.eml
```

### Common POP3 Commands <a href="#common-pop3-commands" id="common-pop3-commands"></a>

| Command | Description            | Usage           |
| ------- | ---------------------- | --------------- |
| `USER`  | Username               | `USER username` |
| `PASS`  | Password               | `PASS password` |
| `STAT`  | Mailbox stats          | `STAT`          |
| `LIST`  | List messages          | `LIST`          |
| `RETR`  | Retrieve message       | `RETR 1`        |
| `DELE`  | Mark for deletion      | `DELE 1`        |
| `NOOP`  | No operation           | `NOOP`          |
| `RSET`  | Reset                  | `RSET`          |
| `TOP`   | Message header + lines | `TOP 1 10`      |
| `UIDL`  | Unique IDs             | `UIDL`          |
| `QUIT`  | Close connection       | `QUIT`          |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool              | Description            | Primary Use Case  |
| ----------------- | ---------------------- | ----------------- |
| telnet            | Terminal client        | Manual testing    |
| openssl s\_client | SSL/TLS client         | POP3S connection  |
| curl              | Transfer tool          | Automated access  |
| Hydra             | Password cracker       | Brute force       |
| Nmap              | Network scanner        | Service detection |
| Metasploit        | Exploitation framework | Automated testing |

### Security Misconfigurations <a href="#security-misconfigurations" id="security-misconfigurations"></a>

* ❌ No encryption (port 110)
* ❌ Weak passwords
* ❌ No rate limiting
* ❌ Plaintext authentication
* ❌ No account lockout
* ❌ Outdated server software
* ❌ No TLS enforcement
* ❌ Information disclosure
