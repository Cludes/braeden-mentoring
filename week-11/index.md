---
layout: page
title: "How the Internet Works"
week_number: "11"
release_date: "2026-06-30"
phase_label: "Phase 3"
phase_name: "Cybersecurity Foundations"
phase_color: "#f72585"
permalink: /week-11/
---

> Week 11. This week: understanding what actually happens when you visit a website.
> 
> This is the foundation that all cybersecurity sits on. If you do not understand DNS, HTTP, and TCP, you will not understand why XSS, SQLi, and MITM attacks work.
> 
> This week is theory-heavy. Read carefully and look things up when something is unclear. Use the browser DevTools (F12 > Network tab) while browsing normally to watch real HTTP traffic.

---

# How the Internet Works

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Explain what happens step by step when you type a URL into a browser
- Describe what DNS, HTTP, TCP/IP, and TLS do
- Read an HTTP request and response
- Explain common HTTP status codes
- Understand why HTTPS is required and what it protects

---

## Why This Matters for Security

Every web attack -- XSS, SQL injection, MITM, phishing -- exploits something about how the internet works. Understanding the basics is not optional for cybersecurity.

Before you learn how to attack or defend systems, you need to understand what traffic looks like, how names resolve to addresses, and what happens over the wire.

---

## Step by Step: What Happens When You Visit a Website

Type `https://example.com` into a browser. Here is everything that happens:

**Step 1: DNS lookup** - "What is the IP address of example.com?"

**Step 2: TCP connection** - Your computer opens a connection to that IP address on port 443

**Step 3: TLS handshake** - Browser and server negotiate encryption

**Step 4: HTTP request** - Browser sends a request for the page

**Step 5: HTTP response** - Server sends back the HTML

**Step 6: Rendering** - Browser parses HTML, fetches CSS and JS, renders the page

Each step is a potential attack surface.

---

## IP Addresses

Every device on the internet has an IP address. It is like a postal address for data.

```
IPv4: 192.168.1.1       (4 numbers, 0-255, separated by dots)
IPv6: 2001:db8::1       (newer format, more addresses available)
```

**Private IP addresses** (not reachable from the internet):
```
192.168.x.x     -- your home/office network
10.x.x.x
172.16.x.x - 172.31.x.x
```

**Loopback:**
```
127.0.0.1       -- your own machine ("localhost")
```

---

## DNS: The Internet's Phone Book

DNS (Domain Name System) translates human-readable names to IP addresses.

```
example.com  -->  DNS lookup  -->  93.184.216.34
```

The lookup chain:
1. Your browser checks its cache - "do I already know this?"
2. Your OS checks its cache
3. Your OS asks your DNS resolver (usually your router or ISP)
4. Resolver asks the root DNS servers
5. Root refers to the `.com` TLD nameservers
6. TLD refers to `example.com`'s authoritative nameserver
7. Authoritative server returns the IP address
8. Your browser connects to that IP

Check DNS records yourself:

```bash
nslookup example.com         # basic lookup
dig example.com              # detailed lookup
dig example.com MX           # mail records
dig example.com TXT          # text records (SPF, DKIM, etc)
```

**DNS attack surface:**
- DNS spoofing / cache poisoning - attacker returns a fake IP
- DNS hijacking - attacker controls your DNS resolver
- DNS tunneling - data hidden inside DNS queries to bypass firewalls

---

## Ports

After resolving the IP, the browser connects to a specific **port** on that server. A port is a number (0-65535) that identifies which service to talk to.

| Port | Protocol | Used for |
|------|----------|---------|
| 80 | HTTP | Unencrypted web |
| 443 | HTTPS | Encrypted web |
| 22 | SSH | Secure shell |
| 21 | FTP | File transfer |
| 25 | SMTP | Email sending |
| 53 | DNS | DNS queries |
| 3306 | MySQL | Database |
| 5432 | PostgreSQL | Database |

When you type `https://example.com`, the browser connects to port 443 because that is the default for HTTPS. You can specify a different port: `http://localhost:3000`.

---

## TCP: Reliable Delivery

TCP (Transmission Control Protocol) ensures data arrives in order and without errors.

The **three-way handshake** establishes a connection:

```
Client                     Server
  |                           |
  |----  SYN  --------------> |   "I want to connect"
  |                           |
  | <----- SYN-ACK  --------- |   "OK, I acknowledge"
  |                           |
  |----  ACK  --------------> |   "Connection established"
  |                           |
  |====  DATA EXCHANGE  ====> |
```

After the handshake, data flows. TCP guarantees:
- All packets arrive
- They arrive in the correct order
- Errors are detected and retransmitted

**Security relevance:** A SYN flood attack sends millions of SYN packets without completing the handshake, exhausting the server's connection table (a type of DoS attack).

---

## HTTP: The Language of the Web

HTTP (HyperText Transfer Protocol) is how browsers and servers communicate.

**An HTTP request:**

```http
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0)
Accept: text/html
Cookie: session=abc123
```

Breaking it down:
- `GET` = method (what to do)
- `/index.html` = path (what resource)
- `HTTP/1.1` = protocol version
- `Host`, `User-Agent`, `Accept`, `Cookie` = headers (extra info)

**An HTTP response:**

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 1256
Set-Cookie: session=abc123; HttpOnly; Secure

<!DOCTYPE html>
<html>...
```

Breaking it down:
- `200 OK` = status code
- `Content-Type` = what type of data
- `Set-Cookie` = tell browser to store a cookie
- Then a blank line, then the body

---

## HTTP Methods

| Method | Purpose | Has body? |
|--------|---------|-----------|
| GET | Retrieve data | No |
| POST | Submit data, create resource | Yes |
| PUT | Replace a resource | Yes |
| PATCH | Update part of a resource | Yes |
| DELETE | Delete a resource | No |

GET requests should never change data. They should be safe to repeat and bookmark.

---

## HTTP Status Codes

| Code | Meaning | When you see it |
|------|---------|----------------|
| 200 | OK | Everything worked |
| 201 | Created | Resource was created (POST success) |
| 301 | Moved Permanently | Redirect to new URL |
| 302 | Found (temporary redirect) | Login redirects |
| 400 | Bad Request | Malformed request |
| 401 | Unauthorised | Not logged in |
| 403 | Forbidden | Logged in but not allowed |
| 404 | Not Found | Resource does not exist |
| 500 | Internal Server Error | Bug in server code |
| 503 | Service Unavailable | Server overloaded or down |

**Security relevance:** 401 vs 403 leaks information. 404 vs 403 on secret files -- a 403 confirms the file exists but you are not allowed to access it. A 404 pretends it does not exist. Good security practice is to return 404 for both.

---

## HTTPS and TLS

HTTP sends data as plain text. Anyone on the network can read it.

HTTPS = HTTP + TLS encryption. TLS (Transport Layer Security) encrypts the connection so only the browser and server can read the data.

**The TLS handshake:**
1. Browser connects to server on port 443
2. Server sends its **certificate** (proves identity, contains its public key)
3. Browser verifies the certificate is signed by a trusted Certificate Authority (CA)
4. Browser and server use the certificate to negotiate a shared secret key
5. All further communication is encrypted with that key

What TLS protects against:
- Eavesdropping (reading traffic on public Wi-Fi)
- Tampering (injecting malicious content mid-transfer)

What TLS does NOT protect against:
- A compromised server (if the server is hacked, TLS doesn't help)
- A compromised certificate authority
- The server itself doing something malicious

Look for the padlock icon in your browser. Click it to see the certificate details.

**Security relevance:** HTTP-only sites are a security risk. Avoid submitting forms on HTTP sites. Man-in-the-middle attacks are trivial on unencrypted HTTP.

---

## Cookies and Sessions

HTTP is **stateless** -- each request is independent. The server doesn't remember who you are between requests.

Cookies solve this. When you log in:
1. Server creates a session and stores your data
2. Server sends `Set-Cookie: session_id=abc123` in the response
3. Browser stores the cookie
4. Browser sends `Cookie: session_id=abc123` on every subsequent request
5. Server looks up `abc123` in its session store to know who you are

Cookie security attributes:

| Attribute | What it does |
|-----------|-------------|
| `HttpOnly` | JavaScript cannot read this cookie (prevents XSS theft) |
| `Secure` | Only sent over HTTPS |
| `SameSite=Strict` | Not sent on cross-site requests (prevents CSRF) |
| `Path=/` | Which paths the cookie applies to |
| `Expires=` | When the cookie expires (session cookie if not set) |

Always set `HttpOnly` and `Secure` on session cookies.

---

## HTTP Headers and Security

Several HTTP headers are specifically for security:

```
Content-Security-Policy: script-src 'self'      -- only allow scripts from same origin
X-Frame-Options: DENY                           -- prevent embedding in iframes (clickjacking)
X-Content-Type-Options: nosniff                 -- prevent MIME type sniffing
Strict-Transport-Security: max-age=31536000     -- always use HTTPS
Referrer-Policy: no-referrer                    -- don't send referrer headers
```

Missing these headers is a misconfiguration finding in a security audit.

---

## Same-Origin Policy

A fundamental browser security rule: JavaScript on one website cannot read data from another website.

Origin = scheme + hostname + port:
```
https://example.com:443      -- an origin
http://example.com:80        -- different origin (different scheme + port)
https://api.example.com:443  -- different origin (different subdomain)
```

The same-origin policy is why:
- A malicious `evil.com` script cannot read your `bank.com` cookies
- But XSS that injects code INTO `bank.com` CAN access `bank.com` cookies (because the injected code runs on the same origin)

CORS (Cross-Origin Resource Sharing) is the mechanism for explicitly allowing cross-origin requests when needed.

---

## Tools for Inspecting HTTP Traffic

**Browser DevTools (F12):**
- Network tab: see every request, headers, response, timing
- Application tab: see cookies, local storage, session storage

**curl (command line):**

```bash
curl -v https://example.com                     # verbose, shows headers
curl -I https://example.com                     # only headers (HEAD request)
curl -X POST -d "user=alice&pass=1234" http://example.com/login
curl -H "Cookie: session=abc" https://example.com/profile
```

**Burp Suite:** A proxy tool that intercepts all HTTP traffic from your browser. Used by penetration testers to inspect and modify requests. Free community edition is sufficient for learning.

---

## Quick Reference

| Concept | Summary |
|---------|---------|
| DNS | Translates domain names to IP addresses |
| IP address | Unique address of a device on a network |
| Port | Number identifying which service to connect to |
| TCP | Protocol ensuring reliable, ordered data delivery |
| HTTP | Protocol for browser-server communication |
| HTTPS | HTTP + TLS encryption |
| TLS | Encryption layer protecting data in transit |
| Cookie | Small data stored in browser for session state |
| Status code | Three-digit number in HTTP response (200=OK, 404=not found) |
| Same-origin policy | Browser rule preventing cross-site data access |

---

## What to Read Next

- `01_threat-models-cia.md` - Applying this knowledge to threat modelling
- `02_owasp-top10.md` - The most common web vulnerabilities
- `intermediate/01_network-attacks.md` - Attacks that exploit the concepts in this guide

---

## Activity: Inspect a Real Website's HTTP Traffic

1. Open your browser and press F12 to open DevTools. Click the Network tab
1. Visit https://example.com and look at all the requests that were made
1. Click on the first request (the HTML document). Find and write down: the status code, the Content-Type header, whether Set-Cookie was in the response
1. Open a terminal and run: curl -v https://example.com 2>&1 | head -40
1. Identify in the curl output: where the TLS handshake happens, the HTTP status line, at least 3 response headers
1. Run: dig example.com and write down the IP address returned
1. Run: nmap -sV <the IP from dig> and note which ports are open
1. You know you are done when you can describe the full request path: DNS lookup to port scan results

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between HTTP and HTTPS? What does TLS protect and what does it not protect?
2. What is the difference between a 401 and a 403 status code?
3. Why does a session cookie need the HttpOnly attribute?
4. A website returns a 403 for /admin and a 404 for /secret. What information does this give you?
