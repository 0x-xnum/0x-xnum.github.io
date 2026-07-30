# Printer Hacking

**What is an MFP and MFP Hacking?**

**Multi-Function Peripherals (MFPs)** are devices that offer several functions like printing, copying, scanning, and faxing. Despite their widespread use in offices. However, a successful breach of an MFP can lead to significant security findings, including:

* **Credential Disclosure**
* **File System Access**
* **Memory Access**

MFPs, often found in corporate settings, come with network ports, USB drives, and iPad-like control panels. They integrate with corporate networks for functions like scanning to email or printing from network shares. These integrations expose valuable data and create potential vulnerabilities.

**What Information is at Risk with an MFP?**

MFPs are often linked to **LDAP (Lightweight Directory Access Protocol)**, **SMTP (Simple Mail Transfer Protocol)**, and **network shares**. These integrations provide:

* **Access control** for print, copy, and scan functions.
* **Email lookup** when using scan-to-email features.
* **Access to user home directories** stored on the network.

**MFP-LDAP Integration**

* **LDAP** helps MFPs look up email addresses or allow access to files on the network.
* The MFP needs to have login credentials to access LDAP.
* If someone can access the MFP settings, they can change the LDAP server to a malicious server and capture usernames and passwords.

**The Pass-Back Attack**

This is a method where an attacker changes the MFP settings in the **Embedded Web Service (EWS)** interface. By modifying the LDAP server configuration to point to a malicious LDAP server. instead of logging into the real system, the MFP sends the login info to the attacker.

**Steps to Perform the Attack**:

**Access the EWS**: Most MFPs come with default administrative credentials. You can find these in the **Administrator Guide** or use common defaults (e.g., `admin`/`blank` for HP printers, `admin`/`admin` for Ricoh).

* <https://github.com/RUB-NDS/PRET>&#x20;
* <https://github.com/percx/Praeda>&#x20;
* [http://www.hacking-printers.net/wiki/index.php/Printer\_Security\_Testing\_Cheat\_Sheet](https://www.hacking-printers.net/wiki/index.php/Printer_Security_Testing_Cheat_Sheet)&#x20;

**Change the LDAP Server Address**: Once authenticated, modify the LDAP server settings in the MFP configuration to point to a malicious server (e.g., your own machine running a Netcat listener on port 389).

<figure><img src="https://cdn.prod.website-files.com/601959b8cde20c101809c86a/66e850669c9a07d95e80099f_63702b148fa46855262734d6_HP-Color-LaserJet-MFP-Screencap.webp" alt=""><figcaption></figcaption></figure>

**Capture Credentials**: The next time a user authenticates through the MFP (e.g., for scan-to-email), their LDAP credentials will be sent to the malicious server.

<figure><img src="https://cdn.prod.website-files.com/601959b8cde20c101809c86a/66e850669c9a07d95e8009a5_63702b8ba81428188b9600ad_Command-line-LDAP-server-under-our-control.webp" alt=""><figcaption></figcaption></figure>

#### Attacking Other MFP Configurations

**SMTP Server Attacks**

MFPs (Multifunction Printers) are commonly configured to send emails directly from the device, such as [scanning documents to email](#user-content-fn-1)[^1] or sending alerts. To perform this function, the MFP stores **SMTP credentials** (username and password) for authentication with an SMTP server.

If an attacker gains access to the MFP's settings, they can modify the **SMTP server configuration** to point to a malicious SMTP server they control. As a result, when the MFP attempts to send emails (such as scanning documents), the SMTP credentials used by the printer will be sent to the attacker’s server, allowing the attacker to capture these credentials.

<figure><img src="/files/r8ith5IqW9EA1OciHedt" alt=""><figcaption></figcaption></figure>

#### **Windows Sign-In Attack on MFPs**

Many MFPs allow users to authenticate using **Windows sign-in** via an integrated domain controller (e.g., Active Directory). This method can be used to restrict access to the MFP based on the user's Windows credentials.

If an attacker is able to modify the **domain controller settings** within the MFP configuration, they can point the device to an attacker-controlled domain server. Once this change is made, the next time a legitimate user tries to log in to the MFP, their credentials (username and password) will be sent to the attacker's domain controller instead of the legitimate one.

<figure><img src="/files/bOdCH8KO5GWCJl6qzy83" alt=""><figcaption></figcaption></figure>

resources&#x20;

* **Printer Exploitation Toolkit (PRET)**: [GitHub Repository](https://github.com/RUB-NDS/PRET)
* **Praeda**: [GitHub Repository](https://github.com/percx/Praeda)
* **Printer Security Testing Cheat Sheet**: [Hacking-Printers Wiki](http://www.hacking-printers.net/wiki/index.php/Printer_Security_Testing_Cheat_Sheet)

[^1]: The **scan-to-email** feature on a printer uses **SMTP** to send scanned documents directly to an email address. You place the document on the printer, select the email option, and it sends the scanned file as an attachment to the chosen email address, all without needing a computer.
