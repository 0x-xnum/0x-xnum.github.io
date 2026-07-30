# Enumeration

### Directory Bruteforcing

Directory bruteforcing is the act of guessing valid directories/files in the target web server. Sometimes it's possible to get some interesting pages (such as admin panel, backup/log files, env file, unauthenticated dashboard, etc) through directory bruteforcing. For performing directory bruteforcing, we need a list of interesting endpoints i.e. wordlist. We can use [assetnote](https://wordlists.assetnote.io/) or [trickest](https://github.com/trickest/wordlists/tree/main/technologies) wordlists.

We're going to use [ffuf](https://github.com/ffuf/ffuf) for performing directory bruteforce.

```bash
ffuf -c -w files.txt -u https://target.com/FUZZ
```

### Fetching URLs from Public Archives

Directory bruteforcing is pretty helpful to find out interesting endpoints in the target server. However, it's a part of active recon and causes lots of noise in the web server which can trigger the WAF and block us.

An alternate way of directory bruteforcing would be fetching URLs from public archives such as Wayback Machine, AlienVault's Open Threat Exchange, Common Crawl, etc. These archives crawl the web and store the endpoints which can be viewed later.

In this section, we'll talk about 3 different tools that will fetch URLs from public archives and give us the output.

#### i. WayBackURLs

Waybackurls is a command-line tool for fetching URLs from the Wayback Machine (archive of websites that contains over 858 billion web pages). Sometimes, you can find some juicy information from waybackurls such as passwords, PII data, API keys, etc. We can either use the command-line tool <https://github.com/tomnomnom/waybackurls> or directly browse the URL: `http://web.archive.org/cdx/search/cdx?url=*.target.com/*&collapse=urlkey&output=text&fl=original`\
Since the number of URLs is quite big, we can search for the below keywords to extract sensitive information:

```bash
password
secret
token
access
pwd
api
.json
=http [For SSRF/open redirection]
=%2F [For SSRF/open redirection]
=/ [For SSRF/open redirection]
email=
@
ey [For JWT tokens]
.txt
aws
admin
.js
config
dashboard
oauth
sql
```

These are some common keywords you can search for to extract sensitive data from waybackurls. During your testing, if you encounter some endpoint that reveals juicy info about your account (for example invoices, billing info, etc), you can search for the endpoint in waybackurl to check if you're able to access other users' data. For instance, if `https://target.com/orders/1aq0b2chy9qar3w84chfr7ju5poa6` shows the order info of your order, you can search for the keyword `target.com/orders/` to check if other similar endpoints are logged in Wayback machine.

#### ii. Gau

[Gau](https://github.com/lc/gau) is a similar tool that fetches known URLs from AlienVault's Open Threat Exchange, the Wayback Machine, Common Crawl, and URLScan.

```bash
echo target.com | gau 
```

You can search for the same keywords after gathering the URLs. Also, there are multiple one-liners which will make your work easier. For example, if you want to get the JS files from gau and look for sensitive info inside it, you can use httpx and nuclei for this:

<pre class="language-bash"><code class="lang-bash"><strong>echo target.com | gau | grep ".js" | httpx -content-type | grep 'application/javascript' | awk '{print $1}' | nuclei -t nuclei-templates/http/exposures/
</strong></code></pre>

You can find more such one-liners [here](https://twitter.com/search?f=top\&q=one,%20liner%20%22gau%22\&src=typed_query)

#### iii. Waymore

As per the official repository, the difference between [Waymore](https://github.com/xnl-h4ck3r/waymore) and other tools is that it can also **download the archived responses** for URLs on wayback machine so that you can then search these for even more links, developer comments, extra parameters, etc.

```bash
python3 waymore.py -i bugbase.in -mode U
```

### Parameter Bruteforcing

Sometimes, it's possible to get some hidden parameters in some endpoint which may lead to further attacks such as cross-site scripting, SSRF, open redirection, etc. Generally, it's recommended to look for hidden parameters in pages where there are some kind of forms. For example, during one of my pen tests, I used [arjun](https://github.com/s0md3v/Arjun) for parameter bruteforcing on a signup page. It gave me a parameter `referral` which was vulnerable to XSS.

```bash
arjun -u https://bugbase.in/register -m GET,POST
```

Additionally, [ffuf](https://github.com/ffuf/ffuf#get-parameter-fuzzing) or [param-miner](https://github.com/PortSwigger/param-miner) can also be used for parameter bruteforcing.

### Dorking

Dorking involves using advanced search queries to find sensitive or hidden information on web applications. In this section, we'll discuss about Google Dorking and GitHub Dorking.

#### i. Google Dorking

Google Dorking is the art of using advanced search queries in Google to search for specific keywords, file types, or parameters. Sometimes, developers expose sensitive endpoints (admin panel, log files, etc) to the internet which are later crawled and indexed by Google. We can use advanced search queries from [GHDB](https://www.exploit-db.com/google-hacking-database) or [Bug Bounty Helper](https://dorks.faisalahmed.me/) to find such sensitive pages.

#### ii. GitHub Recon

Many times, developers of the company accidentally push sensitive information into the GitHub repository. We can try to find out this information using GitHub dorks. Here are some keywords that we can search for:

```bash
PASSWORD
PWD
KEY
API
TOKEN
ACCESS_TOKEN
SECRETKEY
CLIENT-SECRET
CLIENT_SECRET
SECRET
@target.com
DEV
PROD
JENKINS
CONFIG
SSH
FTP
MYSQL
ADMIN
AWS
DASHBOARD
BUCKET
ST NO
CVV
GITHUB_TOKEN
=http
OTP
OAUTH
AUTHORIZATION
LDAP
INTERNAL
language:sql
language:json
language:txt
```

All we need to do is search for these keywords in GitHub after `"target.com"`. For example, to look for passwords, we can search `"target.com" password` or `"target.com" pwd` in GitHub. We can also use [GitDorker](https://github.com/obheda12/GitDorker) to automate the entire process.

P.S: Make sure to verify the credentials found from GitHub before reporting.

### Aquatone

[Aquatone](https://github.com/michenriksen/aquatone) is a tool for the visual inspection of websites across a large number of hosts and is convenient for quickly gaining an overview of the HTTP-based attack surface.\
Let's say, you have a list of 1000 subdomains. Of course, you can't go through each of them one by one. Here comes Aquatone handy. It will take the list of subdomains, scan for popular ports that are often used by HTTP services, and take a screenshot of the resolved webpage. Additionally, it will generate a report that will allow us to view the targets in a list or in a graph view. We can look at which pages are identical and which aren’t. We can explore the headers returned for each endpoint and much more stuff.

```bash
cat http.txt | aquatone
```

### Nuclei

Finally, we're going to use [nuclei](https://github.com/projectdiscovery/nuclei) for scanning vulnerabilities on all the subdomains.

```bash
1nuclei -l http.txt -t nuclei-templates/ -severity critical,high,medium,low
```
