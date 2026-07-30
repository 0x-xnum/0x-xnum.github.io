# Evasive Maneuvers

Many APIs in the wild will not be exploited as easily as the deliberately vulnerable applications encountered in this course. Security controls such as web application firewalls (WAFs) and rate limiting can block attacks. These controls may vary from one API provider to another but typically have thresholds for malicious activity that trigger responses. Common triggers for WAFs include:

* Too many requests for resources that do not exist
* Too many requests within a certain time frame
* Common attack attempts (e.g., SQL injection, XSS)
* Abnormal behavior (e.g., tests for authorization vulnerabilities)

Evading security controls requires trial and error, as some may not advertise their presence with response headers and may wait for missteps. Below are effective measures for evading or bypassing restrictions.

***

### String Terminators

String terminators can disrupt API security controls by terminating the processing of input. Common string terminators include:

* `%00`
* `0x00`
* `//`
* `;`
* `%`
* `!`
* `?`
* `[]`
* `%5B%5D`
* `%09`
* `%0a`
* `%0b`
* `%0c`
* `%0e`

These can be strategically placed in different parts of the request, such as the path or POST body. For example, in the following injection attack:

```json
POST /api/v1/user/profile/update
{
  "uname": "hapihacker",
  "pass": "%00'OR 1=1"
}
```

The null byte can bypass input validation measures.

***

### Case Switching

When security controls rely on the literal spelling and case of components, case switching can effectively bypass these controls. For example, in a POST request targeting an IDOR attack against a `uid` parameter:

```http
POST /api/myprofile 
{uid=§0001§}
```

If the API has rate limiting set to 100 requests per minute, you could manipulate the case to create variations:

```http
POST /api/myProfile
POST /api/MyProfile
POST /aPi/MypRoFiLe
```

These different path iterations might cause the API to handle requests differently, allowing you to bypass rate limiting. Using tools like Burp Suite’s Pitchfork attack can help pair attacks with brute-force attempts efficiently.

***

### Encoding Payloads

Encoding payloads can help evade WAFs by altering how payloads are processed. Even if certain characters or strings are blocked, encoded versions may pass undetected. For instance:

* **URL Encoded Payload**: `%27%20%4f%52%20%31%3d%31%3b`
* **Double URL Encoded Payload**: `%25%32%37%25%32%30%25%34%66%25%35%32%25%32%30%25%33%31%25%33%64%25%33%31%25%33%62`

The double encoding might slip past a WAF’s detection.

***

### Payload Processing with Burp Suite

Once a WAF bypass method is discovered, Burp Suite’s Intruder can automate evasive attacks. The **Payload Processing** feature allows you to add rules to each payload before sending.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/Lc9XrEHQSQqQe3srFQEn_Evasion1.PNG)

For instance, if you need to add null bytes before and after a URL-encoded payload, your processing rules should be set to apply encoding first, followed by adding null bytes. An example of processed payloads:

```http
POST /api/v3/user?id=%00%75%6e%64%65%66%69%6e%65%64%00
POST /api/v3/user?id=%00%75%6e%64%65%66%00
POST /api/v3/user?id=%00%28%6e%75%6c%6c%29%00
```

Ensure the payloads have been processed correctly by reviewing the attack results.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/qDzcT4JSUkAVWywCB1NQ_Evasion2.PNG)

***

#### Evasion with Wfuzz

Wfuzz offers capabilities for payload processing, including encoding options. To see available encoders, use:

```bash
wfuzz -e encoders
```

A sample command to use an encoder:

```bash
wfuzz -z file,wordlist/api/common.txt,base64 http://hapihacker.com/FUZZ
```

This command would base64-encode every payload before sending. You can also combine encoders, such as:

```bash
wfuzz -z list,TEST,base64-md5-none
```

This would result in multiple encoded versions of the payload. For example:

```bash
wfuzz -z list,a-b-c,base64-md5-none -u http://hapihacker.com/api/v2/FUZZ
```
