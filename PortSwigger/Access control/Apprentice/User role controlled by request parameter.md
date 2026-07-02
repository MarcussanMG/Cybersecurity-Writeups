---
Category: Access Control
LAB: https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter
Difficulty: Apprentice
---
---

First thing we will do is to log in with the credentials provided `wiener:peter`

![](../../../0.%20Assets/User%20role%20controlled%20by%20request%20parameter-1783017974887.webp)

Then we will move to the `/admin` directory 

![](../../../0.%20Assets/User%20role%20controlled%20by%20request%20parameter-1783018004173.webp)

Then you want to open `Burp Suite` and intercept the `GET` request to the `/admin` directory

Observe that the response sets the cookie `Admin=false`.

![](../../../0.%20Assets/User%20role%20controlled%20by%20request%20parameter-1783018042856.webp)

What you want to do is set it up to `True`

The problem now is that from BurpSuite you cant really press the button

![](../../../0.%20Assets/User%20role%20controlled%20by%20request%20parameter-1783018078662.webp)

But is you inspect the `Pretty` or `Raw` response you will realize that to delete `Carlos` you just have to send a specific request

![](../../../0.%20Assets/User%20role%20controlled%20by%20request%20parameter-1783018125359.webp)


So we will just do that request


![](../../../0.%20Assets/User%20role%20controlled%20by%20request%20parameter-1783018298815.webp)

And remember to set the cookie again

![](../../../0.%20Assets/User%20role%20controlled%20by%20request%20parameter-1783018364562.webp)

Alternatively you can also set the cookie from your browser
![](../../../0.%20Assets/User%20role%20controlled%20by%20request%20parameter-1783018354655.webp)

