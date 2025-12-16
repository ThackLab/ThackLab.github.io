---
title: "THM: Neighbour"
date: 2025-12-17 04:25:00 +1000
categories: [TryHackMe, Easy]
tags: [Linux, http, IDOR]
---

![Logo](/assets/Images/Neighbour/Neighbour.png)

Check out our new cloud service, Authentication Anywhere. Can you find other user's secrets?

This room is all about insecure direct object references or (IDOR).

---

Running initial fast scan to identify open services:
```bash
sudo nmap -T4 -sS -p- --min-rate=10000 TARGET_IP
```

**Results**
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```
---

Heading to http://TARGET_IP we're met with a login page
![Login.php](/assets/Images/Neighbour/Loginphp.png)

---

Viewing the page source shows how to login
```html
<p>Don't have an account? Use the guest account! (<code>Ctrl+U</code>)</p>
<!-- use guest:guest credentials until registration is fixed. "admin" user account is off limits!!!!! 
```
---

Loging in via guest:guest shows a vulnerable path
```
http://TARGET_IP/profile.php?user=guest
```
---

Lets see what happens when we change user=guest to user=admin
![Flag](/assets/Images/Neighbour/neighbour-flag.png)

---

Thats all for this room folks.

---

**Room:** Neighbour | **Platform:** TryHackMe | **Difficulty:** Easy | **Access Method:** IDOR 
