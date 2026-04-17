---
layout: page
title: "Australian Cybersecurity Career Paths"
week_number: "49"
release_date: "2027-03-23"
phase_label: "Phase 10"
phase_name: "Projects, Career & Year Two"
phase_color: "#2ec4b6"
permalink: /week-49/
---

> Week 49. Careers in cybersecurity in Australia.
> 
> You have spent almost a year learning. This week is about where these skills lead and what the Australian industry looks like.
> 
> This is a planning and research week, not a technical lab week. The activity involves research and a conversation with Andrew.

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

## Activity: Career Research: Australian Cybersecurity

1. Research the following job titles on Seek.com.au and LinkedIn Jobs: Security Analyst, Penetration Tester, SOC Analyst, Threat Intelligence Analyst, Bug Bounty Hunter (freelance), DevSecOps Engineer
1. For each title: find 2 job ads and note the required skills, qualifications, and salary range if listed
1. Research the following Australian organisations and what they do: Australian Cyber Security Centre (ACSC), ASD (Australian Signals Directorate), CyberCX, Tesserent, Sense of Security, KPMG Cyber
1. Find one Australian cybersecurity community or event: BSides Canberra, AusCERT, AISA (Australian Information Security Association). Find when the next event is
1. Write a one-page plan: which career path interests you most, what skills you still need, and what your next 6 months of learning should focus on
1. Discuss this plan with Andrew

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between a red team and a blue team role?
2. What does a SOC (Security Operations Centre) analyst do day to day?
3. What are the main cybersecurity certifications recognised in Australia? Which one is most relevant to your chosen path?
4. What is the ACSC and what role does it play in Australian cybersecurity?
