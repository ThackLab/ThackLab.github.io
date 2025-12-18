---
title: "THM: TakeOver"
date: 2025-12-17 12:07:00 +1000
categories: [TryHackMe, Easy]
tags: [subdomain, enumeration, hostfile, certificate]
---

![Banner](/assets/Images/Takeover/banner.png)

This challenge revolves around subdomain enumeration. If you're still abit "FUZZY" on url's, i've got a breakdown below.

![URL](/assets/Images/Takeover/url-explain.png)

---

## Phase 1: Information Gathering

Standard port & service scan yielded:
```bash
PORT      STATE    SERVICE         VERSION
==================================================================
22        open     ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu
80        open     http            Apache httpd 2.4.41 ((Ubuntu))
443       open     ssl/http        Apache httpd 2.4.41 ((Ubuntu))
==================================================================
http-title: Did not follow redirect to https://futurevera.thm/
``` 

- Add host to /etc/hosts

---

## Phase 2: Enumeration

The description states that they're getting their rebuilding support. Quite often youll see support as a subdomain so add "support.futurevera.thm" to your /etc/hosts too.

Navigating to the URL, If we click on the view certificate:
![Warning](/assets/Images/Takeover/cert-warning.png)

We end up with yet another secret service.
![Cert](/assets/Images/Takeover/cert.png)

- Adding that to /etc/hosts yet again and navigating to the URL resolves to support.futurevera.thm. Not what we want.

- If you remember we had port 80 open so if we drop HTTPS to HTTP with the secret URL we're rewarded with the flag.

![Flag](/assets/Images/Takeover/flag.png)

---

Thats all folks!

---


**Room:** TakeOver | **Platform:** TryHackMe | **Difficulty:** Easy | **Flag Method:** Subdomain-Enumeration
