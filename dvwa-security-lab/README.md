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
**Screenshot:** ![Brute Force Low](screenshots/brute-low.png)  
**Why it worked:** There are no server-side protections. The application processes the request immediately, allowing an attacker to automate thousands of login attempts per second without any lockout or delay.

---

### 🟡 Medium Security
**Tool/Method:** Manual entry.  
**Result:** Successful login, but failed attempts resulted in a noticeable delay.  
**Screenshot:** ![Brute Force Medium](screenshots/brute-medium.png)  
**Why it was harder:** The server-side script includes a `sleep(2)` function call whenever a login attempt fails. This adds a 2-second delay to every incorrect guess.  
**Why it still worked:** While the delay slows down a brute force attack significantly, it does not actually stop it. An attacker with enough time can still eventually guess the password.

---

### 🟢 High Security
**Tool/Method:** Manual entry.  
**Result:** Successful login, but requires a unique CSRF token per attempt.  
**Screenshot:** ![Brute Force High](screenshots/brute-high.png)  
**Why it failed / mitigation used:** The High security level implements a CSRF token (anti-automation token) and a random `sleep()` delay. The token must be scraped from the login page and submitted with the credentials, which breaks simple automated brute-force scripts. Additionally, the random delay makes it harder for attackers to predict response times.

---

## 2. Command Injection

### 🔴 Low Security
**Payload:**
```
127.0.0.1; whoami
```
**Result:**  
**Screenshot:**  
**Why it worked:**  

---

### 🟡 Medium Security
**Payload:**  
**Result:**  
**Screenshot:**  
**Why it was harder:**  

---

### 🟢 High Security
**Payload:**  
**Result:**  
**Screenshot:**  
**Why it failed / mitigation used:**  

---

## 3. CSRF

### 🔴 Low Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it worked:**  

---

### � Medium Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it was harder:**  

---

### 🟢 High Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it failed / mitigation used:**  

---

## 4. File Inclusion

### 🔴 Low Security
**Payload:**  
**Result:**  
**Screenshot:**  
**Why it worked:**  

---

### 🟡 Medium Security
**Payload:**  
**Result:**  
**Screenshot:**  
**Why it was harder:**  

---

### 🟢 High Security
**Payload:**  
**Result:**  
**Screenshot:**  
**Why it failed / mitigation used:**  

---

## 5. File Upload

### 🔴 Low Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it worked:**  

---

### 🟡 Medium Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it was harder:**  

---

### 🟢 High Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it failed / mitigation used:**  

---

## 6. Insecure CAPTCHA

### 🔴 Low Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it worked:**  

---

### 🟡 Medium Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it was harder:**  

---

### 🟢 High Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it failed / mitigation used:**  

---

## 7. SQL Injection

### 🔴 Low Security
**Payload:**
```sql
1' OR '1'='1
```
**Result:** Successfully dumped the entire database of users (Usernames and Passwords).  
**Screenshot:** ![SQL Injection Low](screenshots/sql-low.png)  
**Why it worked:** The application directly concatenates user input into the SQL query without any sanitization or use of prepared statements. The `'` character closes the original string, and `OR '1'='1` makes the WHERE clause always true.

---

### 🟡 Medium Security
**Payload:** `1 OR 1=1` (sent via edited HTML value)  
**Result:** Successfully dumped the database after bypassing the dropdown menu.  
**Screenshot:** ![SQL Injection Medium](screenshots/sql-medium.png)  
**Why it was harder:** Direct text input was blocked by a dropdown menu, and the PHP code used `mysql_real_escape_string()` to escape single quotes.  
**Why it still worked:** I used "Inspect Element" to modify the value of the dropdown option to a SQL payload. Since the developer forgot to put quotes around the `$id` variable in the SQL query (e.g., `WHERE user_id = $id`), the `mysql_real_escape_string()` function was useless against numeric-based injection.

---

### 🟢 High Security
**Payload:** `1' OR '1'='1` (entered in the popup window)  
**Result:** Successfully dumped the database from a separate input window.  
**Screenshot:** ![SQL Injection High](screenshots/sql-high.png)  
**Why it failed / mitigation used:** The "High" level simply moved the input form to a separate popup page. This stops some automated scrapers but doesn't fix the underlying SQL injection vulnerability. The backend code remains vulnerable as it still doesn't use prepared statements.

---

## 8. SQL Injection (Blind)

### 🔴 Low Security
**Payload:**  
**Result:**  
**Screenshot:**  
**Why it worked:**  

---

### 🟡 Medium Security
**Payload:**  
**Result:**  
**Screenshot:**  
**Why it was harder:**  

---

### 🟢 High Security
**Payload:**  
**Result:**  
**Screenshot:**  
**Why it failed / mitigation used:**  

---

## 9. Weak Session IDs

### 🔴 Low Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it worked:**  

---

### 🟡 Medium Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it was harder:**  

---

### 🟢 High Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it failed / mitigation used:**  

---

## 10. XSS – DOM

### 🔴 Low Security
**Payload:**
```
<script>alert('DOM XSS')</script>
```
**Result:**  
**Screenshot:**  
**Why it worked:**  

---

### 🟡 Medium Security
**Payload:**  
**Result:**  
**Screenshot:**  
**Why it was harder:**  

---

### 🟢 High Security
**Payload:**  
**Result:**  
**Screenshot:**  
**Why it failed / mitigation used:**  

---

## 11. XSS – Reflected

### 🔴 Low Security
**Payload:**
```html
<script>alert('XSS')</script>
```
**Result:**  
**Screenshot:**  
**Why it worked:**  

---

### 🟡 Medium Security
**Payload:**  
**Result:**  
**Screenshot:**  
**Why it was harder:**  

---

### 🟢 High Security
**Payload:**  
**Result:**  
**Screenshot:**  
**Why it failed / mitigation used:**  

---

## 12. XSS – Stored

### 🔴 Low Security
**Payload:**
```html
<script>alert('Stored XSS')</script>
```
**Result:**  
**Screenshot:**  
**Why it worked:**  

---

### 🟡 Medium Security
**Payload:**  
**Result:**  
**Screenshot:**  
**Why it was harder:**  

---

### 🟢 High Security
**Payload:**  
**Result:**  
**Screenshot:**  
**Why it failed / mitigation used:**  

---

## 13. JavaScript

### 🔴 Low Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it worked:**  

---

### 🟡 Medium Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it was harder:**  

---

### 🟢 High Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it failed / mitigation used:**  

---

## 14. CSP Bypass

### 🔴 Low Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it worked:**  

---

### 🟡 Medium Security
**Method:**  
**Result:**  
**Screenshot:**  
**Why it was harder:**  

---

### 🟢 High Security
**Method:**  
**Result:**  
**Screenshot:**  
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
