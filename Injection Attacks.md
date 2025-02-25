## **5.1 Introduction to Injection Attacks**

Injection attacks are a class of security vulnerabilities where an attacker “injects” malicious input into a program. This input is then interpreted as part of a command or query by the underlying system. The most common form of injection is SQL Injection (SQLi), which exploits insecure code that dynamically builds SQL queries by concatenating unvalidated user input. The result is unauthorized data access or manipulation, and in some cases, full control over the system.
![[Pasted image 20250225160921.png]]
### **5.1.2 SQL Introduction**

Structured Query Language (SQL) is the standard language used to interact with relational databases. SQL is used to perform a variety of operations such as querying, updating, inserting, and deleting data. It provides the syntax and structure for how data is organized in databases. Understanding SQL is essential because it forms the backbone of most data-driven applications.

### **5.1.3 SQL & Databases (DBS)**

Databases (DBS) store and manage data in structured formats. SQL serves as the interface between applications and these databases. Most modern applications rely on databases like MySQL, PostgreSQL, Microsoft SQL Server, and Oracle. SQL commands such as `SELECT`, `INSERT`, `UPDATE`, and `DELETE` are used to manipulate data. The integrity and security of these databases depend on how well SQL queries are constructed and safeguarded against malicious inputs.

### **5.1.4 Parameters in SQL Queries**

Parameters are placeholders used in SQL queries to safely insert user-supplied data. Instead of directly concatenating input into SQL statements, using parameters (or parameterized queries) ensures that user input is treated strictly as data rather than executable code. This practice helps prevent SQL Injection attacks by separating query logic from input values. For example, in PHP using MySQLi or PDO, prepared statements allow you to bind parameters securely.

## **5.2 SQLi types**
SQL Injection methods vary in complexity (see Figure __)
![[Untitled diagram-2025-02-22-181631.png]]
### 5.2.1 In-Band SQL Injection

In-Band SQL Injection, also known as Classic SQLi, is the most common and straightforward form of SQL injection. Attackers use the same communication channel to both launch the attack and retrieve the results. This category includes:

- **Error-Based SQL Injection**: Attackers manipulate inputs to generate database errors, which can reveal valuable information about the database structure.
    
- **Union-Based SQL Injection**: Utilizes the `UNION` SQL operator to combine the results of two or more queries, enabling attackers to extract data from different database tables.
    

### 5.2.2 Inferential (Blind) SQL Injection

In Inferential or Blind SQL Injection, attackers do not receive direct feedback from the database. Instead, they infer information based on the application's behavior and responses. This type includes:

- **Boolean-Based Blind SQL Injection**: Attackers send queries that result in different responses based on whether the injected statement is true or false, allowing them to deduce information from the application's behavior.
    
- **Time-Based Blind SQL Injection**: Attackers execute queries that cause time delays in the database's response, inferring data based on the time taken to respond.
    

### 5.2.3 Out-of-Band SQL Injection

Out-of-Band SQL Injection occurs when attackers cannot use the same channel to launch attacks and retrieve data. Instead, they rely on the database server's ability to make external requests, such as DNS or HTTP, to exfiltrate data. This method is less common and depends on specific features being enabled on the database server.

### 5.2.4 Second-Order (Stored) SQL Injection

Second-Order SQL Injection involves injecting malicious SQL code into an application where it is stored for later execution. The payload is not executed immediately but is triggered when the stored data is subsequently used in a different SQL query, often in a different context or by a different function within the application.

### 5.2.5 Conditional SQL Injection

Conditional SQL Injection involves injecting SQL code that includes conditional statements, allowing attackers to execute different queries based on specific conditions. This technique can be used to bypass authentication or extract specific information based on the application's logic.

## **5.3 SQLi Labs & Prevention**

The following labs will use DVWA stands for **Damn Vulnerable Web Application**. which is a deliberately insecure web application designed to help security professionals and students learn about and practice exploiting common web vulnerabilities in a safe, controlled environment. DVWA covers various attack vectors like SQL injection, XSS, CSRF, and more, making it a popular tool for hands-on learning in web application security, often in line with OWASP principles and guidelines.
### **5.3.1 Lab 1: SQLi to retrieve data (LOW)**

A SQL injection attack consists of insertion or "injection" of a SQL query via the input data from the client to the application. A successful SQL injection exploit can read sensitive data from the database, modify database data (Insert/Update/Delete), execute administration operations on the database (such as shutdown the DBMS), recover the content of a given file present on the DBMS file system and in some cases issue commands to the operating system. SQL injection attacks are a type of injection attack, in which SQL commands are injected into data-plane input in order to effect the execution of predefined SQL commands.

The 'id' variable within this PHP script is vulnerable to SQL injection.

There are 5 users in the database, with id's from 1 to 5. Your mission... to steal passwords!

#### **5.3.1.1 Happy Scenario**
![[Pasted image 20250222141812.png]]
The standard is you first try the happy scenario in which you act as a normal user (normal input)
as shown in figure____ 

#### **5.3.1.2 Test for SQL Injection**

- Enter **`1'`** (with a single quote) and submit.
- If an **error message** appears (e.g., "Syntax error in SQL query"), it means there’s a SQL injection vulnerability but with some escaping applied.
`You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''3''' at line 1`
#### **5.3.1.3 Given Query(LOW Security):**
![[Pasted image 20250222125908.png]]
shown in figure __
php:`$getid = "SELECT first_name, last_name FROM users WHERE user_id = '$id'";`
This SQL query selects the `first_name` and `last_name` from the `users` table where the `user_id` matches the variable `$id`
If the attacker enters:
`3' or 1=1#`
then `$id` becomes:
`'3' or 1=1#`

**note that:** the attacker doesn't know the php code when attacking it's shown here for the purpose of learning.
##### **5.3.1.4 How the Query is Modified:**
sql:`SELECT first_name, last_name FROM users WHERE user_id = '3' or 1=1#';`
🔍 **Breakdown:**
1. **`3'`** → Closes the opening single quote (`'`) in the original query.
2. **`or 1=1`** → This is always true, so the condition applies to all rows.
3. **`#`** → This **comments out** the rest of the query (like a password check or extra conditions), preventing errors.

![[vmware_WPmQwZcxhX 1.png]]
The result of the injection shown in figure __
- **Bypasses Filtering:** Instead of selecting only the user with `user_id = 3`, the **`1=1`** condition ensures **all rows** are returned.
- **Leaks Data:** The attacker gets **all user names** from the database.
- **Possible Authentication Bypass:** If this query were checking credentials, it could allow login without a valid password.


### **5.3.2 Lab 2: SQLi to retrieve data (medium)**

In the Medium security level, the backend **uses `mysqli_real_escape_string()`** to escape special characters in user input. This prevents **simple single-quote-based SQL injections**.
as shown in figure __

The function then escapes special characters in the `unescaped_string`, taking into account the current character set of the connection so that it is safe to place it in a [mysql_query()]. If binary data is to be inserted, this function must be used.
**mysql_real_escape_string()** calls MySQL's library function mysql_real_escape_string, which prepends backslashes to the following characters: `\x00`, `\n`, `\r`, `\`, `'`, `"` and `\x1a`.

#### **5.3.2.1 Given Query(See figure __):**

![[Pasted image 20250222135853.png]]


##### **5.3.2.2 How the Query is Modified(Bypass with Numeric Injection):**

Since `mysqli_real_escape_string()` escapes quotes, try a **numeric injection**:
 - Enter in the input field:
	`1 OR 1=1`
- The query becomes:
	sql
	`SELECT first_name, last_name FROM users WHERE user_id = '1' OR 1=1;`
- Since `1=1` is **always true**, it retrieves all users from the database. as shown in figure __
![[Pasted image 20250222152123.png]]
Some databases might add a **LIMIT clause** or extra conditions that break the injection.  
To ignore the rest of the query, use SQL comments (`--` or `#`):

`1 OR 1=1 --` 
or                    see figure __
`1 OR 1=1 #`               

- This **terminates the SQL query after `1=1`**, ignoring anything else after. 
![[Pasted image 20250222152302.png]]
### **5.3.3 LAB 3: SQLi to retrieve data (High)**

High mode completely mitigate SQL injection vulnerabilities by processing(Sanitization) the user input first with the functions: (as shown in figure __)
`\$id = stripslashes($id);`
`\$id = mysql_real_escape_string($id);`

![[Pasted image 20250222190516.png]]
#### **5.3.3.1 `$id = stripslashes($id);`**
- The `stripslashes()` function removes backslashes (`\`) from the input.
- This is useful if `magic_quotes_gpc` (an old PHP feature) is enabled, which automatically adds slashes to escape characters like quotes (`'` and `"`).
- **Protection:** Helps prevent breaking out of string values in SQL queries.
#### **5.3.3.2 `$id = mysql_real_escape_string($id);`**
- This function **escapes special characters** (like quotes `'`, `"` and backslashes `\`) so they are treated as text rather than part of an SQL command.
- **Protection:** Prevents an attacker from injecting malicious SQL statements.
#### **5.3.3.3 After Sanitization**
After `mysql_real_escape_string()`, `stripslashes($id)`
Now if the input is 3' or 1=1 #  // 3 or 1=1 # 

the input becomes: 
`3\' OR 1=1 #`

This prevents SQLi because the query would now treat the input as a literal string, making the attack ineffective and no error message anymore.

###### **⚠️ Note: `mysql_real_escape_string()` is Deprecated!**
- This function is **no longer recommended** as it is **deprecated in PHP 5.5+** and removed in PHP 7+.
- **Better Approach:** Use **prepared statements** with `mysqli` or `PDO` to prevent SQLi effectively.

