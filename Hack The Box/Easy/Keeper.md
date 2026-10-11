---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Keeper?tab=play_machine
Difficulty: Easy
Featured:
aliases:
  - Linux
---

---
# Information / Description

![](../../0.%20Assets/Keeper-1791718599133.webp)

![135](../../0.%20Assets/Keeper-1791718609548.webp)
---

# Walkthrough

Let's start with an `nmap` scan to see what the machine is exposing. I went straight for a service and script scan on the two ports that matter here.

```
sudo nmap -sV -sC -Pn -n -p 22,80 -oN results.txt
```


```
sudo nmap -sV -sC -Pn -n -p 22,80 -oN results.txt $T
```

Running it against the target shows only SSH on 22 and an nginx web server on 80.

![](../../0.%20Assets/Keeper-1791719126813.webp)

With almost nothing else to go on, let's look at the website. The page is basically a single line pointing us to a support portal at `tickets.keeper.htb`.

![](../../0.%20Assets/Keeper-1791719179449.webp)


That hostname won't resolve on its own, so let's add both `keeper.htb` and `tickets.keeper.htb` to `/etc/hosts`.

![](../../0.%20Assets/Keeper-1791719284957.webp)


The portal is a Request Tracker (RT) instance, version 4.4.4.

![](../../0.%20Assets/Keeper-1791719309219.webp)


RT ships with well-known default credentials, `root:password`, and nobody changed them here.

![](../../0.%20Assets/Keeper-1791719333217.webp)

We're in as the RT administrator.

![](../../0.%20Assets/Keeper-1791719668992.webp)

Browsing the admin panel, the Users section lists a second account besides root: `lnorgaard` (Lise Nørgaard).

![](../../0.%20Assets/Keeper-1791720966220.webp)

Her user page carries a comment left by an admin: the initial password was set to `Welcome2023!`.

![](../../0.%20Assets/Keeper-1791721064373.webp)

That password was never rotated, so it logs us straight in over SSH as `lnorgaard`.

![](../../0.%20Assets/Keeper-1791721095278.webp)

The user flag is in her home directory, along with an interesting file: `RT30000.zip`.

![](../../0.%20Assets/Keeper-1791721109405.webp)

I pulled the archive back to my box. Since I already had a `penelope` session, I just used its built-in download.

![](../../0.%20Assets/Keeper-1791721181658.webp)

Unzipping it gives two files: `KeePassDumpFull.dmp` and `passcodes.kdbx`. So we have a KeePass database and what looks like a memory dump of KeePass itself.

![](../../0.%20Assets/Keeper-1791721230986.webp)

Let's install KeePassXC so we can work with the database.

![](../../0.%20Assets/Keeper-1791721298778.webp)

Opening `passcodes.kdbx` prompts for a master password we don't have yet.

![](../../0.%20Assets/Keeper-1791721333542.webp)

![](../../0.%20Assets/Keeper-1791721346614.webp)

Before anything fancy, let's try to crack the master password the usual way. Convert the database into a hash with `keepass2john`.

```
keepass2john passcodes.kdbx > keepass.hash
```

![](../../0.%20Assets/Keeper-1791721577763.webp)

Trim the hash file so it starts at the `$keepass$` marker, then throw it at `hashcat`.

```
hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt
```

`rockyou` was getting nowhere, which makes sense for a password like this one. That's when the second file, the memory dump, becomes the real lead.

A `.dmp` sitting next to a KeePass database is a strong hint. KeePass versions before 2.54 are vulnerable to CVE-2023-32784, which lets you recover most of the master password from a process memory dump.

![](../../0.%20Assets/Keeper-1791721835154.webp)

![](../../0.%20Assets/Keeper-1791721841381.webp)


There are several PoCs for `CVE-2023-32784`. I used this one: [keepass-dump-extractor](https://github.com/JorianWoltjer/keepass-dump-extractor/releases). Grab the Linux build from the releases page and unpack it.

```
tar -xzf keepass-dump-extractor-x86_64-unknown-linux-gnu.tar.gz
```

Run it against the dump. It recovers the master password one character at a time, missing only the first one, so I piped the output into a wordlist to feed back into `hashcat`.

![](../../0.%20Assets/Keeper-1791722048397.webp)

```
./keepass-dump-extractor KeePassDumpFull.dmp -f all > wordlist.txt
```

```
hashcat -m 13400 keepass.hash wordlist.txt
```

This time `hashcat` cracks it almost instantly.

![](../../0.%20Assets/Keeper-1791722154395.webp)

The master password turns out to be a Danish dessert, `rødgrød med fløde`.

```
rødgrød med fløde
```

Unlocking the database with it, the Network group holds an entry for the ticketing server owned by `root`.

![](../../0.%20Assets/Keeper-1791722182121.webp)

The entry stores root's password and, more usefully, a PuTTY-format private key in the notes field.

![](../../0.%20Assets/Keeper-1791722336711.webp)

Save the private key to a file and lock down its permissions.

```
chmod 600 id_rsa
```

Trying `ssh -i id_rsa root@$T` fails with an `error in libcrypto` message. That's because the key is in PuTTY's own format, which OpenSSH can't read directly.

![](../../0.%20Assets/Keeper-1791722410082.webp)


The quickest fix is to use PuTTY itself, which understands its own key format. This [guide](https://www.dell.com/support/kbdoc/en-gb/000217157/connectrix-b-series-how-to-use-putty-for-ssh-key-based-authentication) walks through key-based auth in PuTTY. You could also convert the key to OpenSSH format with `puttygen`, but loading it straight into PuTTY is faster.

Load the private key under Connection > SSH > Auth.

![](../../0.%20Assets/Keeper-1791722875610.webp)

Set the target IP and port 22 in the Session tab.

![](../../0.%20Assets/Keeper-1791722831975.webp)

Set the username to `root` under Connection > Data.
![](../../0.%20Assets/Keeper-1791722857463.webp)

Then hit Open to connect.

PuTTY authenticates with the key and drops us in as root.

![](../../0.%20Assets/Keeper-1791722891428.webp)

And the root flag is ours.

![](../../0.%20Assets/Keeper-1791722610539.webp)