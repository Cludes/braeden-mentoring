---
layout: page
title: "Python Web Scraping"
week_number: "27"
release_date: "2026-10-20"
phase_label: "Phase 6"
phase_name: "Python Automation"
phase_color: "#118ab2"
permalink: /week-27/
---

> Week 27. Web scraping -- extracting data from websites using Python.
> 
> Scraping is used in security for: passive reconnaissance (finding company info without touching their servers), collecting data from public sources, automated testing. It is also widely used in data analysis and automation.
> 
> Key rule: check robots.txt before scraping. Be polite -- add delays between requests. Only scrape public data.
> 
> Install first: pip install requests beautifulsoup4

---

# Python Automation: Files, Web Scraping, APIs, and Regex

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Read, write, and parse text files in Python
- Send HTTP requests with the `requests` library
- Scrape data from a web page with BeautifulSoup
- Match and extract text patterns using regular expressions
- Automate repetitive IT tasks with Python scripts

---

## File I/O

Reading and writing files is one of the most common tasks in scripting. Every log file, config, CSV, and report is a file.

### Reading Files

```python
# Read entire file as a string
with open("log.txt", "r") as f:
    contents = f.read()

# Read line by line (memory-efficient for large files)
with open("log.txt", "r") as f:
    for line in f:
        print(line.strip())   # strip() removes trailing \n

# Read all lines into a list
with open("log.txt", "r") as f:
    lines = f.readlines()
```

Always use `with open(...)`. It automatically closes the file even if an exception occurs.

### Writing Files

```python
# Write (creates file, overwrites if it exists)
with open("output.txt", "w") as f:
    f.write("Hello\n")
    f.write("World\n")

# Append to existing file
with open("log.txt", "a") as f:
    f.write("New log entry\n")

# Write multiple lines at once
lines = ["line1\n", "line2\n", "line3\n"]
with open("output.txt", "w") as f:
    f.writelines(lines)
```

### Paths

```python
from pathlib import Path

p = Path("reports/scan_2026.txt")

p.exists()          # True/False
p.parent            # Path("reports")
p.name              # "scan_2026.txt"
p.stem              # "scan_2026"
p.suffix            # ".txt"
p.read_text()       # reads file contents (shortcut)
p.write_text("...")  # writes file contents (shortcut)

# List all .txt files in a folder
for f in Path("logs").glob("*.txt"):
    print(f.name)
```

### CSV Files

```python
import csv

# Read CSV
with open("data.csv", "r") as f:
    reader = csv.DictReader(f)   # headers become dict keys
    for row in reader:
        print(row["ip_address"], row["port"])

# Write CSV
with open("results.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["ip", "port", "service"])
    writer.writeheader()
    writer.writerow({"ip": "192.168.1.1", "port": 80, "service": "http"})
```

---

## HTTP Requests with `requests`

The `requests` library is the standard way to make HTTP requests in Python.

```bash
pip install requests
```

### GET Request

```python
import requests

response = requests.get("https://api.github.com/users/octocat")

print(response.status_code)   # 200
print(response.headers)       # dict of headers
data = response.json()         # parse JSON body
print(data["name"])           # "The Octocat"
```

### Checking for Errors

```python
response = requests.get("https://api.example.com/data")

# Option 1: manual check
if response.status_code != 200:
    print(f"Error: {response.status_code}")

# Option 2: raise an exception automatically for 4xx/5xx
response.raise_for_status()   # raises requests.HTTPError if not 2xx
```

### Query Parameters

```python
params = {"q": "python security", "per_page": 5}
response = requests.get("https://api.github.com/search/repositories", params=params)
# becomes: https://api.github.com/search/repositories?q=python+security&per_page=5
```

### POST Request

```python
payload = {"username": "alice", "password": "hunter2"}
headers = {"Content-Type": "application/json"}

response = requests.post(
    "https://httpbin.org/post",
    json=payload,     # automatically sets Content-Type and serialises to JSON
)
print(response.json())
```

### Sessions (for cookies and auth)

```python
session = requests.Session()

# Log in once
session.post("https://example.com/login", data={"user": "alice", "pass": "1234"})

# Session carries cookies on subsequent requests
profile = session.get("https://example.com/profile")
```

### Timeouts and Error Handling

```python
try:
    response = requests.get("https://api.example.com", timeout=10)
    response.raise_for_status()
    return response.json()
except requests.exceptions.Timeout:
    print("Request timed out")
except requests.exceptions.ConnectionError:
    print("Could not connect")
except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e}")
```

---

## Web Scraping with BeautifulSoup

Web scraping extracts data from HTML pages. Use it when there is no API.

```bash
pip install requests beautifulsoup4
```

```python
import requests
from bs4 import BeautifulSoup

url = "https://books.toscrape.com"
response = requests.get(url)
soup = BeautifulSoup(response.text, "html.parser")

# Find elements
title = soup.find("h1").text                       # first h1
all_links = soup.find_all("a")                     # all links
prices = soup.find_all("p", class_="price_color")  # by class

for price in prices:
    print(price.text.strip())

# CSS selectors (same as JavaScript querySelector)
items = soup.select("article.product_pod")         # all product cards
for item in items:
    name = item.select_one("h3 a")["title"]        # get attribute
    price = item.select_one(".price_color").text
    print(name, price)
```

### Scraping Multiple Pages

```python
import requests
from bs4 import BeautifulSoup
import time

results = []

for page in range(1, 6):   # pages 1-5
    url = f"https://books.toscrape.com/catalogue/page-{page}.html"
    response = requests.get(url)
    soup = BeautifulSoup(response.text, "html.parser")

    for item in soup.select("article.product_pod"):
        results.append({
            "title": item.select_one("h3 a")["title"],
            "price": item.select_one(".price_color").text,
        })

    time.sleep(1)   # be polite -- don't hammer the server

print(f"Scraped {len(results)} books")
```

### Ethics of Scraping

- Check the site's `robots.txt` file: `https://site.com/robots.txt` - it tells you what you are allowed to scrape
- Do not scrape at high speed (add `time.sleep(1)` between requests)
- Do not scrape personal data without a legal basis
- If the site has an API, use it instead

---

## Regular Expressions

Regex lets you match, find, and extract patterns in text.

```python
import re
```

### Core Functions

```python
text = "Server at 192.168.1.100 responded with 200 OK"

re.search(r"\d+\.\d+\.\d+\.\d+", text)     # find first IP - returns match object or None
re.findall(r"\d+", text)                    # find ALL numbers - returns list ['192', '168', '1', '100', '200']
re.sub(r"\d{3,}", "XXX", text)             # replace - 'Server at XXX.XXX.X.XXX responded with XXX OK'
re.match(r"Server", text)                   # match at START of string only
re.split(r"\s+", text)                      # split on whitespace
```

### Pattern Syntax

| Pattern | Matches |
|---------|---------|
| `.` | Any single character (except newline) |
| `\d` | Digit (0-9) |
| `\w` | Word character (a-z, A-Z, 0-9, _) |
| `\s` | Whitespace (space, tab, newline) |
| `\D`, `\W`, `\S` | NOT digit, word, whitespace |
| `^` | Start of string |
| `$` | End of string |
| `*` | 0 or more |
| `+` | 1 or more |
| `?` | 0 or 1 (optional) |
| `{3}` | Exactly 3 |
| `{2,5}` | 2 to 5 |
| `[abc]` | One of a, b, or c |
| `[^abc]` | Not a, b, or c |
| `(group)` | Capture group |
| `a\|b` | a or b |

### Practical Examples

```python
# Extract all IP addresses from a log file
log = open("access.log").read()
ips = re.findall(r"\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b", log)

# Extract email addresses
emails = re.findall(r"[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}", text)

# Check if a string is a valid Australian mobile number
def is_mobile(s):
    return bool(re.match(r"^04\d{8}$", s))

# Find lines in a log that contain HTTP errors
errors = [line for line in log.splitlines() if re.search(r"HTTP/\d\.\d [45]\d{2}", line)]

# Extract groups
match = re.search(r"(\d{1,3}(?:\.\d{1,3}){3}):(\d+)", "192.168.1.1:8080")
if match:
    ip = match.group(1)    # "192.168.1.1"
    port = match.group(2)  # "8080"
```

---

## Automating File System Tasks

```python
import os
import shutil
from pathlib import Path

# List files
for f in Path("/var/log").iterdir():
    print(f.name)

# Create directories
Path("reports/2026").mkdir(parents=True, exist_ok=True)

# Move/rename
shutil.move("old_name.txt", "new_name.txt")

# Copy
shutil.copy("source.txt", "backup/source.txt")

# Delete
Path("temp.txt").unlink()             # delete file
shutil.rmtree("temp_folder")          # delete folder and contents

# Check disk usage
import shutil
total, used, free = shutil.disk_usage("/")
print(f"Free: {free // (2**30)} GB")
```

---

## Running Shell Commands from Python

```python
import subprocess

# Run a command, wait for it to finish, capture output
result = subprocess.run(
    ["nmap", "-sV", "192.168.1.1"],
    capture_output=True,
    text=True,
    timeout=30
)

print(result.stdout)    # standard output
print(result.stderr)    # error output
print(result.returncode)  # 0 = success

# Simple one-liner for trusted commands
output = subprocess.check_output(["whoami"], text=True).strip()
```

Never build shell commands by concatenating user input. Use a list of arguments instead:

```python
# WRONG - command injection vulnerability
os.system(f"nmap {user_input}")

# RIGHT - arguments are passed safely, not interpreted by shell
subprocess.run(["nmap", user_input], capture_output=True)
```

---

## Putting It Together: A Simple Recon Script

```python
#!/usr/bin/env python3
"""Quick passive recon: DNS lookup + HTTP headers for a domain."""

import re
import sys
import socket
import requests

def recon(domain: str):
    print(f"\n=== Recon: {domain} ===\n")

    # DNS
    try:
        ip = socket.gethostbyname(domain)
        print(f"IP address: {ip}")
    except socket.gaierror as e:
        print(f"DNS lookup failed: {e}")
        return

    # HTTP headers
    for scheme in ("https", "http"):
        try:
            r = requests.head(f"{scheme}://{domain}", timeout=5, allow_redirects=True)
            print(f"\nHTTP headers ({scheme.upper()}):")
            for header in ("Server", "X-Powered-By", "X-Frame-Options",
                           "Content-Security-Policy", "Strict-Transport-Security"):
                value = r.headers.get(header, "NOT SET")
                print(f"  {header}: {value}")
            break
        except requests.exceptions.RequestException:
            continue

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python recon.py <domain>")
        sys.exit(1)
    recon(sys.argv[1])
```

---

## Common Mistakes

1. **Not closing files.** Always use `with open(...)`.

2. **Scraping without a delay.** Add `time.sleep(1)` between requests.

3. **Using `os.system()` with user input.** Use `subprocess.run(["cmd", arg])` instead.

4. **Not handling request timeouts.** Always set `timeout=` on `requests.get()`.

5. **Overly greedy regex.** `.*` matches as much as possible. Use `.*?` (lazy) to match as little as possible.

---

## Activity: HackerNews Headline Scraper

1. Scrape the front page of https://news.ycombinator.com
1. Extract all story titles, URLs, and point scores
1. Filter to only stories with more than 100 points
1. Save the results to headlines.csv with columns: title, url, points
1. Then also fetch each story URL's domain (just the hostname part, not the full URL) and count how many stories come from each domain
1. Print the top 5 domains by story count to the terminal
1. You know you are done when headlines.csv exists, contains only high-scoring stories, and the domain counts are accurate

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between soup.find() and soup.find_all()?
2. What does soup.select('.classname') do and how is it different from soup.find_all(class_='classname')?
3. Why should you add time.sleep(1) between scraping multiple pages?
4. A site shows data you want to scrape but its robots.txt says Disallow: /. What should you do?
