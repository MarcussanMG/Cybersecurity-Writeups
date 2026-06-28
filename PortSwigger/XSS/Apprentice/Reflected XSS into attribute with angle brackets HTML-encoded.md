---
Category: XSS
LAB: https://0aca00fe03cb9dab814c9eb500a700e6.web-security-academy.net/?search=hello%22%3E
Difficulty: Apprentice
---
---

In this lab, the problem is that whenever we try to input a `> or <` it is going to be HTML-encoded, here is an example

![](../../../0.%20Assets/Reflected%20XSS%20into%20attribute%20with%20angle%20brackets%20HTML-encoded-1782687532509.webp)

So we know it is not going to be possible to insert  `<>`, or at least pretty hard, and the angled brackets are necessary to enter HTML tags, so we need to find another way.

We know that the `Input` looks like this

```
<input type="" placeholder="" name="search" value="">
```

So we know 3 things now:

1. Angled brackets are out of band
2. We are not inside of a `<script>` tag
3. We are not inside of a `comment`

So the one thing we can do is typing this :

```
"onmouseover="alert(1)
```


![](../../../0.%20Assets/Reflected%20XSS%20into%20attribute%20with%20angle%20brackets%20HTML-encoded-1782688710321.webp)

![](../../../0.%20Assets/Reflected%20XSS%20into%20attribute%20with%20angle%20brackets%20HTML-encoded-1782688777504.webp)
