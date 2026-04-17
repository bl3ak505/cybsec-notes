### 📌 A03: Injection 

**Definition**

- Occurs when untrusted, attacker-controlled data is sent to an interpreter as part of a command or query — tricking it into executing unintended instructions.

> The OG of web vulns. SQL injection alone has been destroying databases since the 90s and still isn't dead 💀

---

**Core Concepts**

- **Root cause**: application trusts user input and passes it directly to an interpreter without sanitization
- Affects any interpreter: SQL, OS shell, LDAP, XPath, NoSQL, HTML/JS (XSS), XML (XXE)
- Two main goals for an attacker: **extract data** or **execute commands**
- Input comes from _anywhere_: URL params, form fields, HTTP headers, cookies, JSON body
- **SQLi** is the most common/well-known but injection is a whole family of attacks
- **Second-order injection**: payload stored first, executed later (sneaky 😈)

---

**How It Works**

**SQL Injection flow:**

- App builds query by string concatenation:
    
    ```sql
    SELECT * FROM users WHERE username = '$input';
    ```
    
- Attacker inputs: `' OR '1'='1`
- Query becomes:
    
    ```sql
    SELECT * FROM users WHERE username = '' OR '1'='1';
    ```
    
- `1=1` is always true → returns all rows → auth bypass ✅

**Types of SQLi:**

- **In-band (Classic)**: results returned directly in the response
    - _Error-based_: force DB errors that leak schema info
    - _Union-based_: append `UNION SELECT` to pull data from other tables
- **Blind SQLi**: no visible output — infer data from app behavior
    - _Boolean-based_: ask true/false questions (`AND 1=1` vs `AND 1=2`)
    - _Time-based_: use `SLEEP(5)` — if response delays, condition was true
- **Out-of-band**: data exfiltrated via DNS/HTTP callbacks (rare, used when everything else fails)

**Command Injection flow:**

- App runs OS command with user input:
    
    ```php
    system("ping " . $_GET['host']);
    ```
    
- Attacker inputs: `8.8.8.8; whoami`
- Shell executes: `ping 8.8.8.8` then `whoami` → RCE 💥

---

**Key Terminology**

|Term|Meaning|
|---|---|
|SQL Injection (SQLi)|Injecting SQL code into a database query|
|Command Injection|Injecting OS commands into a shell execution call|
|LDAP Injection|Injecting into LDAP directory queries|
|XSS (Cross-Site Scripting)|Injecting JS into HTML rendered by a browser|
|XXE|XML External Entity — injecting into XML parsers|
|NoSQL Injection|Injecting into MongoDB/CouchDB queries (uses JSON operators)|
|Parameterized Query|Prepared statement — separates code from data, prevents SQLi|
|ORM|Object Relational Mapper — abstraction layer that can prevent SQLi if used correctly|
|Second-order injection|Payload stored in DB, injected when retrieved/used later|
|UNION attack|SQLi technique to retrieve data from other tables|
|Blind SQLi|Injection where output isn't directly visible in response|
|WAF|Web Application Firewall — can detect/block injection attempts (bypassable 😅)|

---

**Important Technical Details**

- **SQLi detection**: single quote `'` → triggers DB error = confirmed injection point
- **sqlmap** automates the entire SQLi exploitation chain:
    
    ```bash
    sqlmap -u "http://target.com/page?id=1" --dbssqlmap -u "http://target.com/page?id=1" -D dbname --tablessqlmap -u "http://target.com/page?id=1" -D dbname -T users --dump
    ```
    
- **UNION attack** requires matching column count:
    
    ```sql
    ' UNION SELECT NULL,NULL,NULL--
    ```
    
    Keep adding NULLs until no error → found column count
- **Comment syntax** varies by DB:
    - MySQL: `--`, `#`
    - MSSQL/Oracle: `--`
    - Oracle: needs `FROM DUAL` in SELECT
- **NoSQL injection** (MongoDB) example:
    
    ```json
    {"username": {"$gt": ""}, "password": {"$gt": ""}}
    ```
    
    → `$gt: ""` matches anything → auth bypass
- **Stacked queries** (MSSQL/PostgreSQL): allows multiple statements via `;` → can lead to RCE via `xp_cmdshell` on MSSQL
- Prevention: **parameterized queries / prepared statements** — this fully neutralizes SQLi because input is never interpreted as code

---

**Real-World Example**

- **Heartland Payment Systems (2008)**: SQLi → 130 million credit cards stolen. Still one of the largest data breaches ever 😬
- **Classic auth bypass**:
    - Username: `admin'--`
    - Password: `anything`
    - Query: `SELECT * FROM users WHERE username='admin'--' AND password='anything'`
    - Everything after `--` is commented out → logs in as admin with no password check ✅
- **Bobby Tables**: `Robert'); DROP TABLE Students;--` — yes, the XKCD comic is a real attack vector 😂

---

> 🔑 **CEH angle:** Know all SQLi types cold — especially blind vs in-band. Know that parameterized queries / prepared statements are THE fix. Be familiar with sqlmap usage. XSS and XXE are often tested as separate injection sub-categories. WAF bypass techniques (encoding, case variation, comments) are bonus points. 🔒