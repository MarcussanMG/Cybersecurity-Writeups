---
Category: Information disclosure
LAB: https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-via-backup-files
Difficulty: Apprentice
---
---

This is the description of the lab:

>This lab leaks its source code via backup files in a hidden directory. To solve the lab, identify and submit the database password, which is hard-coded in the leaked source code.

In this case we will look for `/robots.txt` where this type of hidden directory's are sometimes disclosed so that they are not indexed by the crawlers


![](../../../0.%20Assets/Source%20code%20disclosure%20via%20backup%20files-1784473395672.webp)
![](../../../0.%20Assets/Source%20code%20disclosure%20via%20backup%20files-1784473391535.webp)

And there we go 

Now we go into `/backup` and see if we can see any information disclosed

![](../../../0.%20Assets/Source%20code%20disclosure%20via%20backup%20files-1784473424515.webp)
![](../../../0.%20Assets/Source%20code%20disclosure%20via%20backup%20files-1784473441858.webp)

And now we just need to input this string into the box and we solve the lab

![](../../../0.%20Assets/Source%20code%20disclosure%20via%20backup%20files-1784473469240.webp)

