# Neighbor - https://tryhackme.com/room/neighbour

We first perform an Nmap scan of the target IP address:

`nmap -sV -sC <ip address>`

* `-sV` - probe open ports and determine service/version info of any running services
* `-sC` - run default scripts on those discovered services

```
Starting Nmap 7.80 ( https://nmap.org ) at 2022-11-26 10:40 PST
Nmap scan report for 10.10.228.136
Host is up (0.15s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.53 ((Debian))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.53 (Debian)
|_http-title: Login
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.33 seconds
```

And we discover 2 accessible services: a web server and SSH.

Visitng the webserver:

![](https://i.imgur.com/3xsWPe3.png)

We do not have any credentials, so we'll use the guest credentials we get from viewing the source code:

![](https://i.imgur.com/Exfoelk.png)

Logging in, we see that the user we logged in as is displayed in the `user` parameter of the URL:

![](https://i.imgur.com/qSskVfG.png)

If we change the parameter to another user, such as admin...

![](https://i.imgur.com/Z8C0gQ8.png)

We get our flag!
