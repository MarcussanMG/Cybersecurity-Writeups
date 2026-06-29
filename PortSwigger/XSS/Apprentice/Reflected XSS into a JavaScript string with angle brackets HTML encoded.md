---
Category: XSS
LAB: https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-html-encoded
Difficulty: Apprentice
---
---

This is a reflected XSS so input forms are our best friends

![](../../../0.%20Assets/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20HTML%20encoded-1782694506122.webp)

Here we have one

So, why don't we try a normal search first to test the functionality of the website

![](../../../0.%20Assets/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20HTML%20encoded-1782694615535.webp)

Great, so as we can see the input ends up in a `string` and then URI encodes it.

Very well, we will find success if we can get out of the string

I crafted this payload

```
';alert(1)//
```

![](../../../0.%20Assets/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20HTML%20encoded-1782695454954.webp)

And because JavaScript executes its code line by line, it will effectively create an empty variable and then run our code


![](../../../0.%20Assets/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20HTML%20encoded-1782694717461.webp)

