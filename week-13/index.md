---
layout: page
title: "OWASP Top 10: The Most Common Web Vulnerabilities"
week_number: "13"
release_date: "2026-07-14"
phase_label: "Phase 3"
phase_name: "Cybersecurity Foundations"
phase_color: "#f72585"
permalink: /week-13/
---

> Week 13. The OWASP Top 10 is the industry standard list of the most critical web security risks.
> 
> Every web developer and every security person knows this list. If you can explain all 10 and describe how to prevent each one, you already know more about web security than most developers.
> 
> Read each vulnerability carefully. For each one, ask yourself: have I seen this in any website I have used?

---

# OWASP Top 10: The Most Critical Web Vulnerabilities

**Pillar:** Cybersecurity
**Level:** Beginner
**Read time:** 20 minutes

## What and Why

The OWASP Top 10 is a list of the most critical security risks in web applications, published by the Open Web Application Security Project. Every developer working on a web application should know these by name, what they look like in code, and how to prevent them.

---

## A01: Broken Access Control

Users can access data or functions they should not be able to.

**Example:**
```
GET /api/invoice?id=1001   (your invoice)
GET /api/invoice?id=1002   (someone else's invoice -- should be blocked)
```

**Fix:** Check authorisation on every request server-side. Never rely on the frontend to hide things.

---

## A02: Cryptographic Failures

Sensitive data transmitted or stored without proper encryption.

**Examples:**
- Storing passwords with MD5 or SHA-1 (fast hashing = easy to crack)
- HTTP (not HTTPS) for login forms
- Encryption keys hardcoded in source code

**Fix:** Use bcrypt/argon2 for passwords. Enforce HTTPS everywhere. Store secrets in environment variables, not code.

---

## A03: Injection

Untrusted data sent to an interpreter as part of a command or query.

**SQL injection example:**
```python
# Vulnerable
query = "SELECT * FROM users WHERE name = '" + username + "'"

# Safe (parameterised)
cursor.execute("SELECT * FROM users WHERE name = ?", (username,))
```

**Other types:** OS command injection, LDAP injection, template injection (SSTI).

**Fix:** Parameterised queries. Never construct queries with string concatenation using user input.

---

## A04: Insecure Design

Flaws in the architecture or design that no amount of coding fixes can solve.

**Example:** A password reset that emails a link but never invalidates old links, letting an attacker use a stolen link days later.

**Fix:** Threat model during design, not after. Use proven patterns (use proven auth libraries, do not build your own session management from scratch).

---

## A05: Security Misconfiguration

Correct software, wrong settings.

**Examples:**
- Default credentials left in place (`admin` / `admin`)
- Stack traces exposed to users in production
- Unused ports open on the server
- Directory listing enabled on a web server

**Fix:** Harden defaults before deploying. Automate configuration checks in CI.

---

## A06: Vulnerable and Outdated Components

Using libraries with known CVEs.

**Example:** A `requirements.txt` pinned to `requests==2.6.0` from 2015, which has multiple security advisories.

**Fix:** Run `pip-audit` or `npm audit` in CI. Pin dependencies and review updates regularly.

---

## A07: Identification and Authentication Failures

Broken or missing controls around who is who.

**Examples:**
- Allowing unlimited password attempts (brute force)
- Weak session tokens (sequential IDs, short tokens)
- Not invalidating sessions on logout

**Fix:** Rate limit login attempts. Use cryptographically random session tokens. Invalidate server-side on logout.

---

## A08: Software and Data Integrity Failures

Assuming code or data pipelines have not been tampered with.

**Example:** A CI pipeline that pulls a third-party script with `curl | bash` without verifying a checksum. An attacker who compromises that script's host now owns your build.

**Fix:** Verify checksums or signatures on downloaded artifacts. Pin GitHub Actions to commit hashes, not mutable tags.

---

## A09: Security Logging and Monitoring Failures

Not knowing when you're being attacked.

**What should be logged:**
- Login attempts (success and failure)
- Privilege changes
- API errors at unusual rates
- Access to sensitive data

**Fix:** Log in structured format (JSON). Alert on anomalies. Ensure logs cannot be deleted by the attacker (write to a separate, append-only store).

---

## A10: Server-Side Request Forgery (SSRF)

The server fetches a URL supplied by the user, allowing the attacker to reach internal services.

**Example:**
```
POST /fetch-url
{"url": "http://169.254.169.254/latest/meta-data/"}
```

On AWS, that URL returns the instance metadata including IAM credentials.

**Fix:** Validate and allowlist URLs. Block requests to private IP ranges. Use a dedicated network egress with tight firewall rules.

---

## Quick Reference Card

| # | Name | One-line fix |
|---|------|-------------|
| A01 | Broken Access Control | Server-side authz on every request |
| A02 | Cryptographic Failures | bcrypt, HTTPS, no hardcoded secrets |
| A03 | Injection | Parameterised queries, never string concat |
| A04 | Insecure Design | Threat model at design time |
| A05 | Security Misconfiguration | Harden defaults, automate checks |
| A06 | Vulnerable Components | Dependency scanning in CI |
| A07 | Auth Failures | Rate limits, strong tokens, invalidate on logout |
| A08 | Integrity Failures | Verify checksums, pin deps |
| A09 | Logging Failures | Log auth + errors, alert on anomalies |
| A10 | SSRF | Allowlist URLs, block internal ranges |

## Checklist

- [ ] Can you name all 10 from memory
- [ ] Can you give a code example of SQL injection and its fix
- [ ] Have you run a dependency audit on a project you work on
- [ ] Do you know what your app logs on failed login

## Next Steps

- Lab: `activities/cybersecurity/medium/01_xss-demo.md`
- Lab: `activities/cybersecurity/medium/03_sqli-playground.md`
- Guide: `intermediate/01_network-attacks.md`
- Quiz: `learning/quizzes/03_cybersecurity.md`

---

## Activity: Spot the Vulnerabilities

1. Read through each of the 10 vulnerabilities in the guide
1. For EACH of the following 3 code snippets below, identify which OWASP vulnerability it contains and explain why:
1. Snippet A (Python/Flask): query = 'SELECT * FROM users WHERE username = \'' + username + '\''
1. Snippet B (JavaScript): document.getElementById('output').innerHTML = userInput
1. Snippet C (Python): password_hash = md5(password)
1. For each snippet: name the vulnerability, explain what an attacker could do with it, and write the fixed version
1. You know you are done when you have identified all 3 vulnerabilities correctly with valid fixes

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between SQL injection and XSS? Which one attacks the server and which attacks other users?
2. Why is storing passwords as MD5 hashes a security vulnerability?
3. What is IDOR (Insecure Direct Object Reference)? Give a concrete example.
4. What is the most effective single defence against SQL injection?
