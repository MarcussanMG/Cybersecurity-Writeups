---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Dog?sort_by=created_at&sort_type=desc
Difficulty: Easy
Featured:
aliases:
---

---
# Information / Description

![](../../0.%20Assets/Dog-1791969481754.webp)
![](../../0.%20Assets/Dog-1791969488343.webp)

---

# Walkthrough

Let's start with an `nmap` scan to see what the machine is exposing. First a full port sweep.

```
nmap $T -sS -Pn -n -p- --min-rate 5000 -oG openPorts
```

```
nmap $T -sS -Pn -n -sVC --min-rate 5000 -oN results.txt -p 22,80
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-14 11:21 +0200
Nmap scan report for 10.129.231.223
Host is up (0.021s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 97:2a:d2:2c:89:8a:d3:ed:4d:ac:00:d2:1e:87:49:a7 (RSA)
|   256 27:7c:3c:eb:0f:26:e9:62:59:0f:0f:b1:38:c9:ae:2b (ECDSA)
|_  256 93:88:47:4c:69:af:72:16:09:4c:ba:77:1e:3b:3b:eb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Backdrop CMS 1 (https://backdropcms.org)
| http-robots.txt: 22 disallowed entries (15 shown)
| /core/ /profiles/ /README.md /web.config /admin
| /comment/reply /filter/tips /node/add /search /user/register
|_/user/password /user/login /user/logout /?q=admin /?q=comment/reply
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-git:
|   10.129.231.223:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: todo: customize url aliases.  reference:https://docs.backdro...
|_http-title: Home | Dog
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.65 seconds
```



Only SSH and HTTP are open, but the detailed scan is already generous: the web server runs Backdrop CMS, `robots.txt` lists a pile of CMS paths, and best of all there's an exposed `.git` directory. Let's look at `robots.txt` first.

![](../../0.%20Assets/Dog-1791969758761.webp)

The `.git` folder is browsable, which means the whole site source is sitting there for the taking.

![](../../0.%20Assets/Dog-1791969935366.webp)

Let's pull the whole repository down with `git-dumper`.

```
./git-dumper http://10.129.231.223/.git/ .
```


Among the recovered files, `settings.php` holds the Backdrop database connection string, including the password `BackDropJ2024DS2024`.

![](../../0.%20Assets/Dog-1791969997112.webp)

The database user is `root`, which won't get us into the CMS, and trying that password against the admin panel directly went nowhere. We still need a valid username.

While I was at it, I confirmed the Backdrop version from `core/profiles/minimal/minimal.info`: 1.27.1.

![](../../0.%20Assets/Dog-1791970293483.webp)


The git history also leaks the domain, `dog.htb`, in the commit author's email.

![](../../0.%20Assets/Dog-1791971325658.webp)

So let's search the dumped source for anything tied to `dog.htb`. I used live grep from my Neovim setup to dig through it quickly.

One of the config files references an email address: `tiffany@dog.htb`. That gives us a username.

![](../../0.%20Assets/Dog-1791972817084.webp)

Pairing that username with the password from `settings.php` is worth a shot.

```
username: 'tiffany',
password: 'BackDropJ2024DS2024'
```

The password was reused, and we land in the Backdrop admin dashboard as an administrator.

![](../../0.%20Assets/Dog-1791972915765.webp)

An admin in Backdrop can install modules, and a malicious module means code execution. This [exploit](https://github.com/rvzsec/backdrop-rce) automates it, packaging a PHP web shell as an installable module and uploading it.

Start a listener to catch the shell.

```
penelope -O -p 1337 -a -i tun0
```


The exploit drops a web shell and gives us command execution as `www-data`. From there I sent a Bash reverse shell back to `penelope` for a proper interactive session.

![](../../0.%20Assets/Dog-1791976031260.webp)

Listing `/home` shows two real users: `jobert` and `johncusack`.

![](../../0.%20Assets/Dog-1791976101113.webp)

That `BackDropJ2024DS2024` password keeps paying off. It fails for `jobert`, but `su johncusack` accepts it, so the password was reused for a system account too.

![](../../0.%20Assets/Dog-1791976130692.webp)

Now as `johncusack`, let's grab the user flag.

It's in johncusack's home directory.

![](../../0.%20Assets/Dog-1791976293230.webp)

Now for root. Checking sudo rights, johncusack can run `/usr/local/bin/bee` as any user.

![](../../0.%20Assets/Dog-1791976205806.webp)

`bee` is Backdrop's command-line tool, the equivalent of Drupal's drush. Its `eval` subcommand runs arbitrary PHP, so running it under sudo means running PHP as root.

![](../../0.%20Assets/Dog-1791976236494.webp)

Running it from the Backdrop root, `sudo bee eval 'system(id)'` executes as root.

![](../../0.%20Assets/Dog-1791976600837.webp)

Swapping in `system(bash)` gives a full root shell.

![](../../0.%20Assets/Dog-1791976614300.webp)

And the root flag is ours.

![](../../0.%20Assets/Dog-1791976627018.webp)