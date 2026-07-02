---
Category: Access Control
LAB: https://portswigger.net/web-security/access-control/lab-user-role-can-be-modified-in-user-profile
Difficulty: Apprentice
---
---

# Description of lab

This lab has an admin panel at `/admin`. It's only accessible to logged-in users with a `roleid` of 2.

Solve the lab by accessing the admin panel and using it to delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

---

So what we are going to do is first log-into the website

![](../../../0.%20Assets/User%20role%20can%20be%20modified%20in%20user%20profile-1783027365254.webp)

Once we are logged in

![](../../../0.%20Assets/User%20role%20can%20be%20modified%20in%20user%20profile-1783027493432.webp)

try to access the `/admin` and in the request we intercepted with burp change the `roleid` to a `2` and then we complete the lab by deleting the user carlos