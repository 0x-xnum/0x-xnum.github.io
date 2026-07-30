# rpcbind

**`Default Port: 111`**

**rpcbind** is used by **RPC (Remote Procedure Call)** services. An RPC service is a server-based service that fulfills remote procedure calls. `rpcbind` is used to determine which services can respond to incoming requests to perform the specified service.

### Connect <a href="#connect" id="connect"></a>

Confirming that the rpcbind service runs on a target machine is usually the first step.

```
nc -z -v -u <target-ip> 111
```

### Recon <a href="#recon" id="recon"></a>

When connected to a machine rpcbind, you can use the `rpcinfo` tool to learn the details of rpcbind service.

```
rpcinfo -p <target-ip>
```

### Enumeration <a href="#enumeration" id="enumeration"></a>

You can run the `rpcenum` script to determine the rpc service on the target and collect information. You can use the -v flag for more details.

```
rpcenum -v <target-ip>
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

There are various rpcbind modules in Metasploit. For example, you can use the following Metasploit commands for the `rpcbind_cgi_mainenv` vulnerability

```
msfconsole
use auxiliary/gather/rpcbind_cgi_mainenv
set RHOST <target-ip>
run
```

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### RPC Dictionary Attack <a href="#rpc-dictionary-attack" id="rpc-dictionary-attack"></a>

If you successfully exploited vulnerabilities and obtained a username and password, you can create an RPC bridge as follows using `rpcclient`

```
rpcclient -U "username%password" <target-ip>
```

#### Service Abuse <a href="#service-abuse" id="service-abuse"></a>

If a certain RPC service created a vulnerability on the target machine, you can abuse this service to influence behaviors on the target system or hinder its proper operation. For example, you can stop the following RPC service:

```
rpcclient -U "username%password" <target-ip> -c 'stop service_name'
```
