---
Category: Access Control
LAB: https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality
Difficulty: Apprentice
---
---

This is a pretty simple lab, demonstrating an unprotected admin functionality
The goal of the lab is to delete the user `carlos`

To do this we need to access the admin panel, how do we do this?

We can access the `/robots.txt` to check for directories that the owner of the website does not want users to access or `Crawlers` to crawl

- [Robots.txt explanation](https://en.wikipedia.org/wiki/Robots.txt)

![](../../../0.%20Assets/Unprotected%20admin%20functionality-1783009475371.webp)

As we can see we find a directory called `administrator-panel`, let's try to enter and see if we get blocked or we are asked for credentials, if we don't then there is an `access control vulnerability`

![](../../../0.%20Assets/Unprotected%20admin%20functionality-1783009608409.webp)

As we can see there where no credentials needed, meaning there is a vulnerability and from here we can delete the user carlos.

