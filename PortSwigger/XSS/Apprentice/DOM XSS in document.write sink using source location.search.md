---
Category: XSS
LAB: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink
Difficulty: Apprentice
---
---
# Explanation

A **DOM XSS (Document Object Model Cross-Site Scripting)** vulnerability occurs when **client-side JavaScript** takes untrusted input (such as a URL parameter) and inserts it into the page in an unsafe way. Unlike reflected or stored XSS, the **server is not responsible** for injecting the payload.

### How it works

1. The attacker crafts a malicious URL.
2. The victim opens the URL.
3. JavaScript on the page reads the malicious input (e.g., from `location.search` or `location.hash`).
4. The script inserts the input into the DOM using an unsafe method like `innerHTML` or `document.write()`.
5. The browser executes the injected JavaScript.

### Example

```
const search = new URLSearchParams(location.search).get("search");document.getElementById("results").innerHTML = search;
```

If the victim visits:

```
https://example.com/?search=<img src=x onerror=alert(1)>
```

The browser executes:

```
<div id="results">    <img src="x" onerror="alert(1)"></div>
```

### Key difference

- **Reflected XSS:** The **server** injects the payload into the HTML response.
- **Stored XSS:** The **server** stores and later serves the payload to users.
- **DOM XSS:** The **browser's JavaScript** creates the vulnerability by modifying the DOM with untrusted data. The malicious payload is **never part of the server's HTML response**.

# LAB


![](DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search-1782673329097.webp)

so as we can see here, what we look for in the search bar appears reflected in the `DOM`, trying to fetch a file with the name of what we looked for.

- Something worth noting is that what we are typing is already being reflected inside of an `HTML tag`,  and this is done in an insecure way, its java script doing it directly in the DOM
-

So what we are going to do is to close the tag where the `img` tag closes so we can add a different type of tag, in this case `script` and then our script


![](DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search-1782673542112.webp)

This is how it would look

![](../../../0.%20Assets/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search-1782674382825.webp)

- As we can see we are closing the `img` tag with our input, so we can then add our own tag



![](DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search-1782673548262.webp)
