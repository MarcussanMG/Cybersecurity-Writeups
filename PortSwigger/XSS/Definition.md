---

---

---

**Cross-Site Scripting (XSS)** is a web vulnerability that allows an attacker to inject **malicious JavaScript** into a web application. When another user visits the vulnerable page, the injected script executes **in the victim's browser** with the permissions of that website.

### Simple example

Imagine a comment section that doesn't sanitize user input.

Instead of posting:

```
Hello everyone!
```

An attacker posts:

```
<script>alert(document.cookie)</script>
```

When another user views the comment, their browser executes:

```
alert(document.cookie);
```

The attacker could replace `alert()` with code that:

- Steals session cookies
- Redirects users to phishing pages
- Logs keystrokes
- Performs actions as the victim (if protections are missing)
- Modifies the webpage content

### Main types of XSS

- **Reflected XSS** – The payload comes from the HTTP request and is reflected immediately in the response.
- **Stored XSS** – The payload is stored on the server (e.g., comments, forums, profiles) and executed whenever users view it.
- **DOM-Based XSS** – The vulnerability exists entirely in client-side JavaScript, where unsafe DOM manipulation causes execution.

### Example payload

```
<script>alert('XSS')</script>
```

### Prevention

- Validate and sanitize user input.
- Encode output based on the context (HTML, JavaScript, URL, attributes).
- Use safe DOM APIs (`textContent` instead of `innerHTML` when possible).
- Implement a strong Content Security Policy (CSP).
- Mark cookies as `HttpOnly` to reduce the impact of cookie theft.

> **Key idea:** XSS is **JavaScript execution in another user's browser because a web application improperly handles untrusted input.**