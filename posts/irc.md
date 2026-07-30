# IRC

**`Default Port: 6667`**

**IRC (Internet Relay Chat)**, is a protocol and communication system that allows users to engage in real-time text-based conversations. In this article, we will examine the pentesting techniques for IRC.

### Connect <a href="#connect" id="connect"></a>

#### Connecting to an IRC Server <a href="#connecting-to-an-irc-server" id="connecting-to-an-irc-server"></a>

You can connect to an IRC server using various IRC clients. For example, on Linux, you can use the `irssi` client with the following command:

```
irssi -c <irc-server-ip>
```

This command allows you to connect to a specific IRC server.

#### Login with Nickname and Username <a href="#login-with-nickname-and-username" id="login-with-nickname-and-username"></a>

```
/nick <nickname>
/user <username> <hostname> <servername> :<realname>
```

```
/nick pentester
/user pentester localhost localhost :Pentester
```

### Recon <a href="#recon" id="recon"></a>

#### Identifying an IRC Server <a href="#identifying-an-irc-server" id="identifying-an-irc-server"></a>

You can use `Nmap` to check if an IRC service is running on a specific host:

```
nmap -p 6667 X.X.X.X
```

This command checks if there is a service running on port 6667 of the specified IP address, which is commonly used by IRC.

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

You can use `Netcat` or a similar tool to perform banner grabbing and retrieve information about the IRC service:

```
nc -nv X.X.X.X 6667
```

This command collects banner information from the service running on port 6667 of the given IP address.

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Listing Channels and Users <a href="#listing-channels-and-users" id="listing-channels-and-users"></a>

```
/list
```

This command lists all available channels on the server.

```
/names <channel>
```

This command lists all users in a specific channel.

#### Collecting User Information <a href="#collecting-user-information" id="collecting-user-information"></a>

```
/whois <nickname>
```

This command retrieves detailed information about a specific user, including their hostname, server, and real name.

#### Packet Analysis <a href="#packet-analysis" id="packet-analysis"></a>

You can use packet analysis tools like Wireshark to capture and analyze IRC traffic. This can help in understanding communication patterns and identifying potential weaknesses.

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Default Credentials and Vulnerabilities <a href="#default-credentials-and-vulnerabilities" id="default-credentials-and-vulnerabilities"></a>

Check for default credentials or weak authentication mechanisms. For example, try common nicknames and usernames like `admin` or `root`.

#### Brute Force Attacks <a href="#brute-force-attacks" id="brute-force-attacks"></a>

You can perform brute-force attacks to guess weak passwords using tools like `hydra`:

```
hydra -l <username> -P /path/to/passwords.txt <target_ip> irc -s 6667
```

This command attempts to brute-force the specified IRC server.

#### Exploiting Misconfigurations <a href="#exploiting-misconfigurations" id="exploiting-misconfigurations"></a>

Look for misconfigured IRC servers that allow actions without proper authentication, such as joining restricted channels or sending messages to all users.

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Privilege Escalation <a href="#privilege-escalation" id="privilege-escalation"></a>

After gaining access, attempt to escalate privileges to higher-level accounts or operators (ops) within channels. You can use commands like:

```
/mode <channel> +o <nickname>
```

This command attempts to give operator privileges to the specified user in the channel.

#### Data Analysis and Manipulation <a href="#data-analysis-and-manipulation" id="data-analysis-and-manipulation"></a>

Once you have access, analyze and manipulate data within the IRC server. For example, you can monitor private messages or alter channel topics.

#### Hijacking Sessions <a href="#hijacking-sessions" id="hijacking-sessions"></a>

Target and hijack active user sessions to capture session information and manipulate sessions to your advantage. For example, you could use a tool like `Ettercap` to perform a man-in-the-middle attack on IRC traffic.

#### Example of IRC Commands <a href="#example-of-irc-commands" id="example-of-irc-commands"></a>

| Command                            | Description                                          |
| ---------------------------------- | ---------------------------------------------------- |
| /list                              | List all available channels on the server.           |
| /names \<channel>\\                | List all users in a specific channel.                |
| /whois \<nickname>\\               | Retrieve detailed information about a specific user. |
| /mode \<channel>\ +o \<nickname>\\ | Give operator privileges to a user in a channel.     |

***

This structure provides a comprehensive overview of pentesting activities directed towards IRC services. Always ensure you operate within ethical and legal boundaries while conducting such tests and have the appropriate authorization.

<br>

1. * ```bash
     ```
