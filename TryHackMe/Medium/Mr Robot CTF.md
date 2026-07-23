---
Category: Web
LAB: https://tryhackme.com/room/mrrobot
Difficulty: Medium
Featured:
aliases:
---

---
# Information / Description

| `Victim IP` | 10.128.141.227 |
| ----------- | -------------- |
![](../../0.%20Assets/Mr%20Robot%20CTF-1784827002475.webp)

---

# Walkthrough

We will first start with an `nmap` as usual, to discover the services on the open ports of the machine:

```
sudo nmap -sS -sV -sC -p- -Pn -n 10.128.141.227 -T5
```

```
Nmap scan report for 10.128.141.227
Host is up (0.030s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE  SERVICE VERSION
22/tcp  closed ssh
80/tcp  closed http
443/tcp closed https
```

let's leave `ssh` for later so let's start with the webserver

When we enter the webserver we encounter a terminal with different functionalities:

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826373530.webp)

None of them really do anything (Or I couldn't find what to do with them, they are just videos of the `Mr.Robot` series for the most part) so I used `whatweb` to try to understand how the website was made.

```
whatweb http://10.128.141.227
```

```
┌──(marc㉿martin)-[~]
└─$ whatweb http://10.128.141.227/     
http://10.128.141.227/ [200 OK] Apache, Country[RESERVED][ZZ], HTML5, HTTPServer[Apache], IP[10.128.141.227], Script, UncommonHeaders[x-mod-pagespeed], X-Frame-Options[SAMEORIGIN]
```

And didn't really find much information, just that is hosted in an `apache` web server.

Next thing I did was to search for `robots.txt` and `sitemap.xml` to see if I could find any relevant information. 

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826532795.webp)

And bingo, we find what looks like a dictionary, we will see what to do with it later and the first flag

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826567665.webp)

```flag1
073403c8a58a1f80d943455fb30724b9
```

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826588953.webp)

We will copy this into a file for later, it might be useful.

And checking `sitemap.xml`

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826790912.webp)

Nothing really

Next thing I did was to read the source code of the webserver to see if I could find any information 

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826829750.webp)

Bingo, this is a `wordpress`

Okay cool, let's first do some manual enumeration and then we will move to automated tools

First thing I did was to search for `/wp-admin` or `/wp-login` to see If we could find a place where we could brute force our way in 

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826925117.webp)

And we where lucky

Let's look in the `wp-content` and see if we find anything.

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826964477.webp)

Seems like directory listing is not enabled

Cool let's use `wp-admin` before we brute force the login to see what extra information we get.