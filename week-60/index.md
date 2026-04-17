---
layout: page
title: "Container Supply Chain Security"
week_number: "60"
release_date: "2027-06-08"
phase_label: "Phase 12"
phase_name: "SecDevOps"
phase_color: "#10b981"
permalink: /week-60/
---

> Week 60. Supply chain security for containers.
> 
> When you pull a Docker image you are trusting the entire supply chain behind it -- the base OS, every installed package, every dependency. Trivy scans images for known CVEs. SBOMs give you a manifest of everything inside. Sigstore/cosign lets you cryptographically sign images.
> 
> Install Trivy: https://aquasecurity.github.io/trivy

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

## Activity: Scan Images and Generate an SBOM

1. Scan a popular Docker image: trivy image python:3.11 -- how many HIGH and CRITICAL CVEs does it have?
1. Scan python:3.11-slim -- compare the CVE count. Understand why slim images are preferred
1. Scan your own recon tool image from Week 41. Fix at least 2 HIGH CVEs by updating base image or pinning a dependency version
1. Generate an SBOM for your image in CycloneDX format: trivy image --format cyclonedx --output sbom.json your-image
1. Add Trivy to your GitHub Actions workflow: fail the build if any CRITICAL CVEs exist
1. Research cosign (https://github.com/sigstore/cosign). Sign your Docker image and verify the signature
1. You know you are done when your CI pipeline scans images, your recon tool image has no CRITICAL CVEs, and you understand what an SBOM is

---

## Check-in Questions

Write your answers down before your next session.

1. What is a software bill of materials (SBOM) and why does it matter?
2. What is the difference between a CVE and a misconfiguration finding in Trivy?
3. Why are base image updates one of the most important security practices for containers?
4. What is Sigstore and what problem does image signing solve?
