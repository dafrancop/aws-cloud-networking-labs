# ☁️ AWS Cloud Networking Labs

Hands-on AWS networking and security projects covering VPC design, traffic security, peering, monitoring, IAM, and S3 — built as part of my journey toward **AWS Solutions Architect – Associate** certification.

**Author:** David Franco | IT Consultant | [LinkedIn](https://linkedin.com/in/dfrancop)

---

## Projects

| # | Project | Key Concepts |
|---|---------|-------------|
| 01 | [VPC Traffic Flow & Security](01-vpc-traffic-security/) | Route tables, security groups, NACLs, internet gateways |
| 02 | [Launching VPC Resources](02-launching-vpc-resources/) | EC2, SSH key pairs, public/private server isolation, security group chaining |
| 03 | [Testing VPC Connectivity](03-testing-vpc-connectivity/) | SSH, ping, curl, ICMP troubleshooting, layered security debugging |
| 04 | [Creating a Private Subnet](04-private-subnet/) | Private subnets, dedicated route tables, deny-all custom NACLs |
| 05 | [VPC Peering](05-vpc-peering/) | Multi-VPC architecture, peering connections, cross-VPC routing |
| 06 | [VPC Monitoring with Flow Logs](06-vpc-monitoring-flow-logs/) | VPC Flow Logs, CloudWatch Logs Insights, IAM roles for logging |
| 07 | [Access S3 from a VPC](07-access-s3-from-vpc/) | S3, AWS CLI, access keys, IAM best practices, object uploads |
| 08 | [Cloud Security with AWS IAM](08-cloud-security-iam/) | IAM users, groups, policies (JSON), tag-based access control, policy simulator |

---

## Skills Demonstrated

- **VPC Architecture:** Public/private subnets, multi-AZ, CIDR planning, internet gateways, NAT concepts
- **Network Security:** Security groups (resource-level), NACLs (subnet-level), least-privilege, deny-by-default
- **Cross-VPC Communication:** VPC peering, route table configuration, bidirectional routing
- **Monitoring & Observability:** VPC Flow Logs, CloudWatch log groups, Logs Insights queries (top talkers, traffic analysis)
- **Identity & Access Management:** IAM users, groups, roles, JSON policies, tag-based conditions, policy simulator
- **Storage & Networking:** S3 access from VPC, AWS CLI, access keys vs. IAM roles
- **Troubleshooting:** Systematic layer-by-layer debugging (routes → NACLs → security groups → application)

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                     AWS Cloud                        │
│                                                      │
│  ┌──────────────VPC-1 (10.1.0.0/16)──────────────┐  │
│  │                                                │  │
│  │  ┌─────────────┐    ┌──────────────┐           │  │
│  │  │ Public Sub   │    │ Private Sub   │          │  │
│  │  │ 10.1.0.0/24 │    │ 10.1.1.0/24  │          │  │
│  │  │  ┌───────┐  │    │  ┌────────┐  │          │  │
│  │  │  │  EC2  │  │    │  │  EC2   │  │          │  │
│  │  │  └───────┘  │    │  └────────┘  │          │  │
│  │  └──────┬──────┘    └──────────────┘          │  │
│  │         │                                      │  │
│  │    ┌────┴─────┐                                │  │
│  │    │ Internet │         VPC Peering             │  │
│  │    │ Gateway  │      ◄──────────────►          │  │
│  │    └──────────┘                                │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
│  ┌──────────────VPC-2 (10.2.0.0/16)──────────────┐  │
│  │                                                │  │
│  │  ┌─────────────┐                               │  │
│  │  │ Public Sub   │     ┌────────────────┐       │  │
│  │  │ 10.2.0.0/24 │     │  CloudWatch     │       │  │
│  │  │  ┌───────┐  │     │  Flow Logs      │       │  │
│  │  │  │  EC2  │  │     │  Logs Insights  │       │  │
│  │  │  └───────┘  │     └────────────────┘       │  │
│  │  └──────┬──────┘                               │  │
│  │    ┌────┴─────┐                                │  │
│  │    │ Internet │                                │  │
│  │    │ Gateway  │                                │  │
│  │    └──────────┘                                │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
│  ┌───────────┐    ┌───────────┐                      │
│  │    S3      │    │   IAM     │                      │
│  │  Buckets   │    │  Policies │                      │
│  └───────────┘    └───────────┘                      │
└─────────────────────────────────────────────────────┘
```

---

## How to Use This Repo

Each project folder contains:
- **README.md** — Overview, architecture, step-by-step walkthrough, troubleshooting, and key takeaways
- **diagrams/** — Architecture diagrams and screenshots (add your own console screenshots here)

Projects are ordered by complexity — start at 01 and work through sequentially.

---

## Certifications

- ✅ CompTIA A+
- ✅ Google Project Management
- 🎯 *Pursuing:* AWS Solutions Architect – Associate
