---
Category: SQL-Injection
LAB: https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column
Difficulty: Practitioner
---

---

We are going to be following the same principles as in this lab:

- [SQL injection UNION attack, retrieving data from other tables](PortSwigger/SQL%20Injection/Practitioner/SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables.md)
	- https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables

So in this lab Ill go over what we have done already shortly and move 

this is the description of the lab:

>The database contains a different table called `users`, with columns called `username` and `password`.
>To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the `administrator` user.


First we will find a vulnerable point in the application, in this case the filtering of categories looks like a possible entry:


![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column-1784449346678.webp)


Next we will start figuring out the amount of columns the table we are trying to retrieve data contains. For this (remember past labs) we use `Order by`

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column-1784449364630.webp)

```
' order by 2 --
```

In this case I tried different numbers for the payload and came to the conclusion that 2 was the right amount of columns

now we will proceed by identifying the vulnerable field

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column-1784450018213.webp)

In this case it appears to be the second field

```
' Union select NULL, 'test' --
```

and now we need to exfiltrate the data from that vulnerable field, in my case I used this payload:

```
' union select NULL, username || ':' || password   from users--
```

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column-1784450116059.webp)

basically we `concatenated` 2 results into one column and separated them with `:` to do this in `SQL` you need to use the `|| <separator> ||` syntax

| ***Username***  |     ***Password***     |
| :-------------: | :--------------------: |
| `administrator` | `fst838iduev8f56fdzqv` |
|     carlos      |  me299p6cfr7e5kyrf81e  |
|     wiener      |  ljj8obqgbrskm3mouf7t  |

And to solve the lab we need to log-in as the `administrator`, so go into `"my account"` and log in with the credentials we found

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column-1784450261409.webp)


![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column-1784450273836.webp)

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column-1784450282080.webp)