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
**Tool/Method:**  
**Result:**  
**Screenshot:**  
**Why it worked:**  

---

### 🟡 Medium Security
**Tool/Method:**  
**Result:**  
**Screenshot:**  
**Why it was harder:**  

---

### 🟢 High Security
**Tool/Method:**  
**Result:**  
**Screenshot:**  
**Why it failed / mitigation used:**  

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
