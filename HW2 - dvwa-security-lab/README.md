## DVWA Security Report

## Command Injection

### Security Level: Low

**Payload Used:** `127.0.0.1; ls`

**Result:** Success - Both the ping command and the directory listing command executed, displaying the contents of the current directory after the ping results.

**Screenshot:**
![Command Injection - Low Security](screenshots/command-injection-low.png)

**Explanation of why it worked:**
The application passes user input directly to the system shell without any validation or sanitization. The semicolon (;) acts as a command separator, allowing multiple commands to be chained together. Since the input is not checked to ensure it contains only an IP address, the system executes both the intended ping command and the injected ls command.

**Explanation of why it failed at higher level:**
*This section will be completed after testing Medium and High security levels.*
