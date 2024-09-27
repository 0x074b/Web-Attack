# Table Of Contents
- [Definition](#definition)
- [Types of XSS](#types-of-xss)
   * [DOM-based XSS](#dom-based-xss)
   * [Reflected XSS](#reflected-xss)
   * [Stored XSS](#stored-xss)
- [Scenario](#scenario)
   * [Stealing credentials](#stealing-credentials)
   * [Hijacking sessions](#hijacking-sessions)
   * [Compromising a downloads page](#compromising-a-downloads-page)
- [Preventing XSS Attacks](#preventing-xss-attacks)
   * [Validate and sanitize user-provided data](#validate-and-sanitize-user-provided-data)
   * [HTML Encoding](#html-encoding)
   * [Use a security encoding Library](#use-a-security-encoding-library)
   * [Use a Web Application Firewall](#use-a-web-application-firewall)
- [How NOT to prevent XSS attacks](#how-not-to-prevent-xss-attacks)

# Definition
Cross-site scripting occurs when attackers or malicious users can manipulate a web site or web application to return malicious JavaScript to users. When this malicious JavaScript is executed in the user’s browser, all of the user’s interactions with the site (including but not limited to authentication and payment) can be compromised by the attacker.

# Types of XSS
## DOM-based XSS
This type of XSS occurs when user input is manipulated in an unsafe way in the DOM (Document Object Map) by JavaScript. For example, this can occur if you were to read a value from a form, and then use JavaScript to write it back out to the DOM. If an attacker can control the input to that form, then they can control the script that will be executed. Common sources of DOM-based XSS include the ```eval()``` function and the ```innerHTML``` attribute, and attacks are commonly executed through the URL.

```
const username = document.getElementById('username_input');
const username_box = document.getElementById('username_box');
user_name_box.innerHTML = username;
```

To exploit this vulnerability, you could insert a malicious script into the input that would be executed:

```<script>window.alert("Cross site scripting has occurred!");</script>```

## Reflected XSS
Reflected XSS is similar to DOM-based XSS: it occurs when the web server receives an HTTP request, and "reflects" information from the request back into the response in an unsafe manner. An example would be where the server will place the requested application route/URL in the page that is served back to the user. An attacker can construct a URL with a malicious route that contains JavaScript, such that if a user visits the link, the script will execute.

Malicious URLs containing cross-site scripting are commonly used as social engineering helpers in phishing emails or malicious links online.

Here’s an example — given a route that will 404,

```GET https://example.com/route/that/will/404```

a vulnerable server might generate the response like so:

```
<h1>404</h1>
<p> Error: route "/route/that/will/404 was not found on the server</p>
```

An attacker could exploit this by constructing a URL like this:

```https://example.com//route/that/will/404/<script>alert('XSS!');```

When the user loads the page, the URL will be templated into the page, the script tags will be interpreted as HTML, and the malicious script will execute.

## Stored XSS
Stored XSS occurs when user-created data is stored in a database or other persistent storage and is then loaded into a page. Common examples of types of applications that do this include forums, comment plugins, and similar applications. Stored XSS is particularly dangerous when the stored content is displayed to many or all users of the application, because then one user can compromise the site for any user that visits it, without requiring that they click on a specific link.

For example, suppose that a forum thread’s posts are stored in a database and that they’re loaded whenever someone visits the thread and displayed. A malicious user could leave a comment that contains malicious JavaScript between ```<script></script>``` tags in their post, and then the script would execute in the browser of any user that visits the page.

For example, their post in the threat might look something like this:

```This is some text replying to the thread <script>alert('XSS');</script>```

# Scenario
## Stealing credentials
Suppose that an attacker has discovered a cross-site scripting vulnerability in a login page on a website. They could inject JavaScript to add an event listener to the form, such that whenever it is submitted it captures the username and password of the user that’s trying to log in and sends them to a server controlled by the attacker:

![image](https://github.com/user-attachments/assets/2aeb58fc-d3ca-4c93-953d-7fd69374d1dc)

## Hijacking sessions
Suppose that our attacker has discovered a stored XSS vulnerability in a forum page. For the sake of this example, the forum is storing session without the ```HttpOnly``` attribute.

The attacker could inject a script to grab the session cookie of anyone that is logged in to the forum that views the thread, and could impersonate their user on the forum or the site at large:

![image](https://github.com/user-attachments/assets/98670d32-199e-47f8-82a4-2d4396b9e905)

Similar methods can be used to grab tokens from ```localStorage``` and ```sessionStorage```.

## Compromising a downloads page
Suppose that the attacker has compromised the download page of a website with a cross-site scripting attack. They could use an XSS payload to modify the download links so that instead of attempting to download the intended software, they point to malicious software hosted on the attacker’s server. When users load the page and attempt to download the intended software, they are served malware from the attacker’s server:

![image](https://github.com/user-attachments/assets/ebab2b18-9bf2-49f0-ae2b-04a0c06dac3d)

# Preventing XSS Attacks
XSS vulnerabilities are incredibly easy to create by accident. To prevent them, you need to put in place good coding practices, code review processes, and multiple layers of defense.

The easiest way to prevent XSS would be to never allow users to supply data that is rendered into the page, but the fact is that this isn’t a practical answer, since most applications store and manipulate user input in some form. Unfortunately, there is no single foolproof way to prevent XSS. Therefore, it is important to have multiple layers of defense against cross-site scripting.

## Validate and sanitize user-provided data
User data should be validated on the front end of sites for correctness (e.g. email and phone number formatting), but it should also always be validated and sanitized on the backend for security. Depending on the application, you may be able to whitelist alphanumeric characters and blacklist all other characters. However, this solution is not foolproof. It may help mitigate attacks, but it cannot prevent them entirely.

## HTML Encoding
Any time that you are rendering user-provided data into the body of the document (e.g. with the ```innerHTML``` attribute in JavaScript), you should HTML encode the data. However, this may not always prevent XSS if you are placing user-provided data in HTML tag attributes and is not effective against placing untrusted data inside of a ```<script></script>``` tag. If you decide to place user-provided data in HTML tag attributes, ensure that you are always using quotes around your attributes.

## Use a security encoding Library
For many languages and frameworks, there are security encoding libraries that can help prevent XSS. For example, OWASP has one such library for Java. You should consider using a similar library for your web projects.

## Use a Web Application Firewall
It may seem like overkill, but there are web application firewalls designed to specifically prevent common web attacks such as XSS and SQL Injection. Using a web application firewall (WAF) is not necessary for most applications, but for applications that require strong security, they can be a great resource. One such WAF is ModSecurity, which is available for Apache, Nginx, and IIS. Check out their wiki for more information.

# How NOT to prevent XSS attacks
There are lots of great ways to mitigate and prevent XSS attacks, but there are also lots of really bad ways to try and prevent it. Here are some common ways that people try to prevent XSS that are unlikely to be successful:

- searching for ```<``` and ```>``` characters in user-supplied data
- searching for ```<script></script>``` tags in user-supplied data
- using regexes to try and filter out script tags or other common XSS injections

In reality, XSS payloads can be extremely complicated, and can also be extremely obfuscated. Here’s an example:

```<BODY onload!#$%&()*~+-_.,:;?@[/|\]^`=alert("XSS")>```

Cybercriminals often have extremely robust tools that can be used to attempt to bypass filters by obfuscating their XSS payloads. A homebrew regex is probably not going to cut it.





