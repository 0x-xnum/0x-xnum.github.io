# Incorrect Default Permissions

### Description <a href="#id-486a" id="id-486a"></a>

This weakness describes a case where software sets unintended permissions to directories, files or other objects during the installation process. As a result, a malicious user might be able to bypass intended security restrictions.

Most modern operating systems support access control lists (ACL) that are used to distinguish access rights for different users and groups. In modern operating systems a principal (**e.g.** process or threat acting on behalf of a user) acts upon objects.

Access to these objects (**e.g.** files, directories, registry keys, etc.) is crucial for security mechanisms implemented in different operating systems and can influence system behaviour depending on permissions imposed upon key components of the operating system.

### **1.1** Linux- / UNIX-based systems

#### **1. Permissions in Linux/UNIX**

* Every file and directory has **three types of users** who can access it:
  * **User (Owner)** – The person who owns the file.
  * **Group** – A group of users who share access.
  * **Others** – Everyone else on the system.
* Each user type has **three types of permissions**:
  * **Read (r)** – Can view the file’s content.
  * **Write (w)** – Can modify or delete the file.
  * **Execute (x)** – Can run the file (if it’s a script or program).

#### **2. Changing File Ownership & Groups**

* **`chown`** – Changes the owner of a file.

  ```bash
  sudo chown user file.txt  # Changes owner to 'user'
  ```
* **`chgrp`** – Changes the group of a file.

  ```bash
  sudo chgrp group file.txt  # Changes group to 'group'
  ```

***

#### **3. Special Permissions: Setuid & Setgid**

* These **special bits** allow programs to run with the permissions of their owner or group, even if run by another user.
  * **Setuid (Set User ID)** – Runs a program as the file’s **owner**.
  * **Setgid (Set Group ID)** – Runs a program as the file’s **group**.
* Example: The `ping` command needs admin (root) privileges to send network packets. Since it is owned by **root**, the **setuid bit** allows normal users to run it with root permissions.

  ```bash
  ls -l /bin/ping
  -rwsr-xr-x 1 root root 64424 Jan  1 12:34 /bin/ping
  ```

  * The **`s`** in `rws` indicates **setuid** is enabled.

  **Attack Scenario (Setuid Binary Modification)**

  * If an attacker **gains write access** to a Setuid binary (e.g., `/bin/ping`), they can replace it with a malicious script that runs **as root**.
  * If `/bin/ping` were writable (`-rwsrwxrwx`), an attacker could do:

    ```bash
    echo '#!/bin/bash' > /bin/ping
    echo 'whoami' >> /bin/ping
    chmod +x /bin/ping
    ```

    * Now, running `ping` would execute **arbitrary commands as root**.

### **1.2** Windows-based systems

* Before **Windows NT**, only simple file attributes (like read-only) controlled access.
* **Modern Windows versions (NT and later) use ACLs**, which provide more detailed control over who can access what.
* Permissions are managed through:
  * **Graphical Interface (GUI)** – Right-click a file/folder → **Properties** → **Security Tab**.
  * **Command Line (`icacls`)** – Used to view and modify permissions.

<figure><img src="https://miro.medium.com/v2/resize:fit:367/0*vcItjon1Zh6M8z9x.png" alt="" height="517" width="367"><figcaption></figcaption></figure>

To check who has access to the `C:\` drive, use:

```sh
icacls C:
```

Example output:

```sh
C:\Users\Administrator>icacls C:
C: PC01\Administrator:(F) 
NT AUTHORITY\SYSTEM:(F)
BUILTIN\Administrators:(F)
PC01\Administrator:(OI)(CI)(IO)(F)
NT AUTHORITY\SYSTEM:(OI)(CI)(IO)(F)
BUILTIN\Administrators:(OI)(CI)(IO)(F)Successfully processed 1 files; Failed processing 0 files 
```

* **(F)** → Full control
* **(OI)** → Object Inherit (applies to files inside the folder)
* **(CI)** → Container Inherit (applies to subfolders)
* **(IO)** → Inherit Only (doesn’t apply to the folder itself)

#### **Registry Permissions & Security**

* The **Windows Registry** stores important system settings.
* Permissions for registry keys are managed using **`regedit.exe`** (Registry Editor).
* **Weak registry permissions** can let attackers modify system settings, install malware, or escalate privileges.

📌 **Example Risk:**\
If the registry key controlling a critical system process has **"Everyone: Full Control"**, any user (even without admin rights) could change it and disrupt the system.

### 2. Potential impact <a href="#b725" id="b725"></a>

This weakness is primarily **locally exploitable**, meaning an attacker usually needs some level of access to the system before they can take advantage of it. However, once they have that access, the consequences can be severe. Incorrect permissions on files and applications can lead to **unauthorized access to sensitive data, data tampering, and even full system compromise**.

### 3. Attack patterns <a href="#id-43e6" id="id-43e6"></a>

The following CAPEC patterns correspond to this weakness:

> **❏** CAPEC-1: [Accessing Functionality Not Properly Constrained by ACLs](http://capec.mitre.org/data/definitions/1.html)\
> **❏** CAPEC-19: [Embedding Scripts within Scripts](http://capec.mitre.org/data/definitions/19.html)\
> **❏** CAPEC-81: [Web Logs Tampering](http://capec.mitre.org/data/definitions/81.html)\
> **❏** CAPEC-127: [Directory Indexing](http://capec.mitre.org/data/definitions/127.html)\
> **❏** CAPEC-169: [Footprinting](http://capec.mitre.org/data/definitions/169.html)

Incorrect permissions vulnerability is described in WASC Threat Classification as a weakness under\
**❏** WASC-17:(Improper Filesystem Permissions).

### 4. Severity and CVSS Scoring <a href="#id-67be" id="id-67be"></a>

This real-world example demonstrates **incorrect default permissions** in the **"btinstall" installation script**, which sets **world-writable** permissions on all files inside `/frameworkgui/`.

* Logged in as an **unprivileged guest user**.
* Ran `ls -la` to check file permissions.
* Found that **all files** in `/frameworkgui/` are **world-writable (`-rwxrwxrwx`) :**

<figure><img src="https://miro.medium.com/v2/resize:fit:525/0*wR-IiC3_WKMzJU59.png" alt="" height="229" width="525"><figcaption></figcaption></figure>

Now we will try to read the “config” file and then modify the agentpoll.pl script:

<figure><img src="https://miro.medium.com/v2/resize:fit:471/0*6R7GhHx_UaBaWWfw.png" alt="" height="85" width="471"><figcaption></figcaption></figure>

As a result of this vulnerability, any local user has full access to files within the “/frameworkgui/” directory.

> ***Credits**:* [*https://www.immuniweb.com/*](https://www.immuniweb.com/)
