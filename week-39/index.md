---
layout: page
title: "Advanced Recon for Bug Bounty"
week_number: "39"
release_date: "2027-01-12"
phase_label: "Phase 8"
phase_name: "Bug Bounty"
phase_color: "#f77f00"
permalink: /week-39/
---

> Week 39. Advanced reconnaissance for bug bounty.
> 
> Recon is where bounties are won. Most researchers find the same bugs on the obvious targets. Good recon finds assets and endpoints that other researchers have not tested yet.
> 
> This week you build a recon workflow you will use for every future target.

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

# Network Attacks and Defence

**Pillar:** Cybersecurity
**Level:** Intermediate
**Read time:** 20 minutes
**Prerequisite:** `beginner/01_threat-models-cia.md`, `beginner/02_owasp-top10.md`

## Scope

This guide covers the network layer: how traffic moves, how attackers intercept or disrupt it, and how to defend it. Hands-on labs follow at the end.

---

## How Traffic Actually Moves

Before you can attack or defend a network, you need the model in your head:

```
Application    HTTP, DNS, SMTP
Transport      TCP, UDP      (ports, connections)
Network        IP            (routing, addresses)
Data Link      Ethernet      (MAC addresses, local segment)
Physical       Cables, Wi-Fi
```

Attackers typically operate at layer 2 (local network sniffing), layer 3 (IP spoofing, routing attacks), or layer 7 (application protocol attacks).

---

## Key Attack Techniques

### 1. Port Scanning

Reconnaissance step. Find what services are listening before targeting them.

```bash
nmap -sV -T4 192.168.1.1          # Service version scan
nmap -p 1-1000 192.168.1.0/24     # Scan common ports across a subnet
nmap --script vuln 192.168.1.1    # Run NSE vulnerability scripts
```

**What to defend:** Firewall rules that allow only ports you need. Log port scan patterns (many connection attempts in short time from one IP).

### 2. Man-in-the-Middle (MITM)

Attacker positions themselves between two parties and intercepts or modifies traffic.

**ARP spoofing (LAN):**
The attacker sends fake ARP replies telling the network "the gateway's MAC address is mine". Traffic intended for the router flows through the attacker's machine instead.

**Defence:**
- HTTPS everywhere (traffic is encrypted even if intercepted)
- HSTS (forces HTTPS, prevents SSL stripping)
- Dynamic ARP Inspection on managed switches
- VPN for sensitive communications on untrusted networks

### 3. Denial of Service

Overwhelming a service until it becomes unavailable.

| Type | How it works | Scale |
|------|-------------|-------|
| Volumetric | Flood bandwidth with UDP/ICMP | Gbps |
| Protocol | Exploit TCP (SYN flood) | Moderate |
| Application | HTTP request flood | Small, hard to filter |
| Amplification | Use DNS/NTP as reflectors (small request, huge response) | Very large |

**Defence:** Rate limiting at edge, anycast routing, CDN with DDoS scrubbing (Cloudflare, AWS Shield), SYN cookies on TCP stack.

### 4. DNS Attacks

| Attack | Effect |
|--------|--------|
| DNS poisoning | Replace legitimate DNS answer with attacker's IP |
| DNS hijacking | Modify DNS settings on router or host |
| Subdomain takeover | Claim an abandoned DNS record pointing to a cloud resource |
| DNS tunnelling | Exfiltrate data encoded in DNS queries |

**Defence:** DNSSEC, encrypted DNS (DoH/DoT), monitor for unexpected subdomain records.

### 5. Packet Analysis (Offensive)

Capturing traffic on a network segment to extract credentials or session tokens.

```bash
# Capture all traffic on interface eth0
tcpdump -i eth0 -w capture.pcap

# Filter for HTTP traffic only
tcpdump -i eth0 port 80 -w http.pcap

# Open in Wireshark for analysis
wireshark capture.pcap
```

On a switched network, you need to be on the same segment or perform ARP spoofing first to see others' traffic.

**Defence:** Encryption (HTTPS, SSH, TLS). On unencrypted protocols, move to encrypted equivalents.

---

## Network Hardening Checklist

```
Perimeter
  [ ] Firewall rules deny by default, allow only required ports
  [ ] SSH restricted to known IP ranges, not 0.0.0.0
  [ ] No telnet, FTP, or other plaintext admin protocols
  [ ] All external services behind a load balancer or reverse proxy

Internal
  [ ] Network segmented by function (web, app, database tiers)
  [ ] No direct internet access from database servers
  [ ] Internal traffic encrypted where it crosses segment boundaries

Monitoring
  [ ] Port scan detection (unusual connection attempt volume)
  [ ] Outbound traffic anomaly detection (data exfil)
  [ ] DNS query logging enabled
  [ ] Firewall logs shipped to SIEM
```

## Checklist

- [ ] Can you explain what ARP spoofing achieves and why HTTPS still protects against it
- [ ] Run a port scan against your own machine (`nmap localhost`) and interpret the output
- [ ] Capture 30 seconds of your own traffic with `tcpdump` and open it in Wireshark
- [ ] Identify one open port on your machine that could be closed

## Next Steps

- Lab: `activities/cybersecurity/medium/02_port-scanner.md`
- Lab: `activities/cybersecurity/medium/06_packet-analysis.md`
- Guide: `advanced/01_pentesting-methodology.md`
- Quiz: `learning/quizzes/03_cybersecurity.md`

---

## Activity: Full Recon on Your Target Program

1. Use the program you selected last week
1. Run subdomain enumeration: amass enum -passive -d target.com and curl 'https://crt.sh/?q=%.target.com&output=json'
1. Probe all found subdomains with httpx to find live servers
1. For each live subdomain, check the HTTP headers for technology hints
1. Search for JavaScript files and extract URLs and endpoints from them: curl https://target.com/app.js | grep -oE '"(/[^"]+)"'
1. Check for exposed .git, .env, sitemap.xml, robots.txt on all subdomains
1. Document everything in a structured recon file with sections: subdomains, live hosts, interesting endpoints, potential attack points
1. You know you are done when your recon file has at least 10 findings across the categories

---

## Check-in Questions

Write your answers down before your next session.

1. What is amass and what is the difference between passive and active enumeration?
2. Why are JavaScript files particularly valuable during recon?
3. You find an exposed .git folder on a subdomain. What can you extract from it and how?
4. What is a wayback machine query and how does it help find old endpoints?
