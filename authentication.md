# PortSwigger — Authentication Labs
### Naman Yuvraj

6/14 In Progress 🔄 | Tools: Burp Suite Pro, Kali Linux

---

## Lab 1 — Username Enumeration via Different Responses ✅
**Vulnerability:** Username enumeration via different error messages

Send the login request to the Intruder and do the Sniper attack by adding username.

```
Username found: applications
Password found: 131313
```

Sniper attack on username → different response length reveals valid username, same method for password.

![Lab 1](screenshots/authentication/lab1.png)

---

## Lab 2 — Username Enumeration via Subtly Different Responses ✅
**Vulnerability:** Minor difference in error message reveals valid username

Send the login request to Intruder and do Sniper attack on username field.

```
Invalid username → "Invalid username or password."
Valid username   → "Invalid username or password"
                   (dot missing at the end)
```

Use Grep Extract in Intruder options to catch this difference.
Same method for password after finding valid username.

![Lab 2](screenshots/authentication/lab2.png)

---

## Lab 3 — Password Reset Broken Logic ✅
**Vulnerability:** Password reset token not validated against username

```
Step 1 → Forget the password of username 'wiener'
Step 2 → Click the email link
Step 3 → Change the password = 'password'
Step 4 → Intercept it in Repeater
Step 5 → Change the password token = 'x'
Step 6 → Change the username to victim = 'carlos'
Step 7 → Login using username = carlos, password = 'password'
```

Token not tied to username — changing both token and username resets any account's password.

![Lab 3](screenshots/authentication/lab3.png)

---

## Lab 4 — Username Enumeration via Subtly Different Responses ✅
**Vulnerability:** Username enumeration via response analysis

```
Step 1 → Send login to Intruder
Step 2 → Paste usernames and do Sniper attack
         All show same: 'Invalid username or password'
Step 3 → Send login to Repeater
Step 4 → Change to username NOT in the wordlist
         Result → Different behavior detected

Username found: auction
Password found: 12345678
```

Valid username was outside the wordlist — manual testing in Repeater revealed it.

![Lab 4](screenshots/authentication/lab4.png)

---

## Lab 5 — Username Enumeration via Response Timing ✅
**Vulnerability:** Valid username causes longer server response time

```
X-Forwarded-For: [IP]   (used to bypass IP rate limiting)

wiener   → response time: 1087ms  (valid username = slower)
adserver → response time: 1034ms  (invalid = faster)

Password: starwars
Status code: 302 (successful login)
```

Valid username takes longer — server checks password hash only for valid users.

![Lab 5](screenshots/authentication/lab5.png)

---

## Lab 6 — Broken Brute Force Protection: IP Block ✅
**Vulnerability:** IP block resets after successful login

```
After 3 wrong attempts → login locked for 1 min

Fix: Make alternating list —
carlos → wrong password   (1 fail)
wiener → correct password (counter resets)
carlos → wrong password   (1 fail again)
wiener → correct password (counter resets)
(never reaches 3 fails)

Send to Intruder → add both username and password
Attack type: Pitchfork
Resource Pool: new pool, maximum concurrent requests = 1

Password found: george
```

IP block resets on valid login — alternating list bypasses the protection completely.

![Lab 6](screenshots/authentication/lab6.png)
