---
Category: OS Command Injection
LAB: https://portswigger.net/web-security/os-command-injection/lab-simple
Difficulty: Apprentice
---
---

To solve this lab we need to find a vulnerable command injection vulnerability in this case is disclosed in the description of the lab:

>This lab contains an OS command injection vulnerability in the product stock checker.
   The application executes a shell command containing user-supplied product and store IDs, and returns the raw output from the command in its response.
   To solve the lab, execute the `whoami` command to determine the name of the current user.

And they even disclose what we need to do to solve the lab.

Let's open `Burp` and see what this looks like:

![](../../../0.%20Assets/OS%20command%20injection,%20simple%20case-1784453374976.webp)

So when we check the stock, this is what the request looks like:

![](../../../0.%20Assets/OS%20command%20injection,%20simple%20case-1784453408287.webp)

and the lab tells us that one of the parameters is vulnerable, lets try with `productId` and as we know in Linux we can concatenate commands with `|` so we will modify the `productId` to:

```
1|whoami
```

![](../../../0.%20Assets/OS%20command%20injection,%20simple%20case-1784453722335.webp)

![](../../../0.%20Assets/OS%20command%20injection,%20simple%20case-1784454084018.webp)