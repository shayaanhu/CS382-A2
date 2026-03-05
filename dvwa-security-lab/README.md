# DVWA Security Lab Report
**Course:** CS382 – Cybersecurity  
**Tool:** Damn Vulnerable Web Application (DVWA) via Docker  

---

## Environment Setup

- **Docker Image:** `vulnerables/web-dvwa`
- **Container Port:** `8080:80`
- **Access URL:** `http://localhost:8080`
- **Login:** `admin` / `password`

```bash
docker pull vulnerables/web-dvwa
docker run -d --name dvwa -p 8080:80 vulnerables/web-dvwa
```

---

## Vulnerabilities Tested

1. [Brute Force](#1-brute-force)
2. [Command Injection](#2-command-injection)
3. [CSRF](#3-csrf)
4. [File Inclusion](#4-file-inclusion)
5. [File Upload](#5-file-upload)
6. [Insecure CAPTCHA](#6-insecure-captcha)
7. [SQL Injection](#7-sql-injection)
8. [SQL Injection (Blind)](#8-sql-injection-blind)
9. [Weak Session IDs](#9-weak-session-ids)
10. [XSS – DOM](#10-xss--dom)
11. [XSS – Reflected](#11-xss--reflected)
12. [XSS – Stored](#12-xss--stored)
13. [JavaScript](#13-javascript)
14. [CSP Bypass](#14-csp-bypass)

---

## 1. Brute Force

### 🔴 Low Security
**Tool/Method:** Manual entry (simulating a dictionary attack).  
**Result:** Successfully logged in using `admin` / `password`.  
**Screenshot:** ![Brute Force Low](screenshots/bruteforce_low.png)  
**Why it worked:** There are no server-side protections. The application processes the request immediately, allowing an attacker to automate thousands of login attempts per second without any lockout or delay.

---

### 🟡 Medium Security
**Tool/Method:** Manual entry.  
**Result:** Successful login, but failed attempts resulted in a noticeable delay.  
**Screenshot:** ![Brute Force Medium](screenshots/bruteforce_med.png)  
**Why it was harder:** The server-side script includes a `sleep(2)` function call whenever a login attempt fails. This adds a 2-second delay to every incorrect guess.  
**Why it still worked:** While the delay slows down a brute force attack significantly, it does not actually stop it. An attacker with enough time can still eventually guess the password.

---

### 🟢 High Security
**Tool/Method:** Manual entry.  
**Result:** Successful login, but requires a unique CSRF token per attempt.  
**Screenshot:** ![Brute Force High](screenshots/bruteforce_high.png)  
**Why it failed / mitigation used:** The High security level implements a CSRF token (anti-automation token) and a random `sleep()` delay. The token must be scraped from the login page and submitted with the credentials, which breaks simple automated brute-force scripts. Additionally, the random delay makes it harder for attackers to predict response times.

---

## 2. Command Injection

### 🔴 Low Security
**Payload:** `127.0.0.1; whoami; pwd; ls`  
**Result:** Displays the ping results followed by the username `www-data`, the current directory `/var/www/html/vulnerabilities/exec`, and the list of files in that directory.  
**Screenshot:** ![Command Injection Low](screenshots/command_injection_low.png)  
**Why it worked:** The application directly pipes the user input into the `shell_exec()` function. By using the `;` separator, we can terminate the ping command and execute our own commands (`whoami`, `pwd`, `ls`).

---

### 🟡 Medium Security
**Payload:** `127.0.0.1 & ls`  
**Result:** Displays the ping results, followed sequentially by the output of the `ls` command (which shows `help`, `index.php`, and `source`).  
**Screenshot:** ![Command Injection Medium](screenshots/command_injection_med.png)  
**Why it was harder:** The developer implemented a blacklist that removes `&&` and `;`. If you use the Low payload, the `;` is removed, and the command becomes broken.  
**Why it still worked:** The blacklist was incomplete. It didn't filter the single `&` character, which allows us to run commands asynchronously.

---

### 🟢 High Security
**Payload:** `127.0.0.1 |ls` (Critical: **No space** after the pipe)  
**Result:** Displays only the output of `ls` (`help`, `index.php`, `source`). The ping output is piped away.  
**Screenshot:** ![Command Injection High](screenshots/command_injection_high.png)  
**Why it failed / mitigation used:** The developer expanded the blacklist to include `&`, `;`, `|`, `-`, `$`, etc. However, there is a typo in the blacklist: they filtered `'| '` (pipe followed by a space) instead of just `'|'`.  
**Why it still worked:** By removing the space between the pipe and our command, we bypass the filter and execute the command. This demonstrates that blacklisting is extremely difficult to get right.

---

## 3. CSRF

### 🔴 Low Security
**Payload:** `http://localhost:8080/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change`  
**Result:** The admin password is changed to `hacked` without the user submitting the form — just by visiting the crafted URL.  
**Screenshot:** ![CSRF Low](screenshots/csrf_low.png)  
**Why it worked:** The application processes GET parameters directly using the session cookie, with no check that the request came from the actual form. An attacker can forge a request by crafting a URL.

---

### 🟡 Medium Security
**Method:** The `Referer` header check — bypassed using a crafted HTML form pointed at DVWA.  
**Result:** Password change is blocked when using the raw URL (Referer doesn't match), but can be bypassed.  
**Screenshot:** ![CSRF Medium](screenshots/csrf_med.png)  
**Why it was harder:** The server now checks the HTTP `Referer` header to ensure the request originated from the same site.  
**Why it still worked:** The `Referer` check only verifies that the string `localhost` appears anywhere in the Referer URL. An attacker can host a malicious page on a site like `http://localhost.evil.com/` and still bypass this check.

---

### 🟢 High Security
**Method:** CSRF token required per request.  
**Result:** Attack is blocked — each request requires a unique, unpredictable CSRF token.  
**Screenshot:** ![CSRF High](screenshots/csrf_high.png)  
**Why it failed / mitigation used:** High security generates a unique `user_token` for every page load that is tied to the session. Without scraping the page first to obtain this token, any forged request will be rejected by the server.

---

## 4. File Inclusion

### 🔴 Low Security
**Payload:**  
**Result:**  
**Screenshot:** ![File Inclusion Low](screenshots/file_inclusion_low.png)  
**Why it worked:**  

---

### 🟡 Medium Security
**Payload:**  
**Result:**  
**Screenshot:** ![File Inclusion Medium](screenshots/file_inclusion_med.png)  
**Why it was harder:**  

---

### 🟢 High Security
**Payload:**  
**Result:**  
**Screenshot:** ![File Inclusion High](screenshots/file_inclusion_high.png)  
**Why it failed / mitigation used:**  

---

## 5. File Upload

### 🔴 Low Security
**Method:**  
**Result:**  
**Screenshot:** ![File Upload Low](screenshots/file_upload_low.png)  
**Why it worked:**  

---

### 🟡 Medium Security
**Method:**  
**Result:**  
**Screenshot:** ![File Upload Medium](screenshots/file_upload_med.png)  
**Why it was harder:**  

---

### 🟢 High Security
**Method:**  
**Result:**  
**Screenshot:** ![File Upload High](screenshots/file_upload_high.png)  
**Why it failed / mitigation used:**  

---

## 6. Insecure CAPTCHA

### 🔴 Low Security
**Method:**  
**Result:**  
**Screenshot:** ![Insecure CAPTCHA Low](screenshots/captcha_low.png)  
**Why it worked:**  

---

### 🟡 Medium Security
**Method:**  
**Result:**  
**Screenshot:** ![Insecure CAPTCHA Medium](screenshots/captcha_med.png)  
**Why it was harder:**  

---

### 🟢 High Security
**Method:**  
**Result:**  
**Screenshot:** ![Insecure CAPTCHA High](screenshots/captcha_high.png)  
**Why it failed / mitigation used:**  

---

## 7. SQL Injection

### 🔴 Low Security
**Payload:**
```sql
1' OR '1'='1
```
**Result:** Successfully dumped the entire database of users (Usernames and Passwords).  
**Screenshot:** ![SQL Injection Low](screenshots/sql_injection_low.png)  
**Why it worked:** The application directly concatenates user input into the SQL query without any sanitization or use of prepared statements. The `'` character closes the original string, and `OR '1'='1` makes the WHERE clause always true.

---

### 🟡 Medium Security
**Payload:** `1 OR 1=1` (sent via edited HTML value)  
**Result:** Successfully dumped the database after bypassing the dropdown menu.  
**Screenshot:** ![SQL Injection Medium](screenshots/sql_injection_med.png)  
**Why it was harder:** Direct text input was blocked by a dropdown menu, and the PHP code used `mysql_real_escape_string()` to escape single quotes.  
**Why it still worked:** I used "Inspect Element" to modify the value of the dropdown option to a SQL payload. Since the developer forgot to put quotes around the `$id` variable in the SQL query (e.g., `WHERE user_id = $id`), the `mysql_real_escape_string()` function was useless against numeric-based injection.

---

### 🟢 High Security
**Payload:** `1' OR '1'='1` (entered in the popup window)  
**Result:** Successfully dumped the database from a separate input window.  
**Screenshot:** ![SQL Injection High](screenshots/sql_injection_high.png)  
**Why it failed / mitigation used:** The "High" level simply moved the input form to a separate popup page. This stops some automated scrapers but doesn't fix the underlying SQL injection vulnerability. The backend code remains vulnerable as it still doesn't use prepared statements.

---

## 8. SQL Injection (Blind)

### 🔴 Low Security
**Payload:**  
**Result:**  
**Screenshot:** ![SQL Injection Blind Low](screenshots/sqli_blind_low.png)  
**Why it worked:**  

---

### 🟡 Medium Security
**Payload:**  
**Result:**  
**Screenshot:** ![SQL Injection Blind Medium](screenshots/sqli_blind_med.png)  
**Why it was harder:**  

---

### 🟢 High Security
**Payload:**  
**Result:**  
**Screenshot:** ![SQL Injection Blind High](screenshots/sqli_blind_high.png)  
**Why it failed / mitigation used:**  

---

## 9. Weak Session IDs

### 🔴 Low Security
**Method:**  
**Result:**  
**Screenshot:** ![Weak Session IDs Low](screenshots/session_low.png)  
**Why it worked:**  

---

### 🟡 Medium Security
**Method:**  
**Result:**  
**Screenshot:** ![Weak Session IDs Medium](screenshots/session_med.png)  
**Why it was harder:**  

---

### 🟢 High Security
**Method:**  
**Result:**  
**Screenshot:** ![Weak Session IDs High](screenshots/session_high.png)  
**Why it failed / mitigation used:**  

---

## 10. XSS – DOM

### 🔴 Low Security
**Payload:**
```html
<script>alert('DOM XSS')</script>
```
**Result:**  
**Screenshot:** ![XSS DOM Low](screenshots/xss_dom_low.png)  
**Why it worked:**  

---

### 🟡 Medium Security
**Payload:**  
**Result:**  
**Screenshot:** ![XSS DOM Medium](screenshots/xss_dom_med.png)  
**Why it was harder:**  

---

### 🟢 High Security
**Payload:**  
**Result:**  
**Screenshot:** ![XSS DOM High](screenshots/xss_dom_high.png)  
**Why it failed / mitigation used:**  

---

## 11. XSS – Reflected

### 🔴 Low Security
**Payload:**
```html
<script>alert('XSS')</script>
```
**Result:**  
**Screenshot:** ![XSS Reflected Low](screenshots/xss_reflected_low.png)  
**Why it worked:**  

---

### 🟡 Medium Security
**Payload:**  
**Result:**  
**Screenshot:** ![XSS Reflected Medium](screenshots/xss_reflected_med.png)  
**Why it was harder:**  

---

### 🟢 High Security
**Payload:**  
**Result:**  
**Screenshot:** ![XSS Reflected High](screenshots/xss_reflected_high.png)  
**Why it failed / mitigation used:**  

---

## 12. XSS – Stored

### 🔴 Low Security
**Payload:**
```html
<script>alert('Stored XSS')</script>
```
**Result:**  
**Screenshot:** ![XSS Stored Low](screenshots/xss_stored_low.png)  
**Why it worked:**  

---

### 🟡 Medium Security
**Payload:**  
**Result:**  
**Screenshot:** ![XSS Stored Medium](screenshots/xss_stored_med.png)  
**Why it was harder:**  

---

### 🟢 High Security
**Payload:**  
**Result:**  
**Screenshot:** ![XSS Stored High](screenshots/xss_stored_high.png)  
**Why it failed / mitigation used:**  

---

## 13. JavaScript

### 🔴 Low Security
**Method:**  
**Result:**  
**Screenshot:** ![JavaScript Low](screenshots/javascript_low.png)  
**Why it worked:**  

---

### 🟡 Medium Security
**Method:**  
**Result:**  
**Screenshot:** ![JavaScript Medium](screenshots/javascript_med.png)  
**Why it was harder:**  

---

### 🟢 High Security
**Method:**  
**Result:**  
**Screenshot:** ![JavaScript High](screenshots/javascript_high.png)  
**Why it failed / mitigation used:**  

---

## 14. CSP Bypass

### 🔴 Low Security
**Method:**  
**Result:**  
**Screenshot:** ![CSP Bypass Low](screenshots/csp_low.png)  
**Why it worked:**  

---

### 🟡 Medium Security
**Method:**  
**Result:**  
**Screenshot:** ![CSP Bypass Medium](screenshots/csp_med.png)  
**Why it was harder:**  

---

### 🟢 High Security
**Method:**  
**Result:**  
**Screenshot:** ![CSP Bypass High](screenshots/csp_high.png)  
**Why it failed / mitigation used:**  

---

## Docker Inspection

```bash
docker ps
```
**Output:**  

```bash
docker inspect dvwa
```
**Output (key fields):**  

```bash
docker logs dvwa
```
**Output:**  

```bash
docker exec -it dvwa /bin/bash
ls /var/www/html
```
**Output:**  

**Analysis:**
- **Application files location:**  
- **Backend technology:**  
- **How Docker isolates the environment:**  

---

## Security Analysis

**Q1. Why does SQL Injection succeed at Low security?**  

**Q2. What control prevents it at High?**  

**Q3. Does HTTPS prevent these attacks? Why or why not?**  

**Q4. What risks exist if this app is deployed publicly?**  

**Q5. OWASP Top 10 Mapping:**

| Vulnerability | OWASP Top 10 Category |
|---|---|
| Brute Force | A07:2021 – Identification & Auth Failures |
| Command Injection | A03:2021 – Injection |
| CSRF | A01:2021 – Broken Access Control |
| File Inclusion | A05:2021 – Security Misconfiguration |
| File Upload | A04:2021 – Insecure Design |
| Insecure CAPTCHA | A07:2021 – Identification & Auth Failures |
| SQL Injection | A03:2021 – Injection |
| SQL Injection (Blind) | A03:2021 – Injection |
| Weak Session IDs | A07:2021 – Identification & Auth Failures |
| XSS (DOM / Reflected / Stored) | A03:2021 – Injection |
| JavaScript | A05:2021 – Security Misconfiguration |
| CSP Bypass | A05:2021 – Security Misconfiguration |

---
