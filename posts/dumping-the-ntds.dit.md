# Dumping the NTDS.dit

What is NTDS.dit :&#x20;

it is a databserd that used to store thall the user ingormaiton

* **User Information**
* **Group Information**
* **Security Descriptors**
* **and oh yeah, Password Hashes**

#### **Step-by-Step Guide to Extract NTDS.DIT Data**

**Step 1: Using secretsdump.py to Extract NTDS.dit Data**

Use the `secretsdump.py` tool from the Impacket suite to dump the NTDS.dit database remotely.

```bash
secretsdump.py MARVEL.local/hawkeye:'Password1@'@192.168.92.129
```

* `MARVEL.local`: The domain name of the target AD environment.
* `hawkeye`: The username of the domain user we’re authenticating as.
* `Password1@`: The password for the user account.
* `192.168.92.129`: The IP address of the domain controller.

This command retrieves:

* Usernames.
* NTLM password hashes.
* Data from the NTDS.dit file.

This depicts how an attacker with valid credentials can extract sensitive information remotely.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*yoXSRg0fdKhoe3Dy6VgQmQ.png" alt="" height="293" width="700"><figcaption><p>Using secretsdump.py to Extract NTDS.dit Data</p></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*b5mpHG1cPubaoIqrWFh5Ug.png" alt="" height="330" width="700"><figcaption><p>Using secretsdump.py to Extract NTDS.dit Data</p></figcaption></figure>

\
Step 2: Dumping Only NTLM Hashes

If you are only interested in NTLM password hashes, you can use the `-just-dc-ntlm` flag to limit the output.

```bash
secretsdump.py MARVEL.local/hawkeye:'Password1@'@192.168.92.129 -just-dc-ntlm
```

We obtain the NTLM hashes of all accounts in the domain. These hashes can now be used for offline password cracking.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*zWPXzyKpL41htRGfK63htw.png" alt="" height="232" width="700"><figcaption><p>Dumping NTLM Hashes</p></figcaption></figure>

#### Step 3: Saving the Extracted Hashes <a href="#d08d" id="d08d"></a>

To organize the extracted hashes for cracking, we save them in a text file.

```bash
mousepad ntds.txt
```

<figure><img src="https://miro.medium.com/v2/resize:fit:414/1*Gp_1D_ZOGmMNOjyQ4jy7zQ.png" alt="" height="73" width="414"><figcaption></figcaption></figure>

The NTLM hashes are now stored in a file named ntds.txt for further processing.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*oboe06nusfeEdcG7Cvmcbg.png" alt="" height="251" width="700"><figcaption><p>Mousepad — ntds.txt</p></figcaption></figure>

#### Step 4: Cracking NTLM Hashes with Hashcat <a href="#a929" id="a929"></a>

```
hashcat -m 1000 ntds.txt rockyou.txt
```

* `-m 1000`: Specifies the hash type (1000 = NTLM).

Hashcat compares each word in the wordlist against the NTLM hashes to find a match, revealing the plaintext passwords.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*lzm9m9JAhTXGOcgJLH1wzA.png" alt="" height="280" width="700"><figcaption><p>Cracking NTLM Hashes</p></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*LhAJpNWKZWw1esoONO4CSw.png" alt="" height="429" width="700"><figcaption><p>Cracking NTLM Hashes</p></figcaption></figure>

We retrieve the plaintext passwords for user accounts whose hashes match entries in the wordlist.

#### Step 5: Viewing Cracked Passwords <a href="#id-8b64" id="id-8b64"></a>

After cracking the hashes, we can list all cracked passwords to analyze them further.

```
hashcat -m 1000 ntds.txt rockyou.txt --show
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*nyYKFLlxwYb0MHMBvQwAgg.png" alt="" height="171" width="700"><figcaption></figcaption></figure>

The cracked passwords are displayed in a clear format, showing the hash, and the corresponding plaintext password.

### Step 6: Organizing Cracked Credentials <a href="#e993" id="e993"></a>

To simplify analysis and usage, we prepare a list of the cracked credentials.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*zgxI5rFkROotDoeqde6P4g.png" alt="" height="315" width="700"><figcaption></figcaption></figure>

This organized list makes it easier to identify which accounts have weak passwords and prioritize further exploitation.

### How Attackers Exploit This: <a href="#id-0d54" id="id-0d54"></a>

* Attackers can use the dumped credentials to authenticate as legitimate users, bypassing security controls.
* Cracked hashes enable privilege escalation, allowing attackers to target sensitive resources.

### Mitigations <a href="#id-6b11" id="id-6b11"></a>

1. **Strong Password Policies**: Enforce complex passwords that are resistant to dictionary attacks.
2. **Limit Account Privileges**: Use the principle of least privilege to minimize the impact of compromised accounts.
3. **Enable Logging and Monitoring**: Detect and respond to suspicious activity, such as unexpected NTDS.dit access.
4. **Implement Multi-Factor Authentication (MFA)**: Even with stolen credentials, MFA adds a layer of security.
