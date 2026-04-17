---
layout: page
title: "Linux Command Line"
week_number: "9"
release_date: "2026-06-16"
phase_label: "Phase 2"
phase_name: "Python, Linux & Git"
phase_color: "#7209b7"
permalink: /week-09/
---

> Week 9. This week: the Linux terminal.
> 
> Every cybersecurity tool runs here. Every server runs Linux. CTF challenges require it. The terminal is not optional.
> 
> If you are on Windows: install WSL (Windows Subsystem for Linux) by opening PowerShell as Administrator and running: wsl --install
> Then open Ubuntu from the Start menu.
> 
> If you are on Mac: your Terminal app runs a Unix shell that works the same way.
> 
> Goal this week: everything you do should be done from the terminal. No file explorer. No GUI. Just the command line.

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

## Activity: CLI Speed Run Challenge

1. Open a terminal
1. Complete these tasks using only command line commands -- no file explorer or GUI:
1. Create a folder called ctf-practice with 3 subfolders: tools, notes, flags
1. Create a file called README.txt in ctf-practice with the text 'My CTF workspace' using echo and redirect
1. Copy README.txt into all three subfolders
1. In the notes folder, create a file called targets.txt. Add 5 made-up IP addresses to it (one per line) using echo >>
1. Use grep to search targets.txt for lines containing '192'
1. Use find to list all .txt files inside ctf-practice and its subfolders
1. You know you are done when all files exist and find outputs at least 4 results

---

## Check-in Questions

Write your answers down before your next session.

1. What does pwd print?
2. What is the difference between > and >> when redirecting output to a file?
3. What does chmod +x script.sh do and why do you need it before running a script?
4. What does the pipe symbol | do? Give one example.
