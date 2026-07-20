---
Category: Path Traversal
LAB: https://portswigger.net/web-security/file-path-traversal/lab-validate-file-extension-null-byte-bypass
Difficulty: Practitioner
Featured: "True"
---
---

This is the description of the lab:

>This lab contains a path traversal vulnerability in the display of product images.
>The application validates that the supplied filename ends with the expected file extension.
>To solve the lab, retrieve the contents of the `/etc/passwd` file.

My first idea is to do path traversal and then in burp change the extension of the file to the expected one 

![](../../../0.%20Assets/File%20path%20traversal,%20validation%20of%20file%20extension%20with%20null%20byte%20bypass-1784457508997.webp)

As we can appreciate it is expecting `jpg` files, so let's try directly changing the extension of the file to accommodate this specification

![](../../../0.%20Assets/File%20path%20traversal,%20validation%20of%20file%20extension%20with%20null%20byte%20bypass-1784457611566.webp)

And it didn't work out of the box, so let's tweak it a bit

`%00` is a `NULl byte` so when the system opens the file it interprets the `NULL byte` as an end so it doesn't read the `.jpg` but the webapp does so we make both happy :)

![](../../../0.%20Assets/File%20path%20traversal,%20validation%20of%20file%20extension%20with%20null%20byte%20bypass-1784457740941.webp)

And that is the solved lab

![](../../../0.%20Assets/File%20path%20traversal,%20validation%20of%20file%20extension%20with%20null%20byte%20bypass-1784457801907.webp)