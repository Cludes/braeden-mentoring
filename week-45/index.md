---
layout: page
title: "Cloud Security Overview"
week_number: "45"
release_date: "2027-02-23"
phase_label: "Phase 9"
phase_name: "DevOps & Cloud Security"
phase_color: "#4cc9f0"
permalink: /week-45/
---

> Week 45. Cloud basics and cloud security.
> 
> Almost every modern company runs on AWS, Azure, or Google Cloud. Security misconfigurations in cloud environments are the most common cause of major data breaches -- exposed S3 buckets, overpermissioned IAM roles, public databases.
> 
> This week you understand the cloud model and learn what security issues to look for.

---

# Cloud Fundamentals

## What Cloud Actually Is

Cloud = other people's computers with good APIs. The value is: pay for what you use, scale on demand, delegate maintenance of physical infrastructure.

---

## Service Models

| Model | You manage | Provider manages | Examples |
|-------|-----------|-----------------|---------|
| IaaS | OS, runtime, app | Hardware, network | EC2, GCE, Azure VMs |
| PaaS | App, data | Everything else | Cloud Run, Heroku, App Engine |
| SaaS | Config only | Everything | Gmail, Salesforce, GitHub |

**Rule:** use the highest abstraction that fits your requirements. PaaS > IaaS for most web apps.

---

## Core Building Blocks

### Compute

```
Virtual Machine (VM)     Rented physical server. Full control, full responsibility.
Container Service        Run Docker containers without managing servers (ECS, Cloud Run).
Serverless Function      Run code in response to events, pay per invocation (Lambda, Cloud Functions).
Kubernetes Service       Managed k8s cluster (EKS, GKE, AKS).
```

### Storage

```
Object Storage    Files as objects with HTTP API. Infinite scale, cheap.
                  Examples: S3, GCS, Azure Blob Storage
                  Use for: static files, backups, ML datasets, logs

Block Storage     Like a hard drive. Attached to a VM.
                  Examples: EBS (AWS), Persistent Disk (GCP)
                  Use for: database data, VM root volumes

File Storage      Shared filesystem. Multiple VMs can mount it.
                  Examples: EFS (AWS), Filestore (GCP)
                  Use for: shared config, legacy apps that need NFS
```

### Networking

```
VPC (Virtual Private Cloud)
  Your private network in the cloud. Other tenants cannot reach it.

Subnet
  A range of IP addresses inside a VPC.
  Public subnet: has route to internet gateway (can receive traffic).
  Private subnet: no route to internet (databases, internal services live here).

Security Group / Firewall Rule
  Stateful firewall for a resource. Allow traffic on specific ports from specific sources.
  Default: deny all. Open only what you need.

Load Balancer
  Distributes traffic across multiple instances.
  Application LB: HTTP/HTTPS, path-based routing.
  Network LB: TCP/UDP, lower latency.

CDN (Content Delivery Network)
  Caches static assets at edge locations globally.
  Examples: CloudFront, Cloud CDN, Azure Front Door.
```

### Identity and Access (IAM)

```
User        A person or service with credentials.
Role        A set of permissions. Assign to users or services.
Policy      Defines what actions are allowed on which resources.

Principle of least privilege: grant only the permissions needed for the task.
Never: root credentials in code, wildcard permissions in production.
```

---

## Regions and Availability Zones

```
Region          A geographic area (e.g. us-east-1, eu-west-2).
                Data stays in the region unless you move it.

Availability Zone (AZ)
                A physically separate data centre within a region.
                Deploy across 2+ AZs to survive a single AZ failure.

Edge Location
                A CDN point of presence, closer to end users.
```

---

## Shared Responsibility Model

```
Provider is responsible for:   Security OF the cloud (hardware, network, physical)
You are responsible for:       Security IN the cloud (IAM, encryption, patching your OS/app)
```

Common mistakes that are your responsibility:
- S3 bucket left public
- EC2 instance with port 22 open to 0.0.0.0/0
- No encryption on database at rest
- IAM role with `*` permissions

---

## Cost Basics

```
On-Demand       Pay by the hour/second. No commitment. Most expensive per unit.
Reserved        1 or 3-year commitment. Up to 70% cheaper.
Spot/Preemptible
                Spare capacity, can be interrupted. Up to 90% cheaper.
                Use for: batch jobs, ML training, anything resumable.
Free Tier       Most providers offer always-free and 12-month free tiers.
```

**Biggest cost mistakes:**
- Leaving large instances running when idle
- Not setting up billing alerts
- Storing logs/data in expensive storage classes without lifecycle policies
- NAT gateway costs for high-egress workloads

---

## Checklist

- [ ] Can explain the difference between IaaS, PaaS, SaaS
- [ ] Know the difference between object, block, and file storage
- [ ] Understand VPC, subnets, security groups
- [ ] Know the shared responsibility model
- [ ] Set up billing alerts before spending any real money

## Next

[AWS Core Services](../intermediate/01_aws-core-services.md)

---

## Activity: AWS Free Tier: Set Up and Secure an S3 Bucket

1. Create a free AWS account at https://aws.amazon.com/free (requires a credit card but will not charge you for free tier usage)
1. Create an S3 bucket. By default, try to make it public. Observe what AWS now warns you about
1. Enable Block Public Access on the bucket. Verify you can no longer access it without authentication
1. Enable S3 bucket versioning and access logging
1. Use the AWS CLI (pip install awscli, then aws configure) to list your buckets: aws s3 ls
1. Search HackerOne's disclosed reports for 'S3 bucket' -- read two real reports about misconfigured buckets
1. You know you are done when your bucket is private, versioning is on, logging is on, and you understand why all three matter

---

## Check-in Questions

Write your answers down before your next session.

1. What is the shared responsibility model in cloud security?
2. Why are exposed S3 buckets such a common security finding?
3. What is IAM and what is the principle of least privilege in that context?
4. What is the difference between IaaS, PaaS, and SaaS?
