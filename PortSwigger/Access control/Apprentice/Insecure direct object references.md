---
Category: Access Control
LAB: https://portswigger.net/web-security/access-control/lab-insecure-direct-object-references
Difficulty: Apprentice
---
---
# Lab description

This lab stores user chat logs directly on the server's file system, and retrieves them using static URLs.

Solve the lab by finding the password for the user `carlos`, and logging into their account.

---
# Walkthrough


So this lab has a live chat where you can download the transcript of the chat

![](../../../0.%20Assets/Insecure%20direct%20object%20references-1784477784860.webp)

![](../../../0.%20Assets/Insecure%20direct%20object%20references-1784477999940.webp)

This downloads a file with a number as its name and it happens to be the transcript of the chat at that certain moment


![](../../../0.%20Assets/Insecure%20direct%20object%20references-1784477751661.webp)


I figured that there was no number 1 or 0, it directly started at 2

So i changed the file name to `1.txt`

![](../../../0.%20Assets/Insecure%20direct%20object%20references-1784477971441.webp)

Now that we know the password we just need to log as `carlos`

![](../../../0.%20Assets/Insecure%20direct%20object%20references-1784478084200.webp)