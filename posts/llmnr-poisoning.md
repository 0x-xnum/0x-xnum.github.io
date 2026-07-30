# LLMNR Poisoning

**What is LLMNR Poisoning?**

LLMNR (Link-Local Multicast Name Resolution) is a protocol that allows computers on the same local network to resolve hostnames without needing a DNS server. When a computer's DNS query fails, it broadcasts an LLMNR request across the network, asking other devices to provide the required information. LLMNR, a successor to the older NetBIOS protocol, operates similarly to NetBIOS Name Service (NBT-NS), which is also used as a fallback for name resolution within local networks.

***

**How LLMNR Poisoning Works:**

**Attacker Prepares:** The attacker runs a tool like [Responder](https://github.com/SpiderLabs/Responder) to listen for LLMNR queries on the network:

```bash
sudo responder -I eth0 -dwP
```

**Event Trigger:** When a device broadcasts an LLMNR request (e.g., looking for a hostname), the attacker intercepts the request and responds maliciously, pretending to be the requested device.&#x20;

<figure><img src="/files/UmMF3V4ruXJriafuGmt8" alt=""><figcaption></figcaption></figure>

**Sensitive Data Captured:** The victim's system sends sensitive details to the attacker, including:

* The victim's IP address (e.g., `192.168.138.137`)
* Domain and username (e.g., `MARVEL\fcastle`)
* Password hash

**Hash Cracking:** The attacker can then take the captured password hash offline and attempt to crack it using a tool like Hashcat:

```bash
hashcat -m 5600 <hashfile.txt> rockyou.txt
```

Module `5600` is for NTLMv2 hashes. You can find other modules with:

```bash
hashcat --help | grep LLMNR
```

***

**How to Mitigate LLMNR Poisoning in Active Directory:**

**Disable LLMNR:** Navigate to: `Computer Configuration > Administrative Templates > Network > DNS Client`\
Set **Turn OFF Multicast Name Resolution** to disable LLMNR.

**Disable NBT-NS:** Go to: `Network Connections > Network Adapter Properties > TCP/IPv4 Properties > Advanced tab > WINS tab`\
Select **Disable NetBIOS over TCP/IP**.

**Verify Mitigation:** Run the following commands to confirm:

* For LLMNR (PowerShell):

  ```powershell
  (Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows NT\DNSClient" -name EnableMulticast).EnableMulticast
  ```

  A result of "`0"` confirms LLMNR is disabled.
* For NBT-NS (Command Prompt):

  ```powershell
  ewmic nicconfig get caption,index,TcpipNetbiosOptions
  ```

  A result of "`2"` confirms NBT-NS is disabled.
