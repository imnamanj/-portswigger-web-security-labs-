### Lab 1 — Username Enumeration via Different Responses ✅
**Difficulty:** Apprentice  
**Vulnerability:** Username enumeration via different error messages

**What I did:**
- Intercepted login POST request in Burp Suite
- Sent to Intruder for Sniper attack on username field
- Identified valid username from different response length
- Ran second Sniper attack on password field
- Successfully logged in with found credentials

**Step 1 — Find Valid Username (Sniper Attack):**
```
→ Login request → Send to Intruder
→ Attack Type: Sniper
→ Mark username field as payload position
→ Payload: Username wordlist (Burp built-in list)
→ Run attack
→ Filter by response length — different length = valid username

Found Username: applications
```

**Step 2 — Find Password (Sniper Attack):**
```
→ Same request with found username
→ Mark password field as payload position
→ Payload: Password wordlist (Burp built-in list)
→ Run attack
→ Filter by response length or status code 302

Found Password: 131313
```

**Credentials Found:**
```
Username: applications
Password: 131313
```

**What I Learned:**
- Different error messages reveal if username is valid
- "Invalid username" vs "Incorrect password" = username enumeration
- Burp Intruder Sniper automates username and password brute force
- Response length difference identifies valid usernames
- Always use same error message for both invalid user and wrong password

---

### Lab 2 — Username Enumeration via Subtly Different Responses ✅
**Difficulty:** Apprentice  
**Vulnerability:** Minor difference in error message reveals valid username

**What I did:**
- Intercepted login request and sent to Intruder
- Used Grep Extract to capture error message differences
- Found subtle difference in response for valid username
- Brute forced password using found username

**Payloads Used:**
```
Step 1 → Intruder → Sniper attack on username
Step 2 → Options → Grep Extract
         Add error message to extract
Step 3 → Look for subtle difference:
         Invalid user → "Invalid username or password."
         Valid user   → "Invalid username or password"
                        (missing full stop!)
Step 4 → Note valid username
Step 5 → Brute force password with same method
```

**What I Learned:**
- Developers sometimes make tiny mistakes in error messages
- Even one character difference can reveal valid usernames
- Grep Extract in Burp helps find subtle response differences
- Always normalize error messages in production applications

---

### Lab 3 — Password Reset Broken Logic ✅
**Difficulty:** Apprentice  
**Vulnerability:** Password reset token not validated against username

**What I did:**
- Used forgot password feature for own account (wiener)
- Intercepted password reset request in Burp Repeater
- Changed password reset token to invalid value
- Changed username parameter to victim (carlos)
- Reset carlos password successfully and logged in

**Step by Step:**
```
Step 1 → Click "Forgot password"
Step 2 → Enter own username: wiener
Step 3 → Click email link received
Step 4 → Intercept new password request in Burp
Step 5 → Send to Repeater

Step 6 → In Repeater change:
         token=valid-token → token=x
         username=wiener  → username=carlos
         password=anything → password=password

Step 7 → Send request
Step 8 → Login with:
         Username: carlos
         Password: password
```

**What I Learned:**
- Password reset token must be validated against the username
- Server should reject token if username doesn't match
- Broken logic allows resetting any user's password
- Always tie reset tokens to specific user accounts
- This is a Business Logic vulnerability combined with Auth failure

---

### Lab 4 — Username Enumeration via Subtly Different Responses ✅
**Difficulty:** Practitioner  
**Vulnerability:** Username enumeration through response analysis

**What I did:**
- Sent login to Intruder and ran Sniper attack on username
- All responses showed same message: "Invalid username or password"
- Sent request to Repeater
- Tested username NOT in the wordlist
- Found valid username that way

**Process:**
```
Step 1 → Login → Intruder → Sniper attack
         Payload: Username wordlist
         Result: All show "Invalid username or password"
         (No obvious difference)

Step 2 → Send to Repeater
Step 3 → Try username NOT in wordlist:
         username=auction
         Result → Different behavior detected

Step 4 → Brute force password for auction:
         password=12345678
```

**Credentials Found:**
```
Username: auction
Password: 12345678
```

**What I Learned:**
- Sometimes valid username is NOT in common wordlists
- Testing usernames outside wordlist can reveal valid accounts
- Repeater helps manually test individual usernames
- Always test edge cases beyond automated wordlists
- Response analysis requires careful manual observation

---

### Lab 5 — Username Enumeration via Response Timing ✅
**Difficulty:** Practitioner  
**Vulnerability:** Valid usernames cause longer response time during password check

**What I did:**
- Used X-Forwarded-For header to bypass IP-based rate limiting
- Found valid username by measuring response time difference
- Valid username took longer = server was checking password hash
- Brute forced password using Pitchfork attack

**Understanding the Timing Attack:**
```
Valid username + wrong password:
→ Server checks password hash
→ Hash comparison takes time
→ Response: ~1087ms (slower)

Invalid username + wrong password:
→ Server rejects immediately
→ No hash comparison needed
→ Response: ~1034ms (faster)

Timing difference reveals valid username
```

**Bypass IP Rate Limiting:**
```
Add header to every request:
X-Forwarded-For: [different IP each request]

This tricks server into thinking
each request comes from different IP
```

**Timing Results:**
```
wiener   → 1087ms (valid username = slower)
adserver → 1034ms (invalid = faster)
```

**Find Password:**
```
→ Pitchfork attack with valid username
→ Payload: Password wordlist
→ Filter by Status Code 302 = login success

Found Password: starwars
Status Code: 302 (redirect = logged in)
```

**What I Learned:**
- Response timing reveals valid usernames even with same error message
- Password hashing takes measurable time for valid users
- X-Forwarded-For bypasses simple IP-based rate limiting
- Pitchfork attack tests one username with many passwords
- Status code 302 means redirect = successful login

---

### Lab 6 — Broken Brute Force Protection: IP Block ✅
**Difficulty:** Practitioner  
**Vulnerability:** IP block resets after successful login

**What I did:**
- Found that 3 wrong attempts locks IP for 1 minute
- Discovered that successful login resets the attempt counter
- Created alternating username list: carlos, wiener, carlos, wiener
- Used Pitchfork attack with max 1 concurrent request
- Found carlos password by resetting counter after each 2 attempts

**Understanding the Logic:**
```
Normal behavior:
Attempt 1 → carlos : wrong   (1 fail)
Attempt 2 → carlos : wrong   (2 fails)
Attempt 3 → carlos : wrong   (3 fails = LOCKED)

Bypass logic:
Attempt 1 → carlos  : wrong password  (1 fail)
Attempt 2 → wiener  : correct password (RESET counter)
Attempt 3 → carlos  : wrong password  (1 fail again)
Attempt 4 → wiener  : correct password (RESET again)
(Never reaches 3 fails for carlos)
```

**Username List Created:**
```
carlos
wiener
carlos
wiener
carlos
wiener
(alternating — same count as password list)
```

**Burp Intruder Setup:**
```
→ Attack Type: Pitchfork
→ Position 1: username field
→ Position 2: password field

Username payload list:
carlos, wiener, carlos, wiener...

Password payload list:
[wrong passwords], peter, [wrong], peter...
(wiener's password "peter" on even positions)

Resource Pool Settings:
→ Create new resource pool
→ Maximum concurrent requests: 1
→ (Must be 1 to maintain order)
```

**Result:**
```
Found carlos password: george
Status code 302 = successful login ✅
```

**What I Learned:**
- IP block that resets on valid login is bypassable
- Alternating valid login resets the brute force counter
- Pitchfork attack pairs usernames with passwords in order
- Maximum concurrent requests must be 1 to maintain sequence
- Rate limiting must be based on username not just IP
