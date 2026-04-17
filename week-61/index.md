---
layout: page
title: "Secrets Management with HashiCorp Vault"
week_number: "61"
release_date: "2027-06-15"
phase_label: "Phase 12"
phase_name: "SecDevOps"
phase_color: "#10b981"
permalink: /week-61/
---

> Week 61. Secrets management.
> 
> Hardcoded secrets are one of the most common and most damaging vulnerability classes. API keys in GitHub, database passwords in config files, AWS credentials in environment variables -- all have caused major breaches.
> 
> HashiCorp Vault is the industry standard for secrets management. This week you run it locally and integrate it with a Python app.

---

# DevSecOps Pipeline

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Explain what DevSecOps is and how it differs from traditional security reviews
- Design a CI/CD pipeline with integrated security checks at each stage
- Configure SAST, dependency scanning, secret scanning, container scanning, and IaC scanning in GitHub Actions
- Understand secrets management and why environment variables are not enough
- Identify what each security tool catches and what it misses

---

## What is DevSecOps?

DevSecOps means integrating security checks into the development and deployment process rather than doing a single security review before release.

The goal is to catch vulnerabilities when they are cheapest to fix: at the point they are introduced.

**Traditional approach:**
```
Code -> Build -> Test -> Security review (weeks/months later) -> Deploy
```

**DevSecOps approach:**
```
Code -> [SAST + secrets scan] -> Build -> [dep scan] -> 
[container scan] -> [IaC scan] -> Deploy -> [DAST + monitoring]
```

---

## The Pipeline Stages

### Stage 1: Static Application Security Testing (SAST)

Scans source code for security issues without running it.

**Python: Bandit**
```yaml
- name: SAST - Bandit
  run: |
    pip install bandit
    bandit -r . -ll -ii --exit-zero -f json -o bandit-report.json
    bandit -r . -ll -ii   # fail on medium+ severity
```

**JavaScript: Semgrep**
```yaml
- name: SAST - Semgrep
  uses: returntocorp/semgrep-action@v1
  with:
    config: p/owasp-top-ten
```

### Stage 2: Secret Scanning

Prevents credentials, API keys, and tokens from being committed.

**detect-secrets:**
```yaml
- name: Secret scan
  run: |
    pip install detect-secrets
    detect-secrets scan --baseline .secrets.baseline
    detect-secrets audit .secrets.baseline
```

**Gitleaks:**
```yaml
- name: Gitleaks
  uses: gitleaks/gitleaks-action@v2
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Stage 3: Dependency Scanning

Checks third-party libraries for known CVEs.

**pip-audit:**
```yaml
- name: Dependency audit
  run: |
    pip install pip-audit
    pip-audit --requirement requirements.txt --strict
```

**npm audit:**
```yaml
- name: npm audit
  run: npm audit --audit-level=high
```

### Stage 4: Container Image Scanning

Scans the built Docker image for CVEs in the OS and installed packages.

**Trivy:**
```yaml
- name: Build image
  run: docker build -t my-app:${{ github.sha }} .

- name: Trivy image scan
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: my-app:${{ github.sha }}
    exit-code: 1
    severity: CRITICAL
    format: sarif
    output: trivy-results.sarif
```

### Stage 5: IaC Scanning

Scans Terraform, CloudFormation, or Kubernetes manifests for misconfigurations.

**Checkov:**
```yaml
- name: Checkov IaC scan
  uses: bridgecrewio/checkov-action@v12
  with:
    directory: ./terraform
    framework: terraform
    soft_fail: false
    check: CKV_AWS_*
```

---

## Complete GitHub Actions Workflow

```yaml
name: DevSecOps Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0   # full history for gitleaks

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: SAST - Bandit
        run: |
          pip install bandit
          bandit -r . -ll -ii

      - name: Secret scan - Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Dependency audit
        run: |
          pip install pip-audit
          pip-audit -r requirements.txt --strict

      - name: Build Docker image
        run: docker build -t app:${{ github.sha }} .

      - name: Container scan - Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: app:${{ github.sha }}
          exit-code: 1
          severity: CRITICAL,HIGH

      - name: IaC scan - Checkov
        uses: bridgecrewio/checkov-action@v12
        with:
          directory: ./terraform
          soft_fail: false
```

---

## Secrets Management

### Why Not Environment Variables?

Environment variables are readable by any process in the same environment, logged by many tools, and often accidentally printed in CI output.

### HashiCorp Vault

Vault is the industry standard for secrets management.

```python
import hvac

client = hvac.Client(url='http://vault:8200', token=os.environ['VAULT_TOKEN'])

# Read a secret
secret = client.secrets.kv.v2.read_secret_version(
    path='myapp/database',
    mount_point='secret'
)
db_password = secret['data']['data']['password']
```

### AWS Secrets Manager

```python
import boto3, json

client = boto3.client('secretsmanager', region_name='ap-southeast-2')
secret = json.loads(
    client.get_secret_value(SecretId='prod/myapp/db')['SecretString']
)
db_password = secret['password']
```

In CI, use GitHub Actions secrets for CI credentials, but fetch application secrets from Vault or Secrets Manager at runtime.

---

## IaC Scanning with tfsec and Checkov

Both tools scan Terraform for security misconfigurations. They overlap but have differences.

```bash
# tfsec
tfsec ./terraform

# Checkov
checkov -d ./terraform --framework terraform

# Common findings both catch:
# - S3 buckets without encryption
# - S3 buckets with public access
# - Security groups with 0.0.0.0/0 ingress on sensitive ports
# - CloudTrail not enabled
# - RDS without encryption at rest
# - Lambda without VPC
```

---

## What Each Tool Catches

| Tool | What it finds | What it misses |
|------|--------------|----------------|
| Bandit | Python security anti-patterns (eval, subprocess shell=True) | Logic flaws, business logic bugs |
| Gitleaks | Hardcoded secrets and API keys | Secrets passed as parameters |
| pip-audit | CVEs in Python dependencies | Zero-days, logic issues |
| Trivy | OS and library CVEs in containers | Runtime configuration issues |
| Checkov | IaC misconfigurations | Runtime misconfigurations |
| DAST | Runtime web vulnerabilities (XSS, SQLi) | Source-level issues |

No single tool catches everything. Layered scanning is essential.

---

## Key Things to Remember

- Fast pipelines get used; slow pipelines get bypassed
- Each security gate should fail fast and give actionable output
- SAST has false positives - tune severity thresholds to avoid alert fatigue
- Secrets in git are compromised even after deletion - rotate immediately
- DevSecOps is not a tool, it is a culture of shared ownership of security

---

## Activity: Store and Retrieve Secrets with Vault

1. Run Vault in dev mode: docker run --cap-add=IPC_LOCK -e VAULT_DEV_ROOT_TOKEN_ID=myroot -p 8200:8200 vault
1. Set the environment: export VAULT_ADDR=http://127.0.0.1:8200 && export VAULT_TOKEN=myroot
1. Store a secret: vault kv put secret/myapp db_password=supersecret api_key=abc123
1. Retrieve it: vault kv get secret/myapp -- then retrieve just one field: vault kv get -field=db_password secret/myapp
1. Write a Python script that retrieves secrets from Vault using the hvac library (pip install hvac) instead of reading from environment variables
1. Create a Vault policy that only allows read access to secret/myapp. Create a token using that policy and verify it cannot write
1. Scan your existing Python projects with detect-secrets (pip install detect-secrets): detect-secrets scan . -- review any findings
1. You know you are done when your Python app reads secrets from Vault and you can explain why this is better than environment variables

---

## Check-in Questions

Write your answers down before your next session.

1. Why is storing secrets in environment variables considered insecure?
2. What is a Vault policy and how does it enforce least privilege?
3. What is dynamic secrets and why is it a security improvement over static credentials?
4. Name three places where developers commonly accidentally commit secrets.
