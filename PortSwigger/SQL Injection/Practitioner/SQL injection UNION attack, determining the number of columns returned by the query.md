---
Category: SQL-Injection
LAB: https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns
Difficulty: Practitioner
---
---


This is a very simple lab, as we explored in different labs we are going to be using the `UNION` statement in `SQL`

`UNION` joins two result tables vertically, matching values by column position.

So both `SELECT`s need the same number of columns, otherwise SQL would not know where to place the extra/missing value.

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query-1784408296696.webp)

We could try manually by for example `union select NULL, NULL ..` and add `NULl` recursively until we don't encounter an error in the website but there is an easier way and that is to use the `Order by <number of columns>` query

`ORDER BY 3` tells SQL to sort by result column 3.

If there are only 2 columns, `ORDER BY 3` errors because that column does not exist.  
Therefore, the highest `ORDER BY n` that works is the number of returned columns.

>It's important to note that if the number is bigger we will receive the error if it is smaller than the number of columns it will be okay because that is a column we can order by so we need to find the biggest number that doesn't trigger an error


So let's try this first


![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query-1784408544487.webp)

This looks like a possible vulnerable field ill start by using the following payload:

```
' order by 4 --
```

remember:
- `'` | finishes our input and the rest of the input becomes `SQL`
- `order by 4` | our `SQL statement`
- `--` | comment in `SQL` so the rest of the query doesn't execute

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query-1784408749783.webp)

it is `URL encoded` that's why the `%27 ...` appear

Now we will try with a smaller number

```
' order by 3 --
```

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query-1784408814428.webp)

and we didn't receive an error so now we can use `union` and in this case to fill the columns we will use `NULL` because we are making the union to an inexistent table, this will change in further labs

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query-1784408887247.webp)![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query-1784408908575.webp)

And there you go now we know the table the website is working with, has 3 columns.