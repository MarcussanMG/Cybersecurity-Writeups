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

So let's try this first



