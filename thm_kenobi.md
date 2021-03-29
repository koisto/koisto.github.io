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

