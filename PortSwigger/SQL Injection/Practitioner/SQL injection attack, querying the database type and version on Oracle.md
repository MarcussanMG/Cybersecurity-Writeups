---
Category: SQL-Injection
LAB: https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle
Difficulty: Practitioner
---
---

# Introduction

Okay, so for this lab, we need to find a way of querying the database to retrieve the version of the `DBMS`

Let's start small by doing a quick look-up on the internet to figure out how to do that

I found this post: [post](https://www.techonthenet.com/oracle/questions/version.php)

Where they explain that you can do it this way:

```
SELECT * FROM v$version;
```

So now we know that `v$version` is what we need to query.

Also we know that `Oracle` databases need from a table to query from, if no table is mentioned, we won't be able to do the query.
- There is a table called `DUAL` that serves this matter.

Okay so we find a search functionality where what we select becomes an input in the URL, let's try tampering it to see if its a 

Great, so what we want to know is not in the realm of what the search the URL  is prepared for, so we need to get out of it to be able to query other tables (where the information we want is laying around)



![](../../../0.%20Assets/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle-1782861567471.webp)

How can we do this?

- We can exploit this by using the `UNION` parameter

![](../../../0.%20Assets/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle-1782861772285.webp)

This is a pretty stupid example but it was just a quick draw to visualize the idea, and it is also wrong, because of 1 problem.

- `UNION` statements, need to specify the same amount of columns as the first statement we are connecting it to , because it shows the results together, so if columns are missing or extra columns appear, it will be complicated for the database to represent it and instead it throws an error


![](../../../0.%20Assets/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle-1782862111565.webp)

So one thing we can do to solve this is to add `NULL` for example or a `'example test'` to fill those empty spaces.

This is where `error-based SQLi` takes its name from, we don't know how many columns the server is calling so it is a game of trial an error.

# Testing payloads

In this case I tried this payload to try to identify the amount of columns we are going to be working with

``` payload 
 UNION select NULL from dual -- -
```

![](../../../0.%20Assets/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle-1782862324650.webp)

> The URL looks like that because it is `URL encoded` we will get to that in another lab but for now think that it does it automatically.

Okay cool, so we know that we need more than 1 parameter (check the previous payload and if you look closely you will see that we are using `NULL` as a spacer and calling `Dual` to not get an error)
- You might be wondering for how long do we have to do this, and the answer is until you don't encounter an error anymore

This is the next payload i am going to try and it's a simple tweak, just adding another spacer:

```
' UNION select NULL, NULL from DUAL -- -
```

![](../../../0.%20Assets/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle-1782862671073.webp)

And there we go, we didn't encounter an error. 

The last thing we need to do is substitute one of the `NULL` parameters for our `v$version` and if it doesn't work try the other position because remember this:

![](../../../0.%20Assets/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle-1782862111565.webp)

We don't know where the space is , we know it's 2 positions now but don't which one might be open.

And apparently i forgot to mention something (I was doing the lab while writing this so i encountered some errors) 
- `v$version` This is the table 
	- `Banner` is what we want

and because `v$version` already is a table we don't need to call `Dual`

```
' UNION select banner, NULL from v$version -- -
```

![](../../../0.%20Assets/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle-1782863006700.webp)