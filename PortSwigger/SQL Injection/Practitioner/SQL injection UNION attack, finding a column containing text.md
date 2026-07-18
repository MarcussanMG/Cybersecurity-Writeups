---
Category: SQL-Injection
LAB: https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text
Difficulty: Practitioner
---
---

This is very similar to the previous lab but in this case we will also need to determine in which of the fields we encounter as positions in the table is actually injectable, because not all of them might be. 

We will do this by following the previous lab ([lab](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns)) and trying the different fields with a random `string`

In this case to make it faster I won't show the testing of finding out what number of columns exist and move directly to show the number and the payload i used and how to figure out which input is injectable

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text-1784409282762.webp)

I used the following payload

```
' order by 3 -- 
```

And determined that there are 3 columns

Knowing this we would substitute the previous payload with

```
' union select NULL, NULL ... -- # with the amount we figured previously

# in this case

' union select NULL, NULL, NULL -- 
```

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text-1784409411789.webp)

And as we can appreciate we don't get an error so we can proceed, we just need to substitute one by one each `NULL` for a `string`

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text-1784409472458.webp)

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text-1784409478556.webp)


In this case the third input is not injectable so let's try the other 2.

And as we can appreciate it was the one on the middle

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text-1784409574280.webp)

and in this case we need to retrieve this exact string:

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text-1784409639449.webp)


![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text-1784409659211.webp)

```
' union select NULL, 'P9vcbf ', NULL --
```

![](../../../0.%20Assets/SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text-1784409676493.webp)
