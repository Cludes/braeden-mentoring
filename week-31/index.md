---
layout: page
title: "Burp Suite: Intercept, Replay, and Fuzz"
week_number: "31"
release_date: "2026-11-17"
phase_label: "Phase 7"
phase_name: "Intermediate Cybersecurity"
phase_color: "#e63946"
permalink: /week-31/
---

> Week 31. Phase 7: intermediate cybersecurity and real tools.
> 
> Burp Suite is the standard tool for web application security testing. Every professional pentester and bug bounty hunter uses it. You will use it for the rest of your security career.
> 
> Download Burp Suite Community Edition from https://portswigger.net/burp/communitydownload and install it before this session.
> 
> This week is lab-heavy. Read the guide, then open the PortSwigger Web Academy and work through the labs.

---

# Burp Suite: Web Security Testing Proxy

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Set up Burp Suite as a browser proxy
- Intercept, inspect, and modify HTTP requests
- Use Repeater to manually test requests
- Use Intruder for basic automated parameter fuzzing
- Analyse a web application's attack surface

---

## What is Burp Suite?

Burp Suite is a web security testing platform made by PortSwigger. It sits between your browser and the target server, intercepting every request and response.

This lets you:
- See the raw HTTP traffic including headers, cookies, and POST bodies
- Modify requests before they are sent
- Replay requests with different parameters
- Automate tests across thousands of payloads

The Community Edition is free. It has everything you need to learn. The Professional Edition adds an active scanner and is used in commercial pentesting.

**Download:** https://portswigger.net/burp/communitydownload

---

## Setup: Proxy Your Browser Through Burp

Burp listens on `127.0.0.1:8080` by default. Your browser must send traffic through that port.

**Option 1: Use the built-in Burp Browser (easiest)**

In Burp: Proxy > Open Browser. This opens a Chromium instance preconfigured to route through Burp. No certificate setup needed.

**Option 2: Configure an existing browser**

1. In Firefox: Settings > Network Settings > Manual proxy > HTTP Proxy: `127.0.0.1`, Port: `8080`. Tick "Use for all protocols".

2. Install Burp's CA certificate so HTTPS works:
   - With Burp running, visit `http://burpsuite` in the proxied browser
   - Click "CA Certificate" to download `cacert.der`
   - Firefox: Settings > Privacy > Certificates > Import > select the file

**Test it:**
- Enable intercept in Burp: Proxy > Intercept > Intercept is on
- Visit any HTTP page in the proxied browser
- The request should appear in Burp's Intercept tab and the browser will hang waiting

---

## The Proxy Tab

The Proxy tab shows every request that passes through Burp.

**Intercept tab:** Pause traffic so you can inspect or modify each request before it continues.

```
Controls:
- Forward  -- send the current request on
- Drop     -- discard the request
- Action   -- send to Repeater, Intruder, etc.
- Intercept is off -- toggle to stop pausing requests
```

**HTTP history tab:** A log of every request that has passed through the proxy, even when intercept is off. This is often more useful than intercepting -- browse the target normally and then review what was captured.

Reading an intercepted request:

```http
POST /login HTTP/2
Host: example.com
Cookie: session=abc123
Content-Type: application/x-www-form-urlencoded
Content-Length: 27

username=alice&password=1234
```

Everything you see here can be modified before you forward it.

---

## Repeater: Manually Replay Requests

Repeater lets you modify and resend a request as many times as you want without touching the browser.

**To send a request to Repeater:**
Right-click the request in Intercept or HTTP history > Send to Repeater

**In Repeater:**
- Left pane: the raw request (editable)
- Right pane: the server's response
- Click Send to fire the request
- The response updates in the right pane

**Common uses:**

Testing SQLi: change `id=1` to `id=1'` and check if the response changes or errors.

Testing auth bypass: modify the `session` cookie value and see if you still get a valid response.

Testing IDOR: change `/profile?id=42` to `/profile?id=43` and see if you get someone else's data.

Testing XSS: inject `<script>alert(1)</script>` into a parameter and check if the response reflects it unescaped.

Repeater remembers every request you send. Use the arrow buttons to navigate history.

---

## Decoder Tab

Decoder encodes and decodes common formats:

- Base64: `SGVsbG8=` decodes to `Hello`
- URL encoding: `%3Cscript%3E` decodes to `<script>`
- HTML encoding: `&lt;img&gt;` decodes to `<img>`
- Hex: `48 65 6c 6c 6f` decodes to `Hello`

Paste a value, select the format, click Decode.

Use it when you see an encoded string in a cookie, URL parameter, or POST body and want to know what it is. Also use it to build encoded payloads.

---

## Intruder: Automated Parameter Testing

Intruder sends a single request repeatedly with different payload values.

**To send to Intruder:**
Right-click any request > Send to Intruder

**Step 1: Set positions**
Intruder marks injection points with `§` symbols. Clear the automatic marks with "Clear §". Then highlight the value you want to test and click "Add §".

Example: testing a numeric ID parameter
```
GET /user?id=§1§ HTTP/1.1
```

**Step 2: Set payloads**
Go to the Payloads tab. Payload type options:
- Simple list -- a list of values you provide
- Numbers -- a range of numbers (e.g. 1 to 1000)
- Brute forcer -- all combinations of a character set up to a length

For IDOR testing: numbers 1 to 100.
For password testing: simple list from a wordlist file.
For fuzzing: simple list of common payloads.

**Step 3: Start attack**
Click "Start attack". A results window shows each request and the response length and status code.

Look for:
- Different response lengths (different content = something interesting)
- Different status codes (200 vs 403 vs 500)
- Error messages in responses

**Community Edition limitation:** Intruder is rate-limited in Community Edition (throttled to ~1 request/second). This is enough for learning. Professionals use the paid version or switch to tools like `ffuf` or `hydra` for speed.

---

## Scanner (Pro only, but know it exists)

In the Pro version, the Scanner automatically crawls the target and actively tests for vulnerabilities: XSS, SQLi, SSRF, open redirects, etc.

In Community Edition, you can use the Passive Scanner which analyses requests that pass through the proxy and flags potential issues without sending any extra requests.

An alternative for Community Edition: **PortSwigger Web Academy** has browser-based labs that simulate a full Pro edition environment. https://portswigger.net/web-security

---

## Practical Workflow: Mapping a Target

When you first approach a web application:

1. Turn intercept OFF and browse the entire application normally
2. Log in, navigate every page, submit every form, click every button
3. Open HTTP history and review what was captured:
   - What endpoints exist?
   - What parameters are passed?
   - What cookies are set?
   - Are there any API calls (JSON requests)?
4. Look for: numeric IDs in URLs, token-based auth in headers, hidden form fields
5. Pick interesting requests and send them to Repeater to probe

This is reconnaissance of the application's attack surface.

---

## Burp Tips

**Save your project:** File > Save project (Pro). Community Edition does not persist state between restarts -- take notes.

**Scope:** Go to Target > Scope > Add to scope. Burp will only log and highlight traffic to your defined target, reducing noise.

**Filter HTTP history:** The filter bar at the top of HTTP history lets you show only certain status codes, content types, or search by URL or response body.

**Right-click shortcuts:** Right-clicking any request gives you options to search for, copy, compare, or analyse it.

**Keyboard shortcuts:** Ctrl+R sends to Repeater. Ctrl+I sends to Intruder.

---

## Practice: PortSwigger Web Academy

The best free resource for practising Burp Suite is PortSwigger's own Web Academy:
https://portswigger.net/web-security

It has guided labs for every vulnerability type. The labs run in the browser and give you a preconfigured Burp instance. No setup required.

Start with:
- SQL injection labs
- XSS labs
- Access control labs (IDOR)

Each lab has an apprentice, practitioner, and expert difficulty.

---

## Common Mistakes

1. **Forgetting to turn intercept off.** Every request will hang waiting for you to forward it.

2. **Not installing the CA certificate.** HTTPS requests will fail or show certificate errors.

3. **Testing against live production sites without authorisation.** Use labs, your own apps, or explicitly authorised targets only.

4. **Missing things in HTTP history.** Browse the entire application with intercept OFF first, then review history. Intercept is only useful when you know what you are looking for.

5. **Ignoring response sizes in Intruder.** Interesting results are the ones that are a different size from the baseline.

---

## Activity: PortSwigger Web Academy: SQL Injection Labs

1. Set up Burp Suite with your browser (see the guide for instructions)
1. Log into https://portswigger.net/web-security and go to SQL injection labs
1. Complete the following apprentice labs using Burp Repeater (do NOT use the automated scanner):
1. Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
1. Lab: SQL injection vulnerability allowing login bypass
1. For each lab: intercept the request in Burp, send to Repeater, craft your payload manually, explain in your notes what the payload does and why it works
1. You know you are done when both labs show 'Solved' and you have written explanations for both payloads

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between Burp Intercept and Burp HTTP History?
2. What is Burp Repeater used for?
3. You send 100 requests in Intruder to test an IDOR. How do you identify which responses are interesting without reading every one?
4. What does 'scope' mean in Burp and why should you configure it?
