# Symlink or Hard Link Following

**CWE-59: Improper Link Resolution Before File Access ('Link Following')**

**CWE-59** refers to a security vulnerability that occurs when an application resolves symbolic or hard links to files before properly validating the link’s target. This can allow attackers to control which file or directory the application accesses, potentially leading to unauthorized access to sensitive files, privilege escalation, or other types of attacks.

#### How It Works:

This vulnerability arises when a **program** opens a file path (which may be controlled by the user) and **fails to check whether the path refers to a symbolic link**. Instead, the program blindly follows the link and accesses the file it points to, without any validation. If an attacker can control or manipulate the symbolic link’s target, they can trick the program into accessing **sensitive files** or data, such as system files that should be protected.

***

### **Vulnerable Scenario** (Local and Remote)

**1. Local Attack ( which is likely ) :**

* **Vulnerable Application**: A local file viewer or editor that opens any file without validating whether it is a symlink.
* **Attack Steps**:
  1. The attacker creates a symbolic link (`symlink.txt`) that points to `/etc/passwd.`
  2. The attacker uploads the symbolic link to the server or places it within a directory accessible by the vulnerable application.
  3. The application opens the symbolic link as if it were a regular file, and it ends up reading `/etc/passwd`.
* The attacker gains unauthorized access to sensitive information, such as user credentials stored in `/etc/passwd`.

<figure><img src="/files/9eLONxsS8QguAfEFYdpI" alt=""><figcaption></figcaption></figure>

**2. Remote Attack (via HTTP Request):**

* **Vulnerable Web Application**: A file upload service where users can upload pictures for sharing, without checking for symbolic links.
* **Attack Steps**:
  1. The attacker uploads a symbolic link (e.g., `user_file.txt`) through the web interface. This symlink points to `/etc/passwd` on the server.
  2. The web application stores the file in the server's file storage system without validating whether it's a symlink.
  3. The attacker, knowing the file's location (e.g., `/uploads/user_file.txt`), requests the file via an HTTP request:

     ```http
     GET /uploads/user_file.txt HTTP/1.1
     Host: vulnerable-app.com
     ```
  4. The application, unaware that `user_file.txt` is a symlink, follows it and opens `/etc/passwd` instead of the intended file.
* The attacker can access sensitive system information through the web interface, such as user credentials stored in `/etc/passwd:`

```http
HTTP/1.1 200 OK
Date: Fri, 11 Jan 2025 13:45:00 GMT
Server: Apache/2.4.7 (Ubuntu)
Content-Type: text/plain; charset=UTF-8
Content-Length: 122

root:x:0:0:root:/root:/bin/bash
user:x:1001:1001:user:/home/user:/bin/bash
```

> **Note:** This scenario assumes the server's file system supports symbolic links and the upload mechanism does not sanitize or block symlinks. On some systems, uploading symlinks may require special permissions or may not be supported.

***

### **Vulnerable Application Example**

**Example:** a local web server that lets users upload files.

#### **Normal Behavior**

* You upload a file `photo.jpg` → server saves it as `/uploads/photo.jpg`
* Later, when you or others download it, the server reads `/uploads/photo.jpg` and returns it.

**No problem** here if it’s a real file.

#### **What happens if the app does NOT check for symlinks?**

* The app just opens `/uploads/photo.jpg` **without checking if it’s actually a shortcut**.
* If an attacker can somehow upload a **symlink** instead of a real file, bad things can happen.

**Attacker’s Trick**

Let’s say the attacker wants to steal `/etc/passwd`.

* Attacker uploads a **symlink** called `photo.jpg` that points to `/etc/passwd`.

Example:

```bash
ln -s /etc/passwd photo.jpg
```

Then they upload `photo.jpg` to the server.

#### **Server stores it**

If the server:

* Saves this symlink on the disk
* Does NOT sanitize it (doesn’t check if it’s a symlink)

then later, when *anyone* requests:

```http
GET /uploads/photo.jpg
```

The server says:

> “OK, let me open `/uploads/photo.jpg`...”\
> OS sees it’s a symlink → follows it → actually opens `/etc/passwd`!

The server then sends **/etc/passwd** content to the attacker!

#### **Result: Data Leakage**

Now the attacker sees:

```ruby
root:x:0:0:root:/root:/bin/bash
...
```

***

### **Impact and Exploitation**

As demonstrated, the vulnerable application opens `/uploads/user_file.txt` without checking whether it’s a symbolic link. Since `/uploads/user_file.txt` is a symlink to `/etc/passwd`, the application reads and prints the contents of `/etc/passwd`, exposing sensitive system information. This results in information disclosure and can potentially lead to privilege escalation or further attacks.

***

### **Mitigation Strategy**

To mitigate this vulnerability, always verify the target of symbolic links before accessing them. In Python, this can be done using `os.path.islink()` to detect symbolic links and reject or handle them securely.

Here’s a secure version of the code:

```python
import os

def process_file(file_path):
    """Secure function that checks for symbolic links before opening the file"""
    if os.path.islink(file_path):
        print("Error: Symbolic link detected. Aborting file access.")
        return
    
    print(f"Opening file: {file_path}")
    try:
        with open(file_path, 'r') as f:
            data = f.read()
            print(data)
    except Exception as e:
        print(f"Error: {e}")

# Simulated file upload path
uploaded_file = '/uploads/user_file.txt'
process_file(uploaded_file)
```

In this version, the program first checks whether the file is a symbolic link using `os.path.islink()`. If it is, the program aborts the operation and prevents accessing the target file, blocking exploitation.

***

### Resources

* [CWE-59: Improper Link Resolution Before File Access](https://cwe.mitre.org/data/definitions/59.html)
* [Python `os.path.islink()` Documentation](https://docs.python.org/3/library/os.path.html#os.path.islink)
* [OS Path Documentation for Python](https://docs.python.org/3/library/os.path.html#os.path.islink)
* [Latest vulnerabilities for CWE-59](https://www.cybersecurity-help.cz/vdb/cwe/59/)
