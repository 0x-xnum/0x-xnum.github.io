# Improper Following of a Certificate's Chain of Trust

Secure communication on the internet relies heavily on **Public Key Infrastructure (PKI)** and **SSL/TLS certificates**. These certificates establish **trust** between a client and a server, ensuring that users are connecting to legitimate services rather than malicious ones.

However, when a system fails to properly validate the **certificate chain of trust**, it opens the door to **man-in-the-middle (MITM) attacks, impersonation, phishing, and data interception**. This is classified as **CWE-296: Improper Following of a Certificate's Chain of Trust**.

### **Explanation**

Certificates are used to establish secure communication, ensuring that entities (like websites or services) are who they claim to be. The **chain of trust** relies on:

1. The end-entity certificate (used by the website or service).
2. Intermediate certificate(s) issued by a trusted authority.
3. A root certificate from a trusted Certificate Authority (CA).

When a system improperly follows this chain, it may accept an untrusted or malicious certificate, leading to security risks.

### **Common Causes**

#### **1. Accepting Self-Signed Certificates Without Verification**

**Description:**

* Self-signed certificates are not issued by a trusted **Certificate Authority (CA)**, meaning they cannot be verified by a root of trust.
* Some applications allow users to provide their own certificates but fail to **check the issuer**, making them vulnerable to **MITM attacks**.

**Attack Scenario:**

**Wi-Fi Hotspot MITM Attack**

* A hacker sets up a **malicious public Wi-Fi** network and intercepts traffic using a **self-signed certificate**.
* A victim connects to the Wi-Fi and visits a site over HTTPS.
* If the victim’s browser **does not validate certificates properly**, it will **accept the attacker's self-signed certificate**, allowing the attacker to steal sensitive data.

**Mitigation:**\
Always validate the certificate issuer and ensure it belongs to a **trusted CA**.\
Implement **certificate transparency logs** to detect unauthorized certificates.

***

#### **2. Trusting User-Supplied Certificates Without Validating the Entire Chain**

**Description:**

* Some applications allow users to provide **custom certificates**, but if the entire **certificate chain is not validated**, attackers can use fraudulent certificates.

**Attack Scenario:**

**Bypassing Client Authentication**

* A web service uses **mutual TLS (mTLS)** for authentication.
* An attacker **supplies a fake client certificate**, and if the server **fails to validate the full chain**, it may accept the certificate as valid, granting access.

**Mitigation:**\
\- Always validate the **full chain of trust**, ensuring each certificate is issued by a legitimate CA.\
\- Implement strict **mutual TLS validation**.

***

#### **3. Improper Certificate Pinning Leading to Bypasses**

**Description:**

* **Certificate pinning** ensures that an application only trusts specific certificates, reducing MITM risks.
* However, **improper pinning** can lead to security bypasses if attackers exploit weak implementations.

**Attack Scenario:**

**SSL Pinning Bypass in Mobile Apps**

* A mobile banking app uses **certificate pinning** but only checks the **common name (CN)** instead of verifying the entire certificate.
* An attacker installs a **proxy tool** and **pins their own CA certificate**.
* The app accepts the attacker’s certificate, allowing **MITM attacks** to steal login credentials.

**Mitigation:**\
\- **Pin public keys** instead of full certificates to allow **certificate renewal**.\
\- Use **HPKP (HTTP Public Key Pinning)** or **DNS-based TLSA records** for proper pinning.

***

#### **4. Ignoring Certificate Revocation (CRL/OCSP Bypass)**

**Description:**

* Attackers can use **revoked certificates** if an application does not check for **Certificate Revocation Lists (CRLs)** or **Online Certificate Status Protocol (OCSP)** responses.

**Attack Scenario:**

**Using a Revoked Certificate for Spoofing**

* A hacker obtains a **revoked SSL certificate** from a compromised CA.
* A web application **fails to check OCSP**, so it still trusts the revoked certificate.
* The hacker can impersonate a legitimate service and perform **MITM attacks**.

**Mitigation:**\
\- Enforce **OCSP stapling** to ensure revocation checks are performed.\
\- Require applications to verify **CRLs (Certificate Revocation Lists)**.

***

#### **5. Weak or Incomplete Validation of Certificate Authorities (CAs)**

**Description:**

* Some applications accept **any certificate from a CA**, even if the CA itself is untrusted or compromised.

**Attack Scenario:**

**Using a Rogue CA to Issue Fake Certificates**

* An attacker compromises a **low-security CA** and issues fake certificates for major websites.
* Browsers that trust **all CA-issued certificates** automatically accept the fake ones.
* This enables large-scale **phishing attacks** or MITM attacks.

**Mitigation:**\
\- Maintain a strict **list of trusted root CAs** and **remove untrusted CAs**.\
\- Use **Certificate Transparency Logs** to detect unauthorized certificates.
