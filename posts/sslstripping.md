# SSLStripping \*\*

### What is SSLStripping? <a href="#id-10e1" id="id-10e1"></a>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/0*RFfKJ-LE2lGbw0tx" alt=""><figcaption></figcaption></figure>

MITM stands for man-in-the-middle attack or we can call it SSLStripping, and SSLStrip is a tool. The data can be effectively stolen because the connection is no longer encrypted when the attacker downgrades the website connection from HTTPS to HTTP. Stripped down to "HTTP," or in other terms, an attacker downgrades the connection to the website from HTTPS to HTTP. Attackers are able to retrieve sensitive information as soon as the connection quality declines, and they also have the ability to change that information.

Consider, for instance, a scenario in which an adversary is able to monitor and intercept the traffic with the use of MITM tools such as SSLstrip. On a public network, an attacker could employ an MITM attack to retrieve information on a legitimate user. This information could include the user's credit card details, password, or any number of other sensitive pieces of information. Attackers change the details of a legitimate user and then send him to a server under the attacker's control, where he can then coerce the user into performing undesired tasks.

Using SSLStrip an attacker can jeopardize the integrity and confidentiality of user information; in extreme cases, it can even allow hackers to collect personally identifiable information (PII), health information, and bank account details.

### How does it work? <a href="#id-98f0" id="id-98f0"></a>

When a legitimate user connects to a server — which is typically a secure connection — but an attacker downgrades the connection to plaintext because the legitimate user is connected to a wifi network controlled by the attacker, the attacker is able to intercept the traffic and act as a bridge between the legitimate user and the server. Now the attacker is able to access all of the information that is being transmitted between the User and the Server. As I previously stated, this may include sensitive information such as credit card information, usernames and passwords, and a great deal of other information.

For example: We have a website that works on https i.e. [https://test.com](https://test.com/) and if an attacker is able to degrade it to http then the website becomes [http://test.com](http://test.com/). So in that case user can easily continue to communicate with test.com but due to plaintext protocol attacker will be able to intercept each and every communication that is happening between them.

legit user ⇐ HTTP ⇒ Attacker ⇐ HTTPS ⇒ test.com

As soon as the SSLStripping attack starts, a legit user is communicating to the website on HTTP.

legit user ⇐ HTTP ⇒ Attacker ⇐ HTTP ⇒ test.com

If the application has a vulnerability known as SSLStripping, then an attacker can connect from a particular session while waiting for a response from the server if the application has this vulnerability. Throughout the entirety of this procedure, the legitimate user has no way of knowing whether or not he is connecting to a secure connection, and he also has no way of understanding whether or not the session is being intercepted.

### How can SSLStripping be exploited? <a href="#id-0af2" id="id-0af2"></a>

SSLStripping vulnerability can be exploited for all users who are going to use the attacker controlled Wifi or network. There are multiple ways to perform SSL Strip:

### Use of SSLStrip Tool <a href="#id-66af" id="id-66af"></a>

In Kali Linux we have a couple of tools which can be used to exploit the SSLstripping vulnerability such as Ettercap. These tools can hijack HTTP traffic from existing networks and watch for HTTPS URLs and then redirect them. After redirection it maps those links the same as HTTP links.

### **Ettercap and it's benefits:**

Ettercap is an open source tool that is available in Kali Linux. It can intercept the network traffic and also perform eavesdropping against some of the common protocols. If there is a need to maintain the connection, It can also insert the commands or characters between the network connections. This tools has some feature due to which it is more popular than others tools of SSLstriping, Let us discuss them in detail:

### **How to run it in Kali Linux:**

It requires very basic knowledge in order to run this and this tool is pre installed on kali Linus. Still if anyone wants to install it he can use this URL <https://github.com/Ettercap/ettercap.git>. If users want to gain more knowledge about this tool he can visit this URL <https://github.com/Ettercap/ettercap>. If Ettercap is already installed on Kali Linux then the user can use the below command, in order to execute it.

**Syntax : Ettercap -G** // where -G is used to select the user interface.

When we run the above command it will show the version of "Ettercap", which you can see in the below screenshot also.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/0*tt-9LskUjicsBUvr" alt=""><figcaption></figcaption></figure>

As soon as the user, insert this command. Ettercap prompt will open and it looks like as mentioned below:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/0*jfi1HRGfS-1Us_e2" alt=""><figcaption></figcaption></figure>

In order to extend its functionality users can add Plugins to it that will enhance its basic functionality.

### **Benefits of Ettercap**

Ettercap can perform both active and passive information gathering by using both TCP and UDP protocol. Whenever there is a need to capture gateway logs it is preferred as this tool is capable of MAC base filtering. Sometimes in case of ARP protocols users need to filter the traffic in full duplex mode, Ettercap can also do that. It is capable of creating connections in various environments such as VPN tunneling, SSH connections, and secure HTTP connections.

### By Using In-Secure Public WIFI: <a href="#id-5604" id="id-5604"></a>

Legit users who are connected through the insecure public WIFI, attacker can intercept their traffic by performing MITM attack and using tools such as SSLStrip or Ettercap.

Encryption functionality implemented only on login page

There are so many application developers who encrypt the password on the login page itself. They transmit the password after encrypting it.

### Remediation <a href="#id-1d8d" id="id-1d8d"></a>

Because switching a connection from HTTPS to HTTP might result in a vulnerability of a high severity, companies need to take the appropriate remediation measures in order to protect themselves from the threat posed by this vulnerability.

### Implementation of HTTP Strict Transport Security / HSTS <a href="#id-2fd1" id="id-2fd1"></a>

As soon as an attacker attempts to degrade it or change application from secure channel to insecure channel, application automatically redirects to secure channel or stops working, because after enabling or adding HSTS header on application it protects against downgrading the HTTPS connection to HTTP and then application can only work on secure channel.

How is it implemented?

With the use of a new something that in the web application known as "Strict-Transport-Security." Therefore, it is impossible for it to be possible as soon as the supported browser receives a connection request to downgrade. The owner of the application is responsible for setting the "max-age" attribute with this header. The admin can define the expiration time in the "max-age" field. As a result, web browsers are able to comprehend the fact that websites have to be accessed via HTTPS only during the allotted period of time.

### Enable Certificate pinning <a href="#d816" id="d816"></a>

Certificate pinning somehow reduces the attack surface of the SSLStripping vulnerability as by enabling the certificate pinning on websites or applications, we can specify the browser to accept connection only from these hosts where the SSL certificate conditions meet otherwise reject the rest of connections.

### Enabled secure cookie attribute <a href="#id-14fd" id="id-14fd"></a>

Using the secure attribute in the cookie, it can't be transmitted through unsecure channels so if the attacker has downgraded the connection to HTTP, the cookie will not be transmitted in the request and hence it will be secure from getting his account compromised.
