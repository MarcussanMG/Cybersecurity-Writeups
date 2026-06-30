---
Category: SQL-Injection
LAB: https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle
Difficulty: Practitioner
---
---

Okay, so for this lab, we need to find a way of querying the database to retrieve the version of the `DBMS`

Let's start small by doing a quick look-up on the internet to figure out how to do that

I found this post: [post](https://www.techonthenet.com/oracle/questions/version.php)

Where they explain that you can do it this way:

```
SELECT * FROM v$version;
```

So now we know that `v$version` is what we need to query.

Also we know that `Oracle` databases need from a table to query from, if no table is mentioned, we won't be able to do the query.

Great, so what we want to know is not in the realm of what the search is prepared for, so we need to get out of it.


![](../../../0.%20Assets/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle-1782861567471.webp)

How can we do this?

- If we find a `SQLi vulnerability` we can exploit this by using the `UNION` parameter

![](../../../0.%20Assets/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle-1782861772285.webp)

This is a pretty stupid example but it was just a quick draw to visualize the idea, and it is also wrong, because of 1 problem.

- `UNION` statements, need to 