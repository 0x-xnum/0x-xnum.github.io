# Cryptographic Failure

To better understand **Cryptographic Failures**, I will walk you through the **Cryptographic Failures module** in TryHackMe’s OWASP Top 10 room. In this scenario, we will analyze a vulnerable **"Sense and Sensitivity"** website.

<figure><img src="https://miro.medium.com/v2/resize:fit:770/1*hukT3T6OUb2N-eCqbRDHJA.png" alt="" height="302" width="700"><figcaption></figcaption></figure>

### **Step 1: Navigate through the login page.**

<figure><img src="https://miro.medium.com/v2/resize:fit:770/1*l9f4Mcq9xQ6ftwGcsMR6SA.png" alt="" height="111" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:770/1*pmjtDJSbiPIDLCSp6VqNnA.png" alt="" height="199" width="700"><figcaption></figcaption></figure>

View the source code of the login page.

<figure><img src="https://miro.medium.com/v2/resize:fit:770/1*_CI4j5PsihxbO9U4OR9o-Q.png" alt="" height="410" width="700"><figcaption></figcaption></figure>

> The developer has **left a comment** revealing that the database is stored in the `/assets` directory.

### **Step 2: Navigate to /assets.**

<figure><img src="https://miro.medium.com/v2/resize:fit:685/1*IrQrHJYQJnd6Ga_8nSt2yQ.png" alt="" height="420" width="623"><figcaption></figcaption></figure>

> There is, in fact, a web application database (webapp.db) stored in the /assets directory. Databases stored as files are known as ‘flat-file’ databases.

### **Step 3: Inspect the database file.**

Click on the file to download it.

<figure><img src="https://miro.medium.com/v2/resize:fit:575/1*VgC-aY0rQwfpSwRJb8lNvw.png" alt="" height="118" width="523"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:770/1*a8A5JdmPYla2oqNbv4KgBQ.png" alt="" height="59" width="700"><figcaption></figcaption></figure>

Determine the file type.

```bash
file webapp.db
```

<figure><img src="https://miro.medium.com/v2/resize:fit:766/1*mG1QsgoZswNQf2y_K7hKlA.png" alt="" height="46" width="696"><figcaption></figcaption></figure>

> The database is a SQlite database

Dump the database

```bash
sqlite3 webapp.db
```

<figure><img src="https://miro.medium.com/v2/resize:fit:541/1*Pic7K3ZthCtK8Gk6pt9_jQ.png" alt="" height="61" width="492"><figcaption></figcaption></figure>

Dump the tables.

```sql
.tables
```

<figure><img src="https://miro.medium.com/v2/resize:fit:297/1*4dddkW1jcdDKWa5izK5xyA.png" alt="" height="36" width="270"><figcaption></figcaption></figure>

Dump users.

```sql
PRAGMA table_info(users);
SELECT * FROM users;
```

<figure><img src="https://miro.medium.com/v2/resize:fit:425/1*hSd4QFNto_G18rK1GtOeOQ.png" alt="" height="118" width="386"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:770/1*WMVo5A1hr5hbztVZBQVoug.png" alt="" height="97" width="700"><figcaption></figcaption></figure>

> We now have access to usernames and password hashes. Following the order, the second column contains the usernames, and the third column contains the password hashes.

```
Username:admin 
Password Hash:6eea9b7ef19179a06954edd0f6c05ceb
```

### **Step 4: Crack the hash**

Crack the hash using [CrackStation](https://crackstation.net/).

<figure><img src="https://miro.medium.com/v2/resize:fit:770/1*fl5xA9IxrjJWrTT5YZCdnQ.png" alt="" height="273" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:770/1*izaDQLFa1wqwDzqDDZqJMg.png" alt="" height="57" width="700"><figcaption></figcaption></figure>

> The hash has been identified as an MD5 hash and successfully cracked to reveal the plaintext password ‘**qwertyuiop**’.

### **Step 5: Login with the derived credentials.**

Navigate back to the login page and use the credentials.

```
Username: admin
Password: qwertyuiop
```

<figure><img src="https://miro.medium.com/v2/resize:fit:770/1*hDrvWX4d8-x3N_ZC7mYsxA.png" alt="" height="168" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:770/1*8g-sZZEfU_Ok0J79gBHciQ.png" alt="" height="275" width="700"><figcaption></figcaption></figure>

> We now have admin access and can perform actions, including adding and deleting users.

Cryptographic flaws, including the use of weak algorithms such as MD5, coupled with data exposure, can result in the compromise of sensitive information and systems.
