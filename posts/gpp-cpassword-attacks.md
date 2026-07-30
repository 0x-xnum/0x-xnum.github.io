# GPP / cPassword Attacks

**What Happened?**

* **Group Policy Preferences (GPP)** was a feature introduced in **Windows Server 2008** to simplify the management of local accounts and services across domain-joined machines.
* Admins could use GPP to:
  * Set passwords for local admin accounts.
  * Configure services and other tasks via **Group Policies**.

#### **What Made GPP Vulnerable?**

Here’s where things went wrong:

* GPP allowed **embedded credentials** (e.g., local admin passwords) to be set within XML files.
* These credentials were **encrypted** and stored as `cPassword` values in files like `Groups.xml`.
* The `Groups.xml` file resides in **SYSVOL** – a shared folder on every domain controller that **all authenticated users can access**

The encryption should have kept these passwords safe, right? Well…

**The OOPS Moment**

Microsoft made a critical mistake:

* The **AES encryption key** used to encrypt the cPassword was **hardcoded**.
* Worse yet, they accidentally published this encryption key in official **documentation**.
* Result? Anyone who gets access to the XML file can easily **decrypt the cPassword**.

<figure><img src="/files/15Bncyo9yazI0WSEvl1a" alt=""><figcaption></figcaption></figure>

<https://n1chr0x.medium.com/unwrapping-gpp-exposing-the-cpassword-attack-vector-using-active-htb-machine-4d3b97e0ac43>
