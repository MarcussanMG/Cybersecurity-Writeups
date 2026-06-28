---
Category: XSS
LAB: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-innerhtml-sink
Difficulty: Apprentice
---
---


> When user input is assigned to **`innerHTML`**, the browser parses it as HTML and creates DOM elements. However, **`<script>` tags inserted through `innerHTML` are not executed** for security reasons. Therefore, you need to find another way to execute JavaScript using HTML that the browser will process.

- `innerHTML` ✅ lets you inject HTML.
- `innerHTML` ❌ does **not** execute injected `<script>` tags.
- Therefore, you must look for other HTML constructs that can trigger JavaScript execution.

---
# Lab

So Knowing what i explained at the beginning, we know we have to find a way of executing JavaScript without calling the `<script>` tag.

One work around could be injecting an `<img>` tag that doesn't load the image and calling JavaScript when it doesn't load;

```
<img src=# onerror=alert("1")>
```

![](../../../0.%20Assets/DOM%20XSS%20in%20innerHTML%20sink%20using%20source%20location.search-1782686493965.webp)

Chain of events:

1. `innerHTML` creates an `<img>` element.
2. The browser tries to load the image from `src=#`.
3. The image fails to load.
4. The `onerror` event fires.
5. The JavaScript inside `onerror` executes.

So basically as I explained previously, we create an `img` tag (considering `innerHTML` does not run `<script>` {for security reasons} but allows other HTML tags) We make an error occur on purpose and when an error occurs we run our malicious payload

![](../../../0.%20Assets/DOM%20XSS%20in%20innerHTML%20sink%20using%20source%20location.search-1782686347682.webp)
