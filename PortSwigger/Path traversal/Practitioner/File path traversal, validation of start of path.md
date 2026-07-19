---
Category: Path Traversal
LAB: https://portswigger.net/web-security/file-path-traversal/lab-validate-start-of-path
Difficulty: Practitioner
---
---

This is the description of the lab:

>This lab contains a path traversal vulnerability in the display of product images.
   The application transmits the full file path via a request parameter, and validates that the supplied path starts with the expected folder.
   To solve the lab, retrieve the contents of the `/etc/passwd` file.

So we know it has to start with the path hardcoded from the website, okay no problem, my first idea is to traverse from there instead of the start of the root of the file system, something like this:

```
/path/in/website/../../../../etc/passwd
```

So let's try (the vulnerability resides in the same place as before)

![](../../../0.%20Assets/File%20path%20traversal,%20validation%20of%20start%20of%20path-1784457102640.webp)

And yes we have a hardcoded Path and if we try to start from the root of the filesystem it breaks

so let's try what I anticipated (note that we know the path to the root of the filesystem so we know how many hops backwards we need to do to get there)

![](../../../0.%20Assets/File%20path%20traversal,%20validation%20of%20start%20of%20path-1784457190430.webp)

And there we go, we just need to refresh the page to solve the lab

![](../../../0.%20Assets/File%20path%20traversal,%20validation%20of%20start%20of%20path-1784457212746.webp)