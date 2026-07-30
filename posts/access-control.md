# Access Control

### Lab 1: Unprotected Admin Panel <a href="#lab-1-unprotected-admin-panel" id="lab-1-unprotected-admin-panel"></a>

This website has an unprotected admin panel. Visiting `/robots.txt` shows me this:

```http
User-agent: *
Disallow: /administrator-panel
```

I can access this and delete the `carlos` user, which solves the lab.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-bee6a5dbcbfa48a5c2339422b5dee41023c22e06%252Fportswigger-access-writeup-image.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=99d70e2e&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 2: Unpredictable URL <a href="#lab-2-unpredictable-url" id="lab-2-unpredictable-url"></a>

Same thing as Lab 1, just that the panel is located at a random directory.

Checking the page source reveals this randomised directory:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-066102b3ebeb764a38b138c956611230f4e636ac%252Fportswigger-access-writeup-image-1.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=dade3d02&#x26;sv=2" alt=""><figcaption></figcaption></figure>

It's still unprotected, so deleting `carlos` is trivial.

### Lab 3: User role controlled by request parameter <a href="#lab-3-user-role-controlled-by-request-parameter" id="lab-3-user-role-controlled-by-request-parameter"></a>

Logging in, I see that there is an `Admin` cookie set to `false`.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-1d4c69b8f8e2a80c69697d621e1abae50b5c997d%252Fportswigger-access-writeup-image-2.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=bc9288a3&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Changing this to `true` allows me to view the `/admin` panel.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-4e9502999b9a2117599beb396cfd4fcbe0dd8480%252Fportswigger-access-writeup-image-3.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=8c2c5e33&#x26;sv=2" alt=""><figcaption></figcaption></figure>

To solve the lab, just send a GET request to `/admin/delete?username=carlos`.

### Lab 4: Modifiable User Role <a href="#lab-4-modifiable-user-role" id="lab-4-modifiable-user-role"></a>

Loggin in, I see that there is an update email function. When testing it, I can see that a `roleid` parameter is returned in the response:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-2c98ce46c0c7555e27c4c94f71298b2e8c4718fe%252Fportswigger-access-writeup-image-4.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=cf26d&#x26;sv=2" alt=""><figcaption></figcaption></figure>

The POST request sends a JSON object with the `email` parameter. I added a `roleid` parameter and it worked:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-239c90bc85c557dbb5713c8493993dd98d0e7335%252Fportswigger-access-writeup-image-5.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=a7ece8c5&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This allows me to access the `/admin` panel to delete `carlos`.

### Lab 5: User ID controlled by request <a href="#lab-5-user-id-controlled-by-request" id="lab-5-user-id-controlled-by-request"></a>

This lab requires horizontal privilege escalation, and I have to obtain the API key of `carlos`.

Logging in, I see that the `/my-account` directory uses an `id` parameter with the username:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-c4f613e935c965e9e692b063acf81a3018e09b77%252Fportswigger-access-writeup-image-6.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=7251266b&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Changing `wiener` to `carlos` allows me to view his account:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-087795d5f86d9d95267b9d11acaa70537a26cde1%252Fportswigger-access-writeup-image-7.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=c204b4d5&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 6: Unpredictable User ID <a href="#lab-6-unpredictable-user-id" id="lab-6-unpredictable-user-id"></a>

Same as Lab 5, just that the ID isn't as predictable. When viewing blog posts, I can see a few by `carlos`:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-268ca11ab36d17b01032a0c2b6abb62f8834cb6c%252Fportswigger-access-writeup-image-8.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=4808edc4&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I noticed that the blogs posted by `carlos` contain a different user ID in the URL:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-482679ec5ec751a1f1957a9b6d348084ff9f1211%252Fportswigger-access-writeup-image-9.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=83a4462a&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Visiting `/my-account?id=213bd623-d8a5-4374-8161-fc52861f3bcc` would show us the API key.

### Lab 7: Data Leakage in Redirect <a href="#lab-7-data-leakage-in-redirect" id="lab-7-data-leakage-in-redirect"></a>

This lab uses a plain username as the `id` parameter. When visiting `carlos`, it attempts to redirect me to the login page:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-75bcb4747ebe0b2e74b4aaaadaae451a9026c57c%252Fportswigger-access-writeup-image-10.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=4bda31cd&#x26;sv=2" alt=""><figcaption></figcaption></figure>

However, this does not change the fact that **the user profile is still loaded**.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-884356a1b39981e1781ca44ce6832835ad82cde2%252Fportswigger-access-writeup-image-11.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=3b4dd8ae&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 8: Password Disclosure <a href="#lab-8-password-disclosure" id="lab-8-password-disclosure"></a>

Logging in, I can see that there's a password change feature, with the user's password already typed in:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-62963ed178087e4e61439ec2036a84af402e53fb%252Fportswigger-access-writeup-image-12.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=802760a6&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This value is present in the page source:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-1d8f29bab4d3c13d804af26de9f2670c61c4666f%252Fportswigger-access-writeup-image-13.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=1fb26198&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Visiting `carlos` profile via changing the `id` parameter results in his password being displayed:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-abc29a27726c9b228466545b151de330462d7175%252Fportswigger-access-writeup-image-14.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=3b475637&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Change the `id` to `administrator`, visit their profile and read the password.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-018adff7ce1c245df3ad5952f5c5db9357d547fa%252Fportswigger-access-writeup-image-15.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=65d8e179&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Then, login and delete `carlos`.

### Lab 9: Insecure Direct Object References (IDOR) <a href="#lab-9-insecure-direct-object-references-idor" id="lab-9-insecure-direct-object-references-idor"></a>

This lab has a live chat feature:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-e948be52ac8e44f3a2e7355b4431a910619ba50f%252Fportswigger-access-writeup-image-16.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=b1e817ba&#x26;sv=2" alt=""><figcaption></figcaption></figure>

When using the 'View transcript' function, I can see that `2.txt` is downloaded:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-4752341812dca781ac2ad818481a791a45f8a18f%252Fportswigger-access-writeup-image-17.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=b24ceec&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Changing this to `1.txt` shows me the password of `carlos`:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-5c5b0387671ce99794e8ab76827305275cf0f9c5%252Fportswigger-access-writeup-image-18.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=e15b5eda&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 10: URL-Based Control Bypass <a href="#lab-10-url-based-control-bypass" id="lab-10-url-based-control-bypass"></a>

Attempting to visit the unauthenticated admin panel results in an 'Access denied' being returned:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-4bb3efedc23d1a569586a080dd57c45ef58bf56c%252Fportswigger-access-writeup-image-19.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=556d931&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This lab supports using the `X-Original-URL` header. This header overrides the target URL in requests with the one specified in this header value.

This means that although I cannot visit `/admin` directly, I can do so via using `X-Original-URL`:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-4e057fe1358f0c3373db2d6a6b7f9986dc69f31f%252Fportswigger-access-writeup-image-20.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=ab90d97a&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I can then solve the lab by specifying `/admin/delete` within this header, which would override the target URL of `/`. The `username` parameter can be written in the target URL.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-2de6570c1d0709769b306ef82322d43ebe0b3288%252Fportswigger-access-writeup-image-21.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=ed230ae3&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 11: Method-Based Control Bypass <a href="#lab-11-method-based-control-bypass" id="lab-11-method-based-control-bypass"></a>

To solve this lab, make `wiener` an administrator. This lab also gives us admin access straightaway.

The admin panel has a function to change user privileges.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-0f7f382cd0f2669d448f2ea89e9fe91740bf8117%252Fportswigger-access-writeup-image-22.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=9deea5d4&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This thing sends a POST request to `/admin-roles` with a `username` and `action` parameter. `action` is either set to 'upgrade' or 'downgrade'.

When I logged in as `wiener` and attempted to send the same POST request, I got a 403 Unauthorized.

Changing it to a GET request solves the lab:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-008322247bd2a6ee19aa1708fe0023e57c9bac40%252Fportswigger-access-writeup-image-23.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=da369b79&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 12: Multi-step process with broken control on one step <a href="#lab-12-multi-step-process-with-broken-control-on-one-step" id="lab-12-multi-step-process-with-broken-control-on-one-step"></a>

This lab highlights the fact that it just takes 1 mistake for access control to be completely broken. To solve, make `wiener` the admin from a unprivileged account.

I upgraded `carlos` to administrator, and there was a confirmation page:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-2cf776e1dcf4ae7220ea892806289410f5806a56%252Fportswigger-access-writeup-image-24.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=5692f2d5&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I noticed that replaying this request with the session cookie of `wiener` did not work. However, sending a 'Yes' confirmation (by including `confirmation=true` within the POST parameters) bypasses the authorisation check:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-ce2cef0d82acf67f047815794d6f60d1089dce23%252Fportswigger-access-writeup-image-25.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=83f9116c&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 13: Referer-Based Control <a href="#lab-13-referer-based-control" id="lab-13-referer-based-control"></a>

This lab controls access based on the `Referer` header. When vistiing the admin panel, I noticed the header:

```http
Referer: https://LAB.web-security-academy.net/my-account?id=administrator
```

To solve, just send the GET request used to upgrade a user to admin with the `Referer` header set:

```http
GET /admin-roles?username=wiener&action=upgrade HTTP/2
Host: 0a10007a037b05738391a6ec002d0096.web-security-academy.net
Cookie: session=c3YbUWZ8P3UCEY84ZcYT4DwklMlVcDoa
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a10007a037b05738391a6ec002d0096.web-security-academy.net/admin
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
```

Remember to change the `Cookie` value to the `wiener` cookie.
