---
layout: page
title: "Australian Cybersecurity Law and Ethics: Required Before Week 12"
week_number: "11b"
release_date: "2026-07-03"
phase_label: "Phase 3"
phase_name: "Cybersecurity Foundations"
phase_color: "#f72585"
permalink: /week-11b/
---

> This is a required session before you start any offensive security content.
> 
> From Week 12 onward you will learn techniques that are used both in ethical security research and in cybercrime. The exact same tool -- a port scanner, a web proxy, a password cracker -- is legal in one context and a criminal offence in another. The difference is authorisation.
> 
> Before we continue, you need to read and understand Australian law on this topic. Then you will sign an attestation.
> 
> This is not a formality. A conviction for unauthorised computer access in Australia carries up to 2-10 years imprisonment depending on the offence. It can end a career in IT and security before it starts. These consequences apply to everyone, including NDIS participants.
> 
> Read everything in this session carefully. If any of it is unclear, ask Andrew before signing anything.

---

# Legal and Ethical Use of Cybersecurity Skills in Australia

This document must be read and understood before starting any offensive security content in this curriculum (Phase 3, Week 12 onwards).

A separate signed attestation (`mentoring/braeden/cybersecurity-attestation.md`) must be completed before proceeding.

---

## The Core Rule

The only rule that matters in practice:

**You may only test systems you own, or systems where you have explicit, written, prior authorisation from the person or organisation responsible for that system.**

Everything else in this document explains what happens when that rule is broken.

---

## Australian Law: Computer Offences

### Criminal Code Act 1995 (Commonwealth)

The primary federal law is Part 10.7 of the *Criminal Code Act 1995 (Cth)*. These offences apply across all of Australia.

| Section | Offence | Maximum Penalty |
|---------|---------|----------------|
| 477.1 | Unauthorised access, modification or impairment **with intent to commit a serious offence** (e.g. fraud, theft) | 10 years imprisonment |
| 477.2 | Unauthorised modification of data **causing impairment** to a computer or communications | 10 years imprisonment |
| 477.3 | Unauthorised impairment of electronic communication | 10 years imprisonment |
| 478.1 | Unauthorised access to or modification of restricted data | 2 years imprisonment |
| 478.2 | Unauthorised impairment of data held on a computer disk, credit card, or other device | 2 years imprisonment |
| 478.3 | Possession or control of data **with intent to commit a computer offence** | 3 years imprisonment |
| 478.4 | Producing, supplying or obtaining data **with intent** to commit a computer offence (e.g. writing a tool to attack systems) | 3 years imprisonment |

### What "Unauthorised" Means

The word "unauthorised" is defined in s476.2:

> Access to, modification of, or impairment in respect of data held in a computer is unauthorised if the person is not entitled to cause that access, modification or impairment.

This means:
- You do not need to break a password or bypass security to commit the offence. Simply accessing data you were not supposed to access is sufficient.
- The person who runs a website or service has not given you permission to probe it for vulnerabilities, even if the site is publicly accessible.
- "I was just curious" is not a defence.
- "I was going to tell them" is not a defence (though responsible disclosure may reduce penalty in practice).
- "I did not cause any damage" is not a defence for unauthorised access charges.

### State and Territory Laws

In addition to federal law, each state and territory has its own computer crime legislation. You can be charged under both federal and state law for the same conduct.

| Jurisdiction | Legislation |
|-------------|------------|
| New South Wales | *Crimes Act 1900 (NSW)*, Part 6 (ss 308-308I) |
| Victoria | *Crimes Act 1958 (Vic)*, Part 1 Div 3 (ss 247A-247L) |
| Queensland | *Criminal Code Act 1899 (Qld)*, ss 408D-408E |
| Western Australia | *Criminal Code Act Compilation Act 1913 (WA)*, ss 440A-440F |
| South Australia | *Criminal Law Consolidation Act 1935 (SA)*, ss 86A-86G |
| Tasmania | *Criminal Code Act 1924 (Tas)*, ss 257A-257G |
| Australian Capital Territory | *Criminal Code 2002 (ACT)*, ss 415-424 |
| Northern Territory | *Criminal Code Act 1983 (NT)*, ss 222-227 |

---

## Real Consequences: Beyond the Statute

A criminal conviction for computer offences does not just mean a fine or prison time. It has long-term effects on every part of life.

### Criminal Record

A conviction creates a permanent criminal record. In Australia, serious offences (those carrying more than 30 months imprisonment) may never become "spent" under the *Spent Convictions Act 2000 (Cth)*.

### Employment

- Most IT and cybersecurity employer application forms require disclosure of criminal convictions.
- Government roles, defence contractors, and security-cleared positions require a background check. A computer crime conviction will disqualify you from the vast majority of these roles.
- Professional licensing (e.g. for financial services, healthcare, legal) requires character checks.
- A cybersecurity professional with a conviction for an unauthorised access offence is effectively unemployable in the industry.

### NDIS

Participants in the National Disability Insurance Scheme (NDIS) are not exempt from criminal law. A criminal conviction may:
- Affect NDIS plan reviews and funding decisions
- Affect support worker eligibility for workers who are themselves NDIS participants
- Affect access to certain community supports and programs

The NDIS is a support program, not a shield from legal liability.

### Travel

Australia has visa agreements with many countries. A criminal record for computer crimes can result in:
- Visa refusal for the United States (ESTA/visa waiver ineligibility)
- Entry refusal or additional screening in the United Kingdom, Canada, New Zealand
- Restrictions on travel to other countries that conduct background checks at the border

### Civil Liability

In addition to criminal charges, victims of computer offences can bring civil claims. You can be sued for:
- Loss of data or business
- Costs of remediation (forensics, system rebuild)
- Reputational damage

---

## Authorised Environments: What is Safe

The following environments are specifically designed and authorised for practice:

| Platform | URL | What it is |
|----------|-----|-----------|
| HackTheBox | app.hackthebox.com | Professional lab machines. Completing the VPN setup and connecting to a machine constitutes authorisation. |
| PicoCTF | picoctf.org | Beginner CTF challenges. All challenges are self-contained. |
| TryHackMe | tryhackme.com | Guided learning paths with VMs. Explicit authorisation for each room. |
| Hack The Box Academy | academy.hackthebox.com | Structured courses with isolated lab environments. |
| OWASP WebGoat | Local only (Docker) | Deliberately vulnerable app. Run locally, never expose to a network. |
| DVWA | Local only (Docker/VM) | Deliberately vulnerable web app. Local/isolated only. |
| VulnHub | vulnhub.com | Downloadable VMs for local practice. Host-only network only. |
| Nmap scanme | scanme.nmap.org | Nmap explicitly grants permission to scan this single host for learning. No other hosts. |
| Portswigger Web Academy | portswigger.net/web-security | Browser-based labs for web exploitation. All labs are isolated. |

**Everything else** - including any live website, service, or device not on this list - requires explicit written authorisation before any security testing.

---

## What You Are Never Permitted to Do

The following apply without exception, regardless of intent:

- Scan, probe, or attempt to access systems you do not own without written authorisation
- Use techniques from this curriculum against any live service, app, or device outside a designated lab
- Intercept, capture, or read network traffic on networks you do not administer
- Access, read, copy, or download data you are not authorised to access
- Test your employer's or school's systems without a written scope document signed by the appropriate authority
- Use someone else's credentials (even if shared or found) to access a system
- Assist, instruct, or provide tools to others for any of the above
- Publish, share, or sell exploits, vulnerabilities, or data obtained from any live system

---

## Lab Safety Rules

When running any offensive lab in this curriculum:

1. **Use an isolated network.** VirtualBox/VMware host-only adapter. Do not connect lab VMs to your home network or the internet.

2. **Take a snapshot before every lab.** Revert to the snapshot if something goes wrong. Never run a vulnerable VM without a recovery snapshot.

3. **Do not run deliberately vulnerable VMs (DVWA, Metasploitable) on shared networks** - university Wi-Fi, public hotspots, work networks. These are full of exploitable services.

4. **Do not port-forward vulnerable VM services** to your host machine's external interface.

**VirtualBox host-only setup:**
File > Host Network Manager > Create > assign an IP range (e.g. 192.168.56.0/24). Set all lab VMs to use only this adapter. Your host machine can reach them; the internet and your LAN cannot.

---

## Responsible Disclosure

If you encounter a real vulnerability in a system you did not intend to test, or during authorised testing:

1. **Stop. Do not continue probing.** The moment you have confirmed a vulnerability exists, stop.
2. **Do not exploit, extract, or exfiltrate data** even to prove the severity.
3. **Document privately** - what you found, URL, steps to reproduce. Tell no one until the organisation is notified.
4. **Find the right contact.** Check `https://target.com/.well-known/security.txt`. Look for a bug bounty program on HackerOne or Bugcrowd. If none exists, use the organisation's general security contact or legal contact.
5. **Send a disclosure report** (template below).
6. **Allow 90 days** for a patch before any public disclosure (the Project Zero standard, widely accepted in Australia).
7. **For Australian government systems:** Report to the ACSC at cyber.gov.au or call 1300 CYBER1 (1300 292 371).

### Disclosure Email Template

```
Subject: Security Vulnerability Report - [Brief one-line description]

To the [Organisation] Security Team,

I am writing to responsibly disclose a security vulnerability I discovered
in [system/URL] on [date].

I am a [student/researcher/user] and discovered this [how: while using 
the service normally / during an authorised assessment / accidentally].

Summary:
- Affected URL / component: [exact URL or component]
- Vulnerability type: [e.g. Cross-Site Scripting, SQL Injection, IDOR]
- Severity estimate: [Critical / High / Medium / Low]

Steps to reproduce:
1. [Step 1]
2. [Step 2]
3. [Observe: ...]

Potential impact:
[What an attacker could do with this - be specific]

I have not exploited this vulnerability beyond confirming its existence.
I have not shared this information with any third party and will not do so
for 90 days from today's date to allow time for remediation.

I am not requesting any compensation. I am happy to provide additional
technical details or test a fix.

Please acknowledge receipt of this report.

Regards,
[Full name]
[Email address]
[Date]
```

---

## The Ethical Mindset

Beyond the law, professional security researchers follow a set of principles:

**Do no harm.** The goal of security research is to make systems safer. If your work makes a system less safe, even temporarily, stop and reassess.

**Minimum footprint.** In any authorised test, take only what you need to prove the vulnerability. Do not read actual user data. Do not persist on the system longer than necessary.

**Confidentiality.** Information you collect during a security assessment is confidential. It is not yours to share, publish, or discuss with people outside the scope of the engagement.

**Transparency.** Work you do under authorisation should be documented. If questioned, you should be able to show exactly what you did and why.

**Humility.** Security research is about finding flaws in systems to help protect people. It is not about proving how clever you are, gaining social status, or intimidating others.

---

## Reporting Cybercrime

If you witness cybercrime or someone asks you to participate in an attack:

- **Australian Cyber Security Centre (ACSC):** cyber.gov.au | 1300 CYBER1 (1300 292 371)
- **Australian Federal Police:** afp.gov.au | 131 237
- **ReportCyber (national online reporting):** cyber.gov.au/report
- **Scamwatch (ACCC) for scams and fraud:** scamwatch.gov.au
- **State police:** your local police station or 000 for urgent matters

You are not obligated to report every vulnerability you discover in an authorised lab. You are obligated to report if you witness an active attack on a real system or if you are asked to participate in one.

---

## References

- *Criminal Code Act 1995 (Cth)*, Part 10.7: https://www.legislation.gov.au/Details/C2021C00194
- ACSC: https://www.cyber.gov.au
- Australian Institute of Criminology - Cybercrime: https://www.aic.gov.au/topics/cybercrime
- OWASP Code of Ethics: https://owasp.org/www-policy/operational/code-of-ethics
- HackerOne Disclosure Guidelines: https://www.hackerone.com/disclosure-guidelines

---

## Activity: Read, Discuss, and Sign the Attestation

1. Read the full document in this session (Australian Cybersecurity Law and Ethics)
1. Write down at least 3 questions or things you want to clarify with Andrew before signing
1. Discuss those questions with Andrew in your session
1. If you are satisfied you understand the content, read the attestation at mentoring/braeden/cybersecurity-attestation.md
1. Sign and date the attestation with Andrew present. Andrew signs as mentor. A witness signature is recommended
1. Keep the signed copy. Andrew keeps a copy
1. You know you are done when: you can explain in your own words what makes an action legal vs illegal, you have signed the attestation, and Andrew has countersigned

---

## Check-in Questions

Write your answers down before your next session.

1. Under the Criminal Code Act 1995, what is the maximum penalty for unauthorised modification of data causing impairment?
2. You are browsing a website and notice the URL contains an ID number. You change it to a different number and can see someone else's order details. Have you committed an offence? Explain.
3. A friend on Discord says they found a vulnerable website and asks you to help test it. What do you do?
4. Name three authorised platforms where you are permitted to practise offensive security techniques.
