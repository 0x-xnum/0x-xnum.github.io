# MSRPC

**`Default Port: 135, 593`**

**MSRPC (Microsoft Remote Procedure Call)** is the modified version of DCE/RPC. It forms the basis of network-level service interoperability. MSRPC is the protocol standard for Windows processes that allows a program running on one host to execute a program on another host.

### Connect <a href="#connect" id="connect"></a>

MSRPC services normally listen on ports `135 and 593`; however, they can also run on other ports.

#### Netcat <a href="#netcat" id="netcat"></a>

You can check the connection with the `netcat` tool:

```
nc -vn <TARGET_IP> 135
nc -vn <TARGET_IP> 593
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Using Nmap <a href="#using-nmap" id="using-nmap"></a>

```
nmap -p 135,593 --script=msrpc-enum <TARGET_IP>
```

#### RPC Client <a href="#rpc-client" id="rpc-client"></a>

Windows has an embedded tool for interacting with MSRPC called RPC client that you can use for enumeration.

```
rpcclient -U "" -N <TARGET_IP>
#empty username (-U "")
#no password (-N)
```

```
> serverinfo
> lsaenumsid
> netshareenumall
```

#### Identifying Exposed RPC Services <a href="#identifying-exposed-rpc-services" id="identifying-exposed-rpc-services"></a>

Exposure of RPC services can be determined by querying the RPC locator service and individual endpoints using tools such as rpcdump. This tool identifies unique RPC services, denoted by IFID values, providing service details and communication bindings.

```
rpcdump [-p port] <IP>
```

Tools such as Metasploit can also be used to audit and interact with MSRPC services, primarily focusing on port 135.

```
use auxiliary/scanner/dcerpc/endpoint_mapper
use auxiliary/scanner/dcerpc/hidden
use auxiliary/scanner/dcerpc/management
use auxiliary/scanner/dcerpc/tcp_dcerpc_auditor
```

Among these options, all except tcp\_dcerpc\_auditor are specifically designed for targeting MSRPC on port 135.

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

MSRPC has several interfaces that could be potentially exploited for gaining unauthorized access, remote command execution, enumerating users and domains, accessing public SAM database elements, remotely starting and stopping services, accessing and modifying the system registry, and more. These interfaces include:

* LSA interface (`pipe\lsarpc`)
* LSA Directory Services (DS) interface (`pipe\lsarpc`)
* LSA SAMR interface (`pipe\samr`)
* Server services and Service control manager interface (`pipe\svcctl`), (`pipe\srvsvc`)
* Remote registry service (`pipe\winreg`)
* Task scheduler (`pipe\atsvc`)
* DCOM interface (`pipe\epmapper`)

You can also use the IOXIDResolver interface to identify IPv4 and IPv6 addresses of systems on the network.

#### MSRPC DCOM <a href="#msrpc-dcom" id="msrpc-dcom"></a>

MSRPC DCOM is one of the most dangerous services on Windows systems due to the amount of control it can give an attacker. It should be disabled if not needed. MSRPC endpoints can be abused to execute arbitary code on a remote computer.

```
nmap -p 135 --script=msrpc-dcom-interface-activation <TARGET_IP>
```
