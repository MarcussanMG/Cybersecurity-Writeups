---
Category: Access Control
LAB: https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter
Difficulty: Apprentice
---
---
For this lab, you want to open `Burp Suite` and intercept the `GET` request to the `/admin` directory

Observe that the response sets the cookie `Admin=false`. Change it to `Admin=true`

And then you will be able to delete the user carlos

