---
Category: Access Control
LAB: https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url
Difficulty: Apprentice
---
---

In this lab I tried looking for the `/robots.txt` to see if the the location of the admin panel was disclosed

![](../../../0.%20Assets/Unprotected%20admin%20functionality%20with%20unpredictable%20URL-1783010398798.webp)

But it wasn't so i checked the source code of the website with `Control + u`


![](../../../0.%20Assets/Unprotected%20admin%20functionality%20with%20unpredictable%20URL-1783010468413.webp)

![](../../../0.%20Assets/Unprotected%20admin%20functionality%20with%20unpredictable%20URL-1783010489864.webp)

And I saw this `Script tag` showing off some functionality, in this case it's some JavaScript that checks if you are an admin, and if you are it creates a new tag that redirects to the admin panel, lucky for us the `path` to the admin panel is disclosed so we can try accessing it 

![](../../../0.%20Assets/Unprotected%20admin%20functionality%20with%20unpredictable%20URL-1783010609388.webp)

And because there is a vulnerability and there is no access control we access directly as `"admins"`

Finally we can delete the user `carlos` from here