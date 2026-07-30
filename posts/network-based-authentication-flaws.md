# Network-Based Authentication Flaws

### Real-World Attack Scenarios

#### Scenario 1: IP-Based Authentication

System trusts connections from specific IP:

```python
def get_admin_panel():
    client_ip = request.remote_addr
    
    # VULNERABLE - Trusts IP!
    if client_ip in ['192.168.1.100', '10.0.0.50']:
        return render_template('admin.html')
    
    return "Access Denied", 403
```

**The attack:**

Attacker spoofs source IP or redirects traffic:

```bash
# Method 1: IP Spoofing (network level)
# Attacker sends packets with source IP 192.168.1.100

# Method 2: Proxy through trusted IP
# Attacker finds proxy server at 192.168.1.100
# Routes traffic through proxy
# Server sees trusted IP

# Method 3: X-Forwarded-For Header (if vulnerable)
curl http://example.com/admin \
  -H "X-Forwarded-For: 192.168.1.100"
# If server trusts header, access granted
```

**Result:**

* IP-based access control bypassed
* Admin access gained
* No credentials needed

**Finding it:** Try spoofing IP. Use X-Forwarded-For header. Test from different IPs.

***

#### Scenario 2: Referer Header Authentication

Application uses Referer header to allow requests:

```python
@app.route('/api/data', methods=['POST'])
def get_data():
    referer = request.headers.get('Referer')
    
    # VULNERABLE - Trusts Referer!
    if referer == 'https://example.com/page':
        return get_sensitive_data()
    
    return "Forbidden", 403
```

**The attack:**

Attacker crafts request with spoofed Referer:

```bash
# Attacker's page or tool sends request with spoofed Referer
curl -X POST http://example.com/api/data \
  -H "Referer: https://example.com/page" \
  -H "Origin: https://attacker.com"

# Or via HTML/JavaScript
<img src="https://example.com/api/data">
<!-- Referer header automatically set -->
```

**Result:**

* Referer-based auth bypassed
* Access to protected data
* CSRF-like vulnerability

**Finding it:** Try requests without Referer. Use proxy to modify Referer. Check if endpoint accessible from different origins.

***

#### Scenario 3: Reverse DNS Authentication

System validates reverse DNS:

```python
def verify_host(ip):
    # VULNERABLE - Trusts reverse DNS!
    hostname, _ = socket.gethostbyaddr(ip)
    
    if 'trusted-host.com' in hostname:
        return True
    return False
```

**The attack:**

Attacker controls reverse DNS for their IP:

```bash
# Attacker's IP: 203.0.113.50
# Attacker's ISP allows them to set reverse DNS
# Sets reverse DNS to: trusted-host.com

# When server does reverse DNS lookup:
socket.gethostbyaddr('203.0.113.50')
# Returns: trusted-host.com

# Authentication passes!
```

Or DNS spoofing:

```bash
# Attacker performs DNS poisoning
# Changes reverse DNS response
# Server receives forged hostname
# Authentication passes
```

**Result:**

* Reverse DNS auth bypassed
* Access granted
* Server compromised

**Finding it:** Perform reverse DNS lookup on server. Test from different IPs.

***

#### Scenario 4: Channel Accessible by Unintended Endpoint

Internal API only accessible from internal network:

```
Internal network: 10.0.0.0/8
API endpoint: /internal/admin
Expected access: Only from 10.0.0.0/8

But endpoint also accessible via:
- IPv6 (if not blocked)
- VPN (if misconfigured)
- Load balancer bypass
- Direct IP connection
```

**The attack:**

```bash
# Access via unexpected channel
# IPv6 instead of IPv4
curl -6 http://[::ffff:10.0.0.1]/internal/admin

# Or find load balancer bypass
curl http://internal-ip:8080/admin

# Or connect directly to internal IP
# if firewall misconfigured
```

**Result:**

* Bypass of intended access controls
* Access to internal APIs
* Data exposure

***

#### Scenario 5: Header-Based Authentication Without Validation

Server relies on custom header for auth:

```python
@app.route('/protected')
def protected():
    auth_token = request.headers.get('X-Auth-Token')
    
    # VULNERABLE - Only checks if header exists
    if auth_token:
        return "Secret data"
    
    return "Unauthorized", 401
```

**The attack:**

```bash
# Just send any value in header
curl http://example.com/protected \
  -H "X-Auth-Token: anything"
# Access granted!

# Or
curl http://example.com/protected \
  -H "X-Auth-Token: "  # Empty value still accepted
```

**Result:**

* Authentication bypassed
* Any value in header grants access

***

### Mitigation Strategies

**Never rely on IP address alone**

Use proper authentication:

```python
# Bad
if request.remote_addr == '192.168.1.100':
    grant_admin_access()

# Good
@require_auth
def admin_panel():
    if session['user_id'] and user_is_admin():
        return render_template('admin.html')
```

**Don't trust headers for security decisions**

```python
# Bad
if request.headers.get('X-Admin') == 'true':
    grant_admin_access()

# Good
# Get auth from secure session/token
if session.get('is_admin'):
    grant_admin_access()
```

**Never use Referer for authentication**

```python
# Bad
if request.headers.get('Referer') == 'https://trusted.com':
    allow_request()

# Good
# Use CSRF token or session authentication
@require_csrf_token
def protected():
    pass
```

**Don't rely on reverse DNS**

```python
# Bad
hostname, _ = socket.gethostbyaddr(ip)
if 'trusted.com' in hostname:
    allow()

# Good
# Verify DNS forward and reverse match
hostname, _, addresses = socket.gethostbyname_ex(ip)
if ip in addresses:  # Forward DNS matches
    allow()
```

**Use proper endpoint access control**

```python
# All endpoints should require authentication
@app.route('/api/internal/admin')
@require_auth
@require_admin
def admin_api():
    return admin_data
```

**Implement proper origin validation**

```python
ALLOWED_ORIGINS = ['https://example.com', 'https://app.example.com']

@app.route('/api/data', methods=['POST'])
def api_data():
    origin = request.headers.get('Origin')
    
    if origin not in ALLOWED_ORIGINS:
        return "Forbidden", 403
    
    return get_data()
```

***

&#123;% embed url="<https://cwe.mitre.org/data/definitions/291.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/293.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/300.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/350.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/940.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/941.html>" %}
