# Mimikatz

**What is Mimikatz?**

**Mimikatz** is a post-exploitation tool that can:

* **Dump credentials** stored in memory.
* Generate and manipulate Kerberos tickets.
* Perform Just a few attacks: Credential Dumping, Pass-the-Hash, Over-Pass-the-Hash, Pass- the-Ticket, Silver Ticket, and Golden Ticket

**Download Mimikatz**:

* Visit the official GitHub repository: [Mimikatz Releases](https://github.com/gentilkiwi/mimikatz/releases/tag/2.2.0-20220919).
* Download the `mimikatz_trunk.zip` file.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*GpxYb4kJ4hwie_SHSV4vtA.png" alt="" height="297" width="700"><figcaption></figcaption></figure>

1. **Extract the Files**:
   * Unzip the file to create a directory named **mimikatz\_trunk**.
2. **Navigate to the x64 Folder**:
   * For 64-bit Windows systems, use the files inside the `x64` folder.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*9IjQLY93EHs0Oew1NlZcmw.png" alt="" height="201" width="700"><figcaption><p>Folder x64</p></figcaption></figure>

This preparation ensures you have the necessary binaries to execute Mimikatz commands effectively.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*ijkvf3R0F_dNq1AOj8xhYA.png" alt="" height="181" width="700"><figcaption><p>Files Inside the x64 Folder</p></figcaption></figure>

### **Running Mimikatz**

To begin credential dumping

* **Open a Command Prompt (cmd):** Ensure you run it as an administrator to avoid permission issues.
* **Navigate to the Mimikatz directory:** Use the `cd` command to move to the directory where Mimikatz is stored.

```bash
cd c:\Users\pparker\Downloads\mimikatz_trunk\x64
```

* **Run Mimikatz:** Execute the Mimikatz binary using the command:

```bash
mimikatz.exe
```

This launches the interactive Mimikatz interface.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*5lNcXSH3_VAmYHLasos7Iw.png" alt="" height="189" width="700"><figcaption><p>Interactive Mimikatz Interface</p></figcaption></figure>

**3. Elevating Privileges**

Mimikatz requires **debug privileges** to access sensitive memory areas. To enable these privileges:

* **Run the privilege command:**

```bash
privilege::debug
```

This command enables debug privileges, allowing Mimikatz to interact with system processes and extract credentials. If successful, it returns `Privilege '20' OK`.

<figure><img src="https://miro.medium.com/v2/resize:fit:459/1*vjN4meyKSdmLJvO-fQ29Gw.png" alt="" height="67" width="459"><figcaption></figcaption></figure>

**4. Dumping Credentials**

To retrieve credentials stored in memory, run the foolowaing command :

```bash
sekurlsa::logonpasswords
```

* **Username, domain, and plaintext password:** Identifies the user whose credentials and plaintext password are being extracted.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*POrUfIHzB1ubYn0-ERbyng.png" alt="" height="525" width="700"><figcaption><p>Dumping Credentials — Username, Domain, Password</p></figcaption></figure>

* **Hash password:** Also get the hashed version of the password used by the account.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*cluu-eilsezpsc3mNJRSIw.png" alt="" height="250" width="700"><figcaption><p>Hash Password</p></figcaption></figure>

* **NTLM hash:** A hashed version of the password used for authentication.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*xBEUz2sWhRt_MY8SMcUD1Q.png" alt="" height="339" width="700"><figcaption><p>NTLM Hash</p></figcaption></figure>

**5. Validating the Results**

After running the above commands, carefully review the output for sensitive information. Mimikatz typically presents data in an easy-to-read format, highlighting the credentials associated with each logged-on user session.

**6. Optional Commands for Enhanced Analysis**

Mimikatz offers additional commands to refine the extraction process:

* **Exploring available commands:**

```bash
sekurlsa::
```

This displays a list of available subcommands under the `sekurlsa` module, helping tailor your credential dumping process to specific needs.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*7VSGV505GzBwv3uz3JHxJA.png" alt="" height="364" width="700"><figcaption></figcaption></figure>

### Outcomes of Credential Dumping with Mimikatz <a href="#c2c1" id="c2c1"></a>

After running Mimikatz, you will have access to:

1. **Plaintext Passwords**: Direct passwords stored in memory.
2. **NTLM Hashes**: Used for **Pass-the-Hash** attacks.
3. **Kerberos Tickets**: Useful for ticket-based attacks like **Golden Ticket** and **Pass-the-Ticket**.

These credentials can be used to:

* Access other systems.
* Bypass authentication mechanisms.
* Perform lateral movement in a network.
