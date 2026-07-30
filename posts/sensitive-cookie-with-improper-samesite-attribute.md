# Sensitive Cookie with Improper SameSite Attribute

### Description <a href="#description" id="description"></a>

The `SameSite` attribute controls whether cookies are sent with **cross-site requests**. It helps prevent **Cross-Site Request Forgery (CSRF)** attacks by restricting when cookies are automatically included by the browser.

it tells the browser:

> *“Hey browser, only send this cookie if the request is from my own site — don’t send it for cross-site requests.”*

Possible values for `SameSite` are:

* **`Strict`** — The safest option.

  > The cookie is **only sent for requests from the same site**.\
  > It is *never* sent with requests that come from another site.\
  > This blocks most CSRF attacks because malicious sites can’t trigger authenticated requests.

  **`Lax`** — A balanced option.

  > The cookie is sent for requests from the same site **and for some cross-site requests that are considered safe**, like a user clicking a link or typing a URL (top-level navigation using `GET`).\
  > It is *not* sent for background requests like hidden forms or AJAX, which are commonly used in CSRF.

  **`None`** — The least restrictive option.

  > The cookie is **always sent**, even for cross-site requests.\
  > This is sometimes needed for cross-domain use cases (like third-party services or iframes), but it also opens the door for CSRF if there’s no extra protection.\
  > When using `SameSite=None`, you must also set `Secure` (HTTPS only) — modern browsers enforce this.

If the `SameSite` attribute is **not set** or is set to `None` without other CSRF protections (like anti-CSRF tokens), an attacker can exploit this by crafting cross-site POST requests. The browser automatically includes the cookie, allowing unauthorized actions on behalf of the victim.

#### **Demonstration**

**Example 1: Insecure Cookie Setup**

```javascript
javascriptCopyEditlet sessionId = generateSessionId();
let cookieOptions = { domain: 'example.com' };
// ⚠️ Missing SameSite attribute
response.cookie('sessionid', sessionId, cookieOptions);
```

**Problem:**\
This cookie will be sent with *any request* to `example.com`, including cross-site POSTs.

***

**🧪 CSRF Attack Example**

**Attacker's malicious page:**

```html
htmlCopyEdit<html>
  <form id="evil" action="http://local:3002/setEmail" method="POST">
    <input type="hidden" name="newEmail" value="attacker@example.com" />
  </form>

  <script>
    evil.submit();
  </script>
</html>
```

**What happens:**

1. Victim visits this malicious page while logged in.
2. The form auto-submits a POST request to `http://local:3002/setEmail`.
3. Browser sends the `sessionid` cookie automatically.
4. The server believes it’s an authorized request from the real user — the victim’s email gets changed.

***

#### ✅ **Mitigation**

Always set an appropriate `SameSite` value for sensitive cookies — ideally `Strict` or at least `Lax` — and use CSRF tokens for critical actions.

**Secure version:**

```javascript
javascriptCopyEditlet sessionId = generateSessionId();
let cookieOptions = {
  domain: 'example.com',
  sameSite: 'Strict' // ✅ Prevents cross-site requests from sending this cookie
};
response.cookie('sessionid', sessionId, cookieOptions);
```

### **References**

* [MDN: SameSite cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
* OWASP: Cross-Site Request Forgery (CSRF) Prevention Cheat Sheet
