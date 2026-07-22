---
Category: Web
LAB: https://tryhackme.com/room/lofi
Difficulty: Easy
Featured:
aliases:
---

---
# Information / Description

Want to hear some lo-fi beats, to relax or study to? We've got you covered! 

  

**Access this challenge** by deploying both the vulnerable machine by pressing the green "Start Lab Machine" button located within this task, and the TryHackMe AttackBox by pressing the  "Start AttackBox" button located at the top-right of the page.

Navigate to the following URL using the AttackBox: [http://MACHINE_IP(opens in new tab)](http://machine_ip/) and find the flag in the **root of the filesystem.**

  

Check out similar content on TryHackMe:

- [LFI Path Traversal](https://tryhackme.com/r/room/filepathtraversal)
- [File Inclusion](https://tryhackme.com/room/fileinc)

**Note:** The web page does load some elements from external sources. However, they do not interfere with the completion of the room.



| Victim machine | 10.128.130.137 |
| -------------- | -------------- |


---

# Walkthrough


First thing we need to do is browse to the victims IP

![](../../0.%20Assets/Lo-Fi-1784731238815.webp)

When selecting one of the files we can see this

![](../../0.%20Assets/Lo-Fi-1784731322268.webp)