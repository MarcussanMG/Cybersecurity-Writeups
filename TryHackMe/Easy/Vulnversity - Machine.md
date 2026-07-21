---
Category: Web
LAB: https://tryhackme.com/room/vulnversity
Difficulty: Easy
Featured:
---
---

# Information / Description


| `Machine IP` | 10.129.140.194 |
| ------------ | -------------- |

---

# Walkthrough

## Enumeration

Let's start from the beginning doing an `nmap` scan on the machine to see what we are dealing with

This is the command I used:

```
sudo nmap -sS -sV -sC -p- -Pn -n 10.129.140.194 -T5
```

```
[sudo] password for marc: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-07-21 20:44 CEST
Nmap scan report for 10.129.140.194
Host is up (0.041s latency).
Not shown: 65529 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.5
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e0:fe:3c:ec:f9:58:20:b6:4f:df:25:10:43:8f:dc:20 (RSA)
|   256 2a:c0:5a:ae:36:a9:e3:0f:34:2c:3f:0c:bc:8b:b9:63 (ECDSA)
|_  256 cf:3e:a3:40:4b:dd:da:f1:be:06:93:44:3e:ac:c8:38 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 4
445/tcp  open  netbios-ssn Samba smbd 4
3128/tcp open  http-proxy  Squid http proxy 4.10
|_http-title: ERROR: The requested URL could not be retrieved
|_http-server-header: squid/4.10
3333/tcp open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Vuln University
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: , NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_clock-skew: -1s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-07-21T18:45:27
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.53 seconds
```

We got some interesting services I tried logging into the `ftp` service with `anonymous` access with no luck, let's see if we find credentials on the go

Let's see what `samba` has to offer
![](../../0.%20Assets/Vulnversity%20-%20Machine-1784659880973.webp)

Not much for now let's leave it for later as well

Ill move to the `HTTP` service on port `3333`

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784659708175.webp)

We find a website let's see what is made with, with `whatweb`

```
┌──(marc㉿martin)-[~/Desktop]
└─$ whatweb http://10.129.140.194:3333/                                                      
http://10.129.140.194:3333/ [200 OK] Apache[2.4.41], Bootstrap, Country[RESERVED][ZZ], Email[info@yourdomain.com], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.129.140.194], JQuery, Script, Title[Vuln University]

```

nothing crazy, so let's move to `robots.txt`, `sitemap.xml` and doing a `directory bruteforce`

### robots

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784659798843.webp)

### sitemap

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784659816485.webp)

### Directory brute force

For this we will use `gobuster`

```
gobuster dir -u http://10.129.140.194:3333 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```

These are the directories it found:

```
images               (Status: 301) [Size: 324] [--> http://10.129.140.194:3333/images/]
css                  (Status: 301) [Size: 321] [--> http://10.129.140.194:3333/css/]
js                   (Status: 301) [Size: 320] [--> http://10.129.140.194:3333/js/]
internal             (Status: 301) [Size: 326] [--> http://10.129.140.194:3333/internal/]
```


## Exploitation

None of the directories we found are interesting but `/internal` where we can find a ``upload`` page

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784660938742.webp)

We are going to try to exploit this by uploading a `reverse shell`. In this case we need to know what type of reverse shell and when inspecting the source code of the webserver we find a `.php` file so we will start by uploading a `php` shell

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661024478.webp)

we can find preconfigured shells in `/usr/share/webshells/php/` where we can find `php-reverse-shell.php`

When inspecting the reverse shell we can see we need to change some variables

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661111310.webp)

In this case I can leave the port to `1234` because I don't have any service running on that port but in your case double check, and we need to change the `ip`

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661152516.webp)

I will change it to the one in the `vpn` because is the one the server can access

so `nano php-rever-shell.php` and change the `ip`

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661225778.webp)

And now let's upload it to the server.

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661246986.webp)

We encounter an error telling us that the extension is not allowed

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661260376.webp)

No problem, let's try to bypass this, to do it, let's open `Burp suite`

Open the request, right click and send to `repeater`

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661347808.webp)

I tried setting the extension with the `NULL byte` with no luck
![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661425013.webp)

Let's send the request to the `intruder` to try to brute force the extension

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661524352.webp)

Select the position and add a payload list with different `php` extension names to see if one of them works

And the one that worked was `.phtml`

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661762465.webp)

If we traverse to `/internal/uploads`

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661827765.webp)

Now before we run the `reverse shell` we need to start our `listener` and we will do this with `netcat`

```
sudo nc -nlvp 1234
```

- Bare in mind we need to specify the same port as in the script

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661916363.webp)

And now we just press on the file in the website

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661935986.webp)

And Bingo

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661947947.webp)

let's do some enumeration inside of the server itself

```
whoami
```

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784662004971.webp)

not a surprise there, let's see the `Home directories` of the users to list at least some of the users in the server.

```
ls /home
```

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784661989935.webp)

And we found 2 users
- `bill`
- `ubuntu`

![](../../0.%20Assets/Vulnversity%20-%20Machine-1784662101307.webp)

And there we go, the first flag.

```
8bd7992fbe8a6ad22a63361004cfcedb
```

### Privilege escalation

