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
