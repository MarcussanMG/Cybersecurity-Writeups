---
Category: Path Traversal
LAB: https://portswigger.net/web-security/file-path-traversal/lab-sequences-stripped-non-recursively
Difficulty: Practitioner
---
---

In this lab we need to find a path traversal but the traversal sequences are stripped, so lets start by finding the path traversal in this lab.

Conveniently I found that when you look into a item it fetches an image from the server

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20non-recursively-1784455561478.webp)

the problem resides in that traversal sequences are stripped

so my first approach was to use `~` because you can get to the root of the file system using it

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20non-recursively-1784455665560.webp)

But that appears to be stripped as well 

so my next approach is to double the amount of traversals so that it takes one of the doubled ones leaving a clean traversal

```
....//....//....//..../ ...
```

effectively leaving us with

```
../../../ ...
```

and then calling `/etc/passwd`

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20non-recursively-1784455855978.webp)


![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20non-recursively-1784455916347.webp)

