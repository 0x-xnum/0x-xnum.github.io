# TACACS

**`Default Port: 49`**

**TACACS** (Terminal Access Controller Access-Control System) is a protocol used to manage access to network devices and servers. TACACS+ is a more secure and advanced version that provides authentication, authorization, and accounting.

### Connect <a href="#connect" id="connect"></a>

#### Connect Using TACACS Client <a href="#connect-using-tacacs-client" id="connect-using-tacacs-client"></a>

You can connect to a TACACS server using a TACACS client. For example, on Linux systems, you can use the TACACS client with the following command:

```
tacacs_client -u <username> -p <password> -a <TACACS-server-ip>
```

This command allows you to connect to a specific TACACS server with a given username and password.

### Recon <a href="#recon" id="recon"></a>

#### Identifying a TACACS Server <a href="#identifying-a-tacacs-server" id="identifying-a-tacacs-server"></a>

You can use `Nmap` to check if there is a TACACS service running on a specific host:

```
nmap -p 49 X.X.X.X
```

This command checks if there is a service running on port 49 of the specified IP address.

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

You can use `Netcat` or a similar tool to perform banner grabbing and retrieve information about the TACACS service:

```
nc -nv X.X.X.X 49
```

This command collects banner information from the service running on port 49 of the given IP address.

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### TACACS+ Packet Analysis <a href="#tacacs-packet-analysis" id="tacacs-packet-analysis"></a>

You can use packet analysis tools like Wireshark to capture and analyze TACACS+ packets. TACACS+ traffic operates over TCP port 49.

#### Enumerating TACACS Users <a href="#enumerating-tacacs-users" id="enumerating-tacacs-users"></a>

There may be methods to enumerate TACACS users and groups, although it typically requires proper authorization. If you gain access, you can attempt to retrieve user and group details.

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Default Credentials <a href="#default-credentials" id="default-credentials"></a>

Check for default credentials or weak authentication configurations. For example, try common password combinations like `admin:admin123` or `admin:<blank>`.

#### Brute Force Attacks <a href="#brute-force-attacks" id="brute-force-attacks"></a>

You can perform brute-force attacks to guess weak passwords using tools like `hydra`:

```
hydra -l admin -P /path/to/passwords.txt <target_ip> -s 49 tacacs
```

#### Unauthorized Access <a href="#unauthorized-access" id="unauthorized-access"></a>

Exploit misconfigured access controls or weak authentication mechanisms to gain unauthorized access.

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Privilege Escalation <a href="#privilege-escalation" id="privilege-escalation"></a>

After gaining unauthorized access, try various methods to escalate privileges to higher-level accounts.

#### Data Analysis and Manipulation <a href="#data-analysis-and-manipulation" id="data-analysis-and-manipulation"></a>

Once you have access to sensitive data on network devices, you can analyze or manipulate this data.

#### User Session Hijacking <a href="#user-session-hijacking" id="user-session-hijacking"></a>

Target and hijack active user sessions to capture session information and manipulate sessions to your advantage.

#### Example of TACACS CLI Commands (Hypothetical) <a href="#example-of-tacacs-cli-commands-hypothetical" id="example-of-tacacs-cli-commands-hypothetical"></a>

| Command                                   | Description                                         |
| ----------------------------------------- | --------------------------------------------------- |
| tacacs\_client -l \<entity>\\             | List entities like users or sessions.               |
| tacacs\_client -i \<entity>\ -u \<user>\\ | Display detailed information about a specific user. |
| tacacs\_client -a \<entity>\ -n \<name>\\ | Add a new user/entity to TACACS.                    |
| tacacs\_client -d \<entity>\ -u \<user>\\ | Delete an existing user/entity from TACACS.         |

These commands are typically used for managing TACACS, and you can use them after gaining unauthorized access.

<br>
