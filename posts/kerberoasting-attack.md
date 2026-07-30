# Kerberoasting Attack

#### **How Kerberoasting Works**

* In Active Directory (AD), service accounts are used to run specific services or applications, such as web servers or database systems.
* These services are associated with a Service Principal Name (SPN), which uniquely identifies them.
* Any authenticated AD user can request a Ticket Granting Service (TGS) ticket for any SPN in the domain.
* The TGS is encrypted using the service account's NTLM hash (derived from its password).
* Now the attacker requests a TGS for a service with a known SPN.
* The TGS is then extracted from memory using tools like **Rubeus** or **Impacket**.
* The attacker uses tools like **Hashcat** or **John the Ripper** to brute force or dictionary attack the TGS offline.
* If successful, this reveals the service account's plaintext password.

## How to do it?  <a href="#id-9b08" id="id-9b08"></a>

#### **1. Identify Service Principal Names (SPNs)**

With valid admin or standard user credentials, you can use `GetUserSPNs.py` (from [Impacket](https://github.com/fortra/impacket)) to enumerate SPNs and request TGS tickets for those services.

```bash
GetUserSPNs.py -request -dc-ip <DC_IP> <domain\user>
```

* **`-request`**: Requests TGS tickets for the identified SPNs.
* **`<DC_IP>`**: IP address of the Domain Controller.
* **`<domain\user>`**: The username and domain to authenticate with.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*3COZ_HDecSZ8UH9hq75kLg.png" alt=""><figcaption><p>TGS ticket dump from Attacker’s PC</p></figcaption></figure>

* When you request TGS tickets, the tool will dump the tickets in a format that can be used for offline cracking.
* The extracted **TGS hashes** are encrypted with the NTLM hash of the associated service account's password.

#### **3. Crack the TGS Hash**

* Use **Hashcat**  to brute-force the NTLM hash of the service account password offline.
* The hash type for TGS tickets is **13100** in Hashcat.

```bash
hashcat -m 13100 <hash_file> <rockyou_wordlist>
```

**Important note:** If any of the above test gives a negative result, keep an eye on your Wireshark traffic. Mostly setting up static DHCP or DNS or Gateway IP address solves such issues. This is a very small thing to underestimate which will affect the pentest in a peculiar way.\\

## **Mitigations:** <a href="#f335" id="f335"></a>

* If possible use [group managed service accounts](https://technet.microsoft.com/en-us/library/hh831782%28v=ws.11%29.aspx?f=255\&MSPPError=-2147217396) which have random, complex passwords (>100 characters) and are managed automatically by Active Directory
* Ensure all service accounts (user accounts with Service Principal Names) have long, complex passwords greater than 25 characters, preferably 30 or more. This makes cracking these password far more difficult.
* Service Accounts with elevated AD permissions should be the focus on ensuring they have long, complex passwords.
* Ensure all Service Account passwords are changed regularly

## **Shout outs:** <a href="#id-874a" id="id-874a"></a>

* [Cracking Kerberos TGS Tickets Using Kerberoast — Exploiting Kerberos to Compromise the Active Directory Domain](https://adsecurity.org/?p=2293)
* [Attack Methods for Gaining Domain Admin Rights in Active Directory](https://adsecurity.org/?p=2362)
* [Sneaky Persistence Active Directory Trick #18: Dropping SPNs on Admin Accounts for Later Kerberoasting](https://adsecurity.org/?p=3466)
* [Targeted Kerberoasting (Harmj0y)](http://www.harmj0y.net/blog/activedirectory/targeted-kerberoasting/)
* Tim Medin’s DerbyCon “Attacking Microsoft Kerberos Kicking the Guard Dog of Hades” presentation in 2014 ([slides](https://files.sans.org/summit/hackfest2014/PDFs/Kicking%20the%20Guard%20Dog%20of%20Hades%20-%20Attacking%20Microsoft%20Kerberos%20%20-%20Tim%20Medin%281%29.pdf) & [video](https://www.youtube.com/watch?v=PUyhlN-E5MU\&feature=youtu.be)).
