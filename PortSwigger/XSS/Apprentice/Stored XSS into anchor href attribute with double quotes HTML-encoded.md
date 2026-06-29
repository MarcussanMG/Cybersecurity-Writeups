---
Category: XSS
LAB: https://portswigger.net/web-security/cross-site-scripting/contexts/lab-href-attribute-double-quotes-html-encoded
Difficulty: Apprentice
---
---
So we find a blog where we can add comments, perfect place for `stored XSS`

Once we add a comment we can see the `website` filed fills up the `href` attribute in the `a` tag, something very interesting is that `href` not only accepts protocols like `http/https` but others like `javascript`

![](../../../0.%20Assets/Stored%20XSS%20into%20anchor%20href%20attribute%20with%20double%20quotes%20HTML-encoded-1782693380102.webp)

So we are going to take advantage of this by calling another protocol (JavaScript in this case) and adding our payload  

![](../../../0.%20Assets/Stored%20XSS%20into%20anchor%20href%20attribute%20with%20double%20quotes%20HTML-encoded-1782693552953.webp)

once we post the comment, we will see our name now has a hyperlink


![](../../../0.%20Assets/Stored%20XSS%20into%20anchor%20href%20attribute%20with%20double%20quotes%20HTML-encoded-1782693586322.webp)

If we open the source code

![](../../../0.%20Assets/Stored%20XSS%20into%20anchor%20href%20attribute%20with%20double%20quotes%20HTML-encoded-1782693644653.webp)


and if we click it

![](../../../0.%20Assets/Stored%20XSS%20into%20anchor%20href%20attribute%20with%20double%20quotes%20HTML-encoded-1782693595818.webp)


