---
layout: page
title: "Write and Submit Your First Bug Report"
week_number: "40"
release_date: "2027-01-19"
phase_label: "Phase 8"
phase_name: "Bug Bounty"
phase_color: "#f77f00"
permalink: /week-40/
---

> Week 40. This is the week you either submit your first real report or write a practice report based on a PortSwigger lab.
> 
> If your recon has turned up a potential vulnerability in your target program: verify it, document it fully using the report template, and submit it.
> 
> If your recon has not yet found a reportable bug (this is normal -- it can take weeks): write a practice report based on any PortSwigger Web Academy lab you have completed. The goal is to practise the report writing itself.
> 
> Good report writing is a skill separate from finding bugs. Many valid bugs are rejected due to poor reports.

---

# Bug Bounty Basics: Getting Paid to Find Vulnerabilities

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Explain what a bug bounty program is and how it works
- Find and read a bug bounty scope document
- Set up a basic bug bounty recon workflow
- Write a quality vulnerability report
- Understand what is in scope, out of scope, and the rules of engagement

---

## What is a Bug Bounty Program?

A bug bounty program is an agreement where an organisation pays security researchers to find and report vulnerabilities in their systems before attackers do.

Key facts:
- Many large companies run programs: Google, Facebook, Microsoft, Apple, GitHub, HackerOne, Bugcrowd
- Rewards range from acknowledgement only to tens of thousands of dollars for critical findings
- You must stay within the program's defined scope
- Testing outside scope is still illegal, even if the program exists

---

## Platforms

**HackerOne:** https://hackerone.com
- Largest bug bounty platform
- Many public programs you can join immediately
- Profile tracks your reputation and earnings

**Bugcrowd:** https://bugcrowd.com
- Similar to HackerOne
- Mix of public and private programs

**Intigriti:** https://intigriti.com
- European-focused
- Good for learning, active community

**Synack:** https://synack.com
- Invite only, vetted researchers
- Higher-value programs

Start with HackerOne's public programs. Many have low competition because they are older or have narrow scope.

---

## Reading a Bug Bounty Policy

Before testing anything, read the policy document completely. It defines:

**In-scope assets:** What you are allowed to test. Usually a list of domains, IP ranges, or products.

**Out-of-scope assets:** What you must not test. Often includes internal infrastructure, employee accounts, third-party services.

**Prohibited actions:**
- Social engineering staff
- Physical attacks
- Denial of service
- Accessing real user data beyond what is necessary to prove the bug

**Reward table:** What categories of bugs are rewarded and at what amounts.

**Disclosure policy:** Whether you may share your findings publicly and when.

If the policy does not explicitly permit something, assume it is prohibited.

---

## Setting Up Your Workspace

Before starting any program:

```bash
mkdir ~/bugbounty
mkdir ~/bugbounty/target-name
mkdir ~/bugbounty/target-name/{recon,screenshots,notes,reports}
```

Tools to have ready:

```bash
# Subdomain enumeration
sudo apt install amass
pip install sublist3r

# DNS and HTTP
sudo apt install nmap dnsutils curl

# Burp Suite
# Download from portswigger.net/burp/communitydownload

# ffuf (web fuzzer)
sudo apt install ffuf

# httpx (probe domains for live web servers)
go install github.com/projectdiscovery/httpx/cmd/httpx@latest
```

---

## The Recon Phase

Good recon finds attack surface that other researchers have missed.

**Step 1: Subdomain enumeration**

```bash
# Passive (no direct contact with target)
amass enum -passive -d target.com -o subdomains.txt
curl -s "https://crt.sh/?q=%.target.com&output=json" | jq '.[].name_value' | sort -u

# Active (contacts DNS servers -- only if explicitly in scope)
amass enum -active -d target.com
subfinder -d target.com -o subfinder_results.txt
```

**Step 2: Find live hosts**

```bash
cat subdomains.txt | httpx -silent -o live_hosts.txt
```

**Step 3: Port scan in-scope hosts**

```bash
nmap -iL live_hosts.txt -sV -p 80,443,8080,8443 -oA scan_results
```

**Step 4: Manually browse everything**

Go through every live subdomain with Burp Suite proxied. Look at:
- Login pages
- Registration flows
- Password reset flows
- File upload forms
- APIs (check for `/api/v1/` paths)
- Admin panels
- Mobile app endpoints

**Step 5: Google dorking**

```
site:target.com filetype:pdf
site:target.com ext:log OR ext:sql OR ext:env
site:target.com inurl:admin
site:target.com "API key"
```

---

## What to Look For

Focus on high-impact, high-probability vulnerability classes:

**High impact (large rewards):**
- Authentication bypass
- IDOR (Insecure Direct Object Reference) - access other users' data by changing an ID
- SQL injection in production
- Server-Side Request Forgery (SSRF) - make the server request internal resources
- Account takeover via broken password reset
- Privilege escalation within the app (user becomes admin)

**Medium impact (moderate rewards):**
- Stored XSS - executes in other users' browsers
- CSRF - forces logged-in users to perform actions
- Open redirect (valued if leads to phishing or token theft)
- Sensitive data exposure (API keys, tokens in JS files, `.git` folders exposed)

**Low impact / often not accepted:**
- Reflected XSS without a realistic attack scenario
- Missing security headers alone
- Self-XSS (only affects you)
- Rate limiting issues without meaningful impact
- Clickjacking on non-sensitive pages

---

## Finding Low-Hanging Fruit

**Check `.git` folder exposure:**
```bash
curl https://target.com/.git/HEAD
# If it returns "ref: refs/heads/main" -- the git repo is exposed
# Use git-dumper to download it: pip install git-dumper
# git-dumper https://target.com/.git/ ./downloaded_repo
```

**Check for exposed environment files:**
```bash
curl https://target.com/.env
curl https://target.com/config.php.bak
```

**JS file analysis:**

```bash
# Extract all JavaScript URLs from a page
curl https://target.com | grep -oP '"[^"]*\.js[^"]*"' | tr -d '"'

# Download and search for interesting strings
curl https://target.com/app.js | grep -E "(api_key|apikey|secret|token|password|credential)" -i
```

**IDOR hunting pattern:**

After logging in, find any URL with a numeric or UUID identifier. Copy the request to Burp Repeater. Log into a second test account (create one). Change the ID to the second account's resource. If you can access it from the first account -- that is an IDOR.

---

## Writing a Quality Report

A poor report gets rejected or a lower reward. A good report gets rewarded quickly and establishes your reputation.

**Report structure:**

```markdown
## Title
[Vulnerability Type] in [Component] allows [Impact]

Example: IDOR in /api/v1/invoices allows any authenticated user to access other users' invoices

## Summary
One paragraph. What is the vulnerability, where is it, and what can an attacker do with it?

## Severity
[Critical / High / Medium / Low] -- explain your reasoning

## Steps to Reproduce
1. Log into the application as user A (test@example.com)
2. Navigate to /dashboard/invoices
3. Observe the request: GET /api/v1/invoices/1042
4. Change the invoice ID to 1043 (another user's invoice)
5. The response returns user B's invoice data

## Proof of Concept
[Screenshots or a short video of the reproduction steps]
[Redact any real user data you encountered -- use test accounts only]

## Impact
What can an attacker do with this? How many users are affected? What data is exposed?

## Suggested Remediation
How should this be fixed? (Not always required but appreciated)
```

**Rules for good reports:**
- Use test accounts you control. Never access real user data.
- Screenshots at every step. Annotate them.
- Paste the raw HTTP request from Burp (redact sensitive values).
- Write for someone who has not seen the bug before.
- Be factual. Do not exaggerate impact to inflate reward.

---

## Triage Process

After submitting, a triager at the company will:
1. **New** - report received
2. **Triaged** - confirmed valid and accepted for investigation
3. **Resolved** - fix deployed
4. **Duplicate** - someone else already reported it
5. **N/A (Not Applicable)** - not a valid vulnerability, out of scope, or not impactful enough

Duplicates are common. Do not be discouraged. Move to the next target.

---

## Ethics and Legal Reminders

Bug bounty does not grant unlimited permission:
- You are only authorised to test what is listed in scope
- You must not access, read, or exfiltrate real user data beyond confirming the bug exists
- You must follow the program's rules on public disclosure (most require keeping it private until fixed)
- You must stop testing if you find something highly critical (like a way to access all user data) -- report it immediately, do not explore further
- The program authorisation applies to you personally. You cannot share access with others.

---

## Realistic Expectations

- Your first 10-20 reports will likely be duplicates or N/As
- This is normal and part of learning
- Each N/A or duplicate teaches you what the program is already aware of
- Focus on finding unique attack surface (new subdomains, new features, new API versions)
- Build relationships in the community (HackerOne Hacktivity feed, Twitter/X security community)

---

## What to Read Next

- PortSwigger Web Academy for practising the techniques: https://portswigger.net/web-security
- HackerOne hacktivity (public disclosed reports): https://hackerone.com/hacktivity
- "The Web Application Hacker's Handbook" (book, free PDF widely available)
- `learning/03_cybersecurity/intermediate/03_burp-suite.md`

---

## Activity: Write a Complete Bug Report

1. Choose either: a real potential finding from your recon, or any PortSwigger Web Academy lab you have solved (treat it as if it were a real target)
1. Write a full report using the template from the Bug Bounty Basics guide
1. Required sections: title (vulnerability type + component + impact), summary, severity with justification, step-by-step reproduction, proof of concept (screenshots or curl commands), impact statement, suggested remediation
1. Have Andrew review the report before submitting a real one
1. If submitting a real report: submit it and save the confirmation/submission ID
1. You know you are done when your report is complete, readable by someone who has never seen the bug, and has screenshots at every step

---

## Check-in Questions

Write your answers down before your next session.

1. What makes the difference between a High and a Critical severity rating?
2. Why is the impact section the most important part of a bug report?
3. A programme says 'self-XSS is out of scope'. What is self-XSS and why is it less impactful than regular XSS?
4. You submit a report and it is marked Duplicate. What should you do next?
