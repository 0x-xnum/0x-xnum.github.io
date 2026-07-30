# Transport Layer Security (TLS) and SSL \*\*

TLS (formerly SSL) secures communication between a client (browser) and a server by encrypting data to prevent eavesdropping and tampering. Before TLS, online communication was vulnerable to interception. SSL 2.0 (1995) introduced encryption, authentication, and data integrity, but all SSL versions are now deprecated due to security flaws. Only TLS (preferably TLS 1.3) should be used today.

**How SSL/TLS Works**

A secure connection is established through a **cipher suite**, which consists of:

* **Authentication Algorithm** – Verifies both parties (via certificates).
* **Key Exchange Algorithm** – Determines how encryption keys are shared.
* **Bulk Encryption Algorithm** – Encrypts the data to keep it private.
* **Message Authentication Code (MAC) Algorithm** – Ensures data integrity.

Before communication, the **SSL/TLS handshake** occurs, where the client and server agree on a common cipher suite. If they can't agree, the connection fails.

**Ingredients for SSL/TLS**

SSL/TLS ensures that users connect to the legitimate server, not an imposter. This is done through **digital certificates**, issued and cryptographically secured by **Certificate Authorities (CAs)**. These CAs are globally trusted, and their certificates cannot be forged.

1. **Authentication with Certificates**
   * When a client connects, the server provides its **certificate**.
   * If the domain name matches the certificate, the connection is considered secure.
2. **Key Exchange & Asymmetric Encryption**
   * Certificates use **asymmetric key pairs** (public & private keys).
   * Messages encrypted with the **public key** can only be decrypted by the **private key** (and vice versa).
   * This prevents eavesdroppers from reading messages sent to the server.
3. **Forward Secrecy & Secure Key Exchange**
   * TLS 1.3 only allows key exchange methods that support **forward secrecy** (e.g., **Diffie-Hellman key exchange**).
   * This ensures that if a private key is compromised, past communications remain secure.
   * Diffie-Hellman generates a shared secret **without sending it over the network**, reducing risks.
4. **Bulk Encryption & Secure Communication**
   * After key exchange, **symmetric encryption** (faster and equally secure) encrypts all data.
   * The **Message Authentication Code (MAC) Algorithm** ensures integrity—any tampering is detected, and the connection may be closed.

## **The general workflow is as follows:** <a href="#f221" id="f221"></a>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*tR1VuygqcxcmGg9Gyr3Y2Q.png" alt="" height="827" width="700"><figcaption><p>TLS Example Workflow</p></figcaption></figure>

* **Client Hello** – The client initiates communication and sends a list of supported cipher suites.
* **Server Response** – The server selects the most secure shared cipher suite and sends its public certificate.
* **Certificate Verification** – The client verifies the certificate’s authenticity by checking its signature against a trusted Certificate Authority.
* **Key Exchange** – The client and server negotiate a symmetric encryption key using the Key Exchange Algorithm.
* **Secure Communication Begins** – The client encrypts its first HTTP request using the Bulk Encryption Algorithm.
* **Decryption & Integrity Check** – The server decrypts the request and verifies its integrity using the Message Authentication Code (MAC).
* **Server Response** – The server encrypts and sends the HTTP response, which the client decrypts and verifies.
* **Session Continuation** – This encrypted exchange continues until the session ends. A new session requires a fresh key exchange.
