---
layout: page
title: "Infrastructure as Code with Terraform"
week_number: "55"
release_date: "2027-05-04"
phase_label: "Phase 11"
phase_name: "Advanced DevOps"
phase_color: "#06b6d4"
permalink: /week-55/
---

> Week 55. Infrastructure as Code.
> 
> Terraform lets you define infrastructure (servers, databases, networking, IAM roles) as code files. Instead of clicking around the AWS console, you write HCL, run terraform apply, and your infrastructure exists. This also means infrastructure can be reviewed, versioned, tested, and audited.
> 
> Install Terraform from https://www.terraform.io/downloads before this session.

---

# Infrastructure as Code with Terraform

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Explain what Infrastructure as Code is and why it matters
- Write basic Terraform HCL to provision cloud resources
- Use terraform init, plan, apply, and destroy
- Understand Terraform state and why it must be protected
- Identify the security implications of IaC

---

## What is Infrastructure as Code?

Traditionally, infrastructure was provisioned by clicking around cloud consoles or running ad-hoc scripts. IaC means defining infrastructure in code files that are:

- **Version controlled** - track every change in git
- **Reviewable** - code review before provisioning
- **Repeatable** - same code produces identical environments
- **Auditable** - know who changed what and when

Terraform is the most widely used IaC tool. It supports AWS, Azure, GCP, and hundreds of other providers.

---

## Core Concepts

### HCL (HashiCorp Configuration Language)

Terraform uses HCL, a readable configuration language:

```hcl
resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-unique-bucket-name-2027"

  tags = {
    Environment = "production"
    Owner       = "braeden"
  }
}
```

The structure is: `resource "<provider>_<type>" "<local_name>" { ... }`

### Providers

Providers are plugins that let Terraform talk to cloud APIs. Declare them in a `required_providers` block:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-southeast-2"  # Sydney
}
```

### State

Terraform tracks what it has created in a state file (`terraform.tfstate`). This file contains sensitive information including resource IDs and sometimes credentials.

- Never commit state files to git
- Store state remotely in S3 (with encryption and versioning) for team use

---

## The Terraform Workflow

```bash
terraform init      # download providers, initialise backend
terraform plan      # show what will be created/changed/destroyed
terraform apply     # actually make the changes (prompts for confirmation)
terraform destroy   # tear everything down
```

Always read `terraform plan` output before applying. It shows exactly what will happen.

---

## A Complete Example: Secure S3 Bucket

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-southeast-2"
}

resource "aws_s3_bucket" "secure_bucket" {
  bucket = "braeden-secure-bucket-2027"
}

# Block all public access
resource "aws_s3_bucket_public_access_block" "block" {
  bucket = aws_s3_bucket.secure_bucket.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Enable versioning
resource "aws_s3_bucket_versioning" "versioning" {
  bucket = aws_s3_bucket.secure_bucket.id
  versioning_configuration {
    status = "Enabled"
  }
}

# Enable server-side encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "encryption" {
  bucket = aws_s3_bucket.secure_bucket.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# Enable access logging
resource "aws_s3_bucket_logging" "logging" {
  bucket        = aws_s3_bucket.secure_bucket.id
  target_bucket = aws_s3_bucket.secure_bucket.id
  target_prefix = "access-logs/"
}
```

---

## Variables and Outputs

Hard-coding values like bucket names makes code inflexible. Use variables:

```hcl
variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "staging"
}

variable "bucket_name" {
  description = "S3 bucket name"
  type        = string
  # no default = required at apply time
}

resource "aws_s3_bucket" "bucket" {
  bucket = var.bucket_name

  tags = {
    Environment = var.environment
  }
}

output "bucket_arn" {
  description = "ARN of the created bucket"
  value       = aws_s3_bucket.bucket.arn
}
```

Pass variables at apply time:
```bash
terraform apply -var="bucket_name=my-bucket" -var="environment=prod"
```

Or in a `terraform.tfvars` file (do not commit this if it contains secrets):
```hcl
bucket_name = "my-bucket"
environment = "prod"
```

---

## Remote State Backend

For team environments, store state in S3:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "prod/main.tfstate"
    region         = "ap-southeast-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"  # prevents concurrent applies
  }
}
```

---

## Security Considerations

**Never do these:**
- Commit `terraform.tfstate` to git - it contains resource details and sometimes credentials
- Commit `terraform.tfvars` if it contains secrets
- Use `*` in IAM policies in your Terraform
- Store AWS credentials in Terraform files - use environment variables or IAM roles

**Always do these:**
- Enable encryption on state buckets
- Enable versioning on state buckets so you can roll back
- Use workspaces or separate state files for prod/staging
- Run a security scanner (tfsec, Checkov) before applying

---

## Key Things to Remember

- `terraform plan` is read-only - it never changes anything
- State is the source of truth - if it gets out of sync with reality, you have a problem
- Resources reference each other with `resource_type.local_name.attribute` (e.g. `aws_s3_bucket.bucket.id`)
- Destroy is permanent - there is no undo for real cloud infrastructure

---

## Activity: Provision and Destroy Cloud Infrastructure with Terraform

1. Write a main.tf that provisions an S3 bucket on AWS with versioning and public access blocked
1. Run terraform init, terraform plan -- read the output and understand what will be created
1. Run terraform apply -- verify the bucket exists in the AWS console
1. Add a second resource: an IAM policy that allows read-only access to the bucket. Apply again
1. Run terraform destroy -- verify both resources are gone
1. Store your Terraform state in an S3 backend (remote state). Update main.tf with a backend block
1. You know you are done when you can provision and destroy real cloud infrastructure with a single command

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between terraform plan and terraform apply?
2. What is Terraform state and why does it need to be stored securely?
3. What is idempotency and why does it matter for IaC?
4. What is the risk of storing cloud credentials in a Terraform config file?
