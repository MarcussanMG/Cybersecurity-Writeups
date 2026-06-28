---
Category: XSS
LAB: https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded
Difficulty: Apprentice
---

---
# Reflected XSS Definition

Reflected cross-site scripting (or XSS) arises when an application receives data in an HTTP request and includes that data within the immediate response in an unsafe way.

Suppose a website has a search function which receives the user-supplied search term in a URL parameter:

```
https://insecure-website.com/search?term=gift
```

The application echoes the supplied search term in the response to this URL:

```
<p>You searched for: gift</p>
```

Assuming the application doesn't perform any other processing of the data, an attacker can construct an attack like this:

```
https://insecure-website.com/search?term=<script>/*+Bad+stuff+here...+*/</script>
```

This URL results in the following response:

```
<p>You searched for: <script>/* Bad stuff here... */</script></p>
```

If another user of the application requests the attacker's URL, then the script supplied by the attacker will execute in the victim user's browser, in the context of their session with the application.

- Something worth noting is that for testing purposes while searching for XSS vulnerabilities, we don't use harmful payloads. An example would be

```
<script>alert("1")</script>
```

- This is the most basic kind of `XSS` and it is without counting Filters, WAF's ...
	- If the vulnerability exists and the code runs it will show a pop up with the number 1 written in it.


[PortSwigger Explanation](https://portswigger.net/web-security/cross-site-scripting/reflected)

---

# Lab


So this is what we encounter when we enter the lab

![|407x431](../../../0.%20Assets/Reflected%20XSS%20into%20HTML%20context%20with%20nothing%20encoded-1782660007253.webp)

As we mentioned in the definition of the vulnerability, we know `Reflected XSS` occurs normally in `input` tags or `URL's` so let's do some testing in the input field and the URL

- Let's also remember we will not use harmful payloads.

![](../../../0.%20Assets/Reflected%20XSS%20into%20HTML%20context%20with%20nothing%20encoded-1782663417344.webp)

And as we can see we probe the alert confirming the vulnerability

![](../../../0.%20Assets/Reflected%20XSS%20into%20HTML%20context%20with%20nothing%20encoded-1782663430963.webp)

