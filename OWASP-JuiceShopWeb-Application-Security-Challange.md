# OWASP Juice Shop — Web Application Security Challenges

## **Overview**

Hands-on web application security testing against OWASP Juice Shop — a
deliberately vulnerable training application maintained by OWASP for security
education. This lab covers five vulnerability categories from the OWASP Top 10.

**Category:** Web Application Security / Ethical Hacking  
**Target:** OWASP Juice Shop (intentionally vulnerable training app)  
**Tools:** Browser DevTools, CyberChef, ROT13  
**Focus:** OWASP Top 10

# 1. Sensitive Data Exposure - Confidential Document

Accessed the FTP directory by browsing to the `/ftp` path, revealing files not
meant to be public (including a confidential acquisitions document).

**Takeaway:** Directories exposed without access control leak sensitive files.
*Fix:* enforce authorization on all file paths; never rely on obscurity.

![image.png](image.png)

![image.png](image%201.png)

# 2. DOM-Based Cross-Site Scripting (XSS)

Triggered a DOM XSS challenge using an iframe-based JavaScript payload in the search field. The application rendered unsanitized input in the DOM.

**Takeaway:** Unsanitized user input reaching the DOM allows script execution.
*Fix:* sanitize/encode output; use a strict Content Security Policy (CSP)

`<iframe src="javascript:alert(xss)">`

![image.png](image%202.png)

# 3. SQL Injection — Authentication Bypass

Bypassed the login form using a classic SQLi authentication-bypass payload in the email field (`' OR '1'='1' --`), logging in as the administrator without valid credentials. Accessing the admin panel also revealed registered user data.

**Takeaway:** Unparameterized queries let attackers manipulate SQL logic.
*Fix:* use parameterized queries / prepared statements; never concatenate input.

![image.png](image%203.png)

![image.png](image%204.png)

![image.png](image%205.png)

![image.png](image%206.png)

![image.png](image%207.png)

![image.png](image%208.png)

# 4. Improper Input Validation - UNION-Based Data Extraction

A crafted product-search request using a UNION SELECT returned additional columns (including user email/password fields) in the JSON response - demonstrating how missing input validation exposes backend data.

**Takeaway:** Missing validation + injectable queries = data exfiltration.
*Fix:* parameterize queries, validate input, restrict returned columns.

![image.png](image%209.png)

# 5. Cryptographic Issues — Nested Easter Egg

A multi-stage decoding challenge:

1. Downloaded an encoded file from the FTP directory
2. Decoded it from **Base64** using CyberChef
3. Recognized the result as **ROT13** and decoded again
4. The final URL path revealed a hidden page

**Takeaway:** Encoding is not encryption. Base64 and ROT13 provide zero
confidentiality — they're trivially reversible.
*Fix:* never rely on encoding to protect data; use real cryptography.

![image.png](image%2010.png)

![image.png](image%2011.png)

![image.png](image%2012.png)

![image.png](image%2013.png)

![image.png](image%2014.png)

## **Key Takeaways**

Working through Juice Shop builds practical familiarity with the most common web vulnerabilities and, just as importantly, the defensive fixes for each. Seeing how a simple `' OR '1'='1'` bypasses authentication makes the case for parameterized queries far more concrete than theory alone.