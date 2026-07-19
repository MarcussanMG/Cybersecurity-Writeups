---
Category: Authentication vulnerabilities
LAB: https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses
Difficulty: Apprentice
---
---

This is the description of the lab:

This lab is vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:

- [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)
- [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.

---

So we get 2 wordlists and we need to identify users in the website and the passwords associated to them.

For this we will use `intruder` to brute force possible users and then their passwords.

If we try to log-in with random credentials this will be the request we will get

![](../../../0.%20Assets/Username%20enumeration%20via%20different%20responses-1784468576343.webp)

Then only thing we need to do now is send it to the intruder and set the type of attack to `sniper`

![](../../../0.%20Assets/Username%20enumeration%20via%20different%20responses-1784468679671.webp)

Then just select the value for the username and set a new position for our payloads and paste the user wordlist in the payload configuration, then just press start attack.

We are looking for differences in the length of the response because all of the status codes will be `200` because the request is valid

![](../../../0.%20Assets/Username%20enumeration%20via%20different%20responses-1784468793147.webp)

In this case as we can appreciate it kindly tells us that the `Username` was invalid, but if we check one of the longer or different lengths in responses

![](../../../0.%20Assets/Username%20enumeration%20via%20different%20responses-1784468844583.webp)

We get an `Incorrect password`, so we can safely assume that the username is correct. Knowing correct usernames, now we can do the same thing but changing the payload position to the password and setting the payload list to  the password wordlist effectively doing a brute force attack, and setting the `username` to the username we found, in our case `oracle`

![](../../../0.%20Assets/Username%20enumeration%20via%20different%20responses-1784469015225.webp)

![](../../../0.%20Assets/Username%20enumeration%20via%20different%20responses-1784468986775.webp)

![](../../../0.%20Assets/Username%20enumeration%20via%20different%20responses-1784469072987.webp)

now we just open in browser (right click > open in browser)

![](../../../0.%20Assets/Username%20enumeration%20via%20different%20responses-1784469091423.webp)

![](../../../0.%20Assets/Username%20enumeration%20via%20different%20responses-1784469108954.webp)
