# CTF: FTP → SSH → Root

## **Overview**

A timed exam box (CICCC) demonstrating a complete compromise: enumeration,anonymous FTP access, SSH brute forcing, user flag capture, and privilege
escalation to root via a sudo misconfiguration.

**Category:** Penetration Testing / CTF  
**Target:** CICCC exam machine ("simplectf")  
**Tools:** Nmap, FTP, Hydra  
**Privesc:** sudo vim (GTFOBins technique)

## **1. Enumeration**

### **Open Services**

Two things stood out: FTP (check for anonymous access) and SSH on a non-standard port (2222).

![image.png](image.png)

## **2. Anonymous FTP Access**

Anonymous login was accepted. Browsing to the `pub` directory:

![image.png](image%201.png)

![image.png](image%202.png)

This revealed a valid username — **mitch** — and hinted at a weak password.

## 3. SSH Brute Force

Targeting SSH on port 2222 with the known username:

![image.png](image%203.png)

![image.png](image%204.png)

### User Flag

![image.png](image%205.png)

![image.png](image%206.png)

![image.png](image%207.png)

## 4. Privilege Escalation

Mitch can run **vim** as root with no password. Vim can spawn a shell, so this is a direct path to root (a classic GTFOBins technique)

![image.png](image%208.png)

![image.png](image%209.png)

### Root Flag

![image.png](image%2010.png)

![image.png](image%2011.png)

## Attack Chain

nmap → anonymous FTP (user: mitch) → Hydra SSH (:2222) → user flag → sudo -l → sudo vim shell escape → root flag

## Key Takeaways

- Anonymous FTP leaked a username that seeded the entire attack

- A single overly-permissive sudo entry (`NOPASSWD: vim`) gave instant root

- Blue-team lessons: disable anonymous FTP, enforce strong passwords, and audit  sudo rules — binaries like vim, less, and find should never be blanket-allowed