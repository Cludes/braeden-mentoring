---
layout: page
title: "CTF Mini: Multi-Stage Challenge"
week_number: "36"
release_date: "2026-12-22"
phase_label: "Phase 7"
phase_name: "Intermediate Cybersecurity"
phase_color: "#e63946"
permalink: /week-36/
---

> Week 36. Phase 7 capstone: a multi-stage CTF challenge.
> 
> This challenge combines web exploitation, Linux command line, privilege escalation, and enumeration -- everything from Phase 7.
> 
> This is also a good week for catching up if you have been moving fast, or going deeper on any Phase 7 topic that interested you.

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

## Activity: CTF Mini Challenge + Free Choice

1. Complete the activity at activities/cybersecurity/hard/01_ctf-mini.md -- this is a multi-stage challenge that takes about 2-3 hours
1. The challenge involves: web exploitation to get initial access, Linux enumeration to find next steps, privilege escalation to get root, multiple flags hidden at each stage
1. If you finish early: pick one of these bonus challenges -- complete a new TryHackMe room, find and complete a PicoCTF challenge you have not done before, or start a new HackTheBox machine
1. Write up the CTF mini solution in full: every step, every command, every flag
1. You know you are done when you have all flags and a complete write-up

---

## Check-in Questions

Write your answers down before your next session.

1. Walk through your approach to the CTF mini: what did you enumerate first, what vulnerability did you find, how did you escalate?
2. What took the most time and why?
3. What would you do differently next time?
4. What is one technique from Phase 7 (weeks 31-36) you want to go deeper on?
