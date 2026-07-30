# Mail Server Misconfiguration

First of all now Email Spoofing happens due to two main reasons :

1. **SPF** record not set for the particular email
2. Missing of **DMARC** protocol for the particular email

### **What is a SPF Record ?**

So it basically stands for “Sender Policy Framework”. It is a type of Domain Name System (**DNS**) record that can help to prevent email address forgery or email spoofing.

### **What is a DMARC Protocol ?**

**DMARC** stands for “Domain-based Message Authentication, Reporting & Conformance” is a protocol that uses Sender Policy Framework, (**SPF**) and Domain-Keys identified mail (**DKIM**) to determine the authenticity of an email message.

I found this vulnerability on BaseCamp a Project Management company. I exploited the vulnerability and reported to them, they triaged my report and after some days they resolved the issue and paid me the bounty of **100$**.

It is not obvious that an email having the SPF record will also have the DMARC protocol and vise versa. You must check for both the things. And it is not obvious that if one email of the company has both the things enabled than other emails will also have the same things enabled. You must check for all the emails.

### **How to find it ?**

1. Go to your target company and collect all the possible emails of that company
2. Now check the SPF and DMARC record of all the emails : <https://mxtoolbox.com/>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*h0tuK5wBSRvFeUC4ebmlrA.png" alt="" height="394" width="700"><figcaption><p>SPF Record</p></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*JDmpRyzH1j5EgSvWcVHgCA.png" alt="" height="394" width="700"><figcaption><p>DMARC Record</p></figcaption></figure>

3\. Now if you get an email (<xyz@company.com>), then go for its exploitation

4\. Visit <https://emkei.cz/> in order to exploit the issue

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Sekf2RcCZPqGoL91eUkMLg.png" alt="" height="394" width="700"><figcaption><p>Fake Mailer</p></figcaption></figure>

6\. Here you’ll need to fill **From-email**, **To** and **Subject**

7\. In **From-email** add the company’s email and in **To** add your own email and in **Subject** you can add anything (like Hacking You, Bounty Time etc.)

8\. Check your inbox and you’ll get the email from that company which was sent by you

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Mj8cZ1qeO4LLXdXzq7M_3g.jpeg" alt="" height="1478" width="700"><figcaption><p>Email Spoofed</p></figcaption></figure>

### **Impact :**

Anyone can distribute fake email content/files using a company’s email. As a result, a company will have a reputation loss.
