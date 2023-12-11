# Meow

# TL;DR

- Simple maching with telnet service running. Defaul creds can be used to login and obtian the `flag.txt`.

# Enumeration

- `ping 10.129.1.17` : to test if the server us up and running.
- We scan the host to determine running services:
```
└─$ sudo nmap -sV 10.129.1.17
Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-11 07:08 PST
Nmap scan report for 10.129.1.17
Host is up (0.082s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
23/tcp open  telnet  Linux telnetd
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.02 seconds

```
- From the scan we find `telnet` running on port `23/tcp`.

# Foothold

 - `telnet 10.129.1.17` : is used to connect to the host. Since there are no other services running on the host we can try the default credentials to login.
 - We are able to login using `root` as the username.
 - Once logged in we are able to obtain the `flag.txt`.

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