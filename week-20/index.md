---
layout: page
title: "HackTheBox Tier 1 + What's Next"
week_number: "20"
release_date: "2026-09-01"
phase_label: "Phase 4"
phase_name: "CTF & HackTheBox"
phase_color: "#e63946"
permalink: /week-20/
---

> Week 20. You made it through the first 20 weeks.
> 
> Here is what you have covered:
> - HTML, CSS, JavaScript (built a real portfolio page)
> - Python (Caesar cipher, password analyzer, hash cracker, port scanner)
> - Linux command line
> - Git and GitHub
> - How the internet works (DNS, HTTP, TCP, TLS)
> - CIA triad, threat modelling, OWASP Top 10
> - Password security and hash cracking
> - Network attacks and port scanning
> - XSS and SQL injection (built working demos and fixes)
> - CTF challenges on PicoCTF
> - HackTheBox Tier 0
> - OSINT and passive recon
> 
> This week: move into HackTheBox Tier 1 machines, which are less guided. You have the skills -- trust what you have learned.
> 
> After this week: talk to your mentor about what to focus on next.

---

# Penetration Testing Methodology

**Pillar:** Cybersecurity
**Level:** Advanced
**Read time:** 25 minutes
**Prerequisite:** `intermediate/01_network-attacks.md`, `intermediate/02_web-exploitation.md`

## Before Anything Else

You must have written authorisation before testing any system. In Australia this is governed by the Criminal Code Act 1995 (Cth) s477.1 (unauthorised access) and s477.2 (unauthorised modification). "I was just curious" is not a defence. See `ETHICS.md`.

---

## The Penetration Testing Execution Standard (PTES)

PTES defines 7 phases. Real engagements follow these in order.

```
1. Pre-engagement
2. Intelligence gathering (recon)
3. Threat modelling
4. Vulnerability analysis
5. Exploitation
6. Post-exploitation
7. Reporting
```

---

## Phase 1: Pre-engagement

Define the rules of engagement before you run a single command:

- **Scope**: exactly which hosts, domains, or IP ranges are in scope
- **Out of scope**: what you must not touch (prod databases, third-party services, etc.)
- **Testing window**: dates and times when testing is permitted
- **Emergency contacts**: who to call if you accidentally take something down
- **Rules**: no DoS unless explicitly approved, no persistence without permission, no lateral movement beyond agreed scope

---

## Phase 2: Intelligence Gathering (Recon)

### Passive recon (no direct contact with target)

```bash
# DNS records
dig any target.com
nslookup -type=any target.com

# Subdomain enumeration (certificate transparency logs)
curl "https://crt.sh/?q=%.target.com&output=json" | jq '.[].name_value' | sort -u

# WHOIS
whois target.com

# Google dorks
site:target.com filetype:pdf
site:target.com inurl:admin
"target.com" ext:env OR ext:config

# Shodan (internet-facing assets)
org:"Target Corp"
hostname:target.com port:22
```

### Active recon (direct contact with target)

```bash
# Host discovery
nmap -sn 192.168.1.0/24

# Full port scan, service + version detection
nmap -p- -sV -sC -T4 --open 192.168.1.50

# Web technology fingerprinting
whatweb http://target.com
wappalyzer (browser extension)

# Directory enumeration
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt
feroxbuster -u http://target.com -w common.txt --depth 3
```

---

## Phase 4: Vulnerability Analysis

Map discovered services to known CVEs and configuration weaknesses.

```bash
# Automated web scan
nikto -h http://target.com

# NSE vulnerability scripts
nmap --script vuln -p 80,443 target.com

# CMS-specific scanners
wpscan --url http://target.com --enumerate p,u  # WordPress
```

Combine automated output with manual review. Automated scanners miss business logic flaws, IDOR, and auth bypass.

---

## Phase 5: Exploitation

Work from lowest risk to highest. Demonstrate impact without exceeding scope.

### Web application exploitation

Standard order of operations for a web target:
1. Map all endpoints (Burp Spider or manual crawl)
2. Test each parameter for injection (SQLi, XSS, SSTI, SSRF)
3. Test auth flows: login, registration, password reset, MFA
4. Test access controls: IDOR, privilege escalation, missing auth
5. Test file operations: upload, download, path traversal

### Privilege escalation (Linux)

After gaining a shell:
```bash
# Automated enumeration
curl -sL https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# Manual checks
sudo -l                           # What can this user run as root?
find / -perm -4000 2>/dev/null    # SUID binaries
cat /etc/crontab                  # Cron jobs running as root
env                               # Environment variables (tokens, keys)
```

### Common quick wins

| Finding | Impact | Example |
|---------|--------|---------|
| Default credentials | High | `admin:admin` on web panel |
| Password in URL/GET param | High | `/login?pass=secret` in logs |
| Old, unpatched CMS | High | WordPress 4.x with known RCE CVE |
| IDOR in REST API | Medium-High | `/api/users/123` returns any user |
| Missing auth on admin route | High | `/admin` returns 200 without session |
| Self-signed or expired TLS | Low | Browser warning + no HSTS |

---

## Phase 7: Reporting

The report is the deliverable. A finding with no written evidence is a finding you cannot act on.

### Structure of a finding

```
Title:          Reflected XSS in /search endpoint
Severity:       Medium (CVSS 6.1)
Description:    The q parameter is reflected without sanitisation...
Proof:          [Screenshot] [Request/response pair]
Impact:         Attacker can hijack any user's session via a crafted link
Remediation:    Encode output with html.escape() before rendering; add CSP header
References:     OWASP A03, CWE-79
```

### CVSS severity levels

| Score | Severity |
|-------|---------|
| 0.0 | None |
| 0.1-3.9 | Low |
| 4.0-6.9 | Medium |
| 7.0-8.9 | High |
| 9.0-10.0 | Critical |

---

## Tools Reference

| Category | Tool | Use |
|----------|------|-----|
| All-in-one | Metasploit | Exploit framework, post-exploitation |
| Web proxy | Burp Suite | Intercept, modify, replay HTTP |
| Recon | nmap | Port scanning, service detection |
| Web dirs | gobuster, feroxbuster | Directory and file enumeration |
| Passwords | hashcat, john | Offline password cracking |
| Wireless | aircrack-ng | WPA2 handshake cracking |
| Reporting | Dradis, Serpico | Finding management and report generation |

## Checklist

- [ ] Complete `activities/cybersecurity/hard/01_ctf-mini.md` using the full PTES workflow
- [ ] Write a proper finding report for one vulnerability you found (use the template above)
- [ ] Enumerate a local VM with nmap and then with linpeas
- [ ] Calculate a CVSS score for a finding using the CVSS 3.1 calculator

## Next Steps

- Lab: `activities/cybersecurity/hard/01_ctf-mini.md`
- Lab: `activities/cybersecurity/hard/02_jwt-attacks.md`
- Week: `weeks/week15/week15.md` (Pentesting Methodology and Recon)

---

## Activity: HackTheBox Tier 1: Appointment or Sequel

1. Start HTB Tier 1. Recommended starting machines: 'Appointment' (SQL injection) or 'Sequel' (MySQL)
1. Tier 1 machines give you questions but the approach is less guided than Tier 0
1. For 'Appointment': the vulnerability is SQL injection in a login form -- you know how this works from Week 16
1. For 'Sequel': involves connecting directly to a database with a misconfigured credential
1. Write a full write-up for whichever machine you choose: enumeration steps, vulnerability found, exploitation, flag
1. Optionally: keep a running notes file called htb-notes.md in your ctf-practice folder and commit it to GitHub
1. You know you are done when you have the root flag and a written explanation of every step

---

## Check-in Questions

Write your answers down before your next session.

1. Describe your full methodology for approaching a new HTB machine from nothing to root flag.
2. What is privilege escalation and what are two common ways it happens on Linux?
3. What is the difference between a vulnerability and an exploit?
4. Where do you want to go from here? Web hacking, binary exploitation, malware analysis, bug bounty, red team, blue team, or something else?
