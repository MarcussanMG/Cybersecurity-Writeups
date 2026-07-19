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

So we get 2 wordlists and we need to identify 