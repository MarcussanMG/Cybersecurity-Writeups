---
Category: SQL-Injection
LAB: https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables
Difficulty: Practitioner
---
---

Following the same schema as the previous labs:

- [SQL injection UNION attack, determining the number of columns returned by the query](PortSwigger/SQL%20Injection/Practitioner/SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query.md)
- [SQL injection UNION attack, finding a column containing text](PortSwigger/SQL%20Injection/Practitioner/SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text.md)

But now for retrieving data, normally we wouldn't know the names of the tables and columns of the database we are attacking and we would need to query `information shema` -> [information](https://www.ibm.com/docs/en/informix-servers/14.10.0?topic=tables-information-schema)
but in this case what we need appears in the description of the lab:

>The database contains a different table called `users`, with columns called `username` and `password`.
>To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the `administrator` user.

So as before I will skip what we have already done and only provide the payload used and then proceed with the last part of the lab.


![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables-1784415217109.webp)

```
' order by 2 --
```


![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables-1784415244402.webp)

```
'union select NULL, 'test' -- -
```

And now that we found the number of columns and the vulnerable point, we can get the information we want in this case query the `users` table and retrieve the `usernames` and `passwords` to log in as `administrator`


![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables-1784415463544.webp)

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables-1784415472051.webp)

```
' union select NULL, username from users -- 
```

This will take the vulnerable input field and get the username, and with `UNION` remember we can also call a table not only input our own data so we call that column from the `users` table

and now we will do the same but for the passwords

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables-1784415547023.webp)

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables-1784415555334.webp)

```
' union select NULL, password from users -- 
```


and to finish the lab we just need to go into `My account` and log in with the credentials we found

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables-1784415598900.webp)

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables-1784415607277.webp)