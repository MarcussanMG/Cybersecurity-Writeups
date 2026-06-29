---
Category: XSS
LAB: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink-inside-select-element
Difficulty: Practitioner
---
---
![](../../../0.%20Assets/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element-1782732323288.webp)

![](../../../0.%20Assets/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element-1782732509079.webp)

[Document.write explanation](https://developer.mozilla.org/en-US/docs/Web/API/Document/write)

![](../../../0.%20Assets/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element-1782732547092.webp)

Cool, so we know that what  we input in the URL then gets parsed by JavaScript and creates an HTML tag