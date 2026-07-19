---
Category: Path Traversal
LAB: https://portswigger.net/web-security/file-path-traversal/lab-superfluous-url-decode
Difficulty: Practitioner
---
---
Same idea as before and we can find the the vulnerable point at the same location

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20with%20superfluous%20URL-decode-1784456206329.webp)

But now we need to URL encode

We will go to `Decoder` and `URL-encode` our `traversal`

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20with%20superfluous%20URL-decode-1784456262316.webp)

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20with%20superfluous%20URL-decode-1784456278867.webp)

seems like it doesn't like the file name encoded as well so let's change that

So let's encode everything but the file names

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20with%20superfluous%20URL-decode-1784456375583.webp)

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20with%20superfluous%20URL-decode-1784456397820.webp)

It seems like it doesn't like this either so let's try just encoding the `..`

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20with%20superfluous%20URL-decode-1784456430506.webp)

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20with%20superfluous%20URL-decode-1784456460058.webp)

It didn't like this either so lets try encoding the `/`

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20with%20superfluous%20URL-decode-1784456488425.webp)

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20with%20superfluous%20URL-decode-1784456508620.webp)

it didn't like this one either, lets try only encoding the part that traverses and not the folder containing the file directly

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20with%20superfluous%20URL-decode-1784456643848.webp)

Nope, so let's move one to double encoding because we know the website decodes once and if traverse is found it will strip it.

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20with%20superfluous%20URL-decode-1784456734057.webp)

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20with%20superfluous%20URL-decode-1784456764730.webp)

and it worked! I even double encoded the `/` determining the path inside the `etc` directory

![](../../../0.%20Assets/File%20path%20traversal,%20traversal%20sequences%20stripped%20with%20superfluous%20URL-decode-1784456786113.webp)