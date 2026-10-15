---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Access?tab=play_machine
Difficulty: Easy
Featured:
aliases:
---

---
# Information / Description

![](../../0.%20Assets/Access-1791981908130.webp)
![](../../0.%20Assets/Access-1791981914865.webp)

---

# Walkthrough

Let's start with an `nmap` scan. The first run is a full port sweep: `-sS` is a stealthy SYN scan, `-p-` covers all 65535 ports, `--min-rate 5000` pushes the packet rate so it finishes fast, `-Pn` skips host discovery, and `-oG` saves the output in greppable format so I can pull the open ports out quickly.

```
nmap $T -sS -Pn -n -p- --min-rate 5000 -oG openPorts -vvv
```


Then a focused scan on the ports we found. `-sVC` is shorthand for `-sV` (service version detection) plus `-sC` (the default NSE scripts), which fingerprints each service.

```
nmap $T -Pn -n -sVC --min-rate 5000 -oN results.txt -vvv -p 21,23,80
```


Three ports are open: FTP on 21 with anonymous login allowed, Telnet on 23, and IIS 7.5 on 80. This is an old Windows host.

![](../../0.%20Assets/Access-1792066844621.webp)

The website is just a static MegaCorp page showing a server rack, nothing interactive.

![](../../0.%20Assets/Access-1792066855741.webp)

Directory brute forcing with `gobuster` didn't turn up anything useful, so let's focus on the anonymous FTP.

Logging into FTP as `anonymous` with no password gives us two directories, `Backups` and `Engineer`.

![](../../0.%20Assets/Access-1792066875798.webp)

`Backups` holds `backup.mdb`, a 5.6 MB Microsoft Access database, and `Engineer` holds `Access Control.zip`. I pulled both down with `get`, but notice the `WARNING! bare linefeeds received in ASCII mode` message. That bites us shortly.

![](../../0.%20Assets/Access-1792066910941.webp)




`file backup.mdb` confirms it's a Microsoft Access Database.

![](../../0.%20Assets/Access-1792066940780.webp)

To read it on Linux without Access, I used [mdbtools](https://www.kali.org/tools/mdbtools/).

At first `mdbtools` choked on the file, as if it were corrupt.

![](../../0.%20Assets/Access-1792067387510.webp)

Both files looked corrupted. Going back over my notes, I remembered the ASCII-mode warning from the download.

FTP transfers in ASCII mode by default, which rewrites line endings and so mangles any binary file. A `.mdb` and a `.zip` are binary, so they arrived broken.

![](../../0.%20Assets/Access-1792069186116.webp)

The fix is to switch FTP into `binary` mode before downloading, so the bytes come through untouched.

![](../../0.%20Assets/Access-1792069226997.webp)

With binary mode set, I re-downloaded both files and they opened cleanly.


The zip is password-protected, so let's try cracking it first. `zip2john` turns the archive into a hash John can attack, then `john` runs that hash against the `rockyou` wordlist.

![](../../0.%20Assets/Access-1792069624810.webp)

`rockyou` didn't crack it. That's fine, because the password is likely sitting in the Access database we just recovered. Let's go read it.

`mdb-tables -1` lists every table in the database, one name per line.

```
mdb-tables -1 backup.mdb
```

There are a lot of tables, but `auth_user` is the one that stands out. Rather than read them one at a time, it's easier to export them all to CSV and grep through them.

![](../../0.%20Assets/Access-1792069750254.webp)

I adapted a small loop from [this mdbtools guide](https://www.mdb-viewer.com/en/guides/read-mdb-files-with-mdbtools).
It loops over each table name and uses `mdb-export` to write that table out as its own `.csv` file.

```
for table in $(mdb-tables -1 ../backup.mdb); do mdb-export ../backup.mdb "$table" > "$table.csv"; done
```

Now I can grep across all the CSVs at once.

![](../../0.%20Assets/Access-1792070249119.webp)

Searching for `password` in `auth_user.csv` turns up three accounts, including `engineer` with the password `access4u@security`.

![](../../0.%20Assets/Access-1792070270326.webp)

Here's the full `auth_user` table with the engineer credentials.

![](../../0.%20Assets/Access-1792070316701.webp)

Those credentials don't work on FTP or Telnet directly. But the passwords from the database make an excellent targeted wordlist for the protected zip, so let's point John at them instead of `rockyou`.

This time John cracks the archive almost instantly: the zip password is `access4u@security`.

![](../../0.%20Assets/Access-1792071212837.webp)

Extract the archive with `7z x` using the password we just recovered.

```
7z x 'Access Control.zip'
```

Inside is `Access Control.pst`, an Outlook email archive.

![](../../0.%20Assets/Access-1792071296875.webp)

To read a `.pst` on Linux I used [readpst](https://linux.die.net/man/1/readpst), which converts it into a standard mbox file.

The backslash just escapes the space in the filename.

```
readpst Access\ Control.pst
```

Reading the resulting mbox, there's an email to the `security` account saying its password was changed to `4Cc3ssC0ntr0ller`.

![](../../0.%20Assets/Access-1792071430267.webp)


So we have a fresh set of credentials.

```
security:4Cc3ssC0ntr0ller
```


Telnet is still open, so let's log in there as `security`.

![](../../0.%20Assets/Access-1792071464558.webp)

The user flag is on the desktop.

![](../../0.%20Assets/Access-1792071583035.webp)

Now for root. On Windows, `cmdkey /list` shows the credentials saved in the current user's vault. Here, `security` has stored domain credentials for `ACCESS\Administrator`.

![](../../0.%20Assets/Access-1792073118684.webp)

A saved credential means we can run commands as that user without ever knowing the password. `runas /savecred /user:Administrator` reuses the stored secret, and `/noprofile` skips loading the user profile. I used it to copy the Administrator's `root.txt` into the `security` folder, where we can read it. The second `runas` concatenates it with another file into `flag.txt`, which is just another way to force the output into a location we control.

```
cmdkey /list

runas /noprofile /savecred /user:Administrator "cmd.exe /C copy C:\Users\Administrator\Desktop\root.txt C:\Users\security\root.txt"


runas /noprofile /savecred /user:Administrator "cmd.exe /C copy C:\Users\security\root.txt+C:\Users\security\test.txt C:\Users\security\flag.txt"
```

The copy runs as Administrator and drops the flag somewhere our `security` user can read it.

![](../../0.%20Assets/Access-1792073782509.webp)

And the root flag is ours.

![](../../0.%20Assets/Access-1792073983679.webp)