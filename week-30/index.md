---
layout: page
title: "Python Project: Automated Recon Tool"
week_number: "30"
release_date: "2026-11-10"
phase_label: "Phase 6"
phase_name: "Python Automation"
phase_color: "#118ab2"
permalink: /week-30/
---

> Week 30. Phase 6 project: build a real tool.
> 
> You have learned file I/O, web scraping, regex, and APIs in Python. This week you combine them all into a recon tool that automates passive reconnaissance on a domain.
> 
> This is a portfolio piece. By the end it should be something you could show to someone and say 'I built this.'

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

# Linux Command Line: Your First Terminal Skills

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Navigate the Linux file system using the terminal
- Create, move, copy, and delete files and folders
- View file contents and search inside them
- Understand file permissions
- Run scripts and chain commands together

---

## Why Learn the Command Line?

Every cybersecurity tool runs in the terminal. Servers run Linux. CTF challenges require it. Knowing the command line is not optional if you want to work in IT or security.

The terminal is not scary. It is precise -- you type exactly what you want to happen.

---

## Opening a Terminal

| System | How to open |
|--------|------------|
| Ubuntu / Debian | Ctrl + Alt + T |
| macOS | Cmd + Space, type "Terminal" |
| Windows (WSL) | Start menu, search "Ubuntu" or "WSL" |
| Kali Linux | Right-click desktop > Open Terminal |

The **prompt** looks like this:

```
braeden@kali:~$
```

- `braeden` = your username
- `kali` = the machine name (hostname)
- `~` = your current directory (`~` is shorthand for your home folder)
- `$` = you are a regular user (a `#` means root/admin)

---

## Navigation

```bash
pwd             # print working directory -- shows where you are right now
ls              # list files and folders in current directory
ls -l           # long format -- shows permissions, size, date
ls -la          # long format + show hidden files (starting with .)
ls /etc         # list contents of /etc directory

cd Documents    # change into Documents folder
cd ..           # go up one level (parent directory)
cd ~            # go to home directory
cd /            # go to root directory (top of the file system)
cd /etc/ssh     # absolute path -- works from anywhere
```

**Tab completion:** Start typing a folder name and press Tab. The terminal will complete it. Press Tab twice to see all options if there are multiple matches.

**Up arrow:** Press up to cycle through previous commands.

---

## The File System Layout

Linux has one tree starting at `/` (root):

```
/
├── home/         -- user home folders (/home/braeden/)
├── etc/          -- configuration files
├── var/          -- logs, databases, variable data
├── usr/          -- installed programs and libraries
├── bin/          -- essential system commands
├── tmp/          -- temporary files (cleared on reboot)
├── root/         -- home folder of the root user
└── proc/         -- virtual files representing running processes
```

Your home folder is `/home/your-username/` or `~` for short.

---

## Working With Files

```bash
touch hello.txt             # create an empty file
mkdir projects              # create a folder
mkdir -p a/b/c              # create nested folders

cat hello.txt               # print file contents to screen
less hello.txt              # page through long files (q to quit)
head -20 hello.txt          # first 20 lines
tail -20 hello.txt          # last 20 lines
tail -f /var/log/syslog     # follow a file as it grows (useful for logs)

cp hello.txt backup.txt     # copy a file
cp -r projects/ backup/     # copy a folder (recursively)

mv hello.txt renamed.txt    # rename a file
mv hello.txt ~/Documents/   # move a file to another folder

rm hello.txt                # delete a file (no recycle bin -- gone forever)
rm -r projects/             # delete a folder and everything inside
```

**Warning:** `rm` is permanent. There is no undo. Be careful.

---

## Viewing and Editing Files

```bash
cat file.txt               # print entire file
nano file.txt              # simple text editor (Ctrl+O to save, Ctrl+X to exit)
vim file.txt               # powerful editor (press i to insert, Esc then :wq to save and quit)
```

For beginners, `nano` is easier than `vim`. VS Code with the terminal open is fine for most tasks.

---

## Searching

```bash
grep "error" logfile.txt           # find lines containing "error"
grep -i "error" logfile.txt        # case-insensitive search
grep -r "password" /etc/           # search recursively in a folder
grep -n "TODO" script.py           # show line numbers

find / -name "*.conf"              # find all .conf files everywhere
find . -name "*.py"                # find .py files in current folder
find /var/log -mtime -1            # files modified in the last 1 day
```

Searching for a running process:

```bash
ps aux | grep python               # show processes with "python" in the name
```

---

## Pipes and Redirection

Pipes connect the output of one command to the input of another:

```bash
ls -l | grep ".txt"        # list files, then filter to only .txt files
cat log.txt | grep "ERROR" | head -10   # find errors, show first 10
```

Redirection sends output to a file instead of the screen:

```bash
echo "Hello" > hello.txt       # write to file (overwrites)
echo "World" >> hello.txt      # append to file
ls -l > filelist.txt           # save directory listing to a file
```

---

## Permissions

Every file has three permission groups: **owner**, **group**, **others**

Each group has three permissions: **r** (read), **w** (write), **x** (execute)

View permissions with `ls -l`:

```
-rwxr-xr--  1  braeden  staff  4096  Apr 16  script.py
 |||||||||||
 ||||||||||+-- others can read
 |||||||||+--- others cannot write
 ||||||||+---- others cannot execute
 |||||||+----- group can read
 ||||||+------ group cannot write
 |||||+------- group can execute
 ||||+-------- owner can read
 |||+--------- owner can write
 ||+---------- owner can execute
 |+----------- file type (- = file, d = directory, l = symlink)
```

Change permissions:

```bash
chmod +x script.py          # add execute permission for everyone
chmod 755 script.py         # rwxr-xr-x (owner full, others read+execute)
chmod 600 private.key       # rw------- (owner only -- for SSH keys)
```

Run a script:

```bash
chmod +x myscript.sh
./myscript.sh
```

---

## Users and Sudo

```bash
whoami                       # show current username
id                           # show user ID, group ID, and groups
sudo command                 # run command as root (administrator)
sudo apt install nmap        # install a package as root
su - username                # switch to another user
```

`sudo` stands for "superuser do". Only use it when necessary. Running everything as root is dangerous -- if you make a mistake, there are no limits on the damage.

---

## Package Management (Debian/Ubuntu/Kali)

```bash
sudo apt update                  # refresh the package list
sudo apt upgrade                 # install updates
sudo apt install nmap            # install a package
sudo apt remove nmap             # uninstall a package
apt search "keyword"             # search for a package
dpkg -l | grep python            # list installed packages matching "python"
```

---

## Useful Shortcuts

| Shortcut | What it does |
|----------|-------------|
| Tab | Autocomplete file/command name |
| Up/Down arrows | Navigate command history |
| Ctrl + C | Kill a running command |
| Ctrl + Z | Suspend a command (send to background) |
| Ctrl + L | Clear the screen |
| Ctrl + A | Move cursor to start of line |
| Ctrl + E | Move cursor to end of line |
| `!!` | Repeat the last command |
| `history` | Show all previous commands |
| `history | grep ssh` | Search command history |

---

## Environment Variables

```bash
echo $HOME          # /home/braeden
echo $PATH          # directories where executables are searched
echo $USER          # your username

export MY_VAR="hello"   # set a variable for this session
echo $MY_VAR            # hello
```

---

## Networking Commands

```bash
ip addr                      # show IP addresses (modern)
ifconfig                     # show IP addresses (older, may need installing)
ping google.com              # test connectivity (Ctrl+C to stop)
curl https://example.com     # fetch a webpage
wget https://example.com     # download a file
ss -tlnp                     # show listening ports
netstat -tlnp                # same (older command)
nmap -sV 192.168.1.1         # scan ports on a host (requires nmap)
```

---

## Quick Reference: Most Used Commands

```bash
pwd, ls, cd          # navigate
touch, mkdir         # create files/folders
cat, less, head      # view files
cp, mv, rm           # copy, move, delete
grep                 # search inside files
find                 # find files by name/type
chmod, chown         # change permissions/owner
sudo                 # run as administrator
apt install          # install software
ps aux               # list processes
kill <PID>           # stop a process
echo, export         # variables
|  > >>              # pipe and redirect
```

---

## Common Mistakes

1. **`rm -rf /` or `rm -rf *` in the wrong directory.** Always double-check your current directory with `pwd` before any `rm -r`.

2. **Running everything as root.** Use `sudo` only for the specific command that needs it.

3. **Forgetting `./` before a script.** `script.sh` won't work. `./script.sh` will.

4. **Case sensitivity.** Linux filenames are case-sensitive. `File.txt` and `file.txt` are different files.

5. **Using spaces in file names.** Spaces in file names cause problems in the terminal. Use underscores or hyphens instead.

---

## Practice Ideas

1. Navigate to your home folder. Create a folder called `practice`. Inside it, create three files: `notes.txt`, `ideas.txt`, `todo.txt`.
2. Write "Hello, terminal!" into `notes.txt` using `echo > `. View it with `cat`.
3. Copy `notes.txt` to `backup.txt`. Rename `backup.txt` to `notes_backup.txt`.
4. Search `notes.txt` for the word "Hello" using `grep`.
5. List all files in your home folder, pipe it to `grep` to filter for only `.txt` files.

---

## What to Read Next

- `10_git-basics.md` - Version control from the command line
- Cybersecurity: `learning/03_cybersecurity/intermediate/01_network-attacks.md` - Many commands from this guide apply

---

## Activity: Passive Recon Tool

1. Build a Python CLI tool: python recon.py example.com
1. It should run the following checks automatically:
1. DNS: resolve A, MX, TXT, NS records using the dnspython library (pip install dnspython)
1. Certificate transparency: query https://crt.sh/?q=%.{domain}&output=json to find subdomains
1. HTTP headers: fetch the target URL and check for missing security headers (CSP, HSTS, X-Frame-Options)
1. Technology detection: identify server software from the Server and X-Powered-By headers
1. All results should be saved to a JSON report file: recon_{domain}_{date}.json
1. You know you are done when the tool runs on example.com and produces a complete JSON report with findings in all four categories

---

## Check-in Questions

Write your answers down before your next session.

1. What is certificate transparency and why does crt.sh have subdomain information?
2. Why is passive recon safer and more legally sound than active scanning?
3. How did you structure your JSON output? What fields did you use?
4. What is one thing you would add to this tool if you had another week?
