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

So as before I will skip what we have already done and only provide the payload used and then proceed with 

