---
layout: page
title: "Secret Scanning and Static Analysis"
week_number: "43"
release_date: "2027-02-09"
phase_label: "Phase 9"
phase_name: "DevOps & Cloud Security"
phase_color: "#4cc9f0"
permalink: /week-43/
---

> Week 43. Keeping secrets out of code and finding bugs before they ship.
> 
> Two of the most impactful security practices you can add to any project:
> 1. Secret scanning -- automatically detect API keys, passwords, and tokens before they reach GitHub
> 2. SAST (Static Application Security Testing) -- analyse code for security bugs without running it
> 
> These are used by every professional development team.

---

# CI/CD Fundamentals

**Pillar:** DevOps
**Level:** Beginner
**Read time:** 15 minutes

## What and Why

**CI (Continuous Integration):** Every code change is automatically built and tested the moment it is pushed. You find out within minutes if you broke something, not days later in code review.

**CD (Continuous Delivery/Deployment):** After CI passes, the change is automatically packaged and either staged for release (Delivery) or deployed directly to production (Deployment).

The goal: code goes from a developer's laptop to production in a predictable, repeatable, automated way.

---

## The Pipeline Model

A CI/CD pipeline is a sequence of stages that a code change passes through:

```
Developer pushes code
        |
        v
  [Trigger] (push to branch, pull request opened)
        |
        v
  [Build] Compile, install dependencies
        |
        v
  [Test] Unit tests, integration tests, linting
        |
        v
  [Security Scan] Dependency audit, SAST (optional)
        |
        v
  [Package] Build Docker image, create artifact
        |
        v
  [Deploy to Staging]
        |
        v
  [Acceptance Tests] (optional)
        |
        v
  [Deploy to Production] (manual gate or automatic)
```

A stage failure stops the pipeline. Nothing broken gets deployed.

---

## GitHub Actions

GitHub Actions is the most accessible CI/CD system for individual developers. Workflows are defined as YAML files in `.github/workflows/`.

### A minimal CI workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: pytest

      - name: Lint
        run: ruff check .
```

### Key concepts

| Term | Meaning |
|------|---------|
| `on` | What triggers the workflow (push, PR, schedule, manual) |
| `jobs` | Parallel or sequential units of work |
| `steps` | Ordered actions within a job |
| `uses` | Run a pre-built action from GitHub Marketplace |
| `run` | Run a shell command |
| `env` | Environment variables for the job |
| `secrets` | Encrypted values (API keys, passwords) stored in repo settings |

### Accessing secrets

```yaml
- name: Deploy
  env:
    API_KEY: ${{ secrets.DEPLOY_API_KEY }}
  run: ./deploy.sh
```

Never hardcode secrets in workflow files. They appear in logs and git history.

---

## Docker in CI/CD

Docker makes environments reproducible. The same image that passes tests locally runs in production.

### Build and push an image in CI

```yaml
- name: Build and push Docker image
  run: |
    docker build -t myapp:${{ github.sha }} .
    echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
    docker push myapp:${{ github.sha }}
```

Tag with the commit SHA (`github.sha`) so every build is traceable to exactly the code that produced it.

---

## Branching Strategies

How you branch determines what triggers deployments.

### GitHub Flow (simple)

```
main (always deployable)
  |
  +-- feature/new-login-page
  +-- fix/broken-validation
```

- All work on feature branches
- PR into `main`
- `main` auto-deploys to production after passing CI

Good for small teams and web services that deploy frequently.

### Git Flow (structured releases)

```
main (production)
  |
develop (integration)
  |
  +-- feature/*
  +-- release/*
  +-- hotfix/*
```

Good for versioned software (libraries, mobile apps) with scheduled releases.

---

## Key Metrics

| Metric | What it measures | Target |
|--------|-----------------|--------|
| Pipeline duration | How long CI takes end to end | Under 10 minutes |
| Test pass rate | % of pipeline runs that pass | > 95% |
| Deployment frequency | How often code reaches production | Multiple times per day (ideal) |
| Lead time | Commit to production time | Hours, not weeks |
| MTTR (Mean Time to Restore) | How fast you recover from a broken deploy | Under 1 hour |

These are the DORA metrics. High performers on all four metrics ship faster and more reliably.

## Checklist

- [ ] Can you explain the difference between CI, CD (delivery), and CD (deployment)
- [ ] Have you read a GitHub Actions workflow file and understood every line
- [ ] Do you know where secrets go and why they never go in code
- [ ] Can you name the four DORA metrics

## Next Steps

- Lab: `activities/devops/beginner/01_ci-hello.md`
- Lab: `activities/devops/beginner/02_docker-intro.md`
- Guide: `intermediate/01_monitoring-observability.md`
- Quiz: `learning/quizzes/04_devops.md`

---

## Activity: Secret Scanning and SAST

1. Complete the activity at activities/secdevops/intro/01_secret-scan.md (Gitleaks)
1. Install gitleaks and scan your own repos: gitleaks detect --source . --report-format json --report-path results.json
1. Complete the activity at activities/secdevops/intro/02_sast-basics.md (Bandit and Semgrep)
1. Write a short Python script with 3 deliberate security flaws (SQL injection, shell injection, hardcoded password). Run Bandit on it and verify it finds all three
1. Add gitleaks scanning to your GitHub Actions workflow from last week
1. You know you are done when your Actions workflow includes a secrets scan step and Bandit finds the 3 flaws in your test script

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between SAST and DAST?
2. Why is it important to scan for secrets before a commit reaches GitHub rather than after?
3. Bandit reports a B106 finding (hardcoded password). What does that mean and how would you fix it?
4. What is a false positive in security scanning and why is too many of them a problem?
