# DVWA Security Lab Report
**Course:** CS382 – Cybersecurity  
**Tool:** Damn Vulnerable Web Application (DVWA) via Docker  

---

## Setup Commands

Environment setup commands are in [`commands.md`](commands.md).

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

---

## 1. Brute Force

### Low Security
**Tool/Method:** Manual entry (simulating a dictionary attack).  
**Result:** Successfully logged in using `admin` / `password`.  
**Screenshot:** ![Brute Force Low](screenshots/bruteforce_low.png)  
**Why it worked:** There are no server-side protections. The application processes the request immediately, allowing an attacker to automate thousands of login attempts per second without any lockout or delay.

---

### Medium Security
**Tool/Method:** Manual entry.  
**Result:** Successful login, but failed attempts resulted in a noticeable delay.  
**Screenshot:** ![Brute Force Medium](screenshots/bruteforce_med.png)  
**Why it was harder:** The server-side script includes a `sleep(2)` function call whenever a login attempt fails. This adds a 2-second delay to every incorrect guess.  
**Why it still worked:** While the delay slows down a brute force attack significantly, it does not actually stop it. An attacker with enough time can still eventually guess the password.

---

### High Security
**Tool/Method:** Manual entry.  
**Result:** Successful login, but requires a unique CSRF token per attempt.  
**Screenshot:** ![Brute Force High](screenshots/bruteforce_high.png)  
**Why it failed / mitigation used:** The High security level implements a CSRF token (anti-automation token) and a random `sleep()` delay. The token must be scraped from the login page and submitted with the credentials, which breaks simple automated brute-force scripts. Additionally, the random delay makes it harder for attackers to predict response times.

---

## 2. Command Injection

### Low Security
**Payload:** `127.0.0.1; whoami; pwd; ls`  
**Result:** Displays the ping results followed by the username `www-data`, the current directory `/var/www/html/vulnerabilities/exec`, and the list of files in that directory.  
**Screenshot:** ![Command Injection Low](screenshots/command_injection_low.png)  
**Why it worked:** The application directly pipes the user input into the `shell_exec()` function. By using the `;` separator, we can terminate the ping command and execute our own commands (`whoami`, `pwd`, `ls`).

---

### Medium Security
**Payload:** `127.0.0.1 & ls`  
**Result:** Displays the ping results, followed sequentially by the output of the `ls` command (which shows `help`, `index.php`, and `source`).  
**Screenshot:** ![Command Injection Medium](screenshots/command_injection_med.png)  
**Why it was harder:** The developer implemented a blacklist that removes `&&` and `;`. If you use the Low payload, the `;` is removed, and the command becomes broken.  
**Why it still worked:** The blacklist was incomplete. It didn't filter the single `&` character, which allows us to run commands asynchronously.

---

### High Security
**Payload:** `127.0.0.1 |ls` (Critical: **No space** after the pipe)  
**Result:** Displays only the output of `ls` (`help`, `index.php`, `source`). The ping output is piped away.  
**Screenshot:** ![Command Injection High](screenshots/command_injection_high.png)  
**Why it failed / mitigation used:** The developer expanded the blacklist to include `&`, `;`, `|`, `-`, `$`, etc. However, there is a typo in the blacklist: they filtered `'| '` (pipe followed by a space) instead of just `'|'`.  
**Why it still worked:** By removing the space between the pipe and our command, we bypass the filter and execute the command. This demonstrates that blacklisting is extremely difficult to get right.

---

## 3. CSRF

### Low Security
**Payload:** `http://localhost:8080/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change`  
**Result:** The admin password is changed to `hacked` without the user submitting the form — just by visiting the crafted URL.  
**Screenshot:** ![CSRF Low](screenshots/csrf_low.png)  
**Why it worked:** The application processes GET parameters directly using the session cookie, with no check that the request came from the actual form. An attacker can forge a request by crafting a URL.

---

### Medium Security
**Method:** Bypassing `Referer` check using Chrome DevTools `fetch` modification.
**Result:** Password change is successful after modifying the Referer header to include "localhost".
**Screenshot:** ![CSRF Medium](screenshots/csrf_med.png)  
**Why it was harder:** The server now checks the HTTP `Referer` header to ensure the request originated from the same site.  
**Why it still worked:** The `Referer` check only verifies that the string `localhost` appears anywhere in the Referer URL. By using "Copy as fetch" in the Network tab, pasting it into the Console, and changing the `"referrer"` value to a fake URL containing "localhost" (e.g., `http://evil.com/localhost.html`), the server accepts the forged request.

---

### High Security
**Method:** Required predictable CSRF token. Attempted direct URL injection.
**Result:** Attack is blocked — each request requires a unique, unpredictable CSRF token tied to the user's session.
**Screenshot:** ![CSRF High](screenshots/csrf_high.png)  
**Why it failed / mitigation used:** High security generates a unique `user_token` for every page load that is tied to the session. Without scraping the page first to obtain this embedded hidden token, any forged request will be rejected by the server as invalid.

---

## 4. File Inclusion

### Low Security
**Payload:** `?page=/etc/passwd`
**Result:** Successfully read the server's `/etc/passwd` file, listing all system users.
**Screenshot:** ![File Inclusion Low](screenshots/file_inclusion_low.png)  
**Why it worked:** The PHP `include()` function is used without any validation. It blindly accepts whatever string is passed in the `page` parameter and attempts to open it as a file.

---

### Medium Security
**Payload:** `?page=..././..././..././..././..././..././etc/passwd`
**Result:** Successfully bypassed the `../` filter after 6 directory jumps to read `/etc/passwd`.
**Screenshot:** ![File Inclusion Medium](screenshots/file_inclusion_med.png)  
**Why it was harder:** The developer implemented a filter to remove `../` and `..\` sequences to prevent directory traversal.
**Why it still worked:** The replacement only happens once per match. By inputting `..././`, the filter removes the inner `../`, leaving behind a valid `../` sequence. By doing this 6 times, we successfully traverse back to the root directory from the deep application folder.

---

### High Security
**Payload:** `?page=file:///etc/passwd`
**Result:** Successfully bypassed the "file" prefix requirement using the `file://` protocol.
**Screenshot:** ![File Inclusion High](screenshots/file_inclusion_high.png)  
**Why it failed / mitigation used:** The server implements a strict check where the input MUST start with the word "file". This is intended to whitelist only specific files.
**Why it still worked:** Using the `file:///` URI scheme satisfies the "starts with file" condition while still allowing us to point to any location on the local filesystem.

---

## 5. File Upload

### Low Security
**Method:** Uploaded a simple PHP web shell (`shell.php`).
**Result:** File was uploaded successfully, and server-side execution was achieved by visiting the uploaded file path.
**Screenshot:** ![File Upload Low](screenshots/file_upload_low.png)  
**Why it worked:** There are no server-side checks on the file type or extension. The application simply moves the uploaded file to a public directory, allowing any file (including malicious script) to be executed.

---

### Medium Security
**Method:** MIME-type bypass by changing `Content-Type` to `image/jpeg` using a browser `fetch` command.
**Result:** Successfully uploaded the PHP shell by tricking the server into thinking it was an image.
**Screenshot:** ![File Upload Medium](screenshots/file_upload_med.png)  
**Why it was harder:** The application checks the `Content-Type` header sent by the browser to ensure it matches an allowed image type (e.g., `image/jpeg` or `image/png`).
**Why it still worked:** Browser headers are under the user's control. By using "Copy as fetch" in the Network tab and changing the payload's `Content-Type` to `image/jpeg` in the Console, the server-side validation was bypassed.

---

### High Security
**Method:** PNG Magic Bytes bypass + File Inclusion chaining.
**Result:** Successfully uploaded a PHP shell hidden inside a valid PNG image and executed it via LFI.
**Screenshot:** ![File Upload High](screenshots/file_upload_high.png)  
**Why it failed / mitigation used:** The server uses `getimagesize()` to verify the file is a real image and only allows `.jpg`, `.jpeg`, or `.png` extensions.
**Why it still worked:** I used PowerShell to create a file containing valid PNG magic bytes followed by PHP code:
```powershell
$bytes = [System.Convert]::FromBase64String("iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYAAAAAYAAjCB0C8AAAAASUVORK5CYII="); [System.IO.File]::WriteAllBytes("shell.png", $bytes); Add-Content "shell.png" '<?php phpinfo(); ?>'
```
The server accepted `shell.png` as a valid image. I then executed the code by chaining it with the File Inclusion vulnerability: 
`?page=../../../../hackable/uploads/shell.png`

---

## 6. Insecure CAPTCHA

### Low Security
**Method:** Bypassing Step 1 by sending the Step 2 POST request directly.
**Result:** Successfully changed the password without ever solving the CAPTCHA.
**Screenshot:** ![Insecure CAPTCHA Low](screenshots/captcha_low.png)  
**Why it worked:** The application uses a multi-step process. Step 1 shows the CAPTCHA, and Step 2 changes the password. The server-side script for Step 2 (`low.php`) blindly trusts the `step=2` parameter in the POST request without verifying if the user actually completed Step 1.

---

### Medium Security
**Method:** Forging the `passed_captcha` parameter in the POST request.
**Result:** Bypassed the CAPTCHA verification by adding `passed_captcha=true` to the request body.
**Screenshot:** ![Insecure CAPTCHA Medium](screenshots/captcha_med.png)  
**Why it was harder:** The application now checks for a specific parameter (`passed_captcha`) to decide whether to permit the password change.
**Why it still worked:** This check is flawed because the parameter is sent by the client. By using Intercept (Burp Suite) or DevTools to add `&passed_captcha=true` to the Step 2 request, the server is tricked into thinking the CAPTCHA was solved.

---

### High Security
**Method:** Combined Parameter and User-Agent spoofing.
**Result:** Successfully changed the password by emulating the internal reCAPTCHA verification response.
**Screenshot:** ![Insecure CAPTCHA High](screenshots/captcha_high.png)  
**Why it failed / mitigation used:** The High security level attempts to verify the CAPTCHA response but includes a hardcoded "backdoor" for testing or legacy reasons.
**Why it still worked:** By setting the `g-recaptcha-response` parameter to `hidd3n_valu3` and the `User-Agent` header to `reCAPTCHA`, the server-side logic skips the real Google verification and permits the password change. This requires intercepting the request with a tool like Burp Suite since browsers restrict changing the User-Agent via script.

---

## 7. SQL Injection

### Low Security
**Payload:**
```sql
1' OR '1'='1
```
**Result:** Successfully dumped the entire database of users (Usernames and Passwords).  
**Screenshot:** ![SQL Injection Low](screenshots/sql_injection_low.png)  
**Why it worked:** The application directly concatenates user input into the SQL query without any sanitization or use of prepared statements. The `'` character closes the original string, and `OR '1'='1` makes the WHERE clause always true.

---

### Medium Security
**Payload:** `1 OR 1=1` (sent via edited HTML value)  
**Result:** Successfully dumped the database after bypassing the dropdown menu.  
**Screenshot:** ![SQL Injection Medium](screenshots/sql_injection_med.png)  
**Why it was harder:** Direct text input was blocked by a dropdown menu, and the PHP code used `mysql_real_escape_string()` to escape single quotes.  
**Why it still worked:** I used "Inspect Element" to modify the value of the dropdown option to a SQL payload. Since the developer forgot to put quotes around the `$id` variable in the SQL query (e.g., `WHERE user_id = $id`), the `mysql_real_escape_string()` function was useless against numeric-based injection.

---

### High Security
**Payload:** `1' OR '1'='1` (entered in the popup window)  
**Result:** Successfully dumped the database from a separate input window.  
**Screenshot:** ![SQL Injection High](screenshots/sql_injection_high.png)  
**Why it failed / mitigation used:** The "High" level simply moved the input form to a separate popup page. This stops some automated scrapers but doesn't fix the underlying SQL injection vulnerability. The backend code remains vulnerable as it still doesn't use prepared statements.

---

## 8. SQL Injection (Blind)

### Low Security
**Payload:** `1' AND 1=1 #` (True) vs `1' AND 1=2 #` (False)
**Result:** Successfully confirmed the existence of User ID 1 by observing the change in response message.
**Screenshot:** ![SQL Injection Blind Low](screenshots/sql_injection_blind_low.png)  
**Why it worked:** The input is not sanitized or parameterized. By injecting a boolean condition (`AND 1=1`), we can determine if the original query returned a row. If the page says "User ID exists", the condition was true. If it says "User ID is MISSING", the condition was false.

---

### Medium Security
**Payload:** `1 AND 1=1` (True) vs `1 AND 1=2` (False)
**Result:** Bypassed `mysqli_real_escape_string` by using numeric-based injection.
**Screenshot:** ![SQL Injection Blind Medium](screenshots/sql_injection_blind_med.png)  
**Why it was harder:** The application uses `mysqli_real_escape_string()` to escape quotes and uses a dropdown menu for input.  
**Why it still worked:** Since the SQL query does not wrap the ID in quotes (`WHERE user_id = $id`), escaping quotes has no effect. By intercepting the POST request and changing the ID to a boolean expression, we can infer database data.

---

### High Security
**Payload:** `1' AND 1=1 #` (True) vs `1' AND 1=2 #` (False)
**Result:** Successfully exploited the vulnerability by modifying the `id` cookie.
**Screenshot:** ![SQL Injection Blind High](screenshots/sql_injection_blind_high.png)  
**Why it failed / mitigation used:** The application uses a separate page to set a cookie and includes a `sleep()` function to slow down brute-force/automated attacks.  
**Why it still worked:** The security flaw is the same as the Low level (string concatenation in the query), just with a different input vector (cookies). By manually setting the cookie value to our payload, we bypass the intended input method.

---

## 9. Weak Session IDs

### Low Security
**Method:** Observing incremental session ID values.
**Result:** Successfully predicted the next session ID by observing the consecutive incrementing integers in the `dvwaSession` cookie.
**Screenshot:** ![Weak Session IDs Low](screenshots/wsi_low.png)  
**Why it worked:** The server generates the session ID by simply incrementing an integer counter stored in the session (`last_session_id++`). An attacker can easily guess the session ID of the next user to log in.

---

### Medium Security
**Method:** Identifying time-based session IDs.
**Result:** Predicted session IDs by converting the cookie value to a Unix timestamp.
**Screenshot:** ![Weak Session IDs Medium](screenshots/wsi_med.png)  
**Why it was harder:** The session ID is no longer a simple counter, making it appear randomized at first glance.  
**Why it still worked:** The server uses the PHP `time()` function to generate the ID. Since timestamps are sequential and globally synchronized, an attacker can pinpoint a user's session ID if they know the approximate time of login.

---

### High Security
**Method:** Reversing MD5-hashed incremental IDs.
**Result:** Successfully predicted the next hashed session ID by hashing the next expected integer in the sequence.
**Screenshot:** ![Weak Session IDs High](screenshots/wsi_high.png)  
**Why it failed / mitigation used:** The session ID is an MD5 hash, which hides the underlying sequence from casual observation.  
**Why it still worked:** The underlying value is still just an incremental counter (`md5(count++)`). Hashing a predictable value does not make it secure; it only adds a thin layer of obfuscation that can be trivially bypassed.

---

## 10. XSS (DOM)

### Low Security
**Payload:** `?default=<script>alert('Low_XSS')</script>`
**Result:** Successfully executed the script, triggering an alert box.
**Screenshot:** ![XSS DOM Low](screenshots/xss_dom_low.png)  
**Why it worked:** The application reads the `default` parameter directly from the URL and uses it to update the DOM via `document.write`. There is no sanitization or encoding.

---

### Medium Security
**Payload:** `?default=English</option></select><img src=x onerror=alert('Medium_XSS')>`
**Result:** Bypassed the server-side `<script` block by using an `img` tag event handler.
**Screenshot:** ![XSS DOM Medium](screenshots/xss_dom_med.png)  
**Why it was harder:** The server-side PHP code uses `stripos()` to check if the input contains `<script`. If it does, the request is redirected to the default page.
**Why it still worked:** The protection only looks for a specific tag. By using different tags or event handlers like `onerror` or `onload`, the filter is bypassed while still achieving script execution in the DOM.

---

### High Security
**Payload:** `?default=English#</option></select><img src=x onerror=alert('High_XSS')>`
**Result:** **Success (Successfully bypassed after reload).**
**Screenshot:** ![XSS DOM High](screenshots/xss_dom_high.png)  
**Why it failed / mitigation used:** The server uses a strict whitelist (`switch` statement) that only allows certain language names. Any value other than the allowed ones in the `default` parameter results in a redirect.
**Why it still worked:** 
1. **Fragment Identifier (#) Bypass:** By putting the payload after the `#`, it is not sent to the server. The PHP whitelist only sees `default=English` and accepts it.
2. **Tag Breakout:** The client-side JavaScript reads the whole URL and injects the fragment into the DOM. By using `</option></select>`, we "break out" of the HTML tags that would otherwise prevent script execution.
3. **Trigger:** The browser executes the injected tag (like an `img` with `onerror`) when the page is rendered or reloaded, triggering the alert box.

---

## 11. XSS (Reflected)

### Low Security
**Payload:** `<script>alert('Low_XSS')</script>`
**Result:** Successfully executed the script via the URL parameter.
**Screenshot:** ![XSS Reflected Low](screenshots/xss_reflected_low.png)  
**Why it worked:** The server-side script takes the `name` parameter from the GET request and echoes it directly into the HTML response inside `<pre>` tags without any sanitization or encoding.

---

### Medium Security
**Payload:** `<SCRIPT>alert('Medium_XSS')</SCRIPT>`
**Result:** Bypassed the case-sensitive and non-recursive `str_replace` filter.
**Screenshot:** ![XSS Reflected Medium](screenshots/xss_reflected_med.png)  
**Why it was harder:** The application uses `str_replace( '<script>', '', ... )` to remove potential script tags.
**Why it still worked:** The filter only looks for the literal string `<script>` in lowercase and is not recursive. By nesting the tag or using mixed case (e.g., `<SCRIPT>`), the filter fails to remove the entire malicious payload.

---

### High Security
**Payload:** `<img src=x onerror=alert('High_XSS')>`
**Result:** Bypassed the regex-based script tag filter by using an alternative HTML tag.
**Screenshot:** ![XSS Reflected High](screenshots/xss_reflected_high.png)  
**Why it failed / mitigation used:** The application uses a more robust regular expression `preg_replace()` that case-insensitively blocks any string following the pattern of a script tag.
**Why it still worked:** The focus of the protection is solely on the `<script>` tag. By using different HTML elements (like `<img>`, `<body>`, or `<a>`) combined with JavaScript event handlers (like `onerror`, `onload`, or `onclick`), the blacklist is effectively bypassed.

---

## 12. XSS (Stored)

### Low Security
**Payload:** `<script>alert('Low_Stored')</script>` (in the Message field)
**Result:** Successfully stored the script; it triggers an alert every time the page is loaded.
**Screenshot:** ![XSS Stored Low](screenshots/xss_stored_low.png)  
**Why it worked:** The server accepts input from the guestbook form and saves it directly to the database without any tags being stripped or encoded. When the page renders the entries, the browser executes the script.

---

### Medium Security
**Payload:** `<SCRIPT>alert('Medium_Stored')</SCRIPT>` (in the Name field)
**Result:** Bypassed the message-field sanitization by targeting the "Name" field with mixed-case tags.
**Screenshot:** ![XSS Stored Medium](screenshots/xss_stored_medium.png)  
**Why it was harder:** The "Message" field uses `strip_tags()`, making it very difficult to inject code. The "Name" field has a `maxlength` limit of 10 characters in the HTML.
**Why it still worked:** I used DevTools to increase the `maxlength` of the "Name" input field. The backend only uses a case-sensitive `str_replace('<script>', ...)` on the name, which I bypassed by using uppercase tags.

---

### High Security
**Payload:** `<img src=x onerror=alert('High_Stored')>` (in the Name field)
**Result:** Successfully bypassed the robust regex filter by using an `img` tag event handler in the "Name" field.
**Screenshot:** ![XSS Stored High](screenshots/xss_stored_high.png)  
**Why it failed / mitigation used:** The application uses a robust regex to block script tags in the "Name" field and `strip_tags()` in the "Message" field.
**Why it still worked:** Similar to the Medium level, the protection focuses on a blacklist. By bypassing the HTML length limit on the "Name" field and using an alternative tag like `<img>` with an `onerror` handler, the security measures were circumvented.

---

## 13. CSP Bypass

### Low Security
**Payload:** `source/jsonp.php?callback=alert`
**Result:** Successfully triggered an alert box (showing `[object Object]`).
**Screenshot:** ![CSP Bypass Low](screenshots/csp_low.png)  
**Why it worked:** The CSP header whitelists `'self'`. The application hosts a JSONP endpoint (`source/jsonp.php`) that reflects the `callback` parameter. By using `alert` as the callback, the browser executes `alert({"answer":"15"})`, proving arbitrary code execution.
**Failure Analysis (Previous Attempts):**
- **External Whitelists (Pastebin):** Attempts to use whitelisted domains like `pastebin.com` failed. Modern browsers often block these "untrusted" scripting sources due to security filters or network connectivity issues within the VM environment.
- **Complex Payloads:** Initial payloads using quotes or comments (e.g., `alert('XSS')`) failed with syntax errors in the console because the browser's URL-decoding/parsing of the injected string into a script tag's `src` attribute was inconsistent. Switching to a direct function name like `alert` solved this.

---

### Medium Security
**Method:** Exploiting a static, predictable nonce.
**Payload:** `<script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">alert('Medium_CSP_Bypass')</script>`
**Result:** Successfully executed an inline script by providing the correct nonce.
**Screenshot:** ![CSP Bypass Medium](screenshots/csp_med.png)  
**Why it was harder:** The CSP policy uses a nonce (`nonce-...`), which should prevent the execution of arbitrary inline scripts. 
**Why it still worked:** THE nonce is hardcoded in the source code ("Never going to give you up" in base64). Since the nonce is not randomly generated for each request, an attacker can simply include it in their own script tags to satisfy the CSP requirement.

---

### High Security
**Payload (Browser Console):**
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
**Result:** Successfully executed code by leveraging the `jsonp.php` file on the same origin via manual form injection.
**Screenshot:** ![CSP Bypass High](screenshots/csp_high.png)  
**Why it failed / mitigation used:** The CSP policy is very strict, only allowing scripts from `'self'`. Furthermore, the application UI removes the input field, implying no direct injection is possible.
**Why it still worked:** 
- **Hidden Vulnerability:** Despite the missing UI, the backend PHP code (`high.php`) still processes the `POST['include']` parameter. 
- **Console Injection:** By using the browser console to inject and submit a hidden form, we can force the payload into the page.
- **Trusting 'self':** Since the injected `<script>` tag points to a local file (`source/jsonp.php`), it satisfies the strict `'self'` policy. Using `alert` as the callback triggers the execution.

---

## 14. JavaScript

### Low Security
**Objective:** Pass the "success" phrase check by providing a valid token.  
**Logic:** `token = md5(rot13(phrase))`  
**Bypass:**  
1. Type `success` in the input field.
2. Open the console (F12) and run `generate_token()`.
3. Click "Submit".
**Result:** Successfully authenticated.
**Screenshot:** ![JavaScript Low](screenshots/js_low.png)  
**Why it worked:** The page uses client-side JavaScript to generate a token from a phrase. By manually triggering the generation function after entering the correct phrase, we satisfy the server's check.

---

### Medium Security
**Logic:** `token = reverse("XX" + phrase + "XX")`  
**Bypass:**  
1. Type `success` in the input field.
2. Open the console and run `do_elsesomething("XX")`.
3. Click "Submit".
**Result:** Successfully authenticated.
**Screenshot:** ![JavaScript Medium](screenshots/js_med.png)  
**Why it worked:** Similar to Low, but the logic is moved to an external script and involves a `setTimeout`. Manually calling the final step of the token generation ensures the token matches the new phrase.

---

### High Security
**Logic:** A multi-stage SHA-256 process:
1. `part1 = reverse(phrase)`
2. `part2 = sha256("XX" + part1)`
3. `part3 = sha256(part2 + "ZZ")` (triggers on click)  
**Bypass:**  
1. Type `success` in the input field.
2. Open the console and run:
   ```javascript
   token_part_1("ABCD", 44);
   token_part_2("QX");
   ```
3. Click "Submit".
**Result:** Successfully authenticated.
**Screenshot:** ![JavaScript High](screenshots/js_high.png)  
**Why it worked:** The High level uses heavy obfuscation and timing/event-based triggers. By manually walking through the first two parts of the algorithm in the console, we prepare the correct intermediate state for the final part that triggers upon clicking "Submit".

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
**Output (from my terminal, shortened for readability):**
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

**Additional verification (backend stack):**
```text
PHP 7.0.30-0+deb9u1
Apache/2.4.25 (Debian)
```

**Analysis:**
- **Application files location:** The DVWA web root is `/var/www/html`, and vulnerable modules are under `/var/www/html/vulnerabilities`.
- **Backend technology:** DVWA runs on PHP + Apache HTTP Server in a Debian-based Linux container.
- **How Docker isolates the environment:** The app runs in a separate container namespace with its own filesystem and internal IP (`172.17.0.2`) on Docker bridge network, while only port `80` is exposed to host port `8080`.

---

## Security Analysis

**Q1. Why does SQL Injection succeed at Low security?**
Low security directly concatenates user input into SQL queries with no parameterization. Attackers can close the intended expression and inject boolean logic (`OR '1'='1`) to alter query behavior.

**Q2. What control prevents it at High?**
In this DVWA lab, SQL Injection can still be exploited at High, so the control is incomplete. Real prevention is server-side prepared statements (parameterized queries) with strict input typing and least-privilege DB accounts.

**Q3. Does HTTPS prevent these attacks? Why or why not?**
No. HTTPS protects data in transit (confidentiality/integrity between client and server), but it does not validate input or fix server-side logic flaws like SQLi, XSS, command injection, or file inclusion.

**Q4. What risks exist if this app is deployed publicly?**
- Account takeover and session hijacking
- Database disclosure and credential leaks
- Remote command execution and server compromise
- Persistent client-side compromise through stored/reflected XSS
- Pivoting to internal systems after foothold

**Q5. OWASP Top 10 Mapping:**

| Vulnerability | OWASP Top 10 Category | Rationale |
|---|---|---|
| Brute Force | A07:2021 – Identification and Authentication Failures | Weak login protection and anti-automation controls |
| Command Injection | A03:2021 – Injection | Unsanitized input reaches OS command execution |
| CSRF | A01:2021 – Broken Access Control | Forged state-changing requests accepted with victim session |
| File Inclusion | A05:2021 – Security Misconfiguration | Unsafe file handling and traversal controls |
| File Upload | A05:2021 – Security Misconfiguration | Insecure validation and executable upload exposure |
| Insecure CAPTCHA | A07:2021 – Identification and Authentication Failures | CAPTCHA/auth bypass due weak server checks |
| SQL Injection | A03:2021 – Injection | SQL query manipulation via untrusted input |
| SQL Injection (Blind) | A03:2021 – Injection | Boolean/time-based inference from injected conditions |
| Weak Session IDs | A07:2021 – Identification and Authentication Failures | Predictable session token generation |
| XSS (DOM / Reflected / Stored) | A03:2021 – Injection | Script execution through untrusted browser-side/response input |
| JavaScript | A04:2021 – Insecure Design | Security logic trusted on client side and bypassed |
| CSP Bypass | A05:2021 – Security Misconfiguration | Policy/nonce/jsonp weaknesses allow script execution |

---
