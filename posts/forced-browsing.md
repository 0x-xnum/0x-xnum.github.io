# Forced Browsing

## CWE-425: Unrestricted Access by Path Traversal

### What It Is

CWE-425 occurs when a web application protects resources through *obscurity* rather than actual access controls. The application assumes that if a URL isn't linked from visible pages, users won't discover it—a fundamentally flawed security strategy.

Attackers bypass this "hidden" protection easily by:

* Guessing common filenames and URL patterns
* Examining `robots.txt` and source code comments
* Analyzing backup files and configuration directories
* Running automated scanning tools

### Why It Matters

The core principle: **hidden is not secure**. Any resource accessible via a direct URL should be protected by explicit authentication and authorization checks, regardless of whether it's linked from anywhere.

### Real-World Examples

#### Example 1: Unlocked Admin Panel

A website hosts an admin panel at `/admin.html` but never links to it, assuming obscurity provides security:

```html
<nav>
  <a href="/">Home</a>
  <a href="/about">About</a>
  <a href="/contact">Contact</a>
</nav>
```

**The problem:** Any visitor can type `https://example.com/admin.html` directly into their browser and gain access. The missing link provides zero actual security—only proper authentication controls do.

#### Example 2: robots.txt as a Treasure Map

A web server uses `robots.txt` to prevent search engines from indexing sensitive files, inadvertently revealing their locations:

```http
User-agent: *
Disallow: /backup/
Disallow: /private-documents.html
Disallow: /internal-memo-2024.pdf
```

**The problem:** `robots.txt` is publicly readable and acts as a directory listing for attackers. Well-behaved crawlers respect it, but anyone can simply visit those URLs directly. This file should never be used as a security mechanism.

#### Example 3: Predictable File Naming

An application stores user documents with obvious, sequential filenames:

```python
def save_uploaded_file(file, user_id):
    filename = f"document_{user_id}.pdf"
    filepath = f"/var/www/public/uploads/{filename}"
    file.save(filepath)
    return filepath
```

**The problem:** Attackers systematically access other users' files by incrementing the ID: `document_1234.pdf`, `document_1235.pdf`, `document_1236.pdf`. No access controls verify file ownership.

#### Example 4: Forgotten Backup Files

A developer creates backup copies during maintenance but leaves them in the web directory:

```http
/app/login.php           (current version)
/app/login.php.bak       (backup)
/app/login.php~          (editor backup)
/app/login.php.old       (outdated version)
```

**The problem:** Backup files often contain outdated, vulnerable code, debugging output, or hardcoded credentials. Attackers scan for `.bak`, `.old`, `.backup`, `~`, and `.swp` extensions as a matter of routine.

### How to Fix It

**Don't rely on obscurity.** Implement these controls instead:

* Verify authentication and authorization on every protected resource, not just linked pages
* Store sensitive files (backups, configs, uploads) outside the web root entirely
* Implement consistent access control checks for all protected endpoints
* Never store predictable or sequential identifiers in URLs without validation
* Don't use `robots.txt`, `.htaccess`, or `sitemap.xml` as security mechanisms—they're metadata files, not barriers
* Remove development artifacts (backup files, `.DS_Store`, editor temps) from production
