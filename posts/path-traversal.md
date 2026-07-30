# Path Traversal

Path Traversal vulnerabilities, also known as Directory Traversal, allow attackers to manipulate file paths in order to access restricted files on a server. By exploiting this vulnerability, attackers can gain access to sensitive data, and potentially modify or delete critical files, leading to further exploitation, including full server compromise.

### Vulnerable Scenarios and Examples

### Relative Path

#### Understanding Canonicalization ".." :

Canonicalizing a path is the process of resolving the path to its simplest, absolute form by eliminating redundant elements such as `..` (parent directory) or symbolic links. Here are some examples:

* **Input**: `/var/www/project/../index.html`
  * **Canonicalized Path**: `/var/www/index.html`
* **Input**: `/usr/local/bin/../bin/ls`
  * **Canonicalized Path**: `/usr/local/bin/ls`

Let's say each user has their own directory which stores confidential data. To access the files, the user passes a path relative to this directory. It is obvious that other users' directories are nearby. Then, using the dot-dot-slash sequence ('..' or '../'), attackers may access the files of any user. They easily gain access to the `adminPasswords.txt` file, passing the following string as the path:

```url
../admin/adminPasswords.txt 
```

Note that Windows filenames are delimited by backslash (''). To prevent such an attack, it's not enough to check that the string does not start with '../'. Attackers can use the following string for malicious purposes:

```
myFolder/../../admin/adminPasswords.txt
```

At first, they access the `myFolder` directory and then the directory containing each user's data. Then, attackers access the `admin` directory and get the file.

These examples show a possible way to perform a relative path traversal attack. Note that dot-dot-slash sequences allow an attacker to gain access to any file or directory on the disk.

Your application must be secured so that a user could not access other directories. The easiest way to prevent an attack is to check strings for dot-dot-slash sequences. Unfortunately, that's not enough to ensure complete security.

#### Vulnerable Code Example

In this scenario, user input is directly concatenated with a base directory path without validation, leading to a basic path traversal vulnerability:

```javascript
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();

app.get('/vuln', (req, res) => {
  const filename = req.query.file;
  if (!filename) {
    res.status(400).send('You need a parameter called file.');
    return;
  }
  // Vulnerable file path construction
  const filePath = path.join(__dirname, filename);
  fs.readFile(filePath, 'utf8', (err, data) => {
    if (err) {
      res.status(500).send('Error reading file');
    } else {
      res.send(data);
    }
  });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

#### Attack Example

By accessing the endpoint `/vuln?file=../../etc/passwd`, an attacker can exploit the vulnerability to expose sensitive files like `/etc/passwd`.

#### Prevention Code

To mitigate Relative Path Traversal vulnerabilities, the following secure approach can be implemented:

```javascript
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();

app.get('/vuln', (req, res) => {
  const filename = req.query.file;
  if (!filename) {
    res.status(400).send('You need a parameter called file.');
    return;
  }

  // Sanitize the filename to prevent directory traversal
  const sanitizedFilename = path.basename(filename);
  const filePath = path.join(__dirname, sanitizedFilename);

  fs.readFile(filePath, 'utf8', (err, data) => {
    if (err) {
      res.status(500).send('Error reading file');
    } else {
      res.send(data);
    }
  });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

#### Key Prevention Measures:

1. **Path Sanitization:** By using `path.basename()`, only the filename is retained, discarding any path components that could lead to directory traversal.
2. **Secure Path Construction:** `path.join()` ensures that the constructed path is relative to the intended directory, eliminating the risk of accessing unauthorized files.

***

### Absolute Path Traversal

An absolute path traversal attack is easier to perform. Let's say we use the following JavaScript code to process a user's request:

```javascript
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();

app.get('/vuln', (req, res) => {
  let filePath = req.query.file;
  
  if (!filePath) {
    res.status(400).send('You need a parameter called file.');
    return;
  }
  if (filePath.includes('../')) {
    res.send('Hacker caught!');
    return;
  }

  // Absolute path handling
  filePath = path.resolve(__dirname, filePath);
  fs.readFile(filePath, 'utf8', (err, data) => {
    if (err) {
      res.send('Error reading file.');
    } else {
      res.send(data);
    }
  });
});

app.get('/', (req, res) => {
  res.send('Hi, the /vuln endpoint is what you want');
});

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});

```

In this example, although the application tries to block ../, it may still be vulnerable, You might be able to use an absolute path from the filesystem root, such as :&#x20;

```
filename=/etc/passwd
```

to directly reference a file without using any traversal sequences.

#### Prevention Code

To mitigate Relative Path Traversal vulnerabilities, the following secure approach can be implemented:

```javascript
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();

app.get('/vuln', (req, res) => {
  const filename = req.query.file;
  if (!filename) {
    res.status(400).send('You need a parameter called file.');
    return;
  }

  // Sanitize the filename to prevent directory traversal
  const sanitizedFilename = path.basename(filename); // Strip any path traversal characters (e.g., '..')
  
  // Securely construct the path relative to the current directory
  const resolvedPath = path.resolve(__dirname, sanitizedFilename);

  // Check if the resolved path is within the current directory
  if (!resolvedPath.startsWith(__dirname)) {
    return res.status(400).send('Invalid file path');
  }

  // Optional: You could implement a whitelist to limit accessible files
  const allowedFiles = ['file1.txt', 'file2.txt']; // Replace with your allowed files
  if (!allowedFiles.includes(sanitizedFilename)) {
    return res.status(403).send('Access to this file is not allowed');
  }

  // Check if the file exists before attempting to read it
  fs.exists(resolvedPath, (exists) => {
    if (!exists) {
      return res.status(404).send('File not found');
    }

    // Read the file if it exists
    fs.readFile(resolvedPath, 'utf8', (err, data) => {
      if (err) {
        res.status(500).send('Error reading file');
      } else {
        res.send(data);
      }
    });
  });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});

```

***

### Traversal Sequences Stripped Non-Recursivel

This version demonstrates how a server might attempt to mitigate path traversal attacks by removing traversal sequences like `../`, but only non-recursively, which means only the first occurrence will be stripped:

```javascript
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();

app.get('/vuln', (req, res) => {
  let filename = req.query.file;
  
  if (!filename) {
    res.status(400).send('You need a parameter called file.');
    return;
  }

  // Mitigate path traversal by removing traversal sequences once (non-recursively)
  filename = filename.replace(/\\.\\.\\//g, '');
  
  // Ensure file path is securely constructed
  const filePath = path.join(__dirname, filename);

  fs.readFile(filePath, 'utf8', (err, data) => {
    if (err) {
      if (err.code === 'ENOENT') {
        res.status(404).send('File not found');
      } else {
        res.status(500).send('Internal server error');
      }
    } else {
      res.send(`${data}`);
    }
  });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

While this approach removes one occurrence of a traversal sequence (`../`), attackers could craft input with multiple instances of the sequence, such as `....//....//.../`, which may still allow access to restricted files if the server doesn’t repeatedly sanitize or check for further sequences.

#### Example Attack:

An attacker might pass the filename as `....//....//etc/passwd`. While one `../` is stripped, the remaining traversal sequence (`....//....//`) could still resolve to a directory outside the intended path, allowing access to sensitive files like `/etc/passwd`

#### Solution: Recursive or Multiple Layer Sanitization

Instead of stripping traversal sequences just once, you could recursively sanitize the filename or employ more comprehensive validation checks. Additionally, regular expressions like the one in your code can be expanded to account for various other attack vectors.

***

### **File Path Traversal with URL Encoding or Double Encoding**

If a server strips directory traversal sequences like `../` but does so in a non-recursive and simplistic manner, attackers may exploit the URL encoding mechanism to bypass this defense.

For example:

* `../` can be URL-encoded as `%2e%2e%2f`.
* Double URL-encoded traversal like `..%c0%af` or `..%ef%bc%8f` could also bypass some basic sanitization mechanisms.

If the application only strips `../` but doesn't account for URL encoding or double encoding, attackers could craft a payload like:

* `../../../../../etc/passwd` encoded as `%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2fetc/passwd`.

***

### **Validation of Path Start**

Another defense is to check if the provided filename starts with a specific base folder (e.g., `/var/www/images`). However, this can be bypassed if attackers include the base folder followed by traversal sequences, such as:

* filename=/var/www/images/../../../etc/passwd.

Even though the filename might start with the correct base folder, the traversal sequences allow access to arbitrary paths outside of the intended directory.

The application would only check the start of the path, so the request would be accepted as valid, even though it leads outside the allowed directory.

```javascript
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();
// Define the base folder
// You can change it to your base folder
const baseFolder = '/home/kali/VulnerableLabs/PathTraversal/lab5';
// Vulnerable endpoint
app.get('/vuln', (req, res) => {
    // Get the filename parameter from the query string
    const filename = req.query.filename;
    
    if (!filename) {
        return res.status(400).send('Provide filename parameter \\n\\n your working directory is /home/kali/VulnerableLabs/PathTraversal/lab5');
    }
    // Check if the filename starts with the expected base folder
    if (!filename.startsWith(baseFolder)) {
        return res.status(400).send('Invalid filename');
    }
    // Construct the full path to the file
    const filePath = path.normalize(filename);
    // Read the file content
    fs.readFile(filePath, 'utf8', (err, data) => {
        if (err) {
            return res.status(500).send('Error reading file');
        }
        // Display the file content in the response
        res.send(`${data}`);
    });
});
app.get('/', (req, res) => {
  res.send('Hi to 5th Lab Path Traversal, the /vuln endpoint is what you want');
});
// Start the server
app.listen(3000, () => {
    console.log('Server is running on port 3000');
});
```

***

### **File Path Traversal with Validation of File Extension and Null Byte Bypass**

Some applications validate the file extension (e.g., `.png`) to restrict access to specific file types. However, attackers can bypass this restriction by injecting a null byte (`%00`) after the filename, effectively truncating the file path and allowing access to sensitive files.

**Example:**

* `filename=../../../etc/passwd%00.png`

The null byte (`%00`) terminates the string after `etc/passwd`, so the application will read the file as `../../../etc/passwd`, bypassing the `.png` extension check.

***

### How to prevent a path traversal attack <a href="#e442" id="e442"></a>

The most effective way to prevent path traversal vulnerabilities is to avoid passing user-supplied input to filesystem APIs altogether. Many application functions that do this can be rewritten to deliver the same behavior in a safer way.

If you can’t avoid passing user-supplied input to filesystem APIs, we recommend using two layers of defense to prevent attacks:

* **Validate User Input**:
  * Always validate user input before processing it. This can be achieved by allowing only expected values or characters in the input. A whitelist approach is ideal, where the input is compared against a predefined list of safe values. If a whitelist isn't feasible, use a regular expression to ensure the input contains only safe characters, such as alphanumeric characters.
* **Canonicalize the Path**:
  * After validation, append the input to the base directory and use the filesystem API to canonicalize the path. This step resolves any symbolic links, redundant separators, or references to parent directories (e.g., `..`). It ensures the resulting path is an absolute, unique representation that can't escape the intended directory structure.
* **Verify the Path**:

  * Verify that the canonicalized path starts with the expected base directory. This ensures that the final path resides within the intended directory and prevents unauthorized access to other locations.

  #### Node.js Example for Preventing Path Traversal:

  Below is a Node.js code example demonstrating how to prevent path traversal attacks using these principles:

  ```javascript
  const path = require('path');
  const fs = require('fs');

  // Function to validate user input
  function validateInput(userInput) {
      // Example validation: allow only alphanumeric characters
      const alphanumericRegex = /^[a-zA-Z0-9]+$/;
      return alphanumericRegex.test(userInput);
  }

  // Function to process user input and prevent path traversal
  function processInput(baseDir, userInput) {
      // Validate user input
      if (!validateInput(userInput)) {
          throw new Error('Invalid input');
      }

      // Normalize and sanitize user input
      const userInputSafe = path.normalize(userInput);
      const fullPath = path.join(baseDir, userInputSafe);

      // Canonicalize the path
      const canonicalPath = path.resolve(fullPath);

      // Ensure that the canonical path starts with the expected base directory
      if (!canonicalPath.startsWith(path.resolve(baseDir))) {
          throw new Error('Invalid path');
      }

      // Proceed with filesystem operation using the safe path
      fs.readFile(canonicalPath, (err, data) => {
          if (err) {
              console.error('Error reading file:', err);
          } else {
              console.log('File content:', data.toString());
          }
      });
  }

  // Example usage
  const baseDirectory = '/path/to/base/directory';
  const userInput = 'file.txt'; // User-supplied input
  processInput(baseDirectory, userInput);
  ```

in this code:

* **`validateInput`** function validates the user input. You can customize this function according to your requirements.
* **`processInput`** function processes the user input by appending it to the base directory, canonicalizing the path, and verifying that the canonical path starts with the expected base directory.
* If any validation or verification fails, an error is thrown, preventing a potential path traversal attack.
* Ensure to replace **`/path/to/base/directory`** with the actual base directory in your application.

Remember that this is a basic example. Depending on specific use case and requirements, you may need to enhance the validation and error handling logic. Additionally, consider implementing additional security measures such as access controls and permissions to further mitigate risks.

***

## &#x20; Lab 1 : File path traversal, simple case <a href="#id-3c0d" id="id-3c0d"></a>

Link to lab1: <https://portswigger.net/web-security/file-path-traversal/lab-simple>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*29CNFCg-PEr2Vf6S97GjKQ.png" alt="" height="376" width="700"><figcaption></figcaption></figure>

Aim: Read content of `/etc/passwd`

Click `Access the lab`

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*-z93bKWS0dt25_tqZDYqMg.png" alt="" height="394" width="700"><figcaption></figcaption></figure>

Website will launch on new tab where we will practice.

Then run burpsuite.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*V6WIlDlhMnh2GltMjkOisA.png" alt="" height="417" width="700"><figcaption></figcaption></figure>

In burpsuite under `HTTP history` , click on `filter` and select `images` and `css`

Then reload the lab.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*dDGbN_8AT7l5i2k3aCtBTw.png" alt="" height="394" width="700"><figcaption></figcaption></figure>

In the HTTP history, we can see `filename` parameter is loading various images, we will use one of these request , change the parameter and send the changed request to server.

Select any one of the request, right click and send it to repeater\
(or CTRL + R)

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Sw634RrXvCBKvuGfnJeHyg.png" alt="" height="424" width="700"><figcaption></figcaption></figure>

Our aim is to read the content of `/etc/passwd` ,but we don’t know the current location of the files `filename` parameter is fetching from.\
So we will use `../` which is used to move one directory up.

For example.\
If we are currently in `/var/www/images` folder then command `../` will move us one directory up into `/var/www/` and multiple `../` will move us multiple directory up.\
`../../../../../../../../../../../../` will move us to root directory.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*AaUALx9dTvJbeRTVr1LgIA.png" alt="" height="589" width="700"><figcaption></figcaption></figure>

`../../../../../etc/passwd` will move to 5 directory up and then go to `/etc/passwd` . Thus loading the content of the passwd file.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*3FIuHOTdrW7k_pZHICNmnQ.png" alt="" height="252" width="700"><figcaption></figcaption></figure>

Lab 1 completed successfully.

***

## Lab 2 : [File path traversal](https://portswigger.net/web-security/file-path-traversal), traversal sequences blocked with absolute path bypass <a href="#e56f" id="e56f"></a>

link to lab2: <https://portswigger.net/web-security/file-path-traversal/lab-absolute-path-bypass>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*MwtZtK42Mg8TgTlvVKTANw.png" alt="" height="463" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*RrukPTJ5VsKtF57U7UjB1Q.png" alt="" height="349" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*fVMFkWtlDA7UzCvuu0abtQ.png" alt="" height="450" width="700"><figcaption></figcaption></figure>

Same as before intercept the request and send it to repeater.\
For this lab traversal sequences is blocked but we can bypass this by providing absolute path.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/0*hR3WcXJVbjw2E8qk.png" alt="" height="438" width="700"><figcaption><p>Image explaining absolute path vs relative path.</p></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*AHHAbxIZMAX7RymRnZkBzg.png" alt="" height="376" width="700"><figcaption></figcaption></figure>

By providing absolute path `/etc/passwd` we can successfully complet the lab.

***

## Lab 3 : [File path traversal](https://portswigger.net/web-security/file-path-traversal), traversal sequences stripped non-recursively <a href="#id-1770" id="id-1770"></a>

link to lab: <https://portswigger.net/web-security/file-path-traversal/lab-sequences-stripped-non-recursively>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*ff_npV40zAVBEQ4Jmuwfzw.png" alt="" height="427" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*4h0gIUiLIAYx-bQcC8wXng.png" alt="" height="310" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*0GzQUs1x87cGUFzA3FxTGQ.png" alt="" height="337" width="700"><figcaption></figcaption></figure>

Same as before, send the interesting request to repeater.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*t5EFjFkXnySAwpZPLI9rJw.png" alt="" height="221" width="700"><figcaption></figcaption></figure>

In this lab , there is some kind of filter/ sanitization which is stripping or removing the traversal sequence.

<figure><img src="https://miro.medium.com/v2/resize:fit:460/1*cTW6gO5jzjA1uI4icJYGUA.png" alt="" height="280" width="460"><figcaption><p>stripping traversal sequence <code>../</code></p></figcaption></figure>

We can bypass this stripping easily by:

<figure><img src="https://miro.medium.com/v2/resize:fit:510/1*R6kLc-psHFfsrQAFIWH3Uw.png" alt="" height="310" width="510"><figcaption></figcaption></figure>

by adding extra `../` in between the payload we can bypass this filter.\
Note this will not work in recursive stripping process.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*5WxC2rDPKPyo_oB8q7jMug.png" alt="" height="511" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*QFuXuTJRhQz4Qyv8Y-hyJw.png" alt="" height="245" width="700"><figcaption></figcaption></figure>

***

## Lab 4 : [File path traversal](https://portswigger.net/web-security/file-path-traversal), traversal sequences stripped with superfluous URL-decode <a href="#c85f" id="c85f"></a>

link to lab 4: <https://portswigger.net/web-security/file-path-traversal/lab-superfluous-url-decode>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Ft2_dgp-V_Tf35SSzy9EoA.png" alt="" height="465" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*aoVbC1qhrBi2ACBY3VtgJQ.png" alt="" height="513" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*45i4Lstc0mzQbaZ44TEYdg.png" alt="" height="406" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*WTsg9dZz2_Grt-4L8h9t4g.png" alt="" height="502" width="700"><figcaption></figcaption></figure>

Previous method not working.\
We can come around this time by url encoding the payload .

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*QOqE9seb53yc_n-g_RmWFg.png" alt="" height="531" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*WNJpi1F1gAXUEYGefvT6bA.png" alt="" height="533" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*pBBp-e9K4TGOYUHxBdWcLg.png" alt="" height="423" width="700"><figcaption></figcaption></figure>

By double url encoding `../../../../../../etc/passwd` we can complete the lab.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*9lyCvV1ItGqpNERtKpCDGQ.png" alt="" height="260" width="700"><figcaption></figcaption></figure>

***

## Lab 5 : [File path traversal](https://portswigger.net/web-security/file-path-traversal), validation of start of path <a href="#id-2bf8" id="id-2bf8"></a>

link to lab5: <https://portswigger.net/web-security/file-path-traversal/lab-validate-start-of-path>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*nMI136jSWDHM_V_-of2hnw.png" alt="" height="438" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*lkuoEmDRpiPN_c7RzT72kQ.png" alt="" height="454" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*nD7lAHHpTU2JP5661j3EJg.png" alt="" height="292" width="700"><figcaption></figcaption></figure>

An application may require the user-supplied filename to start with the expected base folder, such as `/var/www/images`. In this case, it might be possible to include the required base folder followed by suitable traversal sequences. For example:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*dYf0MCf_NwqwDvf5D7DwEg.png" alt="" height="419" width="700"><figcaption></figcaption></figure>

***

## Lab 6 : [File path traversal](https://portswigger.net/web-security/file-path-traversal), validation of file extension with null byte bypass <a href="#b85f" id="b85f"></a>

link to lab: <https://portswigger.net/web-security/file-path-traversal/lab-validate-file-extension-null-byte-bypass>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*oFOOB6GXAryVWBlQURlqTQ.png" alt="" height="446" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*ly6k1ASXGpKTCKp-PyP1gA.png" alt="" height="403" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*rEokqLjuYc_6Zco52Ssw4g.png" alt="" height="261" width="700"><figcaption></figcaption></figure>

An application may require the user-supplied filename to end with an expected file extension, such as `.png`. In this case, it might be possible to use a null byte to effectively terminate the file path before the required extension. For example:

`filename=../../../etc/passwd%00.png`\
everything after null byte %00 is ignored while fetching the file.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*vXQtouj3Fei8XFDxIfYX7w.png" alt="" height="427" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*qtHMBwxsyq4AKEJ9G1k4SA.png" alt="" height="247" width="700"><figcaption></figcaption></figure>

Which completes the lab.
