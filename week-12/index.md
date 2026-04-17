---
layout: page
title: "Cybersecurity Foundations: CIA Triad and Threat Models"
week_number: "12"
release_date: "2026-07-07"
phase_label: "Phase 3"
phase_name: "Cybersecurity Foundations"
phase_color: "#f72585"
permalink: /week-12/
---

> Week 12. Welcome to the cybersecurity section of your curriculum.
> 
> The first question in security is not 'how do I hack this?' It is 'what am I protecting, from whom, and what happens if it fails?'
> 
> This week covers the CIA triad (Confidentiality, Integrity, Availability) and how to think systematically about threats. This framework is used in every real security assessment.
> 
> By the end of this week you will be able to threat-model a system, which is a skill you can use immediately.

---

# Threat Modelling and the CIA Triad

**Pillar:** Cybersecurity
**Level:** Beginner
**Read time:** 15 minutes

## What and Why

Before you can defend anything, you need a mental model of what you're defending against and why it matters. Threat modelling is that mental model. The CIA triad is the vocabulary.

## The CIA Triad

Every security control you will ever encounter protects one or more of these three properties:

| Property | Means | Broken by | Example control |
|----------|-------|-----------|-----------------|
| **Confidentiality** | Only authorised parties can read the data | Eavesdropping, data leaks | Encryption, access control |
| **Integrity** | Data has not been altered without authorisation | Tampering, MITM attacks | Hashing, digital signatures |
| **Availability** | Systems and data are accessible when needed | DDoS, ransomware, hardware failure | Redundancy, backups, rate limiting |

A real attack often hits more than one. Ransomware hits **all three**: it encrypts your data (confidentiality broken for you), corrupts or deletes backups (integrity), and locks you out entirely (availability).

## Threat Modelling in 4 Steps

Threat modelling is just structured thinking about what could go wrong. The four steps below work for any system:

### Step 1: What are you protecting?

List your assets. Examples:
- User credentials stored in a database
- Payment card data
- Internal source code
- API keys in environment variables

### Step 2: Who might attack it, and why?

| Threat actor | Motivation | Typical capability |
|---|---|---|
| Script kiddie | Curiosity, boredom | Low (runs existing tools) |
| Cybercriminal | Money | Medium (automation, buying exploits) |
| Insider threat | Revenge, financial | Medium-high (knows the system) |
| Nation state | Espionage, disruption | Very high |

For most projects, your realistic threat is **opportunistic attackers** running automated scanners, not targeted nation-state actors.

### Step 3: What are the attack paths?

Work through common vectors:
- Network: open ports, unencrypted traffic, exposed admin panels
- Application: injection, broken auth, insecure direct object references
- Human: phishing, social engineering, physical access
- Supply chain: compromised dependencies, third-party integrations

### Step 4: What controls reduce the risk?

Map each risk to a control. Controls fall into three categories:
- **Preventive**: stops the attack (firewall, input validation, MFA)
- **Detective**: identifies when an attack happens (logging, SIEM, alerting)
- **Corrective**: recovers after an attack (backups, incident response plan)

## A Simple Threat Model Example

**System:** A Python Flask web app that stores user notes.

| Asset | Threat | Vector | Control |
|---|---|---|---|
| User passwords | Credential theft | SQL injection | Parameterised queries + bcrypt hashing |
| Session tokens | Session hijacking | XSS | HTTPOnly cookies, CSP header |
| Server access | Unauthorised shell | Exposed SSH port | Firewall: restrict SSH to your IP |
| App availability | DoS | Unbounded requests | Rate limiting, auto-scaling |

## Key Vocabulary

| Term | Definition |
|------|------------|
| **Attack surface** | Every point where an attacker can try to enter or extract data |
| **Vulnerability** | A weakness that can be exploited |
| **Exploit** | Code or technique that takes advantage of a vulnerability |
| **Risk** | Likelihood x Impact of a threat |
| **Zero trust** | Verify every request regardless of network location |
| **Defence in depth** | Multiple layers of controls; one failure does not compromise everything |

## Checklist

- [ ] Can you explain CIA to a non-technical person with one real example each
- [ ] Have you listed at least 3 assets in a system you work on
- [ ] Have you identified at least 2 realistic threat actors for that system
- [ ] Have you mapped one attack path from threat actor to asset

## Next Steps

- [02_owasp-top10.md](02_owasp-top10.md) - The 10 most critical web vulnerabilities
- Lab: `activities/cybersecurity/easy/01_phishing-sim.md`
- Quiz: `learning/quizzes/03_cybersecurity.md`

---

## Activity: Threat Model a Web Application

1. Pick a simple web app: either the portfolio you built in Week 6 or a login page you design on paper
1. Using the 4-step threat modelling process from the guide, answer each step in writing:
1. Step 1 - What are you building? Describe the system, its components, and data flows
1. Step 2 - What can go wrong? List at least 6 threats using the STRIDE categories (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege)
1. Step 3 - What are you going to do about it? For each threat, write one mitigation
1. Step 4 - Did you do a good enough job? What would you check first if this went to production?
1. You know you are done when you have at least 6 threats with mitigations and can explain why each threat matters

---

## Check-in Questions

Write your answers down before your next session.

1. Define Confidentiality, Integrity, and Availability. Give one real attack that violates each.
2. A company's website is defaced (replaced with a message by an attacker). Which CIA property was violated?
3. What is the difference between authentication and authorisation?
4. What does STRIDE stand for and what does each letter represent?
