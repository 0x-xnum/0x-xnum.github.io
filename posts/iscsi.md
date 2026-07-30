# ISCSI

**`Default Port: 3260`**

**ISCSI (Internet Small Computer System Interface)** is a protocol used for establishing and managing connections between storage devices over an IP network. It enables storage devices to be shared and accessed remotely, providing block-level access to storage resources.

ISCSI is commonly used in data centers and enterprise environments for storage area networks (SANs) and virtualization deployments.

### Connect <a href="#connect" id="connect"></a>

#### Connect Using ISCSI Initiator <a href="#connect-using-iscsi-initiator" id="connect-using-iscsi-initiator"></a>

```
iscsiadm --mode discoverydb --type sendtargets --portal <target-ip>:<target-port> --discover
```

#### Connect Using ISCSI Target Portal <a href="#connect-using-iscsi-target-portal" id="connect-using-iscsi-target-portal"></a>

You can use tools like ISCSI Initiator or open-iscsi to connect to an ISCSI target portal.

### Recon <a href="#recon" id="recon"></a>

#### Identifying an ISCSI Target <a href="#identifying-an-iscsi-target" id="identifying-an-iscsi-target"></a>

You can use `Nmap` to check if there's an ISCSI target on a target host like this:

```
nmap -p 3260 X.X.X.X
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

```
nc -nv X.X.X.X 3260
```

#### Enumeration <a href="#enumeration" id="enumeration"></a>

#### ISCSI Target Information <a href="#iscsi-target-information" id="iscsi-target-information"></a>

Connect to the ISCSI target and gather information about available LUNs (Logical Unit Numbers), target IQNs (ISCSI Qualified Names), and other configuration details using ISCSI commands.

#### ISCSI Client Tools <a href="#iscsi-client-tools" id="iscsi-client-tools"></a>

Tools like ISCSI Initiator, open-iscsi, and tgtadm can be used for interacting with ISCSI targets and performing enumeration tasks.

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Default Credentials <a href="#default-credentials" id="default-credentials"></a>

Check for default credentials or weak authentication configurations in ISCSI targets, such as targets using default IQNs or no authentication.

#### Unauthorized Access <a href="#unauthorized-access" id="unauthorized-access"></a>

Search for open ISCSI targets that allow unrestricted access, which may be exposed to unauthorized access from the internet.

#### LUN Manipulation <a href="#lun-manipulation" id="lun-manipulation"></a>

Exploit vulnerabilities in ISCSI target configurations to access or manipulate LUNs, potentially gaining unauthorized access to sensitive data.

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Common ISCSI Commands <a href="#common-iscsi-commands" id="common-iscsi-commands"></a>

| Command                  | Description                                 |
| ------------------------ | ------------------------------------------- |
| iscsiadm -m session      | List active ISCSI sessions                  |
| iscsiadm -m node -l      | Log in to an ISCSI target                   |
| iscsiadm -m node -u      | Log out of an ISCSI target                  |
| iscsiadm -m node -o show | Display detailed information about a target |
| iscsiadm -m discovery    | Discover available ISCSI targets            |

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

Extract sensitive data by accessing and manipulating LUNs on the ISCSI target.

#### Ransomware Attacks <a href="#ransomware-attacks" id="ransomware-attacks"></a>

Encrypt data on the ISCSI target and demand a ransom for decryption, exploiting vulnerabilities in ISCSI target configurations.

#### Denial-of-Service (DoS) Attacks <a href="#denial-of-service-dos-attacks" id="denial-of-service-dos-attacks"></a>

ISCSI targets may be susceptible to DoS attacks, disrupting storage access and causing service downtime.

<br>
