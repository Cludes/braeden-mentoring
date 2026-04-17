---
layout: page
title: "Forensics and Steganography"
week_number: "37"
release_date: "2026-12-29"
phase_label: "Phase 7"
phase_name: "Intermediate Cybersecurity"
phase_color: "#e63946"
permalink: /week-37/
---

> Week 37. Forensics and steganography -- two CTF staples that are also real-world skills.
> 
> Digital forensics is the recovery and analysis of data from computers and networks. It is used in incident response and legal investigations.
> 
> Steganography is hiding data inside other data -- images, audio, video. CTF challenges frequently hide flags inside image files.
> 
> This is a lighter week -- good for after the holiday period or if you want a change of pace from exploitation.

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

## Activity: Steganography Activity + Packet Analysis

1. Complete the steganography activity at activities/cybersecurity/medium/04_steganography.md
1. Hide a secret message inside a PNG file using the LSB (Least Significant Bit) technique
1. Then complete the packet analysis activity at activities/cybersecurity/medium/06_packet-analysis.md
1. Analyse a provided .pcap file using Wireshark and Python. Find: all unique IP addresses, the most-requested URL, any HTTP POST data
1. For the steganography part: write a Python encoder AND decoder. Verify that decode(encode(image, message)) == message
1. You know you are done when your steganography encoder/decoder works and you have answered all questions from the packet analysis activity

---

## Check-in Questions

Write your answers down before your next session.

1. What is LSB steganography and why is it hard to detect visually?
2. What tools can you use to detect whether a file has hidden data?
3. In Wireshark, how do you filter to show only HTTP traffic?
4. What is a pcap file and how is it created?
