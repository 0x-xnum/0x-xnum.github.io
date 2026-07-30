# Splunkd

**`Default Port: 8089`**

**Splunkd** is the core component of the Splunk platform, responsible for indexing, searching, and processing data ingested by Splunk. It provides a web interface and APIs for managing and analyzing machine-generated data.

Splunk is widely used for log management, security information and event management (SIEM), and data analytics in enterprise environments.

### Connect <a href="#connect" id="connect"></a>

#### Connect Using Web Interface <a href="#connect-using-web-interface" id="connect-using-web-interface"></a>

You can access the Splunk web interface by navigating to `https://<splunk-server-ip>:8089` in a web browser.

#### Connect Using Splunk CLI <a href="#connect-using-splunk-cli" id="connect-using-splunk-cli"></a>

Splunk CLI commands can be used for various administrative tasks and querying data. You can connect to Splunk using the following command:

```
splunk login -auth <username>:<password> -port 8089 -host <splunk-server-ip>
```

### Recon <a href="#recon" id="recon"></a>

#### Identifying a Splunk Server <a href="#identifying-a-splunk-server" id="identifying-a-splunk-server"></a>

You can use `Nmap` to check if there's a Splunk server running on a target host like this:

```
nmap -p 8089 X.X.X.X
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

You can use tools like `Netcat` to perform banner grabbing and retrieve information about the Splunk service:

```
nc -nv X.X.X.X 8089
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Splunkd API Endpoints <a href="#splunkd-api-endpoints" id="splunkd-api-endpoints"></a>

Splunkd exposes various API endpoints for interacting with the Splunk platform. You can enumerate these endpoints to gather information about the server and available functionalities.

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Default Credentials <a href="#default-credentials" id="default-credentials"></a>

Check for default credentials or weak authentication configurations in Splunk instances, such as `admin:admin` or `admin:<blank>`.

#### Unauthorized Access <a href="#unauthorized-access" id="unauthorized-access"></a>

Exploit misconfigured access controls or weak authentication mechanisms to gain unauthorized access to sensitive data stored in Splunk.

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Common Splunk CLI Commands <a href="#common-splunk-cli-commands" id="common-splunk-cli-commands"></a>

| Command                         | Description                                           |
| ------------------------------- | ----------------------------------------------------- |
| splunk search \<query>          | Perform a search query in Splunk.                     |
| splunk list \<entity>           | List entities like indexes, sources, or sourcetypes.  |
| splunk info \<entity>           | Display detailed information about a specific entity. |
| splunk add \<entity> \<name>    | Add a new entity to Splunk (e.g., index, input).      |
| splunk delete \<entity> \<name> | Delete an existing entity from Splunk.                |

#### Data Manipulation <a href="#data-manipulation" id="data-manipulation"></a>

Manipulate indexed data in Splunk, such as modifying or deleting events, altering timestamps, or injecting fake data.

<br>
