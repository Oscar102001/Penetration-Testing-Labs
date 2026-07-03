# DVWA command injection

## **Overview**

Demonstration of OS command injection against DVWA (Damn Vulnerable Web Application) — a deliberately vulnerable app used for security training. The lab shows how unsanitized input in a "ping a device" feature can be abused to run arbitrary system commands.

**Category:** Web Application Security / Ethical Hacking  
**Target:** DVWA (intentionally vulnerable training app)  
**Vulnerability:** OS Command Injection (OWASP A03: Injection)

---

## **The Vulnerability**

DVWA's "Ping a device" form takes an IP address and passes it directly to a system  `ping` command without sanitization. By appending a command separator (`;`), an attacker can chain additional commands that the server executes.

### **`127.0.0.1; id`**

Returns user ID, group ID, and supplementary groups — confirming code execution
and the privilege level of the web service account.

![image.png](image.png)

### System information

**`127.0.0.1; uname -a`**

Reveals kernel name, hostname, kernel release/version, architecture, and OS.

![image.png](image%201.png)

### Read sensitive files

Lists local user accounts — useful for enumeration and identifying targets.

`127.0.0.1; cat /etc/passwd`

![image.png](image%202.png)

![image.png](image%203.png)

### Network connections

Displays active connections and listening services on the host.

**`127.0.0.1; netstat -antp`**

![image.png](image%204.png)

## Why This Works

The application concatenates user input into a shell command. The `;` separator terminates the intended `ping` command and starts a new one, so anything after it runs with the web server's privileges.

## Impact

Command injection is among the most severe web vulnerabilities — it can lead to full system compromise, data theft, lateral movement, and persistence.

---

## Remediation

- **Never** pass user input to a system shell
- Use language-native libraries instead of shell commands where possible
- If a system call is unavoidable, use strict allow-list input validation
(e.g. validate that input is a well-formed IP address)
- Avoid shell interpolation; use parameterized process execution APIs
- Run services with least privilege to limit blast radius