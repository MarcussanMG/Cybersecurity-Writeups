---
Category: SQL-Injection
LAB: https://portswigger.net/web-security/sql-injection/blind/lab-time-delays
Difficulty: Practitioner
Featured:
---
---

# Information / Description

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

To solve the lab, exploit the SQL injection vulnerability to cause a 10 second delay.

---

# Walkthrough

Okay so we know the cookie is vulnerable, let's start there:

I tried the following payload

```
'sleep(10) --
```

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20time%20delays-1784579211724.webp)

and didn't work, maybe it's because it is the wrong type of database and it is not `MySQL` let's try with other implementations of sleep from other databases

```
'pg_sleep(10) --
```

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20time%20delays-1784579280251.webp)

And this didn't work either

```
'||pg_sleep(10) --
```

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20time%20delays-1784579430892.webp)


> And this one did work, this is because postgres has this method for concatenating