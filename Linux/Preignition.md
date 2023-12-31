# Preignition

# TL;DR

- Simple maching running nginx webserver with a hidden admin portal. We find this web portal using the `gobuster` tool and login using the default credentials `admin:admin`

# Enumeration

- `ping 10.129.85.67` : to test if the server us up and running.
- We scan the host to determine running services:
```
nmap -p- -sV 10.129.85.67
Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-30 20:35 PST
Nmap scan report for 10.129.85.67
Host is up (0.084s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.68 seconds
```
- From the scan we find `nginx` service running on port `80/tcp`.

# Foothold

 - There is no other service running on the target host and only running the http service, signaling that this target might be hosting some explorable web content. We can look at the contents hosted we can open a web browser and navigate to the target's IP addrerss in the URL bar.
 - We see the default post-installation page of the nginx service, meaning that there is the possibility that this web application might not be adequately configured yet, or the default credentials are used to facilitate faster configuration up to the poion of live deployment.
 - Since there are no buttons or any other content on the page. We will have to run `gobuster` to discover other web directories.
 - We can run the following commands if `gobuster` does not already exist on the local host:
 ```
 sudo apt install golang-go
 go install github.com/OJ/gobuster/v3@latest
 ```
 - In case the above commands fail for whatever reason. You can always compile the tool from its source code by running the following commands:
 ```
 sudo git clone https://github.com/OJ/gobuster.git
 cd gobuster
 go get && go build
 go install
 ```
 - Once installed we can run the following command specifing we want to used the directory busting mode with `dir`, `-w {path to the worlist}` and `-u {target IP`:
 ```
gobuster dir -w /usr/share/wordlists/dirb/common.txt -u 10.129.85.67
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.85.67
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/12/30 20:54:16 Starting gobuster in directory enumeration mode
===============================================================
/admin.php            (Status: 200) [Size: 999]
Progress: 4592 / 4615 (99.50%)
===============================================================
2023/12/30 20:54:56 Finished
===============================================================

 ```
 - From the above output we find the `\admin.php` web page. When we visit the new webpage in the browser we are presented with the admin console login.
 - Usually, in situations such as this one, we would need to fire up some brute-forcing tools to attempt logging in with multiple credentials sets for an extened period of time until we hit a valid log-in since we do not have any underlying context about usernames and passwords that might have been registered on this web site as valid administrative accounts.
 - But first we can try some defautl credentials since this is a fresh nginx installation.
 - We try `admin:admin` and we are successfully able to login and obtian the flag presented to us.