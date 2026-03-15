# DVWA Security Lab Report
**Course:** CS382 – Cybersecurity  
**Name:** Muhammad Shayaan Qazi  
**Student ID:** ms08066  
**Tool:** Damn Vulnerable Web Application (DVWA) via Docker  

---

## Setup Commands

All the environment setup commands are in [`commands.md`](commands.md).

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
10. [XSS (DOM)](#10-xss-dom)
11. [XSS (Reflected)](#11-xss-reflected)
12. [XSS (Stored)](#12-xss-stored)
13. [CSP Bypass](#13-csp-bypass)
14. [JavaScript](#14-javascript)

## Additional Sections

15. [Docker Inspection](#docker-inspection)
16. [Security Analysis](#security-analysis)
17. [Bonus: Nginx Reverse Proxy + HTTPS](#17-bonus-nginx-reverse-proxy--https)

---

## 1. Brute Force

### Low Security
**Tool/Method:** Manual entry (basically simulating a dictionary attack).  
**Result:** Logged in with `admin` / `password`.  
**Screenshot:** ![Brute Force Low](screenshots/bruteforce_low.png)  
**Why it worked:** There's nothing stopping you on the server side. No lockout, no delay. You can just keep guessing as fast as you want.

---

### Medium Security
**Tool/Method:** Manual entry.  
**Result:** Still logged in, but each wrong guess had a noticeable delay.  
**Screenshot:** ![Brute Force Medium](screenshots/bruteforce_med.png)  
**Why it was harder:** The server has a `sleep(2)` call whenever a login attempt fails, so every wrong guess takes 2 extra seconds.  
**Why it still worked:** The delay slows things down a lot, but it doesn't actually block you. With enough patience you can still brute-force your way in.

---

### High Security
**Tool/Method:** Manual entry.  
**Result:** Logged in, but each attempt needs a unique CSRF token.  
**Screenshot:** ![Brute Force High](screenshots/bruteforce_high.png)  
**Why it failed / mitigation:** At this level there's both a CSRF token (anti-automation token) and a random `sleep()` delay. You'd have to scrape the login page each time just to get the token, which most basic scripts can't do. The random delay makes timing-based attacks harder too.

---

## 2. Command Injection

### Low Security
**Payload:** `127.0.0.1; whoami; pwd; ls`  
**Result:** Got the ping output plus our injected commands. Showed the username `www-data`, working directory `/var/www/html/vulnerabilities/exec`, and the files there.  
**Screenshot:** ![Command Injection Low](screenshots/command_injection_low.png)  
**Why it worked:** The app just hands whatever you type straight to `shell_exec()`. Using `;` lets us end the ping command and run our own stuff after it.

---

### Medium Security
**Payload:** `127.0.0.1 & ls`  
**Result:** Ping output followed by the `ls` output (`help`, `index.php`, `source`).  
**Screenshot:** ![Command Injection Medium](screenshots/command_injection_med.png)  
**Why it was harder:** The developer added a blacklist that strips out `&&` and `;`.  
**Why it still worked:** They forgot to filter the single `&` character. So we can still run commands using that.

---

### High Security
**Payload:** `127.0.0.1 |ls` (no space after the pipe, this is important)  
**Result:** Just the `ls` output (`help`, `index.php`, `source`). The ping output gets piped away.  
**Screenshot:** ![Command Injection High](screenshots/command_injection_high.png)  
**Why it failed / mitigation:** The blacklist was expanded to include `&`, `;`, `|`, `-`, `$`, etc. But there's a mistake in the code: they filtered `'| '` (pipe + space) instead of just `'|'`.  
**Why it still worked:** If you don't put a space after the pipe, the filter misses it. Basically, blacklists are really easy to get wrong.

---

## 3. CSRF

### Low Security
**Payload:** `http://localhost:8080/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change`  
**Result:** Just visiting this URL changed the admin password to `hacked`, without any form submission.  
**Screenshot:** ![CSRF Low](screenshots/csrf_low.png)  
**Why it worked:** The app processes GET parameters with the session cookie and doesn't check if the request actually came from the form. So anyone can craft a URL that changes the password.

---

### Medium Security
**Method:** Bypassed the `Referer` check using the browser console's `fetch`.  
**Result:** Password changed after faking the Referer header.  
**Screenshot:** ![CSRF Medium](screenshots/csrf_med.png)  
**Why it was harder:** The server checks the `Referer` header to make sure the request came from the same site.  
**Why it still worked:** The check just looks for the string `localhost` anywhere in the Referer. So I used "Copy as fetch" from the Network tab, pasted it in the Console, and changed the `"referrer"` to something like `http://evil.com/localhost.html`. The server accepted it.

---

### High Security
**Method:** Tried direct URL injection.  
**Result:** Blocked. Every request needs a unique CSRF token tied to the session.  
**Screenshot:** ![CSRF High](screenshots/csrf_high.png)  
**Why it failed / mitigation:** The server generates a unique `user_token` on each page load, tied to the session. Without scraping the page first to get this hidden token, any forged request gets rejected.

---

## 4. File Inclusion

### Low Security
**Payload:** `?page=/etc/passwd`  
**Result:** Read the server's `/etc/passwd` file, which lists all system users.  
**Screenshot:** ![File Inclusion Low](screenshots/file_inclusion_low.png)  
**Why it worked:** The PHP `include()` function is used with no validation at all. Whatever you put in the `page` parameter, it tries to open it.

---

### Medium Security
**Payload:** `?page=..././..././..././..././..././..././etc/passwd`  
**Result:** Bypassed the `../` filter and read `/etc/passwd` after 6 directory traversals.  
**Screenshot:** ![File Inclusion Medium](screenshots/file_inclusion_med.png)  
**Why it was harder:** There's a filter that removes `../` and `..\` sequences.  
**Why it still worked:** The filter only runs once. So if you type `..././`, it removes the inner `../` and leaves behind `../`. Doing this 6 times gets you back to root from the deep app folder.

---

### High Security
**Payload:** `?page=file:///etc/passwd`  
**Result:** Bypassed the "file" prefix check using the `file://` protocol.  
**Screenshot:** ![File Inclusion High](screenshots/file_inclusion_high.png)  
**Why it failed / mitigation:** The server requires the input to start with the word "file"—meant to whitelist specific filenames.  
**Why it still worked:** The `file:///` URI scheme starts with "file", so it passes the check while still letting you access any local file.

---

## 5. File Upload

### Low Security
**Method:** Uploaded a basic PHP web shell (`shell.php`).  
**Result:** File uploaded fine, and I could execute it by visiting the upload path.  
**Screenshot:** ![File Upload Low](screenshots/file_upload_low.png)  
**Why it worked:** No checks on file type or extension. The app just moves the file to a public folder and you can run it.

---

### Medium Security
**Method:** Changed the `Content-Type` to `image/jpeg` using a browser `fetch` command.  
**Result:** Uploaded the PHP shell by making the server think it was an image.  
**Screenshot:** ![File Upload Medium](screenshots/file_upload_med.png)  
**Why it was harder:** The app checks the `Content-Type` header to make sure it's an image type like `image/jpeg` or `image/png`.  
**Why it still worked:** The browser headers are under your control. I used "Copy as fetch" in the Network tab and changed the `Content-Type` to `image/jpeg` in the Console, and the server accepted it.

---

### High Security
**Method:** PNG magic bytes bypass + File Inclusion chain.  
**Result:** Uploaded a PHP shell hidden in a valid PNG image and ran it through the LFI vulnerability.  
**Screenshot:** ![File Upload High](screenshots/file_upload_high.png)  
**Why it failed / mitigation:** The server uses `getimagesize()` to verify it's a real image and only allows `.jpg`, `.jpeg`, or `.png` extensions.  
**Why it still worked:** I used PowerShell to make a file with valid PNG magic bytes followed by PHP code:
```powershell
$bytes = [System.Convert]::FromBase64String("iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYAAAAAYAAjCB0C8AAAAASUVORK5CYII="); [System.IO.File]::WriteAllBytes("shell.png", $bytes); Add-Content "shell.png" '<?php phpinfo(); ?>'
```
The server accepted `shell.png` as a valid image. Then I ran the code by chaining it with File Inclusion: 
`?page=../../../../hackable/uploads/shell.png`

---

## 6. Insecure CAPTCHA

### Low Security
**Method:** Skipped Step 1 entirely and sent the Step 2 POST request directly.  
**Result:** Changed the password without solving the CAPTCHA.  
**Screenshot:** ![Insecure CAPTCHA Low](screenshots/captcha_low.png)  
**Why it worked:** The password change is a two-step process. Step 1 shows the CAPTCHA, Step 2 actually does the change. But the code for Step 2 just trusts the `step=2` parameter without checking if you completed Step 1.

---

### Medium Security
**Method:** Added `passed_captcha=true` to the POST request.  
**Result:** CAPTCHA bypassed.  
**Screenshot:** ![Insecure CAPTCHA Medium](screenshots/captcha_med.png)  
**Why it was harder:** Now there's a `passed_captcha` parameter that the server checks before allowing the change.  
**Why it still worked:** The problem is that the parameter comes from the client side. Using DevTools or Burp Suite, you can just tack on `&passed_captcha=true` to the request and the server thinks the CAPTCHA was solved.

---

### High Security
**Method:** Parameter + User-Agent spoofing.  
**Result:** Changed the password by faking the reCAPTCHA verification.  
**Screenshot:** ![Insecure CAPTCHA High](screenshots/captcha_high.png)  
**Why it failed / mitigation:** This level tries to verify the CAPTCHA response, but it has a hardcoded backdoor left in (probably for testing).  
**Why it still worked:** If you set `g-recaptcha-response` to `hidd3n_valu3` and the `User-Agent` header to `reCAPTCHA`, the server skips the real Google verification and lets the password change through. You need something like Burp Suite for this since browsers won't let you change the User-Agent via script.

---

## 7. SQL Injection

### Low Security
**Payload:**
```sql
1' OR '1'='1
```
**Result:** Dumped the entire users table (usernames and passwords).  
**Screenshot:** ![SQL Injection Low](screenshots/sql_injection_low.png)  
**Why it worked:** User input goes straight into the SQL query with no sanitization or prepared statements. The `'` closes the original string, and `OR '1'='1` makes the WHERE clause always true, so it returns everything.

---

### Medium Security
**Payload:** `1 OR 1=1` (edited the HTML dropdown value)  
**Result:** Dumped the database after getting around the dropdown menu.  
**Screenshot:** ![SQL Injection Medium](screenshots/sql_injection_med.png)  
**Why it was harder:** There's no text input, just a dropdown. Plus the code uses `mysql_real_escape_string()` to escape single quotes.  
**Why it still worked:** I used Inspect Element to change the dropdown option value to my SQL payload. Because the developer forgot to wrap `$id` in quotes in the query (i.e. `WHERE user_id = $id`), escaping quotes doesn't matter for numeric injection.

---

### High Security
**Payload:** `1' OR '1'='1` (typed into the popup window)  
**Result:** Dumped the database from the separate input page.  
**Screenshot:** ![SQL Injection High](screenshots/sql_injection_high.png)  
**Why it failed / mitigation:** The "High" level just moves the input to a separate popup page. That might stop some automated scanners, but the actual SQL injection is still there because it still doesn't use prepared statements.

---

## 8. SQL Injection (Blind)

### Low Security
**Payload:** `1' AND 1=1 #` (True) vs `1' AND 1=2 #` (False)  
**Result:** Confirmed User ID 1 exists by watching the response change.  
**Screenshot:** ![SQL Injection Blind Low](screenshots/sql_injection_blind_low.png)  
**Why it worked:** Input isn't sanitized. By injecting `AND 1=1`, if the page says "User ID exists" then our condition was true. If it says "MISSING", it was false. This lets us extract info one bit at a time.

---

### Medium Security
**Payload:** `1 AND 1=1` (True) vs `1 AND 1=2` (False)  
**Result:** Bypassed `mysqli_real_escape_string` with numeric injection.  
**Screenshot:** ![SQL Injection Blind Medium](screenshots/sql_injection_blind_med.png)  
**Why it was harder:** The app uses `mysqli_real_escape_string()` and a dropdown for input.  
**Why it still worked:** The SQL query doesn't put quotes around the ID (`WHERE user_id = $id`), so escaping quotes doesn't help. I intercepted the POST request and changed the ID to a boolean expression.

---

### High Security
**Payload:** `1' AND 1=1 #` (True) vs `1' AND 1=2 #` (False)  
**Result:** Exploited it by modifying the `id` cookie.  
**Screenshot:** ![SQL Injection Blind High](screenshots/sql_injection_blind_high.png)  
**Why it failed / mitigation:** The input is now taken from a cookie set on a separate page, and there's a `sleep()` to slow down automated tools.  
**Why it still worked:** The core flaw is the same: string concatenation in the query. It's just a different input vector (cookies instead of a form). Setting the cookie value to the payload bypasses the intended input method.

---

## 9. Weak Session IDs

### Low Security
**Method:** Looked at the session ID values over multiple requests.  
**Result:** The `dvwaSession` cookie was just an incrementing number (1, 2, 3...).  
**Screenshot:** ![Weak Session IDs Low](screenshots/wsi_low.png)  
**Why it worked:** The server just does `last_session_id++` to generate IDs. Super easy to predict the next one.

---

### Medium Security
**Method:** Checked if the session ID was time-based.  
**Result:** The cookie value was a Unix timestamp.  
**Screenshot:** ![Weak Session IDs Medium](screenshots/wsi_med.png)  
**Why it was harder:** It's not a simple counter anymore, so at first it looks random.  
**Why it still worked:** It's just PHP `time()`. Timestamps are sequential and public knowledge, so if you know roughly when someone logged in, you can guess their session ID.

---

### High Security
**Method:** Checked if the IDs were MD5 hashes of something predictable.  
**Result:** Yes, they were MD5 hashes of incrementing integers.  
**Screenshot:** ![Weak Session IDs High](screenshots/wsi_high.png)  
**Why it failed / mitigation:** The session ID looks like a random MD5 hash, so it's not obvious what's behind it.  
**Why it still worked:** The underlying value is still just a counter (`md5(count++)`). Hashing something predictable doesn't make it secure. It's just obfuscation.

---

## 10. XSS (DOM)

### Low Security
**Payload:** `?default=<script>alert('Low_XSS')</script>`  
**Result:** Alert box popped up.  
**Screenshot:** ![XSS DOM Low](screenshots/xss_dom_low.png)  
**Why it worked:** The app reads the `default` parameter from the URL and uses `document.write` to put it into the page. No sanitization at all.

---

### Medium Security
**Payload:** `?default=English</option></select><img src=x onerror=alert('Medium_XSS')>`  
**Result:** Bypassed the `<script` filter with an `img` tag.  
**Screenshot:** ![XSS DOM Medium](screenshots/xss_dom_med.png)  
**Why it was harder:** The server uses `stripos()` to check if the input has `<script` in it. If it does, it redirects you.  
**Why it still worked:** The check only looks for `<script`. Using a different tag like `<img>` with an `onerror` handler gets around it.

---

### High Security
**Payload:** `?default=English#</option></select><img src=x onerror=alert('High_XSS')>`  
**Result:** Worked after a page reload.  
**Screenshot:** ![XSS DOM High](screenshots/xss_dom_high.png)  
**Why it failed / mitigation:** The server has a strict whitelist (a `switch` statement) that only allows certain language names.  
**Why it still worked:**  
1. The `#` fragment isn't sent to the server—the PHP whitelist only sees `default=English` and lets it through.
2. The client-side JavaScript reads the full URL including the fragment and injects it into the DOM.
3. Using `</option></select>` breaks out of the existing HTML structure, and the `<img onerror=...>` runs our code.

---

## 11. XSS (Reflected)

### Low Security
**Payload:** `<script>alert('Low_XSS')</script>`  
**Result:** Script ran and showed the alert.  
**Screenshot:** ![XSS Reflected Low](screenshots/xss_reflected_low.png)  
**Why it worked:** The server takes the `name` GET parameter and echoes it straight into the HTML inside `<pre>` tags. No sanitization or encoding.

---

### Medium Security
**Payload:** `<SCRIPT>alert('Medium_XSS')</SCRIPT>`  
**Result:** Bypassed the filter using uppercase tags.  
**Screenshot:** ![XSS Reflected Medium](screenshots/xss_reflected_med.png)  
**Why it was harder:** The app uses `str_replace( '<script>', '', ... )` to strip script tags.  
**Why it still worked:** The filter only matches the exact lowercase `<script>` string. Using uppercase like `<SCRIPT>` or mixed case gets past it.

---

### High Security
**Payload:** `<img src=x onerror=alert('High_XSS')>`  
**Result:** Bypassed the regex-based filter with an `img` tag.  
**Screenshot:** ![XSS Reflected High](screenshots/xss_reflected_high.png)  
**Why it failed / mitigation:** The app uses `preg_replace()` with a regex that case-insensitively blocks anything that looks like a script tag.  
**Why it still worked:** The regex only targets `<script>` tags. Using other HTML elements like `<img>`, `<body>`, or `<a>` with JavaScript event handlers (`onerror`, `onload`, etc.) bypasses it entirely.

---

## 12. XSS (Stored)

### Low Security
**Payload:** `<script>alert('Low_Stored')</script>` (in the Message field)  
**Result:** The script got saved and fires every time anyone loads the page.  
**Screenshot:** ![XSS Stored Low](screenshots/xss_stored_low.png)  
**Why it worked:** The guestbook form saves input directly to the database without stripping or encoding any tags. When the page renders those entries, the browser runs the script.

---

### Medium Security
**Payload:** `<SCRIPT>alert('Medium_Stored')</SCRIPT>` (in the Name field)  
**Result:** Got past the Message field's sanitization by going through the Name field instead.  
**Screenshot:** ![XSS Stored Medium](screenshots/xss_stored_medium.png)  
**Why it was harder:** The Message field uses `strip_tags()` which is pretty thorough. The Name field has a 10-character `maxlength` in the HTML.  
**Why it still worked:** I used DevTools to increase the `maxlength` on the Name field. The backend only does a case-sensitive `str_replace('<script>', ...)` on the name, so uppercase tags bypass it.

---

### High Security
**Payload:** `<img src=x onerror=alert('High_Stored')>` (in the Name field)  
**Result:** Bypassed the regex filter using an `img` event handler.  
**Screenshot:** ![XSS Stored High](screenshots/xss_stored_high.png)  
**Why it failed / mitigation:** The Name field now has a regex blocking script tags, and the Message field still uses `strip_tags()`.  
**Why it still worked:** Same idea as Medium—increase the HTML length limit on the Name field, then use `<img>` with `onerror` instead of a script tag to get around the blacklist.

---

## 13. CSP Bypass

### Low Security
**Payload:** `source/jsonp.php?callback=alert`  
**Result:** Alert box popped up (showing `[object Object]`).  
**Screenshot:** ![CSP Bypass Low](screenshots/csp_low.png)  
**Why it worked:** The CSP header allows scripts from `'self'`. There's a JSONP endpoint at `source/jsonp.php` that reflects whatever you put in the `callback` parameter. By using `alert` as the callback, the browser runs `alert({"answer":"15"})`.  
**Things I tried first that didn't work:**
- External whitelisted domains like Pastebin—blocked by the browser or VM network issues.
- Complex payloads with quotes or comments—caused syntax errors. Keeping it to just `alert` was the fix.

---

### Medium Security
**Method:** Used the hardcoded nonce.  
**Payload:** `<script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">alert('Medium_CSP_Bypass')</script>`  
**Result:** Inline script ran fine.  
**Screenshot:** ![CSP Bypass Medium](screenshots/csp_med.png)  
**Why it was harder:** The CSP uses a nonce, which is supposed to prevent arbitrary inline scripts from running.  
**Why it still worked:** The nonce is hardcoded (it's "Never going to give you up" in base64). Since it never changes between requests, you can just include it in your own script tags.

---

### High Security
**Payload (Console):**
```javascript
var f = document.createElement('form');
f.method = 'POST';
f.action = window.location.href;
var i = document.createElement('input');
i.name = 'include';
i.value = '<script src="source/jsonp.php?callback=alert"></script>';
f.appendChild(i);
document.body.appendChild(f);
f.submit();
```
**Result:** Code ran by pointing to the `jsonp.php` file on the same origin through a hidden form.  
**Screenshot:** ![CSP Bypass High](screenshots/csp_high.png)  
**Why it failed / mitigation:** The CSP only allows scripts from `'self'`, and the UI doesn't have an input field anymore, so there's no obvious way to inject anything directly.  
**Why it still worked:**  
- The backend PHP still processes the `POST['include']` parameter even though there's no UI for it.
- Using the console to create and submit a hidden form gets the payload in.
- Since the injected `<script>` points to a file on the same origin (`source/jsonp.php`), it satisfies the `'self'` policy. Using `alert` as the callback triggers execution.

---

## 14. JavaScript

### Low Security
**Objective:** Pass the "success" phrase check with a valid token.  
**Logic:** `token = md5(rot13(phrase))`  
**Steps:**  
1. Type `success` in the input field.
2. Open the console (F12) and run `generate_token()`.
3. Click "Submit".  
**Result:** It worked.  
**Screenshot:** ![JavaScript Low](screenshots/js_low.png)  
**Why it worked:** The page generates a token from a phrase using client-side JavaScript. If you manually call the generation function after entering the right phrase, the token matches what the server expects.

---

### Medium Security
**Logic:** `token = reverse("XX" + phrase + "XX")`  
**Steps:**  
1. Type `success` in the input field.
2. Open the console and run `do_elsesomething("XX")`.
3. Click "Submit".  
**Result:** It worked.  
**Screenshot:** ![JavaScript Medium](screenshots/js_med.png)  
**Why it worked:** Same idea as Low, but the logic is in an external script and uses `setTimeout`. Calling the final step of the token generation manually gets the right token.

---

### High Security
**Logic:** Multi-stage SHA-256:
1. `part1 = reverse(phrase)`
2. `part2 = sha256("XX" + part1)`
3. `part3 = sha256(part2 + "ZZ")` (runs on click)  
**Steps:**  
1. Type `success` in the input.
2. Open the console and run:
   ```javascript
   token_part_1("ABCD", 44);
   token_part_2("QX");
   ```
3. Click "Submit".  
**Result:** It worked.  
**Screenshot:** ![JavaScript High](screenshots/js_high.png)  
**Why it worked:** The code is heavily obfuscated with timing and event-based triggers. Manually calling the first two parts in the console sets up the right state, and clicking Submit fires the last part.

---

## Docker Inspection

```bash
docker ps
```
**Output:**
```text
CONTAINER ID   IMAGE                  COMMAND      CREATED             STATUS          PORTS                                     NAMES
9f029a03bb5a   vulnerables/web-dvwa   "/main.sh"   About an hour ago   Up 44 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   dvwa
```

```bash
docker inspect dvwa
```
**Output:**
```json
[
    {
        "Id": "9f029a03bb5ada048d13e58cbee09c223f928af850a539acea18ab86915a1b00",
        "Created": "2026-03-07T17:26:24.248477238Z",
        "Path": "/main.sh",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 504,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2026-03-07T17:58:56.268386943Z",
            "FinishedAt": "2026-03-07T17:55:18.434727061Z"
        },
        "Image": "sha256:dae203fe11646a86937bf04db0079adef295f426da68a92b40e3b181f337daa7",
        "Name": "/dvwa",
        "Driver": "overlayfs",
        "HostConfig": {
            "NetworkMode": "bridge",
            "PortBindings": {
                "80/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8080"
                    }
                ]
            }
        },
        "Config": {
            "Image": "vulnerables/web-dvwa",
            "Entrypoint": [
                "/main.sh"
            ]
        },
        "NetworkSettings": {
            "Ports": {
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8080"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "8080"
                    }
                ]
            },
            "Gateway": "172.17.0.1",
            "IPAddress": "172.17.0.2"
        },
        "ImageManifestDescriptor": {
            "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
            "digest": "sha256:dae203fe11646a86937bf04db0079adef295f426da68a92b40e3b181f337daa7",
            "size": 1997,
            "platform": {
                "architecture": "amd64",
                "os": "linux"
            }
        }
    }
]
```

```bash
docker logs dvwa
```
**Output (shortened):**
```text
[+] Starting mysql...
Starting MariaDB database server: mysqld.
[+] Starting apache
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
Starting Apache httpd web server: apache2.
==> /var/log/apache2/access.log <==

==> /var/log/apache2/error.log <==
[Sat Mar 07 17:26:26.622102 2026] [mpm_prefork:notice] [pid 288] AH00163: Apache/2.4.25 (Debian) configured -- resuming normal operations
[Sat Mar 07 17:26:26.622188 2026] [core:notice] [pid 288] AH00094: Command line: '/usr/sbin/apache2'

==> /var/log/apache2/other_vhosts_access.log <==

==> /var/log/apache2/access.log <==
172.17.0.1 - - [07/Mar/2026:17:27:09 +0000] "GET /setup.php HTTP/1.1" 200 2003 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:17:27:10 +0000] "GET /setup.php HTTP/1.1" 200 2003 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:17:27:11 +0000] "POST /setup.php HTTP/1.1" 302 337 "http://localhost:8080/setup.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:17:27:11 +0000] "GET /setup.php HTTP/1.1" 200 2175 "http://localhost:8080/setup.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:17:27:13 +0000] "GET /login.php HTTP/1.1" 200 1049 "http://localhost:8080/setup.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:17:27:17 +0000] "POST /login.php HTTP/1.1" 302 336 "http://localhost:8080/login.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:17:27:17 +0000] "GET /index.php HTTP/1.1" 200 3036 "http://localhost:8080/login.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
...
172.17.0.1 - - [07/Mar/2026:18:20:19 +0000] "GET /vulnerabilities/javascript/ HTTP/1.1" 200 1815 "http://localhost:8080/security.php" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:18:20:28 +0000] "POST /vulnerabilities/javascript/ HTTP/1.1" 200 1838 "http://localhost:8080/vulnerabilities/javascript/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:18:20:29 +0000] "GET /.well-known/appspecific/com.chrome.devtools.json HTTP/1.1" 404 539 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36"
172.17.0.1 - - [07/Mar/2026:18:21:20 +0000] "-" 408 0 "-" "-"
172.17.0.1 - - [07/Mar/2026:18:26:45 +0000] "-" 408 0 "-" "-"
```

```bash
docker exec -it dvwa /bin/bash
ls /var/www/html
```
**Output:**
```text
root@9f029a03bb5a:/# ls /var/www/html
CHANGELOG.md  about.php  dvwa         hackable     instructions.php  php.ini      security.php
COPYING.txt   config     external     ids_log.php  login.php         phpinfo.php  setup.php
README.md     docs       favicon.ico  index.php    logout.php        robots.txt   vulnerabilities
root@9f029a03bb5a:/#
```

**Backend stack:**
```text
PHP 7.0.30-0+deb9u1
Apache/2.4.25 (Debian)
```

**Where the files are:** The web root is at `/var/www/html`, and the vulnerable modules live under `/var/www/html/vulnerabilities`.

**What backend it uses:** DVWA runs on PHP and Apache in a Debian-based Linux container.

**How Docker isolates things:** The app runs in its own container with a separate filesystem and network namespace. It gets an internal IP (`172.17.0.2`) on Docker's bridge network, and only port 80 inside maps to port 8080 on the host. So the container is isolated from the rest of the system.

---

## Security Analysis

**Q1. Why does SQL Injection succeed at Low security?**  
Because the code just sticks user input directly into the SQL query with no parameterization. An attacker can close the string and add `OR '1'='1` to make the query return everything.

**Q2. What control prevents it at High?**  
In DVWA, even the High level is still exploitable, so the "control" is incomplete. In actual production, you'd use prepared statements (parameterized queries), strict input typing, and least-privilege database accounts.

**Q3. Does HTTPS prevent these attacks? Why or why not?**  
No. HTTPS only encrypts data in transit between the client and server. It does nothing about server-side logic flaws like SQL injection, XSS, command injection, or file inclusion.

**Q4. What risks exist if this app is deployed publicly?**  
- Account takeover and session hijacking
- Database dumps and credential leaks
- Remote command execution on the server
- Persistent compromise through stored XSS
- Attackers could use it as a foothold to pivot into internal systems

**Q5. OWASP Top 10 Mapping:**

| Vulnerability | OWASP Top 10 Category | Rationale |
|---|---|---|
| Brute Force | A07:2021 – Identification and Authentication Failures | Weak login protection, no lockout |
| Command Injection | A03:2021 – Injection | User input reaches OS command execution |
| CSRF | A01:2021 – Broken Access Control | Forged requests accepted with victim's session |
| File Inclusion | A05:2021 – Security Misconfiguration | Unsafe file handling, no path restrictions |
| File Upload | A05:2021 – Security Misconfiguration | Bad validation, executable files can be uploaded |
| Insecure CAPTCHA | A07:2021 – Identification and Authentication Failures | CAPTCHA bypassed due to weak server-side checks |
| SQL Injection | A03:2021 – Injection | SQL query manipulation through user input |
| SQL Injection (Blind) | A03:2021 – Injection | Boolean-based inference from injected conditions |
| Weak Session IDs | A07:2021 – Identification and Authentication Failures | Predictable session tokens |
| XSS (DOM / Reflected / Stored) | A03:2021 – Injection | Script execution via unsanitized input |
| JavaScript | A04:2021 – Insecure Design | Security logic on the client side, easily bypassed |
| CSP Bypass | A05:2021 – Security Misconfiguration | Weak policies/nonces allow script execution |

---

## 17. Bonus: Nginx Reverse Proxy + HTTPS

### Objective
- Put DVWA behind an Nginx reverse proxy
- Set up HTTPS with a self-signed certificate
- See the difference between HTTP and HTTPS traffic

### Implementation

**Files added:**
- `bonus/docker-compose.yml`
- `bonus/nginx/default.conf`
- `bonus/certs/` (generated cert and key)

**Container setup:**
- `dvwa-backend`: runs DVWA on the internal network only, no host port exposed
- `dvwa-nginx-proxy`: Nginx reverse proxy
  - HTTP: `http://localhost:8081`
  - HTTPS: `https://localhost:8443`

### Self-Signed Certificate

Generated a self-signed certificate using OpenSSL in a Docker container:

```bash
docker run --rm -v "${PWD}/bonus/certs:/out" alpine:3.20 sh -c "apk add --no-cache openssl >/dev/null && openssl req -x509 -nodes -newkey rsa:2048 -keyout /out/dvwa.key -out /out/dvwa.crt -days 365 -subj '/CN=localhost'"
```

### Deployment

```bash
docker compose -f bonus/docker-compose.yml up -d
docker compose -f bonus/docker-compose.yml ps
```

**Output:**

```text
NAME               IMAGE                  COMMAND                  SERVICE        CREATED         STATUS         PORTS
dvwa-backend       vulnerables/web-dvwa   "/main.sh"               dvwa-backend   7 seconds ago   Up 6 seconds   80/tcp
dvwa-nginx-proxy   nginx:1.27-alpine      "/docker-entrypoint.…"   nginx-proxy    7 seconds ago   Up 6 seconds   0.0.0.0:8081->80/tcp, [::]:8081->80/tcp, 0.0.0.0:8443->443/tcp, [::]:8443->443/tcp
```

### Verification

```bash
curl -I http://localhost:8081/login.php
curl -k -I https://localhost:8443/login.php
```

**Output:**

```text
HTTP/1.1 200 OK
Server: nginx/1.27.5
...
```

```text
HTTP/1.1 200 OK
Server: nginx/1.27.5
...
```

### HTTP vs HTTPS: What's the Difference?

Ran verbose curl requests to see the difference:

```bash
curl -v http://localhost:8081/login.php -o NUL
curl -vk https://localhost:8443/login.php -o NUL
```

What I saw:
- The HTTP request just sends the data directly. No TLS handshake at all.
- The HTTPS request does a TLS negotiation first before sending any HTTP data.
- HTTPS needed the `-k` flag because the cert is self-signed and not trusted by default.

What this means:
- **HTTP:** Anyone on the network can read or modify the traffic. No encryption at all.
- **HTTPS:** Traffic is encrypted and integrity-protected, so credentials and cookies are safe from passive interception.
- **Important:** HTTPS does NOT fix server-side bugs like SQLi, XSS, or command injection. It only protects data in transit.

---
