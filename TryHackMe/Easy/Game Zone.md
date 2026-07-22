---
Category: Web
LAB: https://tryhackme.com/room/gamezone
Difficulty: Easy
Featured:
aliases:
---

---
# Information / Description

Learn to hack into this machine. Understand how to use `SQLMap`, crack some passwords, reveal services using a reverse SSH tunnel and escalate your privileges to root!

| `Victim IP` | 10.130.175.208 |
| ----------- | -------------- |

---

# Walkthrough

## Enumeration

First thing we will do as usual is to scan all the ports to see what services are running, and we will do this with `nmap`

```
sudo nmap -sS -sV -sC -p- -Pn -n 10.130.175.208 -T5
```

```
┌──(marc㉿martin)-[~]
└─$ sudo nmap -sS -sV -sC -p- -Pn -n 10.130.175.208 -T5
[sudo] password for marc: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-07-22 15:17 CEST
Nmap scan report for 10.130.175.208
Host is up (0.037s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:ea:89:f1:d4:a7:dc:a5:50:f7:6d:89:c3:af:0b:03 (RSA)
|   256 b3:7d:72:46:1e:d3:41:b6:6a:91:15:16:c9:4a:a5:fa (ECDSA)
|_  256 53:67:09:dc:ff:fb:3a:3e:fb:fe:cf:d8:6d:41:27:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Game Zone
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Okay so we have `ssh` and `apache`, let's start with the `apache` and see if we can enumerate some users from there.

### Apache (:80)

When we open the webserver this is what we encounter.

![](../../0.%20Assets/Game%20Zone-1784726513843.webp)


As we can see there is a `login` form, let's try to perform some `SQLinjection`

This is what a query for checking credentials might look like:

```
SELECT * FROM users WHERE username = <input> AND password = <password>
```

What if 

![](../../0.%20Assets/Game%20Zone-1784726761317.webp)
