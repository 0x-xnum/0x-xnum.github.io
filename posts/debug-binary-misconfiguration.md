# Debug Binary Misconfiguration

Debug Binary Misconfiguration CWE-11 occurs when ASP.NET applications are configured to produce debug binaries deployed to production environments. These binaries give detailed debugging messages and expose sensitive information that should never be available in production.

When debug mode is enabled, the application reveals detailed error messages, stack traces, source code paths, variable values, and internal system information. What's meant for developers troubleshooting during development becomes a roadmap for attackers exploiting production systems.

***

### How It Works

#### Vulnerable Configuration

The web.config file contains the debug mode setting. Setting debug to "true" will let the browser display debugging information.

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.web>
    <compilation defaultLanguage="c#" debug="true"/>
  </system.web>
</configuration>
```

With `debug="true"`, ASP.NET compiles the application with debugging symbols included in the DLL files. When an error occurs, the framework outputs detailed information instead of generic error messages.

***

#### Scenario 1: SQL Injection Discovery via Debug Error

A user-submitted search query triggers a database error on a vulnerable page:

```
GET /search?q=test' OR '1'='1
```

**Without debug mode (secure):**

```
An error occurred processing your request. Please try again later.
```

**With debug mode (vulnerable):**

```
System.Data.SqlClient.SqlException: Incorrect syntax near the keyword 'OR'.

Stack Trace:
  at System.Data.SqlClient.SqlInternalConnection.OnError(SqlException exception, Boolean breakConnection, Action`1 closeAll)
  at [...]/DataAccess.cs:line 145
  at [...]/SearchController.cs:line 42
  
Query: SELECT * FROM products WHERE name = 'test' OR '1'='1' AND active = 1
Connection String: Server=db.internal;User Id=sa;Password=P@ssw0rd123
```

**The attack:** The attacker now knows:

* The exact SQL query structure
* The database user is `sa` (SQL Server admin)
* The password
* The internal database server address
* The vulnerable code is at DataAccess.cs:145

They can now craft targeted SQL injection attacks or directly access the database.

**Finding it:** Intentionally trigger errors with special characters, invalid input, or malformed requests. Check if detailed debugging information is displayed.

***

#### Scenario 2: Exposing Hardcoded Secrets in Stack Traces

A developer left a hardcoded API key in the code for testing:

```csharp
public class PaymentController : Controller {
    private const string StripeKey = "sk_REDACTED_live_4eC39HqLyjWDarht8Zlt5Kda";
    
    public ActionResult ProcessPayment(decimal amount) {
        try {
            var client = new StripeClient(StripeKey);
            // Process payment
        }
        catch (Exception ex) {
            // Log error - this includes full variable states
            Logger.LogError(ex.ToString());
        }
    }
}
```

When the application encounters an error, debug mode captures and displays all variable values in memory at the time of the exception, including `StripeKey`.

**The attack:** An attacker triggers an error and the API key appears in the debug output:

```
System.Exception: Payment processing failed
Message: Invalid amount: -100

Local Variables:
  amount: -100
  client: StripeClient { key: "sk_REDACTED_live_4eC39HqLyjWDarht8Zlt5Kda" }
  
Stack Trace:
  at PaymentController.ProcessPayment(Decimal amount)
```

The attacker can now use the Stripe API key to charge customers or capture payment information.

**Finding it:** Look for sensitive data in error messages and stack traces. Test with intentionally invalid inputs to trigger exceptions. Check if secrets appear in responses.

***

#### Scenario 3: Discovering Application Paths and Code Structure

A developer requests a non-existent page:

```http
GET /admin/secret-panel.aspx
```

**Debug response reveals:**

```
FileNotFoundException: File not found: C:\inetpub\wwwroot\admin\secret-panel.aspx

Stack Trace:
  at System.Web.UI.PageHandlerFactory.GetHandlerForExtensionlessURL()
  at C:\inetpub\wwwroot\App_Code\Authentication.cs:line 87
  
Files in directory:
  admin/
    backup-database.aspx
    user-management.aspx
    reset-passwords.aspx
    deleted-users.aspx
```

**The attack:** The attacker now has:

* The full file system path
* A directory listing of admin pages
* Source code locations
* Information about deleted functionality

They can directly request known pages like `/admin/user-management.aspx` or `/admin/backup-database.aspx`.

**Finding it:** Request non-existent pages. Check if file paths and directory structures are revealed. Look for listings of available pages or code files.

***

#### Scenario 4: Information Disclosure via Exception Details

A third-party API call fails:

```csharp
public ActionResult SyncWithExternalService() {
    var response = HttpClient.Get("https://api.partner.com/data");
    // If this throws an exception, debug mode exposes everything
}
```

**Debug response:**

```
System.Net.Http.HttpRequestException: The SSL certificate verification failed.

Inner Exception: Certificate validation error for api.partner.com
Thumbprint: A1B2C3D4E5F6...

Local Variables:
  response: HttpResponseMessage { StatusCode: 401 }
  apiUrl: "https://api.partner.com/data"
  authToken: "Bearer eyJhbGciOiJIUzI1NiIs..."
  
Connection Details:
  Proxy: corp-proxy.internal:8080
  User Agent: Our-Integration-Service/1.2.3
```

**The attack:** Information disclosure includes:

* Third-party API endpoints and authentication tokens
* Internal proxy addresses
* Certificate details
* Application version numbers
* Authentication schemes

The attacker can now target the third-party service or use the auth token to impersonate the application.

**Finding it:** Trigger requests to external services that fail. Monitor responses for internal infrastructure details and authentication credentials.

***

#### Scenario 5: DLL File Exposure

Debug binaries are compiled DLL files that include debugging symbols.

When debug is enabled, these DLL files are stored in the `bin\` directory:

```
/bin/Application.dll (with debug symbols)
/bin/DataAccess.dll (with debug symbols)
/bin/Authentication.dll (with debug symbols)
```

**The attack:** An attacker can:

1. Download the DLL files
2. Use decompilers (dnSpy, ILSpy) to extract source code
3. Understand the exact logic and vulnerabilities
4. Find hardcoded credentials in the decompiled code
5. Identify all methods and functions
6. Plan attacks based on complete code knowledge

A DLL with debug symbols includes metadata about every variable, function, and class, making decompilation trivial.

**Finding it:** Check if `/bin/` directory is accessible. Try downloading DLL files:

```bash
curl http://example.com/bin/Application.dll -o Application.dll

# Check if the DLL is large and contains debug symbols
# Debug DLLs are significantly larger than release builds
file Application.dll
```

***

### Mitigation Strategies

**Disable debug mode in production** Change the debug mode to false when the application is deployed into production.

```xml
<compilation defaultLanguage="c#" debug="false"/>
```

**Use web.config transformations** Use separate configurations for different environments:

```xml
<!-- web.Release.config -->
<compilation xdt:Transform="Replace" defaultLanguage="c#" debug="false"/>
```

**Implement centralized error handling** Never display raw exceptions to users. Log details server-side:

```csharp
try {
    // Code
}
catch (Exception ex) {
    Logger.LogError(ex); // Log server-side
    return View("Error"); // Generic error to user
}
```

**Disable directory browsing** Prevent access to directories:

```xml
<directoryBrowse enabled="false"/>
```

**Remove debug files from production** Ensure `.pdb` files (symbol files) and debug DLLs are not deployed:

```xml
<!-- Exclude .pdb files from deployment -->
```

**Implement custom error pages** Replace detailed errors with user-friendly messages:

```xml
<customErrors mode="On">
  <error statusCode="500" redirect="~/Error.aspx"/>
</customErrors>
```

**Remove sensitive information from code**

* Don't hardcode credentials
* Don't log sensitive data
* Use environment variables or secure vaults
* Never include secrets in error messages

**Audit web.config regularly**

* Check all production servers
* Verify debug="false"
* Review error handling configuration
* Monitor for suspicious changes

**Use automated scanning** Implement CI/CD checks that fail if debug mode is enabled in production builds.
