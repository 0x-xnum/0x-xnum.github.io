# Classic Authentication Attacks

here, We're talking about brute-force attacks, password spraying, and messing with password resets.

### 1. Password Brute-Force Attacks

* **What It Is**: Basically, you're trying a bunch of username/password combos to get into an API.
* **How It Works**: You send requests with different credentials, usually in JSON format. Don't forget to base64 encode them for authentication!
* **Tools You Can Use**: Check out Burp Suite’s Intruder or Wfuzz for this.
* **Example with Wfuzz**:

  ```bash
  wfuzz -d '{"email":"a@email.com","password":"FUZZ"}' -H 'Content-Type: application/json' -z file,/path/to/rockyou.txt -u http://target/api/auth/login --hc 405
  ```
* **Key Commands**:
  * `-d`: This is where you put the data you're sending.
  * `-H`: Add any headers you need (like Content-Type).
  * `--hc`: Use this to hide certain response codes to keep things tidy.

### 2. Password Spraying

* **What It Is**: Instead of trying lots of passwords on one account, you use a few common passwords across many accounts. This way, you dodge account lockouts.
* **How to Do It**: Grab a short list of likely passwords (think "Password1!", "QWER!@#$") and combine them with a list of usernames you've gathered from earlier recon.
* **Example**: If you've got a JSON response with emails, you can use grep to pull those out:

  ```bash
  grep -oe "[a-zA-Z0-9._]\+@[a-zA-Z]\+.[a-zA-Z]\+" response.json
  ```

### 3. Analyzing Your Results

* **Success Signs**: Keep an eye out for HTTP status codes in the 200s or 300s and any response lengths that stand out from your failed attempts.

### 4. Base64 Encoding

* **Quick Note**: If the API uses base64 encoding, make sure your credentials are encoded properly when you send them.

#### **tools and wordlist :**

* **Wordlist:**  `rockyou.txt`. It's often available on Kali Linux and can be unzipped using `gzip -d /usr/share/wordlists/rockyou.txt.gz`.
* **Mentalist App:** (<https://github.com/sc0tfree/mentalist>)
* **Common User Passwords Profiler (CUPP):** (<https://github.com/Mebus/cupp>)

&#123;% content-ref url="/pages/MGfpgRxzQsWcwBIVA5oJ" %}
[Password Attacks](/password-attacks)
&#123;% endcontent-ref %}
