---
Category: XSS
LAB: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink-inside-select-element
Difficulty: Practitioner
---
---
![](../../../0.%20Assets/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element-1782732323288.webp)

As we can see we can see the `productId` on the `URL` let's see on the source code what we can find regarding that.

`Control+U` 

Instead of focusing on that we can see that the JavaScript checks for the `storeID` parameter

![](../../../0.%20Assets/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element-1782927121916.webp)

We can use `Burp Suite` to figure out what that is

![](../../../0.%20Assets/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element-1782928614270.webp)

![](../../../0.%20Assets/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element-1782928666711.webp)

Great so we know the options in the `select` are the `StoreID` and that the JavaScript looks in the URL for that parameter, and then writes 

[Document.write explanation](https://developer.mozilla.org/en-US/docs/Web/API/Document/write)

![](../../../0.%20Assets/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element-1782732547092.webp)

Cool, so we know that what  we input in the URL then gets parsed by JavaScript and creates an HTML tag but if we look deeper we will see that it only does it with the `storeId` so, let's add to the URL with this payload for testing if we can inject new HTML in the website

we will add this to the `URL`\

```
&storeId=<input string or numbers>
```

![](../../../0.%20Assets/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element-1782927049842.webp)

As we discovered before it creates a new option which is scary because if there is no sanitization we will be able to exploit this adding our own code

![](../../../0.%20Assets/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element-1782927056080.webp)

Cool, so we have an `XSS` vulnerability here, lets try with some simple payloads:

```
<script>alert(1)</script>
```

![](../../../0.%20Assets/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element-1782927222038.webp)

![](../../../0.%20Assets/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element-1782927225742.webp)

And in this case it was that simple payload