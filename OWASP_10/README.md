# OWASP Top 10 Walkthrough
1. [Broken Access Control](#broken_access_control)
2. [Cryptographic Failures](#cryptographic_failures)
3. [Injection](#injection)
4. [Insecure Design](#insecure_design)
5. [Security Misconfiguration](#security_misconfiguration)
6. [Vulnerable & Outdated Components](#vulnerable_&_outdated_components)
7. [Identification & Authentication Failures](#identification_&_authentication_failures)
8. [Software & Data Integrity Failures](#software_&_data_integrity_failures)
9. [Secure Logging & Monitoring Failures](#secure_logging_&_monitoring_failures)
10. [Server-Side_Request_Forgery](#server-side_request_forgery)


## Broken_Access_Control
The 34 Common Weakness Enumerations (CWEs) mapped to Broken Access Control had more occurrences in applications than any other category.

## Cryptographic_Failures
 `Cryptographic Failures` previously `Sensitive Data Exposure`; which was a broad symptom rather than a root cause. The renewed focus of the category is on failures related to cryptography which often leads to sensitive data exposure or system compromise.

## Injection
For common types of injection
1. [SQL Injection](#sql_injection)
2. [Command Injection](#command_injection)
3. [Remote Code Injection](#remote_code_injection)
5. [Cross-site Scripting](#xss/cross-site-scripting)
4. [File Upload Vulnerabilities Vulnerability Exploitation](#file_upload_vulnerabilities)

### SQL_Injection

`SQL Injection` attacks target websites that use an unerlying SQL-database and construct data queries to the database in an insecure manner.

**What is SQL**
`Structured Query Language (SQL)`, extracts data and data structures in relational databases. Relational databases, store data in tables; each row in a table is a `data item` SQL syntax allows applications such as webservers to add rows to the database by using `INSERT` statements, read rows by using `SELECT` statements, update rows by using `UPDATE` statements, and remove rows by using `DELETE` statements. EXAMPLES:

```SQL
INSERT INTO users (email, encrypted_password)
VALUES ('jack@gmail.com', '$10$WMT8Z')

SELECT FROM users WHERE email='jack@gmail.com'
AND encrypted_password = '$10$WMT8Z'

UPDATE TABLE users encrypted_password = '$3D$MW10Y'
WHERE email='jack@gmail.com'

DELETE FROM users WHERE email = 'jack@gmail.com'
```

**SQL Injection Attacks**
SQL injection attacks occur when the web server insecurely constrcts the SQL statement it passes to the database driver. This allows the attacker to pass arguments via the HTTP request that cause the drivers to perform undesired actions. 

EXAMPLE 1:

```HTTP REQUEST
Connection connection = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
Statement statement = connection.createStatement();
String sql = "SELECT * FROM users WHERE email = '" + email + "' AND encrypted_password='" + password + "'";

```

The snippet uses the `email` and `password` parameters taken from the `HTTP request`, and inserts them directly into the SQL statement. Because the parameters aren't checked for SQL control characters (e.g, ') that change the meaning of the SQL statement a hacker can craft input that bypasses the websites authentication system. 


EXAMPLE 2:

```HTTP REQUEST
statement.executeQuery(
    "Select * FROM users WHERE email='jack@gmail.com' --' AND encrypted_password='Z$DSA92H0'");
```

The snippet shows the attacker passing the user `email` parameter as `jack@gmail.com'--`, which terminates the SQL statement early and causes the password-checking logic to not execute. The database driver executes only the SQL statement `--`, and ignores everything that comes after `AND encrypted_password='Z$DSA92H0'`

EXAMPLE 3:

```HTTP REQUEST
statement.executeQuery(
    "Select * FROM users WHERE email='jack@gmail.com';
    DROP TABLE users; --' AND encrypted_password='Z$DSA92H0'");
```

The snippet shows the attacker passing the `email` parameter as `jack@gmail.com; DROP TABLE users;--`. the `;` terminates the first SQL statement `Select * FROM users WHERE email='jack@gmail.com'`, after which the attacker inserts an additional, malicious statement `DROP TABLE users`.

**SQL Injection Mitigations**
Security Control 1:
Security Control 2:
Security Control 3:
Security Control 4:

### xss/cross-site-scripting
`Cross-site Scripting`



## Insecure_Design

`Insecure Design` s a new category from 2021, with a focus on risks related to design flaws. If we genuinely want to “move left” as an industry, it calls for more use of threat modeling, secure design patterns and principles, and reference architectures.

## Security_Misconfiguration

The former category `XML External Entities (XXE)` is part of this category.

## Vulnerable_&_Outdated_Components

`Vulnerable & Outdated Components` was previously `Using Components with Known Vulnerabilities`

## Identification_&_Authentication_Failures

`Identification & Authentication Failures` was previously `Broken Authentication` and is sliding down from the second position, and now includes CWEs that are more related to identification failures.The Increased availability of standardized frameworks has reduced the target surface.

## Software_&_Data_Integrity_Failures

`Software & Data Integrity Failures`is a new category from 2021, focusing on making assumptions related to software updates, critical data, and CI/CD pipelines without verifying integrity.One of the highest weighted impacts from Common Vulnerability and Exposures/Common Vulnerability Scoring System (CVE/CVSS) data mapped to the 10 CWEs in this category.

## Secure_Logging_&_Monitoring_Failures

`Secure Logging & Monitoring Failures` was previously `Insufficient Logging & Monitoring` and moving up from #10 previously. This category is challenging to test for, and isn’t well represented in the CVE/CVSS data. However, failures in this category can directly impact visibility, incident alerting, and forensics.

## Server-Side_Request_Forgery