# API Token Attacks

### **Using Burp Suite for Token Analysis**

1. **Capture API Request**: Proxy your API authentication request to Burp Suite.
2. **Forward to Sequencer**: Right-click on the request and select the option to send it to the Sequencer.
3. **Analyze Randomness**:
   * **Define Token Location**: Specify where the token is located in the response.
   * **Start Live Capture**: Begin capturing live token data.
   * **Evaluate Results**: Look for predictability or weak randomness in generated tokens.

**Example**: Weakly generated tokens can be susceptible to brute-force attacks, allowing unauthorized access to endpoints like `/identity/api/v2/user/dashboard`.

**Manual Load of Bad Tokens**:

* Use Burp Suite’s Manual load option to analyze weakly generated tokens. You can use a weak token example generated from a [this repository ](https://raw.githubusercontent.com/hAPI-hacker/Hacking-APIs/main/bad_tokens)for reference.

***

### **JWT Attacks**

**Overview**:

* JSON Web Tokens (JWTs) are commonly used for API authentication but can have vulnerabilities if misconfigured.

#### **Components of JWTs**:

* **Header**: Contains metadata about the token.
* **Payload**: Contains claims or user information.
* **Signature**: Ensures token integrity.

#### **Using JWT.io for Analysis**:

* Decode JWTs using [JWT.io](https://jwt.io/) to inspect their contents.
* **Capturing a Valid JWT**: If you capture a valid JWT, it may grant unauthorized access to API endpoints based on the payload information.

#### **Tools for JWT Analysis**:

* **JWT\_Tool**: A tool for automating JWT analysis and scanning for vulnerabilities.
* **Commands**: Use specific commands with JWT\_Tool for scanning and analyzing tokens in target applications.

#### **Common JWT Vulnerabilities**:

* **None Algorithm Attack**: If the JWT uses "none" as its signing algorithm, attackers can forge tokens by altering the payload, potentially gaining unauthorized access.

### **JWT Attack Techniques**

1. **JWT Decoding**:
   * Decode parts of JWT for analysis:

     ```bash
     echo <header_part>|base64 -d
     echo <payload_part>|base64 -d
     ```
2. **Analyze JWT Structure**:
   * A JWT typically has three parts: header, payload, and signature.
   * Example command for decoding:

     ```bash
     echo eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ1c2VyYWExQGVtYWlsLmNvbSIsImlhdCI6MTY1ODUwNjQ0NiwiZXhwIjoxNjU4NTkyODQ2fQ|base64 -d
     ```
3. **JWT Signature Check**:
   * JWT signature uses HMAC with a secret:

     ```plaintext
     HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
     ```
4. **Automate with JWT\_Tool**:
   * Use `JWT_Tool` for automated testing:

     ```bash
     jwt_tool -t <target_url> -rh "Authorization: Bearer <JWT_Token>" -M pb
     ```
5. **The None Algorithm Attack**:
   * If a JWT uses "none" as the algorithm, modify payloads freely

### For More Attacks and Techniques about .jwt look at the [jwt\_Hacking](/jwt-hacking) Page.
