# Information Disclosure

### Lab 1: Error Messages <a href="#lab-1-error-messages" id="lab-1-error-messages"></a>

To solve this lab, find the version of a third-part framework used.

There's a `productId` parameter passed that contains an integer. If replaced by a string, it shows the error message:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-db2dfad3f2a0d7e4d3efaf3d1d3c8f0ae8cc20bf%252Fportswigger-information-writeups-image.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=1d3992de&#x26;sv=2" alt=""><figcaption></figcaption></figure>

`Apache Struts 2 2.3.31` solves the lab.

### Lab 2: Debug Page <a href="#lab-2-debug-page" id="lab-2-debug-page"></a>

To solve this lab, find the `SECRET_KEY` variable. This lab left a `phpinfo.php` file on the website:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-b7ecdecd2c045b3b47b6e25481ffac1758812ae1%252Fportswigger-information-writeups-image-2.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=c7a3f5a0&#x26;sv=2" alt=""><figcaption></figcaption></figure>

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-e08aa5c72b290859fb8830219bfbf9fe0ce49b04%252Fportswigger-information-writeups-image-1.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=32062a1b&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Just search for `SECRET_KEY` within the page above.

### Lab 3: Backup Files <a href="#lab-3-backup-files" id="lab-3-backup-files"></a>

To solve this lab, find the hard-coded database password.

I ran a `gobuster` scan on the site:

```bash
$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/common.txt -k -u https://0aa600a30372cc9e8148168900cb007a.web-security-academy.net -H 'Cookie: session=RmNy6J8CwjXBdsoYsPgqghJa4agjIKWc'
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://0aa600a30372cc9e8148168900cb007a.web-security-academy.net
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/analytics            (Status: 200) [Size: 0]
/backup               (Status: 200) [Size: 435]
```

The `/backup` directory contains this:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-bf7aa6bbed7767072a032d500db87bd7a2f948cd%252Fportswigger-information-writeups-image-3.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=4a8ea944&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Within this, there's some Java code for stuff, and it contains the password:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-f1d515380b94a7e04aab17cf94beb2a8814e6187%252Fportswigger-information-writeups-image-4.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=9985119c&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 4: Authentication Bypass <a href="#lab-4-authentication-bypass" id="lab-4-authentication-bypass"></a>

To solve this lab, login as the admin and delete `carlos`. When trying to visit the `/admin` directory, I see this:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-1739d34ba704b5fdcba724ebbd530d37fdcd646a%252Fportswigger-information-writeups-image-5.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=aba89fea&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This gives me an idea to exploit SSRF. Since this lab is about information disclosure, I tried using TRACE to view debug information, and it worked!

This was the response:

```http
HTTP/2 200 OK
Content-Type: message/http
X-Frame-Options: SAMEORIGIN
Content-Length: 614

TRACE /admin HTTP/1.1
Host: 0aa600d304d3392386c44517001f007e.web-security-academy.net
user-agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
accept-language: en-US,en;q=0.5
accept-encoding: gzip, deflate, br
upgrade-insecure-requests: 1
sec-fetch-dest: document
sec-fetch-mode: navigate
sec-fetch-site: none
sec-fetch-user: ?1
te: trailers
cookie: session=cookie
Content-Length: 0
X-Custom-IP-Authorization: <TRUNCATED>
```

There's an X-Custom-IP-Authorization header. I can use this to bypass the 'local user' check.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-c71a41ddd5acf063bbe4c49b7ea4ba914d645d32%252Fportswigger-information-writeups-image-6.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=f155fda7&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I can then visit `/admin/delete?username=carlos` to delete the user (this endpoint is found within the HTML for `/admin`).

### Lab 5: Version Control History <a href="#lab-5-version-control-history" id="lab-5-version-control-history"></a>

There's information disclosure via version control history. When running a `gobuster` scan, I found a `.git` repository.

```bash
$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/common.txt -k -u https://0ac2001804901e37806599da00eb00b0.web-security-academy.net -H 'Cookie: session=d893aVyPRCvqJphQnPppX4O1m5Dbzg1I'
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://0ac2001804901e37806599da00eb00b0.web-security-academy.net
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.git/HEAD            (Status: 200) [Size: 23]
/.git                 (Status: 200) [Size: 1201]
/.git/index           (Status: 200) [Size: 225]
/.git/logs/           (Status: 200) [Size: 548]
/.git/config          (Status: 200) [Size: 157]
```

I can install the entire repository using `wget -r https://LAB.web-security-academy.net/.git`. Afterwards, I can take a look at the `git log` output to see any changes that were made to the repository.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-d73e8167daeda2cd25d91b204dd4aa3c7b6fa2cd%252Fportswigger-information-writeups-image-7.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=e3bd393b&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Using this password, I can then login as the admin and delete `carlos`.
