# Sensitive Data in Error Messages and Debug Code

Information disclosure through error messages occurs when applications expose sensitive information in error pages, logs, debug output, or exception messages. This includes:

Error messages are meant to help developers debug, but in production they become a roadmap for attackers. Stack traces, database queries, file paths, and configuration details reveal the application's architecture and vulnerabilitis.

***

### Real-World Attack Scenarios

#### Scenario 1: Database Error Revealing SQL Injection Points

A user searches products with a single quote:

```
GET /search?query=test'
```

**The vulnerable error response:**

```
Error: You have an error in your SQL syntax; check the manual that corresponds 
to your MySQL server version for the right syntax to use near 
"'test''" at line 1

SQL Query: SELECT * FROM products WHERE name LIKE '%test'%' AND active = 1
```

**What the attacker learns:**

1. Database is MySQL (version from error)
2. Exact SQL query structure: `SELECT * FROM products WHERE name LIKE '%[INPUT]%' AND active = 1`
3. SQL injection is possible (query error shows uncaught exception)
4. There's an `active` field (logic discovery)
5. Table name and column names (`products`, `name`, `active`)

**The attack:**

With this information, attacker crafts SQL injection:

```
/search?query=test' OR '1'='1
/search?query=test' UNION SELECT username, password FROM users--
/search?query=test'; DROP TABLE products;--
```

**Result:**

* Complete SQL injection exploitation
* Database compromise
* Data theft or destruction

**The fix:**

Never show SQL errors to users:

```php
<?php
try {
    $result = $pdo->query("SELECT * FROM products WHERE name LIKE ?", [$query]);
} catch (PDOException $e) {
    // Log error server-side
    error_log($e->getMessage());
    
    // Show generic message to user
    http_response_code(500);
    echo "An error occurred. Please try again.";
}
?>
```

**Finding it:** Submit invalid input (single quotes, special characters). Check error messages for SQL syntax errors, table/column names, query structure.

**Exploit:**

```bash
# Trigger error with SQL injection payload
curl "http://target.com/search?query=test'"

# If error shows SQL query, vulnerability confirmed
```

***

#### Scenario 2: Stack Trace Revealing File Paths and Code

Application has an unhandled exception:

```http
POST /api/user/profile
Content-Type: application/json

{"user_id": "abc"}  // Invalid type
```

**The verbose error response:**

```
Fatal error: Uncaught TypeError: Argument 1 passed to getUserProfile() 
must be of the type int, string given, called in 
/var/www/html/api/user.php on line 42 and defined in 
/var/www/html/src/User/Manager.php:28 Stack trace:
#0 /var/www/html/api/user.php(42): getUserProfile('abc')
#1 /var/www/html/api/router.php(156): User\API->handleRequest()
#2 /var/www/html/index.php(12): Router->dispatch()
#3 {main}

thrown in /var/www/html/src/User/Manager.php on line 28
```

**What the attacker learns:**

1. **Web root is `/var/www/html`** (server architecture)
2. **File structure**:
   * `/api/user.php` - handles user API
   * `/src/User/Manager.php` - user management logic
   * `/api/router.php` - routing logic
3. **Code flow**: How requests are routed and processed
4. **Function names**: `handleRequest()`, `getUserProfile()` (API endpoints)
5. **PHP version and framework** (error format)
6. **Exact line numbers** where errors occur

**The attack:**

With this knowledge, attacker:

1. Enumerates file structure
2. Tests each API endpoint knowing exact code paths
3. Identifies vulnerable functions
4. Finds bypass opportunities in error handling

**Result:**

* Complete application architecture exposed
* Targeted exploitation possible
* Privilege escalation paths identified

**The fix:**

Catch all exceptions and return generic errors:

```php
<?php
try {
    $user = getUserProfile($_POST['user_id']);
    echo json_encode($user);
} catch (Exception $e) {
    // Log error with full details
    error_log($e->getMessage() . "\n" . $e->getTraceAsString());
    
    // Return generic error to user
    http_response_code(400);
    echo json_encode(['error' => 'Invalid request']);
}
?>
```

**Finding it:** Trigger exceptions with invalid input. Check error messages for stack traces, file paths, function names.

***

#### Scenario 3: Debugging Code Left in Production

Application has debugging statements that reveal sensitive data:

```javascript
// Debugging code left in production
app.get('/api/login', (req, res) => {
    const user = authenticateUser(req.body.email, req.body.password);
    
    // DEBUG: Console logging (visible in server logs)
    console.log(`Login attempt for: ${req.body.email}`);
    console.log(`Password hash: ${user.password_hash}`);
    console.log(`API Key: ${user.api_key}`);
    console.log(`Database connection: ${process.env.DB_URL}`);
    
    if (user) {
        res.json({user: user});  // DEBUG: Returns full user object
    }
});
```

**What's leaked:**

1. **Emails** - User enumeration (who has accounts)
2. **Password hashes** - Can be cracked offline
3. **API keys** - Direct service access
4. **Database credentials** - Database compromise
5. **Full user objects** - All user data exposed

**The attack:**

Attacker:

1. Captures API response with full user object
2. Gets user's API key
3. Uses API key to access other services
4. Accesses password hash, runs offline crack
5. Logs in with real password

**Result:**

* Complete user account compromise
* Access to all services
* Database and API compromise

**The fix:**

Remove all debugging code in production:

```javascript
app.get('/api/login', (req, res) => {
    const user = authenticateUser(req.body.email, req.body.password);
    
    if (user) {
        // Return only necessary fields
        res.json({
            token: generateToken(user.id),
            name: user.name
            // Don't return: password_hash, api_key, etc.
        });
    } else {
        res.status(401).json({error: 'Invalid credentials'});
    }
});
```

**Finding it:** Check response bodies for unnecessary data. Search code for `console.log()`, `println()`, `print_r()`. Review debug variables in output.

***

#### Scenario 4: Version Information in Error Pages

Application exposes framework and library versions:

```http
HTTP/1.1 500 Internal Server Error
Server: Apache/2.4.41 (Ubuntu)
X-Powered-By: PHP/7.4.3
X-AspNet-Version: 4.0.30319
Content-Type: text/html

Laravel Framework 8.12.0 Error:
Class 'App\Models\InvalidClass' not found
```

**What the attacker learns:**

1. **Apache 2.4.41** - Version with known CVEs
2. **PHP 7.4.3** - Version with known vulnerabilities
3. **Laravel 8.12.0** - Framework version with known exploits
4. **Application structure** - Uses Models (Laravel MVC)

**The attack:**

Attacker:

1. Looks up CVE-2020-xxxx for Apache 2.4.41
2. Finds public exploit for PHP 7.4.3
3. Searches Exploit-DB for Laravel 8.12.0
4. Finds remote code execution vulnerability
5. Exploits application directly

**Result:**

* Remote code execution
* Server compromise
* Application takeover

**The fix:**

Hide version information:

```php
# Apache httpd.conf
ServerTokens Prod
ServerSignature Off

# PHP php.ini
expose_php = Off

# Custom error pages (don't show version)
<ErrorDocument 500 /errors/500.html>
```

**Finding it:** Check HTTP headers. Look at error pages. Search for version strings in responses.

**Exploit:**

```bash
# Check headers for version info
curl -I http://target.com

# Check error pages
curl http://target.com/nonexistent.php

# Look for version strings in HTML
curl http://target.com | grep -i "version\|powered"
```

***

#### Scenario 5: No Custom Error Page (Information Leakage)

Application displays default server error pages:

```
IIS default error page shows:
- IIS version
- .NET Framework version
- Module that caused error
- Source file path
- Error code and description
```

**What the attacker learns:**

* Exact server configuration
* Installed modules
* File paths
* Known vulnerabilities for that configuration

**The fix:**

Create custom error pages:

```html
<!-- /errors/500.html -->
<!DOCTYPE html>
<html>
<head><title>Error</title></head>
<body>
    <h1>Something Went Wrong</h1>
    <p>We're sorry, an error occurred. Our team has been notified.</p>
    <p>Error reference: [UNIQUE_ID]</p>
</body>
</html>
```

Configure in web server:

```
# Apache .htaccess
ErrorDocument 500 /errors/500.html
ErrorDocument 404 /errors/404.html
ErrorDocument 403 /errors/403.html
```

**Finding it:** Trigger errors and check if default error pages shown instead of custom pages.

***

#### Scenario 6: Sensitive Information in Debug Comments

Developer leaves credentials or sensitive info in comments:

```python
def authenticate():
    # DEBUG: Test account for debugging
    # Username: admin, Password: AdminDebug123!
    
    if username == 'admin' and password == 'AdminDebug123!':
        return True
    
    # DB connection string (left from debugging)
    # postgresql://user:SecurePass456@db.internal.company.com:5432/prod_db
    
    return check_database(username, password)
```

**What the attacker learns:**

1. **Debug credentials**: `admin / AdminDebug123!`
2. **Database credentials**: User and password
3. **Database server**: `db.internal.company.com`
4. **Database name**: `prod_db`
5. **Port**: `5432`

**The attack:**

Attacker:

1. Uses debug credentials to log in as admin
2. Or uses database credentials to connect directly to PostgreSQL
3. Dumps entire database
4. Complete compromise

**Result:**

* Admin access
* Database access
* Complete data breach

**The fix:**

Never leave credentials in comments:

```python
def authenticate():
    # Authenticate user against database
    return check_database(username, password)

# Use environment variables for sensitive data
DB_URL = os.getenv('DATABASE_URL')
```

**Finding it:** Search source code for comments containing: `password`, `key`, `secret`, `TODO`, `FIXME`, `DEBUG`, `HACK`.

**Exploit:**

```bash
# Search source code in git
git log -p | grep -i "password\|secret\|key"

# Or search current code
grep -r "password\|TODO\|FIXME\|DEBUG" src/
```

***

### How to Identify Information Disclosure During Testing

**1. Trigger errors intentionally**

```bash
# SQL syntax errors
curl "http://target.com/search?q=test'"

# Invalid parameters
curl "http://target.com/api/user?id=abc"

# File not found
curl "http://target.com/nonexistent"

# Permission denied
curl "http://target.com/admin" -H "Authorization: Bearer invalid"
```

**2. Check error messages for:**

* SQL syntax and queries
* File paths and directory structure
* Stack traces with function names
* Variable names and values
* Library/framework versions
* Database information
* API endpoints
* Server configuration

**3. Review HTTP headers**

```bash
curl -I http://target.com

# Look for:
# Server: [version]
# X-Powered-By: [version]
# X-AspNet-Version: [version]
```

**4. Examine source code**

```bash
# Search for debug code
grep -r "console.log\|println\|print_r\|var_dump" src/

# Search for credentials in comments
grep -r "password\|api_key\|secret\|TODO" src/

# Check for sensitive data in responses
grep -r "password\|credit_card\|ssn" src/
```

**5. Check for custom error pages**

* Do 404 errors show custom page?
* Do 500 errors show custom page?
* Or do they show server defaults?

**6. Review configuration files**

* Are debug flags enabled in production?
* Are error logs verbose?
* Is sensitive data logged?

***

### Mitigation Strategies

**Create custom error pages**

```html
<!-- Generic, no technical details -->
<h1>Error</h1>
<p>Something went wrong. Please try again.</p>
```

**Log errors server-side, not to user**

```python
try:
    # Code
except Exception as e:
    # Log everything
    logger.error(f"Error: {e}\n{traceback.format_exc()}")
    
    # Show generic message
    return "An error occurred"
```

**Remove debug code in production**

```python
if DEBUG:  # DEBUG only enabled locally
    print(sensitive_data)
```

**Hide version information**

```
ServerTokens Prod
ServerSignature Off
expose_php = Off
```

**Sanitize error messages**

```python
except SQLError as e:
    # Don't show: Error in query "SELECT * FROM users..."
    # Show: "Database error occurred"
    return "A database error occurred"
```

**Never log sensitive data**

```python
# BAD
logger.info(f"Login: {username}, password: {password}")

# GOOD
logger.info(f"Login attempt for: {username}")
```

**Remove credentials from code**

* Use environment variables
* Use secrets management
* Use configuration files (not in git)
* Use credential providers

**Code review**

* Review all error handling
* Check for debug statements
* Verify no credentials in code
* Ensure custom error pages used

***

&#123;% embed url="<https://cwe.mitre.org/data/definitions/209.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/215.htmlhttps://cwe.mitre.org/data/definitions/550.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/756.html>" %}

&#123;% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html>" %}

&#123;% embed url="<https://owasp.org/www-community/Information_Disclosure>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/209.html>" %}

&#123;% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html>" %}
