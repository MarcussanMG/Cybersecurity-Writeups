---
Category: OSCP - TjNull
LAB: https://app.hackthebox.com/machines/Editorial?tab=play_machine
Difficulty: Easy
Featured:
aliases:
  - Linux
---

---
# Information / Description

![](../../0.%20Assets/Soccer-1791726718606.webp)

![203](../../0.%20Assets/Soccer-1791726727733.webp)

---
# Walkthrough


```
nmap 10.129.166.127 -sS -Pn -n -p- -T5 -oG openPorts
```

```
nmap $T -sS -Pn -n -sVC --min-rate 5000 -oN results.txt -p 22,80
```

![](../../0.%20Assets/Editorial-1791728440064.webp)

![](../../0.%20Assets/Editorial-1791730112356.webp)

it automatically redirects here and it doesnt' work unless you add it to the `/etc/hosts`

![](../../0.%20Assets/Editorial-1791730132483.webp)

