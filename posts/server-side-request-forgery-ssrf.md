# Server-Side Request Forgery (SSRF)

### What is SSRF?

SSRF occurs when an attacker can control the URL or IP address used by a server to make HTTP requests. For example, if a web application fetches a user-provided URL without validating it, an attacker could trick the server into accessing internal services like `http://localhost:8080` or external systems under their control. This can lead to severe consequences, such as:

* **Accessing internal systems**: Interacting with services like databases, admin panels, or metadata endpoints (e.g., AWS metadata at `http://169.254.169.254`).
* **Bypassing firewalls**: Reaching firewalled resources that are inaccessible from the public internet.
* **Data exfiltration**: Leaking sensitive data to attacker-controlled servers.
* **Service disruption**: Triggering unauthorized actions on internal or external systems.

#### Example Scenario

Imagine a web application with a feature that fetches a user-provided URL to display a preview of a webpage. The backend processes the request as follows:

```http
GET /preview?url=http://example.com
```

If the application does not validate the `url` parameter, an attacker could supply `http://localhost/admin` or `http://169.254.169.254/latest/meta-data/` to access internal resources. This is the essence of SSRF.

### Identifying SSRF Vulnerabilities

To identify SSRF vulnerabilities, focus on application features that involve server-side requests. These are often entry points where user input influences the destination of HTTP requests. Below are key areas to investigate, along with testing strategies and examples.

### 1. URL Import Features

Applications that fetch external resources, such as image importers or API data fetchers, are prime candidates for SSRF. If the server blindly trusts user-supplied URLs, attackers can target internal endpoints.

**Testing Strategy**:

* Identify endpoints that accept URLs as input (e.g., `/fetch?url=...` or `/import?url=...`).
* Test with internal URLs like `http://localhost`, `http://127.0.0.1`, or `http://internal-service.local`.
* Try sensitive cloud metadata endpoints, such as `http://169.254.169.254/latest/meta-data/` (AWS) or `http://metadata.google.internal/computeMetadata/v1/` (Google Cloud).

**Example**:\
A web application allows users to import images via a URL:

```http
POST /import-image
Content-Type: application/json
{"url": "http://example.com/image.jpg"}
```

Test by submitting:

```http
{"url": "http://localhost:8080/admin"}
```

If the server responds with data from the internal admin panel or an error indicating a connection attempt, you’ve likely found an SSRF vulnerability.

### 2. File Upload Mechanisms

File uploads, especially for formats like PDFs, SVGs, or Office documents, can be exploited if the server processes embedded URLs. For instance, an SVG file might include a reference to an external resource that the server fetches.

**Testing Strategy**:

* Upload or trigger generation of files containing embedded URLs (e.g., `<iframe src="http://localhost">`).
* Test with URLs pointing to internal services or attacker-controlled servers.
* Monitor for server responses or use out-of-band (OAST) tools like Burp Collaborator to detect callbacks.

**Example**:

Inject an iframe pointing to an internal resource:

```xml
<iframe src="http://169.254.169.254/latest/meta-data"></iframe>
```

If the server renders the PDF with this embedded content and processes the request, it may lead to SSRF and exposure of internal services.

**Reference**: [*The PDF Trojan Horse: Leveraging HTML Injection for SSRF and Internal Resource Access*](https://uchihamrx.medium.com/the-pdf-trojan-horse-leveraging-html-injection-for-ssrf-and-internal-resource-access-fbf69efcb33d).

### 3. Headless Browsers / HTML Rendering

Features that generate PDFs, screenshots, or previews using headless browsers are susceptible to SSRF. These systems often parse HTML content and fetch embedded resources, which can be manipulated.

**Testing Strategy:**

* Inject URLs into HTML-like inputs (e.g., `<iframe src="http://internal-service">` or `<img src="http://localhost">`).
* Try accessing internal services, cloud metadata, or local APIs like `http://127.0.0.1:9222/json`.
* Use OAST tools (e.g., Burp Collaborator) to confirm blind SSRF via callbacks.

**Example:**\
A dashboard-building app allows users to export their dashboards to PDF. Each dashboard contains user-supplied JSON data, which gets rendered in a headless browser:

```json
{
  "items": [
    {
      "type": "iframeobject",
      "url": "https://attacker.com"
    }
  ]
}
```

When this data is rendered, the attacker’s iframe loads in the backend Chromium instance. If the attacker hosts HTML with a nested iframe pointing to internal services like `http://127.0.0.1:9222/json`, the server may leak sensitive information like browser tabs or session tokens in the rendered PDF.

**Reference**: [*SSRF on a Headless Browser Becomes Critical*](https://medium.com/@Nightbloodz/ssrf-on-a-headless-browser-becomes-critical-c08daaa1017e).

### 4. Server Status and Monitoring Features

Applications that query internal services (e.g., `/health`, `/status`, `/check`) are often vulnerable to SSRF if they allow users to specify the destination URL.

**Testing Strategy:**

* Identify endpoints that accept a `url` or similar parameter.
* Test with internal IPs like `127.0.0.1` or cloud metadata services (`169.254.169.254`).
* Check response codes, timings, or use response validation (if available) to confirm access.
* Use OAST tools for blind SSRF detection.

**Example:**\
A cloud monitoring system allows setting a URL for uptime checks:

```
bashCopyEditGET /check-status?url=http://monitoring-service
```

Test with:

```
bashCopyEditGET /check-status?url=http://169.254.169.254/computeMetadata/v1/project/project-id
Headers: Metadata-Flavor: Google
```

Even without a visible response body, SSRF can be confirmed through:

* Fast response time (e.g., 2ms) indicating internal access
* Response validation features (e.g., check if response contains a known string)
* Brute-force or binary search over response body to reconstruct sensitive content

**Reference**: [*31k SSRF in Google Cloud Monitoring*](https://nechudav.blogspot.com/2020/11/31k-ssrf-in-google-cloud-monitoring.html).

### 5. Proxy Implementations

**Reference**: [*Server-Side Request Forgery on Havoc C2*](https://blog.chebuya.com/posts/server-side-request-forgery-on-havoc-c2/).

### 6. File Storage Integrations

Integrations with file storage services like **Amazon S3**, **Google Drive**, or **Dropbox** are common in modern web applications, especially for file uploads, downloads, and imports. These integrations often involve server-side HTTP requests — making them **prime SSRF targets**.

**Testing Strategy:**

* **Test Import/Export Features**: Look for upload/import endpoints that take a URL or a file path. Example payloads:

  ```json
  jsonCopyEdit{"url": "http://169.254.169.254/latest/meta-data/"}
  ```

  If the server fetches this internal AWS metadata endpoint, SSRF is confirmed.
* **Manipulate Cloud Bucket/Path**:
  * Try changing the bucket name or file path in S3-style URLs:

    ```
    bashCopyEdithttps://s3.amazonaws.com/mybucket/file.pdf
    → https://s3.amazonaws.com/another-bucket/../../etc/passwd
    ```
* **Use OAST Tools**:
  * Inject out-of-band URLs (e.g., Burp Collaborator, OASTify) to detect server-initiated requests.

#### [Real-World Example: SSRF via `_next/image` in Next.js](https://www.assetnote.io/resources/research/digging-for-ssrf-in-nextjs-apps)

#### 7. Path Parameters and Host Headers

Applications that dynamically construct server-side requests using **path parameters** or **host headers** can be manipulated by attackers to access internal systems. This can lead to **SSRF**, **privilege escalation**, and even **complete account takeovers**, depending on the logic.

#### Testing Strategy

* The app (`abc.victim.com`) uses `X-Forwarded-Host` to determine request origin.
* Attacker sets:

  ```http
  X-Forwarded-Host: attacker.com
  ```
* The app sends SSRF requests to `attacker.com` (attacker's server), expecting auth/permission responses.
* Attacker's server responds with forged authorization:

  ```json
  { "type": "admin" }
  ```
* Application trusts the response and grants **admin access**.
* As an admin, the attacker discovers a flawed *add user* feature — even with `403 Forbidden`, users are created.
* Attacker creates a new **admin user**, achieving **organization-wide takeover**.

**Reference**: [*Host Header Injection to Complete Organization Takeover*](https://medium.com/@_yldrm/host-header-injection-to-complete-organization-takeover-67a8a2ddb188) .

### Blind SSRF

Blind SSRF occurs when an attacker can trigger server-side requests but cannot see the full response, only a status code or indirect feedback. This makes detection challenging but not impossible.

**Detection Techniques**:

* **Out-of-Band (OAST) Testing**: Use tools like Burp Collaborator or Interact.sh to generate a unique domain (e.g., `attacker.oastify.com`) and monitor for HTTP or DNS requests.
* **Time-Based Testing**: Measure response times to infer whether a request was successful (e.g., a delay may indicate a connection to a slow internal service).
* **Error Messages**: Analyze error responses for clues about internal services or connection attempts.

**Example**:\
Test a URL import endpoint with:

```http
POST /import
Content-Type: application/json
{"url": "http://attacker.oastify.com"}
```

If your OAST tool logs an HTTP or DNS request, you’ve confirmed blind SSRF.

**Common Mistake**:\
Bug bounty hunters often report DNS pingbacks (e.g., via Burp Collaborator) as SSRF without further validation. Most platforms consider DNS pingbacks alone out of scope, as they don’t confirm actual request execution. Always verify impact by accessing internal resources or triggering observable effects.

### SSRF with DNS Rebinding

DNS rebinding is a sophisticated technique that combines SSRF with DNS manipulation to bypass the Same-Origin Policy, allowing attackers to access internal resources via a victim’s browser.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*lfJPUMCkS6rE7OH9-Y-6NA.png" alt=""><figcaption></figcaption></figure>

#### How It Works

1. **Domain Acquisition**: The attacker controls a domain (e.g., `example.com`) and configures its DNS with a low TTL[^1] to switch between their server and an internal IP (e.g., `192.168.1.3`).
2. **Victim Interaction**: The victim visits `example.com`, which loads malicious JavaScript from the attacker’s server.
3. **DNS Rebinding**: The DNS record for `example.com` is updated to resolve to the internal IP (e.g., `192.168.1.3`).
4. **Malicious Request**: The JavaScript sends a request to `http://example.com/sensitive-endpoint`, which resolves to the internal service. Since the origin remains the same, the Same-Origin Policy is not violated.
5. **Data Exfiltration**: The response is sent to an attacker-controlled domain (e.g., `harmful.example.com`).

**Example**:\
An internal web application at `http://192.168.1.3/admin` is only accessible within the organization’s network. An attacker:

1. Configures `example.com` to resolve to their server (`203.0.113.1`) with a TTL of 1 second.
2. Tricks a victim into visiting `http://example.com`, which loads malicious JavaScript.
3. Updates the DNS to resolve `example.com` to `192.168.1.3`.
4. The JavaScript sends a GET request to `http://example.com/admin`, retrieving sensitive data.
5. The data is exfiltrated to `http://harmful.example.com`.

**Tools**:

* **rbndr.us**: Generates hostnames for DNS rebinding tests.
* **Singularity**: A DNS rebinding attack framework (GitHub: nccgroup/singularity).
* **DNSrebinder**: A minimal DNS server for testing rebinding (GitHub: mogwailabs/DNSrebinder).

Reference:

[SSRF via DNS Rebinding (CVE-2022–4096)](https://infosecwriteups.com/ssrf-via-dns-rebinding-cve-2022-4096-b7bf75928bb2?source=post_page-----1654c72ee2df---------------------------------------)

[rbndr.us dns rebinding service](https://lock.cmpxchg8b.com/rebinder.html?source=post_page-----1654c72ee2df---------------------------------------)

[GitHub - nccgroup/singularity: A DNS rebinding attack framework.](https://github.com/nccgroup/singularity?source=post_page-----1654c72ee2df---------------------------------------)

### SSRF Filter Bypasses

Modern applications often implement filters to block SSRF attacks, but these can be bypassed using creative techniques. Below are common methods to evade SSRF protections.

#### 1. Open Redirects

If the application has an open redirect vulnerability, attackers can use it to redirect requests to internal services.

**Example**:\
An endpoint `/redirect?url=http://example.com` redirects to the provided URL. Test with:

```http
GET /redirect?url=http://localhost:8080
```

If the server follows the redirect to the internal service, you’ve bypassed the filter.

#### 2. IP Address Obfuscation

Filters often block obvious internal IPs like `127.0.0.1`. Use alternative representations to bypass them:

* **Localhost Range**: `127.0.0.0` to `127.255.255.255` (e.g., `127.0.0.2`).
* **Shortened Form**: `127.1`.
* **Padded Form**: `127.000000000000000.1`.
* **All Zeroes**: `0.0.0.0` or `0`.
* **Decimal Form**: `2130706433` (equivalent to `127.0.0.1`).
* **Octal Form**: `0177.0000.0000.0001`.
* **Hexadecimal Form**: `0x7f000001`.
* **IPv6 Loopback**: `0:0:0:0:0:0:0:1` or `::1`.
* **IPv4-mapped IPv6**: `::ffff:127.0.0.1`.

**Example**:\
Test an endpoint with:

```http
POST /fetch
Content-Type: application/json
{"url": "http://2130706433"}
```

If the server resolves `2130706433` to `127.0.0.1`, the filter is bypassed.

#### 3. Protocol-Based Bypasses

Filters often focus on HTTP/HTTPS but may overlook other protocols like `ftp://`, `file://`, or `gopher://`.

**Example**:\
Test with:

```http
POST /fetch
Content-Type: application/json
{"url": "file:///etc/passwd"}
```

If the server reads the local file, you’ve bypassed the filter.

#### 4. URL Encoding and Parsing Tricks

Filters may fail to normalize encoded or malformed URLs. Try:

* **URL Encoding**: `http://%6c%6f%63%61%6c%68%6f%73%74` (encodes `localhost`).
* **Double Encoding**: `http://%256c%256f%2563%2561%256c%2568%256f%2573%2574`.
* **Null Bytes**: `http://localhost%00@attacker.com`.
* **Mixed Case**: `HTTP://LOCALHOST`.

**Example**:\
Test with:

```http
POST /fetch
Content-Type: application/json
{"url": "http://%6c%6f%63%61%6c%68%6f%73%74"}
```

If the server fetches `http://localhost`, the filter is bypassed.

#### 5. Parser Inconsistencies

Different URL parsers in the application stack (e.g., frontend vs. backend) may interpret URLs differently, allowing bypasses.

**Example**:\
Test with:

```http
POST /fetch
Content-Type: application/json
{"url": "http://localhost#@attacker.com"}
```

If the backend parser ignores the fragment (`#`) and fetches `http://localhost`, you’ve found a bypass.

**Reference**: [*PayloadsAllTheThings*](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Request%20Forgery/README.md)

### Useful Tools for SSRF Testing

Here are some powerful tools to automate and enhance SSRF testing:

* [**SSRFmap** ](https://github.com/swisskyrepo/SSRFmap?source=post_page-----1654c72ee2df---------------------------------------)
* [**lorsrf** ](https://github.com/MindPatch/lorsrf?source=post_page-----1654c72ee2df---------------------------------------)
* [**SSRFPwned** ](https://github.com/blackhatethicalhacking/SSRFPwned?source=post_page-----1654c72ee2df---------------------------------------)
* [**IPFuscator**](https://github.com/vysecurity/IPFuscator?source=post_page-----1654c72ee2df---------------------------------------)
* [**DNSrebinder**](https://github.com/mogwailabs/DNSrebinder?source=post_page-----1654c72ee2df---------------------------------------)

***

## Lab: Basic SSRF against the local server <a href="#e8dc" id="e8dc"></a>

### Lab Description <a href="#id-19aa" id="id-19aa"></a>

This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.

### Lab Solution <a href="#id-97bf" id="id-97bf"></a>

To access the "check stock" feature we click on any of the products as in the screenshot below.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*J5viIMBMiR6ZxiYM" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Now check for this captured request in burp suite.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*zLGX4zoKEfF17XQF" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Now send the request to repeater.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*w6ztha4es-RgUHC4" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Now change the URL of the stockApi to the one given in the lab description as shown in the screenshot below. This modification sends the request back to the server, to a location that is local to the server which is hosting the application.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*NzpQErDi8eeTPP7L" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Finally send this request and the response for the same is as shown in the screenshot below.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*_7leUehIVane3YSF" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Now look for the path that will delete user Carlos and replace the stockApi URL with that as shown in the images below.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*Zd9PwGva-UzCvoxE" alt="" height="518" width="1000"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*aV-6bULEQyw6iUcm" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Finally send this request and as can be seen below the user has been deleted successfully and lab has been solved.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*niGAT0JUbmU2NxZf" alt="" height="518" width="1000"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*j0EzWcS4vWWtyqoD" alt="" height="518" width="1000"><figcaption></figcaption></figure>

## 2- SSRF attacks against other back-end systems <a href="#b308" id="b308"></a>

In many applications, the server can communicate with internal back-end systems that are not directly accessible by users. These systems often have non-routable private IP addresses and are typically protected by network segmentation. However, because they are assumed to be unreachable from the internet, they often have weaker security measures.

In many cases, these internal systems contain sensitive functionality that does not require authentication, assuming that only trusted internal components will interact with them. This makes them prime targets for **Server-Side Request Forgery (SSRF) attacks**.

### **Example: Accessing an Internal Admin Panel**

Consider the same shopping application example from before. Suppose the back-end system has an **administrative interface** hosted at `http://192.168.0.68/admin`, which is only meant to be accessed from inside the company's private network.

An attacker can target an internal admin panel using SSRF:

```http
POST /product/stock HTTP/1.0  
Content-Type: application/x-www-form-urlencoded  
Content-Length: 118  

stockApi=http://192.168.0.68/admin11
```

Since the request originates from the application server (which is inside the organization's network), it may successfully access the private administrative interface, even though the attacker is external.

> Private IP addresses are **reserved address blocks** used for internal network communication. Devices within a private network can communicate with each other using these addresses, but they **cannot be accessed directly from the public internet**.

Now consider this lab from Web Security Academy.

## Lab: Basic SSRF against another back-end system <a href="#id-3271" id="id-3271"></a>

### Lab Description <a href="#id-00cd" id="id-00cd"></a>

This lab has a stock check feature which fetches data from an internal system.

To solve the lab, use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port 8080, then use it to delete the user `carlos`.

### Lab Solution <a href="#id-6ab5" id="id-6ab5"></a>

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*OF6DmMTxqfvXSTfJ" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Access the check stock feature by clicking any product and using the check stock functionality as shown in the above screenshot.

Now check burp suite for the requests captured.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*L2RfoHjy31KCiVMe" alt="" height="518" width="1000"><figcaption></figcaption></figure>

As shown in the screenshot above, the request selected contains the check stock feature.

Carefully note that as mentioned in the lab description, the address provided to us is not complete. To know the complete address where the admin interface is hosted, we will send this request to burp suite intruder and brute force for the correct address.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*e7Pgg4orAjHHEyvF" alt="" height="518" width="1000"><figcaption></figcaption></figure>

As shown in the above screenshot we replace the stockApi value with the URL to the IP given in the lab description which points to port 8080 to access the admin panel.

We replace the “X” with “1” and add the payload position over there as this will be replaced each time to check for the correct IP address.

In next step we set the payload as the numbers from 1 to 255 with step value as 1 as shown in the screenshot below.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*AXCx18w4WjSZkEcw" alt="" height="518" width="1000"><figcaption></figcaption></figure>

The results from burp intruder are:

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*StqZO9QPliHziLe-" alt="" height="518" width="1000"><figcaption></figcaption></figure>

As can be seen in the results above, there is one request with status code 200 which means this is the successful request. The highlighted part is the correct IP address which hosts the admin panel over port 8080.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*D8Q0N53bZNtRm8QV" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Now send this request to repeater.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*1aNw5vxChVSDTAav" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Now in repeater, send the request and in the response look for the URL that will delete the user carlos.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*aperi-6Zcn3X5oMm" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Update the current value of stockApi with this URL and send the request.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*-UsKT-lWH_1AMS9G" alt="" height="518" width="1000"><figcaption></figcaption></figure>

This finally solves the lab.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*dNUv4y6RC9HFI6_F" alt="" height="518" width="1000"><figcaption></figcaption></figure>

## 3- Bypassing Blacklist-Based Input Filters in SSRF

Some applications attempt to prevent **Server-Side Request Forgery (SSRF)** by blocking requests containing specific hostnames (e.g., `127.0.0.1`, `localhost`) or restricted paths (`/admin`). However, **blacklist-based filtering** is often weak and can be **bypassed using various techniques :** &#x20;

#### **1. Using Alternative IP Representations**

* **Decimal notation:** `2130706433` (which equals `127.0.0.1` in decimal).
* **Octal notation:** `017700000001`.
* **Shorthand notation:** `127.1` (a valid alternative for `127.0.0.1`).

**Example:** Instead of using `http://127.0.0.1/admin`, try `http://2130706433/admin`.

#### **2. URL Encoding & Other Encoding Techniques**

* **URL encoding:** `http://127%2E0%2E0%2E1/admin`.
* **Double encoding:** `http://127%252E0%252E0%252E1/admin`.
* **Hex encoding:** `http://0x7f.0x00.0x00.0x01/admin`.
* **Base64 encoding (if applicable):** `aHR0cDovLzEyNy4wLjAuMS9hZG1pbg==` (if the application allows decoding).

#### **3. Case Variation Tricks**

Some blacklist filters **only block lowercase variations** of keywords. If filtering is **case-sensitive**, you can try different capitalizations:

* **Bypassing `localhost`:** `LoCaLhOsT`, `lOcAlHoSt`.
* **Bypassing `/admin`:** `/AdMiN`, `/ADMIN`.

**Tip:** Try a **combination of encoding and case variation** for better success!

#### **4. Using Open Redirects & External URLs**

If an application blocks direct requests to sensitive endpoints, you can **bypass the filter by using an open redirect** on a different domain that forwards the request to the restricted resource.

1. ```http
   https://your-redirect.com/redirect?url=http://127.0.0.1/admin
   ```

#### **5. Leveraging Different Redirect Codes & Protocols**

Web servers use **HTTP status codes** for redirections. Some SSRF filters fail when handling certain **redirect codes** or protocol changes. You can experiment with:

* **301 Moved Permanently**
* **302 Found (Temporary Redirect)**
* **307 Temporary Redirect** (preserves the original HTTP method).
* **308 Permanent Redirect** (similar to 301 but stricter).

**6.Protocol Switching:**

* If an application blocks `http://127.0.0.1/admin`, try redirecting from an **`https://`** URL instead.
* Some filters fail when handling `ftp://` or `file://` protocols.

Now consider this lab from Web Security Academy.

## Lab: SSRF with blacklist-based input filter <a href="#id-5fd0" id="id-5fd0"></a>

### Lab Description <a href="#id-2684" id="id-2684"></a>

This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.

The developer has deployed two weak anti-SSRF defenses that you will need to bypass.

### Lab Solution <a href="#id-7f06" id="id-7f06"></a>

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*c0mkomJp8nSARYxY" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Access the check stock feature by visiting any product and clicking the check stock button as shown in the screenshot below.

Now check burp suite for the request captured.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*fuUPxlHeuoxcB_K3" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Send this request to burp suite repeater and replace the current value of stockApi with the back-end system URL as shown in the screenshot below.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*TRcSUISOUFm4EIzC" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Now send the request and analyse the response.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*1JoaGyI6ahbgl9gb" alt="" height="518" width="1000"><figcaption></figcaption></figure>

As can be seen, the request has been blocked, this means there is some anti-SSRF defense mechanism which is blocking the request.

Now try URL encoding “/” and replacing it with its URL encoded counterpart and also try case variation in case of “localhost”. Finally send the request and analyse the response as shown in the screenshot below.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*mjM754x5wjSOYoR5" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Now try to replace “localhost” with “127.0.0.1” and also try case variation in “admin” leaving “/” URL encoded only. Send the request and analyse the response as shown in the image below.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*mbyb0aneQSYMBUua" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Now try to replace URL encoded “/” with “/” itself and also try some other case variation in “admin” and “localhost” as shown in the screenshot below. Finally send the request and analyse the response.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*0vu2ngT8-0VZHtg1" alt="" height="518" width="1000"><figcaption></figcaption></figure>

This means that the SSRF attack was successful and that we have got the access to admin panel.

Look for the path that will delete the user carlos as shown in the screenshot below.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*I_XhhNW4VS52WFjA" alt="" height="518" width="1000"><figcaption></figcaption></figure>

Modify the value of stockApi as shown in the screenshot below and send the request.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*oR2vzY5OKfr4HVAV" alt="" height="518" width="1000"><figcaption></figcaption></figure>

This finally solves the lab.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*HdeRAnq58rDe9BJR" alt="" height="518" width="1000"><figcaption></figcaption></figure>

The logic behind the lab was using an SSRF payload and by-passing the anti-SSRF security mechanism using payload obfuscation.

## **Bypassing SSRF Filters via Open Redirection**

In such a scenario, even if a URL filter is in place to block **dangerous domains** or **internal resources** (like `localhost` or `127.0.0.1`), an **open redirection vulnerability** may still allow attackers to bypass these filters. This happens when the application allows users to supply URLs that the backend server follows, and it supports HTTP **redirections** (such as HTTP 301, 302, or 307).

If the URL contains a legitimate domain that passes the filter, but redirects to a malicious internal IP or service, SSRF can still occur.

### **Example Scenario:**

Consider the following scenario in a vulnerable application:

The application allows users to check the stock of products from an external API. The user submits a request such as:

```http
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://evil-user.net
```

The URL `/product/nextProduct` contains a parameter `path` that redirects to any URL specified by the attacker:

```url
/product/nextProduct?currentProductId=6&path=http://evil-user.net
```

In this case, it will redirect to `http://evil-user.net`.

An attacker can leverage this vulnerability to bypass the SSRF filter. Instead of providing a **malicious internal URL** (which would be blocked), they use the open redirection URL to reach an internal service or target:

```http
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin
```

The server will:

1. Validate the `stockApi` parameter, which points to a **legitimate domain** (`weliketoshop.net`).
2. Make a request to this external domain, which then redirects to `http://192.168.0.68/admin` (an internal service).
3. The application follows the redirection, **accessing an internal administrative interface**, bypassing internal access controls.

### **How This Works**

The SSRF vulnerability works because the application:

1. **Validates the initial domain** (`weliketoshop.net`), allowing it through the filter.
2. **Follows the redirect** to an internal resource (`http://192.168.0.68/admin`), which the attacker controls.

Now consider this lab from Web Security Academy.

## Lab: SSRF with filter bypass via open redirection vulnerability <a href="#d1f9" id="d1f9"></a>

### Lab Description <a href="#b6d5" id="b6d5"></a>

This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at `http://192.168.0.12:8080/admin` and delete the user `carlos`.

The stock checker has been restricted to only access the local application, so you will need to find an open redirect affecting the application first.

### Lab Solution <a href="#id-2df8" id="id-2df8"></a>

Click on any product and then access the check stock feature as shown in the screenshot below.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*P53kfp4XxrF6vYK7" alt="" height="520" width="1000"><figcaption></figcaption></figure>

Now check burp suite for the intercepted request and send it to repeater.

Now try accessing the admin functionality as done in the screenshot below.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*v58crV7cLo0VPPFt" alt="" height="520" width="1000"><figcaption></figcaption></figure>

The response to the above request is:

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*XD8KD5912kutM8_n" alt="" height="520" width="1000"><figcaption></figcaption></figure>

This means the request to this internal resource is being blocked, hence we need to find an open redirection that can help us access this restricted resource.

On further analysis we get the next product button as shown below and clicking it captures the request for the same in burp suite.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*v-LWrLrmbJyDFPWW" alt="" height="520" width="1000"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*k0k3AeiW_8IU8gRy" alt="" height="520" width="1000"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*QI1rA0tjqP0pkdUA" alt="" height="520" width="1000"><figcaption></figcaption></figure>

As shown in the screenshot above, the request for the next product contains the query parameter path which provides the potential of a possible open redirection.

Open redirection means a security flaw in a web application that allows an attacker to manipulate where the application redirects a user.

Hence we try redirecting this to “google.com” as demonstrated below.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*D_sSC70iEM1zDuXF" alt="" height="520" width="1000"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*pYwQ4fWvLcSwWQNH" alt="" height="520" width="1000"><figcaption></figcaption></figure>

As can be seen, there is indeed an open redirection vulnerability that allows the attacker to control the redirection path.

Since the stockApi has access to the internal resources thus we can not redirect from this request itself but we copy the path of this request and then paste it in the stockApi parameter, further we append the internal path in the query parameter path. This is demonstrated in the screenshots below.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*L_TVB9Oc6JNU55yD" alt="" height="520" width="1000"><figcaption><p>copy the highlighted path</p></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*cmM4KCujccilM7V4" alt="" height="520" width="1000"><figcaption><p>paste it in the stockApi parameter and also append the internal path in the path query parameter</p></figcaption></figure>

Now the response for the above request is:

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*BPaMsV2-8rgi3fu7" alt="" height="520" width="1000"><figcaption></figcaption></figure>

Since the original value of the stockApi was URL encoded hence we encode this value also by selecting it and pressing ctrl+U.

The request is modified as:

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*rnIbuV3THQ3BtK-r" alt="" height="520" width="1000"><figcaption></figcaption></figure>

The response for the above request is:

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*4f4GWBWsHbDgrlNQ" alt="" height="520" width="1000"><figcaption></figcaption></figure>

This shows that we have successfully got access to the admin panel.

Now we look for the request that deletes user Carlos and modify the stockApi value accordingly as demonstrated in the screenshots below.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*h_y5SeXP1VvQ0mp3" alt="" height="520" width="1000"><figcaption><p>modified request</p></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*ZxNQpxX8NJfCuAZY" alt="" height="520" width="1000"><figcaption></figcaption></figure>

This finally solves the lab.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/0*9lBN8EdxbR2TnwSu" alt="" height="520" width="1000"><figcaption></figcaption></figure>

> The logic behind the working of this attack is that, the stockApi parameter had access to the internal resources of the server but it did not allow direct access to them due to strict validation of the request. Thus we look for the request that has an open redirection vulnerability which means that the user has control over the URL where redirection takes place.
>
> In this case the redirection was there in the “next product” request. We copy the path of this request and replace it with the value of stockApi since stock checking feature has access to internal resources and thus redirection to internal resources was not possible from the “next product” request itself.
>
> Finally we replace the path parameter with the value of the path to the internal resource that is admin in this case. This finally gives us the access to the admin panel and the lab is further solved as per the steps explained above.

[^1]: **TTL = Time To Live** — it's how long your browser or computer should remember a DNS result (IP address of a domain).
