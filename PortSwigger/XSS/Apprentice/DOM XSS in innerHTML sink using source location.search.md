---
Category: XSS
LAB: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-innerhtml-sink
Difficulty: Apprentice
---
---


> When user input is assigned to **`innerHTML`**, the browser parses it as HTML and creates DOM elements. However, **`<script>` tags inserted through `innerHTML` are not executed** for security reasons. Therefore, you need to find another way to execute JavaScript using HTML that the browser will process.

The key takeaway isn't a specific payload—it's understanding that:

- `innerHTML` ✅ lets you inject HTML.
- `innerHTML` ❌ does **not** execute injected `<script>` tags.
- Therefore, you must look for other HTML constructs that can trigger JavaScript execution.

---
# Lab

