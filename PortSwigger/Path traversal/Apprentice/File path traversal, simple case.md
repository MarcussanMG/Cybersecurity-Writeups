---
Category: Path Traversal
LAB: https://portswigger.net/web-security/file-path-traversal/lab-simple
Difficulty: Apprentice
---
---

To solve this lab we need to find a request that is requesting a file, in this case it is a blog and it is set-up in a way where when we request a blog, it also requests an image for the blog, what we can do is getting that request, sending it to the repeater and modifying it.

![](../../../0.%20Assets/File%20path%20traversal,%20simple%20case-1783428490118.webp)

what we are going to do now is go back in the hierarchy tree from Linux and then get the `/etc/passwd` file

![](../../../0.%20Assets/File%20path%20traversal,%20simple%20case-1783428561859.webp)


In this case i used a lot of `..` to go back in the tree, because in Linux normally once you are at the root `/` you can't really go back further and we don't know where in the filesystem the website is being hosted  from 