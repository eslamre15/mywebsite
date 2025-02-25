<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pentesting intro</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            margin: 20px;
            padding: 20px;
        }
        h1, h2, h3 {
            color: #333;
        }
        a {
            color: blue;
            text-decoration: none;
        }
        a:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>
    <h1>Injection Attacks</h1>
## **5.1 Introduction to Injection Attacks**

Injection attacks are a class of security vulnerabilities where an attacker "injects" malicious input into a program. This input is then interpreted as part of a command or query by the underlying system. The most common form of injection is SQL Injection (SQLi), which exploits insecure code that dynamically builds SQL queries by concatenating unvalidated user input. The result is unauthorized data access or manipulation, and in some cases, full control over the system.

![Injection Attack](Pasted%20image%20250225160921.png)

### **5.1.2 SQL Introduction**

Structured Query Language (SQL) is the standard language used to interact with relational databases. SQL is used to perform a variety of operations such as querying, updating, inserting, and deleting data. It provides the syntax and structure for how data is organized in databases. Understanding SQL is essential because it forms the backbone of most data-driven applications.

### **5.1.3 SQL & Databases (DBS)**

Databases (DBS) store and manage data in structured formats. SQL serves as the interface between applications and these databases. Most modern applications rely on databases like MySQL, PostgreSQL, Microsoft SQL Server, and Oracle. SQL commands such as `SELECT`, `INSERT`, `UPDATE`, and `DELETE` are used to manipulate data. The integrity and security of these databases depend on how well SQL queries are constructed and safeguarded against malicious inputs.

### **5.1.4 Parameters in SQL Queries**

Parameters are placeholders used in SQL queries to safely insert user-supplied data. Instead of directly concatenating input into SQL statements, using parameters (or parameterized queries) ensures that user input is treated strictly as data rather than executable code. This practice helps prevent SQL Injection attacks by separating query logic from input values. For example, in PHP using MySQLi or PDO, prepared statements allow you to bind parameters securely.

## **5.2 SQLi Types**

SQL Injection methods vary in complexity (see Figure below)

![SQLi Types](Untitled%20diagram-2025-02-22-181631.png)

### **5.2.1 In-Band SQL Injection**

In-Band SQL Injection, also known as Classic SQLi, is the most common and straightforward form of SQL injection. Attackers use the same communication channel to both launch the attack and retrieve the results. This category includes:

- **Error-Based SQL Injection**: Attackers manipulate inputs to generate database errors, which can reveal valuable information about the database structure.
- **Union-Based SQL Injection**: Utilizes the `UNION` SQL operator to combine the results of two or more queries, enabling attackers to extract data from different database tables.

### **5.2.2 Inferential (Blind) SQL Injection**

In Inferential or Blind SQL Injection, attackers do not receive direct feedback from the database. Instead, they infer information based on the application's behavior and responses. This type includes:

- **Boolean-Based Blind SQL Injection**: Attackers send queries that result in different responses based on whether the injected statement is true or false.
- **Time-Based Blind SQL Injection**: Attackers execute queries that cause time delays in the database's response, inferring data based on the time taken to respond.

### **5.2.3 Out-of-Band SQL Injection**

Out-of-Band SQL Injection occurs when attackers cannot use the same channel to launch attacks and retrieve data. Instead, they rely on the database server's ability to make external requests, such as DNS or HTTP, to exfiltrate data. This method is less common and depends on specific features being enabled on the database server.

### **5.2.4 Second-Order (Stored) SQL Injection**

Second-Order SQL Injection involves injecting malicious SQL code into an application where it is stored for later execution. The payload is not executed immediately but is triggered when the stored data is subsequently used in a different SQL query, often in a different context or by a different function within the application.

### **5.2.5 Conditional SQL Injection**

Conditional SQL Injection involves injecting SQL code that includes conditional statements, allowing attackers to execute different queries based on specific conditions. This technique can be used to bypass authentication or extract specific information based on the application's logic.

## **5.3 SQLi Labs & Prevention**

The following labs will use **DVWA** (*Damn Vulnerable Web Application*), a deliberately insecure web application designed to help security professionals and students learn about and practice exploiting common web vulnerabilities in a safe, controlled environment.

### **5.3.1 Lab 1: SQLi to Retrieve Data (LOW)**

A SQL injection attack consists of inserting or "injecting" a SQL query via the input data from the client to the application. A successful SQL injection exploit can:

- Read sensitive data from the database
- Modify database data (Insert/Update/Delete)
- Execute administrative operations on the database
- Recover content from the DBMS file system
- Issue commands to the operating system

The `'id'` variable within this PHP script is vulnerable to SQL injection.

#### **5.3.1.1 Happy Scenario**

![Happy Scenario](Pasted%20image%20250222141812.png)

#### **5.3.1.2 Test for SQL Injection**

- Enter **`1'`** (with a single quote) and submit.
- If an **error message** appears (e.g., "Syntax error in SQL query"), it means there’s a SQL injection vulnerability.

```sql
SELECT first_name, last_name FROM users WHERE user_id = '3' or 1=1#;
```

![Injection Result](vmware_WPmQwZcxhX%201.png)

### **5.3.2 Lab 2: SQLi to Retrieve Data (Medium)**

In the Medium security level, the backend **uses `mysqli_real_escape_string()`** to escape special characters in user input, preventing simple single-quote-based SQL injections.

```php
$id = mysqli_real_escape_string($conn, $id);
```

#### **5.3.2.1 Bypassing with Numeric Injection**

Try a **numeric injection**:

```sql
1 OR 1=1 --
```

![Medium SQLi](Pasted%20image%20250222152302.png)

### **5.3.3 Lab 3: SQLi to Retrieve Data (High)**

In High mode, SQL injection vulnerabilities are mitigated using sanitization functions:

```php
$id = stripslashes($id);
$id = mysql_real_escape_string($id);
```

![High Security](Pasted%20image%20250222190516.png)

After applying these sanitization functions, SQL injection is no longer possible as input is safely escaped and treated as a literal string.

---

### **⚠️ Note: `mysql_real_escape_string()` is Deprecated!**

- Deprecated in PHP 5.5+ and removed in PHP 7+.
- **Better Approach:** Use **prepared statements** with `mysqli` or `PDO` to prevent SQLi effectively.
