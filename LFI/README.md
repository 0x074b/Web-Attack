# Table of Contents

- [Definition](#definition)
- [Identifying](#identifying)
- [PHP Wrappers](#php-wrappers)
- [LFI via /proc/self/environ](#lfi-via-procselfenviron)
- [Null Byte Technique](#null-byte-technique)
- [Truncation LFI Bypass](#truncation-lfi-bypass)
- [Log File Contamination](#log-file-contamination)
- [Email a Reverse Shell](#email-a-reverse-shell)
- [References](#references)

# Definition
Local File Inclusion (LFI) allows an attacker to include files on a server through the web browser. This vulnerability exists when a web application includes a file without correctly sanitising the input, allowing and attacker to manipulate the input and inject path traversal characters and include other files from the web server.

example of PHP code vulnerable to local file inclusion.
``` 
<?php
$file = $_GET[‘file’];
if(isset($file)) {
  include(“pages/$file”);
} else {
  include(“index.php”);
}?>
```

# Identifying
LFI vulnerabilities are easy to identify and exploit. Any script that includes a file from a web server is a good candidate for further LFI testing, for example:

```/script.php?page=index.html```

A penetration tester would attempt to exploit this vulnerability by manipulating the file location parameter, such as:

```/script.php?page=../../../../../../../../etc/passwd```

The above is an effort to display the contents of the ```/etc/passwd``` file on a **UNIX/Linux** based system.

# PHP Wrappers
PHP has a number of wrappers that can often be abused to bypass various input filters.

## PHP Expect Wrapper
PHP ```expect://``` allows execution of system commands, unfortunately the expect PHP module is not enabled by default.

```php?page=expect://ls```

The payload is sent in a POST request to the server such as:

```/fi/?page=php://input&cmd=ls```

Example using ```php://input``` against DVWA:

![image](https://github.com/user-attachments/assets/715dc32f-1aea-4899-b765-262469f5d7a5)

Web Application Response:

![image](https://github.com/user-attachments/assets/754ef1b3-9c85-4717-bff3-35a793d295bb)

```php://filter```

allows a pen tester to include local files and base64 encodes the output. Therefore, any base64 output will need to be decoded to reveal the contents.

```vuln.php?page=php://filter/convert.base64-encode/resource=/etc/passwd```

![image](https://github.com/user-attachments/assets/e6bfc8f4-fb9e-49c6-83bc-ccf03c961699)

Base64 decoding the string provides the ```/etc/passwd``` file:

![image](https://github.com/user-attachments/assets/1b0e5194-b307-4020-b469-e3c6d392e34c)

php://filter can also be used without base64 encoding the output using:

```?page=php://filter/resource=/etc/passwd```

![image](https://github.com/user-attachments/assets/4419d47c-95c7-4ba1-a509-2f1f8be18a3c)

## PHP ZIP Wrapper LFI
The zip wrapper processes uploaded .zip files server side allowing a penetration tester to upload a zip file using a vulnerable file upload function and leverage he zip filter via an LFI to execute. A typical attack example would look like:

- 1. Create a PHP reverse shell
- 2. Compress to a .zip file
- 3. Upload the compressed shell payload to the server
- 4. Use the zip wrapper to extract the payload using: ```php?page=zip://path/to/file.zip%23shell```
- 5. The above will extract the zip file to shell, if the server does not append .php rename it to shell.php instead

If the file upload function does not allow zip files to be uploaded, attempts can be made to bypass the file upload function (see: OWASP file upload testing document).

# LFI via /proc/self/environ
If it’s possible to include ```/proc/self/environ``` via a local file inclusion vulnerability, then introducing source code via the User Agent header is a possible vector. Once code has been injected into the User Agent header a local file inclusion vulnerability can be leveraged to execute ```/proc/self/environ``` and reload the environment variables, executing your reverse shell.

## Useful Shells
Useful tiny PHP back doors for the above techniques:

```<? system(‘uname -a’);?>```

# Null Byte Technique
Null byte injection bypasses application filtering within web applications by adding URL encoded "Null bytes" such as ```%00```. Typically, this bypasses basic web application blacklist filters by adding additional null characters that are then allowed or not processed by the backend web application.

Some practical examples of null byte injection for LFI:

```vuln.php?page=/etc/passwd%00```
```vuln.php?page=/etc/passwd%2500```

# Truncation LFI Bypass
Truncation is another blacklist bypass technique. By injecting long parameter into the vulnerable file inclusion mechanism, the web application may "cut it off" (truncate) the input parameter, which may bypass the input filter.

# Log File Contamination
Log file contamination is the process of injecting source code into log files on the target system. This is achieved by introducing source code via other exposed services on the target system which the target operating system/service will store in log files. For example, injecting PHP reverse shell code into a URL, causing syslog to create an entry in the apache access log for a 404 page not found entry. The apache log file would then be parsed using a previously discovered file inclusion vulnerability, executing the injected PHP reverse shell.

After introducing source code to the target systems log file(s) the next step is identifying the location of the log file. During the recon and discovery stage of penetration testing the web server and likely the target operating system would have been identified, a good starting point would be looking up the default log paths for the identified operating system and web server (if they are not already known by the consultant). FuzzDB’s Burp LFI payload lists can be used in conjunction with Burp intruder to quickly identify valid log file locations on the target system.

Some commonly exposed services on a **Linux/UNIX** systems are listed below:

## Apache/Nginx
Inject code into the web server access or error logs using netcat, after successful injection parse the server log file location by exploiting the previously discovered LFI vulnerability. If the web server access/error logs are long, it may take some time execute your injected code.

# Email a Reverse Shell
If the target machine relays mail either directly or via another machine on the network and stores mail for the user www-data (or the apache user) on the system then it’s possible to email a reverse shell to the target. If no MX records exist for the domain but SMTP is exposed it’s possible to connect to the target mail server and send mail to the www-data/apache user. Mail is sent to the user running apache such as www-data to ensure file system permissions will allow read access the file ```/var/spool/mail/www-data``` containing the injected PHP reverse shell code.

First enumerate the target system using a list of known **UNIX/Linux** account names:

![image](https://github.com/user-attachments/assets/9dce5cad-b7b9-46b6-b5f2-0b0ab3209bff)

The following screenshot shows the process of sending email via telnet to the www-data user:

![image](https://github.com/user-attachments/assets/8160b5c9-a560-4cab-a77c-36a1b7b967a0)

![image](https://github.com/user-attachments/assets/5832238e-2e3f-468d-969e-dbbe9b47b38e)

![image](https://github.com/user-attachments/assets/ff2de270-0617-44e6-9f2d-f7c61dc5c8e0)

# References

- https://www.owasp.org/index.php/PHP_File_Inclusion
- DVWA (used for LFI examples): http://www.dvwa.co.uk/






















