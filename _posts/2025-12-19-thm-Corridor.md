---
title: "THM: Corridor"
date: 2025-12-19 12:34:00 +1000
categories: [TryHackMe, Easy]
tags: [IDOR, Endpoints, Hex, Hash, Web]
---

![Banner](/assets/Images/Corridor/banner.jpeg)

---

**SUMMARY:**
Corridor is a web based, IDOR lab where initial recon on the website shows an interactive corridor and the corrosponding hashes to each door. There are 13 doors in total with the first hash being the integer "1". Developers start counting from 0 so we MD5 hashed it and navigated to it, giving us the flag. 

---

Website:
![Site](/assets/Images/Corridor/doors.png)

Source Code:
![Source](/assets/Images/Corridor/source.png)
 
Flag:
![Flag](/assets/Images/Corridor/flag.png)

---

Thats all folks!

---

**Room:** Corridor | **Platform:** TryHackMe | **Difficulty:** Easy | **Focus:** IDOR
