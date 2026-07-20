---
Category: SQL-Injection
LAB: https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses
Difficulty: Practitioner
Featured: yes
---
---

# Information / Description

This is the description of the lab

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and no error messages are displayed. But the application includes a `Welcome back` message in the page if the query returns any rows.

The database contains a different table called `users`, with columns called `username` and `password`. You need to exploit the blind SQL injection vulnerability to find out the password of the `administrator` user.

To solve the lab, log in as the `administrator` user.

## Hint

>You can assume that the password only contains lowercase, alphanumeric characters.

---

# Walkthrough

So from the description of the lab we know a couple things:

	1. Coockie = vulnerable input
	2. If error not welcome back message
	3. Table = users 
		1. columns = username & password
	4. User = administrator
		1. Password = lowercase & 0-9

Cool, we have quite some information on the table but we will work around it no worries.

Let's go by order and identify the cookie that is vulnerable, we will do this with burp suite.

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784546721199.webp)

As we can appreciate in the image, we get the `Welcome back!` message displayed so we know nothing is broken yet, my idea to figure out which of the two cookies is the vulnerable one is to add any `SQL` statements like `'` at the end of one or the other to figure out which one of the two is the problematic one.

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784546827019.webp)

Doesn't look like the `session` cookie is a problem let's try with the `TrackingId` cookie:

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784546880099.webp)

Bingo, we have vulnerable field.

Next, let's try injecting some `SQL` code to the cookie and not break the `Welcome back!` message in the process

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784546965467.webp)

```sql - payload
' and 'a' = 'a' --
```

In this case it is a simple Boolean query to test if we can work around the problem of the website breaking and we can, now we need to exfiltrate the data from the `users` table and get the password for the `administrator`
- We already know that information luckily so we can just go directly for the passwords


```
' and (select substring(password,1,1) from users where username='administrator') = 'a'
```


![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784547971769.webp)

If this statement equals true (meaning the first letter of the password is `a` ) then we will get the `welcome back!` message

## Note to remember

>We are using `substring` and comparing the result to something else because we don't get the information displayed from the database, it's a conditional response 

---
![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784547231131.webp)

Okay now we know the first letter of the password is not `a`. This would take a very long time so lets use the intruder to "brute force" the password

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784548155315.webp)

In the payload configuration, in the list i added `all the letters from the alphabet` and the `numbers from 0-9` also I selected the position for the payload to be the letter we where checking previously.

select `sniper attack` and the start the attack

Because all the requests will be correct all of them will be `200 OK` so we will know which of the requests was the good one (giving us the first letter of the password) when the length of the response changes (the `welcome back!` message appears)

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784548470162.webp)

There we go first letter is `u`, now we need to do the same, changing the `initial position` of the `substring` to 2, 3,4 ... until we reach a point where all of the responses `don't return welcome back`

## First position

| Password | u   |
| -------- | --- |

## Second position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784548600668.webp)

leave everything as is but change the position in the `intruder`

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784548636558.webp)


| Password | uq  |
| -------- | --- |

## Third position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784548698913.webp)


| Password | uqf |
| -------- | --- |

## Forth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784548740807.webp)

| Password | uqfq |
| -------- | ---- |


## Fifth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784548776970.webp)

| Password | uqfqt |
| -------- | ----- |

## Sixth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784548853175.webp)

| Password | uqfqt1 |
| -------- | ------ |

## Seventh position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784548888184.webp)

| Password | uqfqt1q |
| -------- | ------- |
## Eighth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549009012.webp)

| Password | uqfqt1qe |
| -------- | -------- |

## Nineth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549056792.webp)

| Password | uqfqt1qei |
| -------- | --------- |

## Tenth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549123706.webp)

| Password | uqfqt1qeit |
| -------- | ---------- |
## Eleventh position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549163515.webp)

| Password | uqfqt1qeitm |
| -------- | ----------- |

## Twelfth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549199783.webp)

| Password | uqfqt1qeitmg |
| -------- | ------------ |


## Thirteenth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549291259.webp)

| Password | uqfqt1qeitmg1 |
| -------- | ------------- |

## Fourteenth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549340869.webp)

| Password | uqfqt1qeitmg1a |
| -------- | -------------- |

## Fifteenth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549438140.webp)

| Password | uqfqt1qeitmg1a9 |
| -------- | --------------- |
## Sixteenth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549506781.webp)

| Password | uqfqt1qeitmg1a9i |
| -------- | ---------------- |

## Seventeenth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549550566.webp)


| Password | uqfqt1qeitmg1a9ig |
| -------- | ----------------- |

## Eighteenth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549655492.webp)


| Password | uqfqt1qeitmg1a9ig5 |
| -------- | ------------------ |

## Nineteenth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549694515.webp)

| Password | uqfqt1qeitmg1a9ig5g |
| -------- | ------------------- |

## Twentieth position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549766092.webp)

| Password | uqfqt1qeitmg1a9ig5ga |
| -------- | -------------------- |
## Twenty-first position

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549946930.webp)

And here we hit all no `welcome back!` so we know we are done with the password this being


| Password | uqfqt1qeitmg1a9ig5ga |
| -------- | -------------------- |

![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784549994075.webp)


![](../../../0.%20Assets/Blind%20SQL%20injection%20with%20conditional%20responses-1784550007165.webp)


---

# Disclamer

We could have done this with the `Cluster bomb attack` as well so we don't have to go one by one and also try to figure out the length of the password by doing `LENGTH(password)>"your number to test"` but i figured this things out mid way through the lab and the length after i finished it