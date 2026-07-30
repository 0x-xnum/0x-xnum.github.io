# Data Transmission Without Encryption

Data Transmission Without Encryption  occurs when sensitive data is transmitted over a network without encryption. Applications send passwords, authentication tokens, credit card numbers, personal information, and other sensitive data in plain text or with weak encryption, allowing anyone with network access to intercept and read it.

This is one of the oldest and most exploited vulnerabilities in web applications. It's not about attacking the application itself—it's about intercepting data in transit before it reaches the application.

***

### Real-World Attack Scenarios

#### Scenario 1: HTTP Login Without HTTPS

A website's login form submits credentials over HTTP (unencrypted):

```html
<form action="http://example.com/login" method="POST">
  <input type="email" name="email" placeholder="Email">
  <input type="password" name="password" placeholder="Password">
  <button type="submit">Login</button>
</form>
```

**The attack:**

An attacker on the same network (coffee shop WiFi, corporate network, ISP) uses a packet sniffer to capture traffic:

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

email=user@company.com&password=MySecurePassword123
```

The attacker now has the user's credentials in plain text. They can:

* Log in as the user
* Access sensitive data
* Perform actions on behalf of the user
* Reset the password and lock out the real owner

**Finding it:** Check if login endpoints use HTTPS. Test if credentials are sent over HTTP. Use browser developer tools or network analysis tools to verify encryption.

**Exploit:**

```bash
# Listen for HTTP traffic on the network and capture credentials
tcpdump -i eth0 'tcp port 80' -A | grep -i "password"

# Or use urlsnarf to capture form submissions
urlsnarf -i eth0
```

***

#### Scenario 2: API Tokens in Plaintext

A mobile app communicates with an API using HTTP and includes authentication tokens in the Authorization header:

```http
GET /api/user/profile HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**The vulnerability:**

The token is transmitted unencrypted. An attacker on the network:

* Captures the token
* Uses it to make authenticated API requests
* Accesses the user's data
* Performs actions without the user knowing

**The damage:** Unlike passwords, tokens often don't change after compromise. The attacker maintains access until the token expires. If the app doesn't validate token origin or request context, the attacker has complete access.

**Finding it:** Intercept mobile app traffic with tools like Burp Suite or mitmproxy. Check if API tokens are transmitted over HTTP. Test if captured tokens work from different networks/devices.

**Exploit:**

```bash
# Intercept and inspect mobile app traffic
mitmproxy -i eth0

# Capture Authorization headers and use them to make authenticated requests
curl -H "Authorization: Bearer <captured_token>" https://api.example.com/user/profile
```

***

#### Scenario 3: Session Cookies Over HTTP

A website sets session cookies over HTTP:

```http
HTTP/1.1 200 OK
Set-Cookie: sessionid=abc123def456; Path=/
```

The session cookie is transmitted unencrypted. An attacker captures it:

```http
GET /admin HTTP/1.1
Host: example.com
Cookie: sessionid=abc123def456
```

**The result:** The attacker impersonates the user without knowing their password. They have full access to the user's account.

**Finding it:** Check cookies in browser DevTools. Use network analysis tools to see if cookies are sent over HTTP. Test if captured cookies work from different browsers/machines.

**Exploit:**

```bash
# Capture HTTP traffic to extract session cookies
tcpdump -i eth0 'tcp port 80' -A | grep -i "cookie"

# Use captured cookie to impersonate the user
curl -H "Cookie: sessionid=abc123def456" https://example.com/admin
```

***

#### Scenario 4: Payment Information Over HTTP

An e-commerce site transmits credit card data over HTTP during checkout:

```http
POST /process-payment HTTP/1.1
Host: shop.example.com

card_number=4111111111111111&expiry=12/25&cvv=123&amount=99.99
```

**The attack:**

An attacker on the network captures credit card information from multiple transactions:

```
4111111111111111, 4222222222222222, 4333333333333333...
```

They now have payment data for fraud, sale on dark web, or identity theft.

**Legal consequence:** PCI DSS (Payment Card Industry Data Security Standard) explicitly requires encryption for credit card transmission. Companies violating this face fines up to $100,000+ per incident.

**Finding it:** Test checkout flows over HTTP. Capture payment requests to verify encryption. Check if PII and payment data are ever transmitted unencrypted.

**Exploit:**

```bash
# Intercept HTTP traffic during payment processing
mitmproxy -i eth0

# Capture credit card data from payment requests
# POST /process-payment
# card_number=4111111111111111&expiry=12/25&cvv=123
```

***

#### Scenario 5: API Endpoints Accepting Both HTTP and HTTPS

A company implements HTTPS correctly but also leaves HTTP endpoints active. Many users and apps access the HTTP version out of habit or misconfiguration:

```
https://api.example.com/data          (encrypted)
http://api.example.com/data           (unencrypted - still works)
```

**The vulnerability:**

Developers assume users will use HTTPS, but:

* Legacy apps use HTTP
* Load balancers redirect HTTP to HTTPS, but packets are still captured in transit
* Users manually type http\:// instead of https\://
* Attackers perform SSL stripping attacks (downgrade HTTPS to HTTP)

**Finding it:** Test both http\:// and https\:// versions of endpoints. Check if HTTP traffic is rejected or allowed. Use tools like curl to test unencrypted access.

**Exploit:**

```bash
# Test if HTTP endpoint is accessible
curl http://api.example.com/data

# If this returns sensitive data instead of a redirect, it's vulnerable
# Even if HTTPS is configured, HTTP access means data can be captured during transit
```

***

#### Scenario 6: Configuration Data in Plaintext

An application stores and transmits API keys, database credentials, and configuration secrets in plaintext files over HTTP:

```http
GET /config.json HTTP/1.1

{
  "db_password": "admin123",
  "api_key": "sk_REDACTED_live_4eC39HqLyjWDarht...",
  "stripe_secret": "sk_REDACTED_live_51234567890..."
}
```

**The attack:**

An attacker captures the configuration file and gains access to:

* Database credentials
* Third-party API keys
* Internal service credentials
* Master secrets for entire infrastructure

With these credentials, they can:

* Access the database directly
* Use API integrations fraudulently
* Access cloud infrastructure
* Move laterally through the network

**Finding it:** Scan for unencrypted transmission of configuration files. Check if secrets are exposed in source code repositories, logs, or accessible endpoints.

**Exploit:**

```bash
# Attempt to access configuration files over HTTP
curl http://example.com/config.json

# Or monitor network traffic to capture config data
tcpdump -i eth0 'tcp port 80' -A | grep -i "password\|api_key\|secret"
```

***

#### Scenario 7: DNS Queries Without Encryption

A mobile app makes DNS queries over UDP (unencrypted) to resolve API endpoints:

```
DNS Query: api.example.com (unencrypted)
Response: 192.168.1.100
```

**The attack:**

An attacker performs DNS spoofing:

* Intercepts the DNS query
* Responds with their own IP address
* The app connects to the attacker's server instead of the real API
* Man-in-the-middle attack succeeds

**The solution:** Use DNS over HTTPS (DoH) or DNS over TLS (DoT) to encrypt DNS queries.

**Finding it:** Monitor DNS traffic from applications. Check if apps use encrypted DNS. Test if DNS responses can be spoofed.

**Exploit:**

```bash
# Capture unencrypted DNS queries
tcpdump -i eth0 'udp port 53' -A

# Perform DNS spoofing to redirect the app to attacker's server
# Respond with attacker IP for api.example.com queries
```

***

### Mitigation Strategies

**Enforce HTTPS everywhere** Use HTTPS for all endpoints, not just login or payment. There's no performance justification for HTTP anymore.

**Implement HSTS (HTTP Strict Transport Security)** Tell browsers to always use HTTPS for your domain:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

**Secure cookies properly** Set the Secure flag to prevent transmission over HTTP:

```
Set-Cookie: sessionid=abc123; Secure; HttpOnly; SameSite=StricDisable HTTP entirely Don't accept HTTP requests. Redirect all HTTP to HTTPS with a 301 status code.
```

**Use certificate pinning for mobile apps** Pin expected certificates to prevent MITM attacks even with compromised CAs.

**Encrypt sensitive data at rest and in transit**

* TLS for transmission
* AES-256 for storage
* Separate encryption for backups

**Validate certificates properly** Verify certificate validity, expiration, and CN/SAN matching.

**Use strong cipher suites** Disable weak ciphers like SSL 3.0, RC4, and DES. Use TLS 1.2+ with modern cipher suites.

**Monitor and alert on unencrypted requests** Log and flag any attempts to transmit sensitive data unencrypted.

**Use DNS encryption** Implement DoH or DoT to prevent DNS spoofing attacks.

***

###
