# ADB Android Debug Bridge

**`Default Port: 5555`**

The **Android Debug Bridge (ADB)** is a command-line tool that allows communication with an Android device. If not configured properly, it can be exploited to access sensitive data or even full command of the device.

### Connect <a href="#connect" id="connect"></a>

To establish a connection to an ADB service, the default TCP/IP port is 5555.

```
adb connect <ip>:<port>
```

```
adb devices
```

### Recon <a href="#recon" id="recon"></a>

#### Identifying an ADB Server <a href="#identifying-an-adb-server" id="identifying-an-adb-server"></a>

You can use `Nmap` to check if there's an ADB server on a target host like this:

```
nmap -p 5555 X.X.X.X
```

### Attack Vectors <a href="#attack-vectors" id="attack-vectors"></a>

ADB can be exploited in various ways due to misconfiguration or failure to secure the device properly:

**Unrestricted shell access:** With an ADB shell, you have Linux command-line access which means you can essentially perform any operation on the device.

**Ghost Framework:** A robust framework for exploiting ADB to gain control of a device. You can clone the repository from GitHub, navigate into the directory, and start using it.

```
git clone https://github.com/EntySec/ghost
cd ghost
chmod +x install.sh
./install.sh
ghost
```

You can now connect to a device, control it, see device info, list/transfer files, and more.

```
# Connect to a device.
ghost> connect <ip>:<port>

# See device info
ghost> deviceinfo

# List files
ghost> ls
```

**Push and Pull data:** ADB allows you to transfer data to and from a device. This means you can copy sensitive data or push malicious files.

### Post-Exploitation <a href="#post-exploitation" id="post-exploitation"></a>

#### Common ADB Commands <a href="#common-adb-commands" id="common-adb-commands"></a>

| Command                          | Description                                  | Usage                                                 |
| -------------------------------- | -------------------------------------------- | ----------------------------------------------------- |
| `adb devices`                    | List of connected Android devices            | `adb devices`                                         |
| `adb shell`                      | Open a remote shell to the device            | `adb shell`                                           |
| `adb install <APK>`              | Install an APK onto the connected device     | `adb install example.apk`                             |
| `adb uninstall <PACKAGE>`        | Uninstall an app from the connected device   | `adb uninstall com.example.app`                       |
| `adb pull <REMOTE> <LOCAL>`      | Copy a file from the device to your computer | `adb pull /sdcard/example.txt .`                      |
| `adb push <LOCAL> <REMOTE>`      | Copy a file from your computer to the device | `adb push example.txt /sdcard/`                       |
| `adb logcat`                     | View the device log output                   | `adb logcat`                                          |
| `adb reboot`                     | Reboot the device                            | `adb reboot`                                          |
| `adb shell am start <PACKAGE>`   | Launch an app on the device                  | `adb shell am start -n com.example.app/.MainActivity` |
| `adb shell pm list packages`     | List all installed packages on the device    | `adb shell pm list packages`                          |
| `adb shell dumpsys`              | Dump system information                      | `adb shell dumpsys`                                   |
| `adb shell screencap`            | Capture a screenshot of the device screen    | `adb shell screencap /sdcard/screen.png`              |
| `adb shell input keyevent <KEY>` | Simulate a key press on the device           | `adb shell input keyevent KEYCODE_POWER`              |
