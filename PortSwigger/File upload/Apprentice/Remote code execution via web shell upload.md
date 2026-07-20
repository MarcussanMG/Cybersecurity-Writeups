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

Let's craft a `PHP web shell` that only calls for that file and see if we are lucky.

I do not know much about `PHP` so i looked online and found this function:

```
file_get_contents()
```

- https://www.php.net/manual/en/function.file-get-contents.php

the usage it's pretty simple just need to type the path to the file inside the function

And this is the script I came up with:

- We `echo` (print) the output of the file we are looking for and then finish the statement with `;`

```
<?php echo file_get_contents(/home/carlos/secret); ?>
```

![](../../../0.%20Assets/Remote%20code%20execution%20via%20web%20shell%20upload-1784556120866.webp)

Now just upload this as a file


![](../../../0.%20Assets/Remote%20code%20execution%20via%20web%20shell%20upload-1784553929638.webp)

![](../../../0.%20Assets/Remote%20code%20execution%20via%20web%20shell%20upload-1784553936505.webp)

![](../../../0.%20Assets/Remote%20code%20execution%20via%20web%20shell%20upload-1784556109054.webp)

And as we can see in Burp we know where the location of the file is because it is trying to render it in our account

![](../../../0.%20Assets/Remote%20code%20execution%20via%20web%20shell%20upload-1784554067593.webp)

And i am encountering an error, i checked the solution and I should be doing it right so ill take it as solved