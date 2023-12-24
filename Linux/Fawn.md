# Fawn

# TL;DR

- Misconfigured ftp serivce running on the target machine that allows us to login using the `anonymous` username and any password.

# Enumeration

- `ping 10.129.1.14` : to test if the server us up and running.
- We scan the host to determine running services:
```
sudo nmap -sV 10.129.1.14
Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-23 21:36 PST
Nmap scan report for 10.129.1.14
Host is up (0.085s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.04 seconds
```
- From the scan we find `ftp` service running on port `21/tcp`.

# Foothold

 - In order to interact with the `ftp` service we can use the `ftp` command on our host.
 - `sudo apt install ftp -y` to install if it does not already exist on our host.
 - We can connect to the target host using `ftp 10.129.1.14`
 - When we try to connect to the host we are prompted for a username (A typical misconfiguration for running FTP service allows an `anonymous` account to access the service like any other authenticated user).
 - The `anonymous` username can be used as an input when prompted, followed by any password whatsoever since the service will disregard the password for the specific account.
 - We see the commands we can run by typing `help`
 - We find the `flag.txt` file in the current directory. We can download it using the following command: `get flag.txt`