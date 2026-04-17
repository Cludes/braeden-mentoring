---
layout: page
title: "Build and Publish a Security Tool"
week_number: "46"
release_date: "2027-03-02"
phase_label: "Phase 10"
phase_name: "Projects, Career & Year Two"
phase_color: "#2ec4b6"
permalink: /week-46/
---

> Week 46. Build a real security tool and publish it.
> 
> You have built small tools throughout this curriculum. This week you build something polished and publish it publicly on GitHub.
> 
> A public security tool demonstrates: Python skills, security knowledge, documentation ability, and that you can finish and ship something. It is one of the best things you can put on a portfolio.

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

# Git Basics: Track and Share Your Code

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Explain what Git is and why developers use it
- Create a local Git repository
- Save changes with commits
- View the history of a project
- Push code to GitHub and clone a repo

---

## What is Git?

Git is a tool that tracks every change you make to your code. It stores a complete history so you can:

- See exactly what changed and when
- Go back to any previous version
- Work on multiple features at the same time without breaking things
- Collaborate with other developers

Without Git, you end up with folders like `project_v2_FINAL_v3_REALLY_FINAL/`. With Git, you have one folder and a complete history.

**Git** is the tool. **GitHub** is a website that hosts Git repositories online. They are different things. You can use Git without GitHub, but most developers use both.

---

## Installing Git

```bash
# Check if git is installed
git --version

# Install on Debian/Ubuntu/Kali
sudo apt install git

# macOS (if not already installed)
xcode-select --install
```

First-time setup (do this once):

```bash
git config --global user.name "Braeden"
git config --global user.email "your@email.com"
git config --global core.editor "nano"   # or "code" for VS Code
```

---

## The Three Areas

Understanding these three areas is the key to understanding Git:

```
Working Directory    Staging Area (Index)    Repository (.git/)
       |                     |                      |
  Edit files          git add <file>         git commit -m "..."
       |                     |                      |
  Your changes     Changes ready to save    Saved snapshot
```

1. **Working directory** - files on disk. You edit them normally.
2. **Staging area** - you choose which changes to include in the next commit.
3. **Repository** - the saved history. Every commit is a permanent snapshot.

---

## Starting a New Project

```bash
mkdir my-project
cd my-project
git init
```

This creates a hidden `.git/` folder. That folder IS the repository -- never delete it.

---

## The Daily Workflow

```bash
# 1. Check what has changed
git status

# 2. See the exact changes (line by line)
git diff

# 3. Stage the files you want to commit
git add index.html
git add style.css
git add .           # stage EVERYTHING in current folder (use with care)

# 4. Commit with a message
git commit -m "Add homepage and basic styles"

# 5. Check the history
git log
git log --oneline   # compact version
```

That is the core loop. You will do steps 1-5 dozens of times a day.

---

## Writing Good Commit Messages

A commit message is a permanent record of what you did and why.

Good:
```
Add password validation to signup form
Fix broken link in navigation menu
Remove unused CSS variables
```

Bad:
```
update
fix stuff
asdfgh
```

Format: start with a verb, be specific, keep it under 72 characters.

---

## Viewing History

```bash
git log                     # full log
git log --oneline           # one line per commit
git log --oneline -10       # last 10 commits
git show abc1234            # show what changed in a specific commit
git diff HEAD~1             # compare to previous commit
```

---

## Undoing Things

```bash
# Unstage a file (undo git add)
git restore --staged index.html

# Discard changes to a file (goes back to last commit -- cannot undo)
git restore index.html

# Undo the last commit but keep the changes in your working directory
git reset HEAD~1

# View old version of a file
git show abc1234:index.html
```

---

## Branches

A branch is an independent copy of your code. Use branches when:
- Building a new feature (don't break the working version)
- Fixing a bug (keep main clean while you work)

```bash
git branch                      # list branches
git branch feature-login        # create a new branch
git checkout feature-login      # switch to that branch
git checkout -b feature-login   # create AND switch (shortcut)

# Do your work, commit normally
git add .
git commit -m "Add login page"

# Switch back to main
git checkout main

# Merge your work in
git merge feature-login

# Delete the branch (clean up)
git branch -d feature-login
```

---

## .gitignore

A `.gitignore` file tells Git which files to never track. This is important for:
- Secrets and passwords (API keys, `.env` files)
- Generated files (compiled code, `node_modules/`)
- OS files (`.DS_Store`, `Thumbs.db`)

Create a `.gitignore` file in your project root:

```
# Secrets
.env
*.key
*_credentials*

# Node.js
node_modules/

# Python
__pycache__/
*.pyc
venv/

# Operating system
.DS_Store
Thumbs.db

# IDE
.vscode/settings.json
.idea/
```

---

## GitHub: Putting Your Code Online

**Step 1:** Create an account at https://github.com

**Step 2:** Create a new repository on GitHub (click the + button)

**Step 3:** Connect your local repo to GitHub

```bash
git remote add origin https://github.com/your-username/my-project.git
git branch -M main
git push -u origin main
```

**Step 4:** After that, to push new commits:

```bash
git push
```

**Cloning someone else's repo:**

```bash
git clone https://github.com/username/repo-name.git
cd repo-name
```

---

## Pulling Changes

If someone else (or you from another computer) pushed changes to GitHub:

```bash
git pull     # download and merge remote changes
```

Do this at the start of every session to make sure you are working with the latest version.

---

## Quick Reference

| Command | What it does |
|---------|-------------|
| `git init` | Create a new repository |
| `git clone <url>` | Download an existing repository |
| `git status` | Show what has changed |
| `git diff` | Show exact line changes |
| `git add <file>` | Stage a file |
| `git commit -m "msg"` | Save a snapshot |
| `git log --oneline` | View history |
| `git push` | Upload commits to GitHub |
| `git pull` | Download latest changes |
| `git branch` | List branches |
| `git checkout -b name` | Create and switch to branch |
| `git merge name` | Merge branch into current |
| `git restore <file>` | Discard file changes |

---

## Common Mistakes

1. **Committing passwords or API keys.** Once pushed to GitHub, everyone can see it. Always check `.gitignore` before `git add .`.

2. **Giant first commit.** Commit small, focused changes often. Do not let hundreds of file changes pile up.

3. **Not writing a message.** `git commit` without `-m` opens an editor. Exit with `:wq` if it opens Vim unexpectedly.

4. **Working directly on main.** Use branches for features. Keep main clean.

5. **Forgetting to `git pull` before starting work.** You end up out of sync with collaborators.

---

## What to Read Next

- `learning/02_software-engineering/beginner/01_git-workflows.md` - Branching strategies, commit conventions
- GitHub Docs: https://docs.github.com/en/get-started

---

## Activity: Build and Release a Security Tool

1. Choose one of these projects (or propose your own to Andrew):
1. Option A: Expand your passive recon tool from Week 30 into a full CLI tool with multiple modules (DNS, HTTP headers, crt.sh, technology detection, report generation) and a clean --help output
1. Option B: Build a password audit tool that checks a list of passwords against HIBP API (https://haveibeenpwned.com/API/v3), scores each one, and reports weak or compromised passwords
1. Option C: Build a web security header checker that scans a URL and produces a colour-coded report of present/missing security headers with explanations
1. Whichever you choose: write a proper README.md (what it does, how to install, how to use, example output), tag a v1.0 release on GitHub, push the Docker image to Docker Hub
1. You know you are done when a stranger could find your repo, follow the README, and run the tool without asking you anything

---

## Check-in Questions

Write your answers down before your next session.

1. What makes a good README.md? What sections should it always have?
2. What is semantic versioning (semver) and what does v1.2.3 mean?
3. Why does your tool need a requirements.txt file?
4. How would you add automated tests to your tool so CI catches regressions?
