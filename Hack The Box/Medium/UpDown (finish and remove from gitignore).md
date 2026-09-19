---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/UpDown?sort_by=created_at&sort_type=desc
Difficulty: Medium
Featured:
aliases:
---

---
# Information / Description

![](../../0.%20Assets/UpDown-1789741901745.webp)

---

# Walkthrough

We will start with the basics, let's do an `nmap` scan to see what this machine has to offer.

First let's find the ports

```
nmap -sS -p- $T --min-rate 5000 -Pn -n -oG openPorts
```

- `$T` is a variable i created to store the IP of the target machine

We are  storing it in a `grepable` format because i have a little functionality called `"ExtractPorts"` in my `zsh` that takes a file and with grep copies the open ports to the clipboard do we don't have to write them manually and/or scan for all ports again

```
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

So let's version the ports now

```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 9e:1f:98:d7:c8:ba:61:db:f1:49:66:9d:70:17:02:e7 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDl7j17X/EWcm1MwzD7sKOFZyTUggWH1RRgwFbAK+B6R28x47OJjQW8VO4tCjTyvqKBzpgg7r98xNEykmvnMr0V9eUhg6zf04GfS/gudDF3Fbr3XnZOsrMmryChQdkMyZQK1HULbqRij1tdHaxb
IGbG5CmIxbh69mMwBOlinQINCStytTvZq4btP5xSMd8pyzuZdqw3Z58ORSnJAorhBXAmVa9126OoLx7AzL0aO3lqgWjo/wwd3FmcYxAdOjKFbIRiZK/f7RJHty9P2WhhmZ6mZBSTAvIJ36Kb4Z0NuZ+ztfZCCDEw3z3bVXSVR/cp0Z0186gkZv8w8c
p/ZHbtJB/nofzEBEeIK8gZqeFc/hwrySA6yBbSg0FYmXSvUuKgtjTgbZvgog66h+98XUgXheX1YPDcnUU66zcZbGsSM1aw1sMqB1vHhd2LGeY8UeQ1pr+lppDwMgce8DO141tj+ozjJouy19Tkc9BB46FNJ43Jl58CbLPdHUcWeMbjwauMrw0=
|   256 c2:1c:fe:11:52:e3:d7:e5:f7:59:18:6b:68:45:3f:62 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKMJ3/md06ho+1RKACqh2T8urLkt1ST6yJ9EXEkuJh0UI/zFcIffzUOeiD2ZHphWyvRDIqm7ikVvNFmigSBUpXI=
|   256 5f:6e:12:67:0a:66:e8:e2:b7:61:be:c4:14:3a:d3:8e (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIL1VZrZbtNuK2LKeBBzfz0gywG4oYxgPl+s5QENjani1
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Is my Website up ?
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

looks like the intrusion is going to be through a web port

While we inspect the website I will run a gobuster in the `background`

```
gobuster dir -u http://$T -w /usr/share/wordlists/dirb/big.txt
```

And this is how the website looks

![](../../0.%20Assets/UpDown-1789742273876.webp)
To me this immediately calls for `Command injection`

First of all I see a domain down there, so let's add that to our `/etc/hosts`

![](../../0.%20Assets/UpDown-1789742433644.webp)

Trying to do some command injection we find an interesting error message:

![361](../../0.%20Assets/UpDown-1789742543181.webp)

so we have to find a way to bypass this filter

Fidgeting a bit with the website i found that (this is a HTB machine so it doesn't have connectivity meaning any website we test will be though of as not online) but 127.0.0.1 or localhost should be online, and now we can see the debug mode working

![](../../0.%20Assets/UpDown-1789743344889.webp)

I was trying to get the filter bypass to work and i couldn't, so I ran this payload list from intruder in burp suite to see which payloads could work, and work our way up from there

[Repo](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Command%20Injection/Intruder/command_exec.txt)

![](../../0.%20Assets/UpDown-1789743926404.webp)