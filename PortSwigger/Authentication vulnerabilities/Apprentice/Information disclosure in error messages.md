---
Category: Information disclosure
LAB: https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-in-error-messages
Difficulty: Apprentice
---
---

This is the description for the lab:

>This lab's verbose error messages reveal that it is using a vulnerable version of a third-party framework. To solve the lab, obtain and submit the version number of this framework.

So lets see what we can find

In this case we can see that when we enter one of the products, we can see there is a function called `productId` and it is expecting a number, what if we give a string instead?

![](../../../0.%20Assets/Information%20disclosure%20in%20error%20messages-1784471248919.webp)


And that is the answer

