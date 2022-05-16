# OWASP Top 10 Walkthrough
- [OWASP Top 10 Walkthrough](#owasp-top-10-walkthrough)
  - [Broken_Access_Control](#broken_access_control)
  - [Cryptographic_Failures](#cryptographic_failures)
  - [Injection](#injection)
    - [SQL_Injection](#sql_injection)
    - [xss/cross-site-scripting](#xsscross-site-scripting)
  - [Insecure_Design](#insecure_design)
  - [Security_Misconfiguration](#security_misconfiguration)
  - [Vulnerable_&_Outdated_Components](#vulnerable__outdated_components)
  - [Identification_&_Authentication_Failures](#identification__authentication_failures)
  - [Software_&_Data_Integrity_Failures](#software__data_integrity_failures)
  - [Secure_Logging_&_Monitoring_Failures](#secure_logging__monitoring_failures)
  - [Server-Side_Request_Forgery](#server-side_request_forgery)
- [References](#references)


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

**SQL Injection Mitigation**
Security Control 1: Parameterized Statements

To protect against SQL injection attacks, your code needs to construct SQL strings using bind parameters. Bind parameters are placeholder characters that the database driver will safely replace with some supplied inputs. A SQL statement containing bind parameters is called a parameterized statement.

EXAMPLE 1:

``` HTTP Request
Connection connection = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
Statement statement = connection.createStatement();
String sql = "SELECT * FROM users WHERE email = ? AND encrypted_password = ?";
statement.executeQuery(sql,email,password);
```

The snippet shows a SQL query in parameterized form using`?` as the bind parameter. The code then binds the input values for each parameter to the statement `statement.executeQuery(sql,email,password);`, asking the database driver to insert the parameter values into the SQL statement while securely handling any control characters.

If an attacker were to attempt to hack the snippet above using SQL injection by passing in a username of `jack@gmail.com'--`, the  securely constructed SQL statement will defuse the attack, as shown below:

```HTTP REQUEST
statement.executeQuery(
  "SELECT * FROM users WHERE email = ? AND encrypted_password = ?",
  "billy@email.com'--,",
  "Z$DSA92H0");
```

In the above snippet, the database driver makes sure not to terminate the SQL statement early, this `SELECT` statement will safely return no users, and the attack will fail.

Similar injection attacks may be possible whenever a web-sever communicates with a backend by constructing a statement in the backend's native language (e.g., MongoDB, Apache Cassandra, Redis, Memcached, LDAP). Ensure understanding of how each communication works before using in development code.

Security Control 2: Object-Relational Mapping (ORM)

Many web server libraries and frameworks abstract away the explicit construction of SQL statements in code and allow you to access data objects by using object-relational mapping. Object-relational mapping (ORM) libraries map rows in database tables to code objects in memory, meaning the developer generally doesn’t have to write their own SQL statements in order to read from and update the database. This architecture protects against SQL injection attacks under most circumstances, but can still be vulnerable if custom SQL statements are used—so it’s important to understand how your ORM works behind the scenes.

ActiveRecord is an object-relational mapping (ORM). It's a layer of Ruby code that runs between your database and your logic code.

When you need to make changes to the database, you'll write Ruby code, and then run "migrations" which makes the actual changes to the database.

The ORM that people are probably most familiar with is the Ruby on Rails ActiveRecord framework. 

Example 1:

```Ruby on Rails
User.find_by(email: "jack@gmail.com")

```
In the above script, we use Ruby on Rails code that looks up a user by email in a way that is protected against injection attacks because the SQL is abstracted away. Because ORMs use bind parameters under the hood, they protect against injection attacks in most cases. However, most ORMs also have backdoors that allow the developer to write raw SQL if needed. 

Example 1:

``` Ruby on Rails
def find_user(email, password)
  User.where("email = '" + email + "' and encrypted_password = '" + password + "'")
end
```

Because the above code passes part of the SQL statements as a raw string, an attacker can pass in special characters to manipulate the SQL statement that Rails generates. If the attacker can set the password variable to `'` OR `1=1`, they could run a SQL statement that disables the password check: 

```SQL
SELECT * FROM users WHERE email='jack@gmail.com' AND encrypted_password ='' OR 1=1
```
Parameterization of `email` and `password` can mitigate this issue:

```Ruby on Rails
def find_user(email, encrypted_password)
  User.where(["email = ? and encrypted_password = ?", email, encrypted_password])
end
```


### xss/cross-site-scripting
`Cross-site Scripting`

TODO: Add Content


## Insecure_Design

`Insecure Design` s a new category from 2021, with a focus on risks related to design flaws. If we genuinely want to “move left” as an industry, it calls for more use of threat modeling, secure design patterns and principles, and reference architectures.

TODO: Add Content

## Security_Misconfiguration

The former category `XML External Entities (XXE)` is part of this category.

TODO: Add Content

## Vulnerable_&_Outdated_Components

`Vulnerable & Outdated Components` was previously `Using Components with Known Vulnerabilities`

TODO: Add Content

## Identification_&_Authentication_Failures

`Identification & Authentication Failures` was previously `Broken Authentication` and is sliding down from the second position, and now includes CWEs that are more related to identification failures.The Increased availability of standardized frameworks has reduced the target surface.

## Software_&_Data_Integrity_Failures

`Software & Data Integrity Failures`is a new category from 2021, focusing on making assumptions related to software updates, critical data, and CI/CD pipelines without verifying integrity.One of the highest weighted impacts from Common Vulnerability and Exposures/Common Vulnerability Scoring System (CVE/CVSS) data mapped to the 10 CWEs in this category.

TODO: Add Content

## Secure_Logging_&_Monitoring_Failures

`Secure Logging & Monitoring Failures` was previously `Insufficient Logging & Monitoring` and moving up from #10 previously. This category is challenging to test for, and isn’t well represented in the CVE/CVSS data. However, failures in this category can directly impact visibility, incident alerting, and forensics.

TODO: Add Content

## Server-Side_Request_Forgery

TODO: Add Content

# References
Mcdonald, M., 2020. Web Security for Developers. 1st ed. San Francisco: No Startch Press.
