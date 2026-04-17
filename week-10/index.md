---
layout: page
title: "Git and GitHub: Track and Share Your Code"
week_number: "10"
release_date: "2026-06-23"
phase_label: "Phase 2"
phase_name: "Python, Linux & Git"
phase_color: "#7209b7"
permalink: /week-10/
---

> Week 10. This week: Git and GitHub.
> 
> Git is how every professional developer tracks their work. It is also how security researchers publish tools, write-ups, and proof-of-concept exploits.
> 
> You will use Git for the rest of your career. Getting it set up now means all your future projects will be backed up and versioned.
> 
> First: create a free account at https://github.com if you do not already have one.
> Then: run git config --global user.name 'Your Name' and git config --global user.email 'your@email.com'

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

## Activity: Put Your Portfolio on GitHub

1. Create a new repository on GitHub called 'portfolio' (click the + button in the top right)
1. In your terminal, navigate to the folder containing your portfolio.html from Week 6
1. Run git init, then git add portfolio.html style.css script.js
1. Commit with: git commit -m 'Initial portfolio page'
1. Follow the GitHub instructions to push to your new remote repo
1. Go to your repo settings on GitHub, find Pages, set Source to main branch
1. Wait 1-2 minutes and visit https://your-username.github.io/portfolio -- your page is live
1. Make a small change to portfolio.html, commit and push it, confirm the change appears on the live site
1. You know you are done when your portfolio is publicly accessible via GitHub Pages

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between git add and git commit?
2. What is the staging area and why does it exist?
3. What files should never be committed to a public GitHub repo?
4. What does git log --oneline show you?
