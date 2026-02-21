# 02 — Launching VPC Resources (EC2)

## Overview
Launched EC2 instances into public and private subnets within a custom VPC, configured SSH access with key pairs, and used security group chaining to restrict private server access to only the public server.

## Architecture

```
┌──────────────── VPC (10.0.0.0/16) ────────────────┐
│                                                     │
│  ┌──────────────────┐    ┌──────────────────┐      │
│  │  Public Subnet    │    │  Private Subnet   │     │
│  │  10.0.0.0/24      │    │  10.0.1.0/24      │    │
│  │                   │    │                   │     │
│  │  ┌─────────────┐ │    │  ┌─────────────┐  │    │
│  │  │ Public EC2   │ │    │  │ Private EC2  │  │    │
│  │  │ (web server) │─┼────┼─►│ (backend)   │  │    │
│  │  └─────────────┘ │    │  └─────────────┘  │    │
│  │                   │    │                   │     │
│  │  SG: SSH from     │    │  SG: SSH from     │    │
│  │  anywhere (0.0.0.0)│   │  Public SG only   │    │
│  └────────┬──────────┘    └──────────────────┘     │
│      ┌────┴──────┐                                  │
│      │  Internet │        (No IGW route)            │
│      │  Gateway  │                                  │
│      └───────────┘                                  │
└─────────────────────────────────────────────────────┘
         │
    Internet / SSH
```

## AWS Services Used
- Amazon VPC, Subnets (public + private)
- EC2 (2 instances), Key Pairs (.pem)
- Security Groups (chained)
- VPC Creation Wizard + Resource Map

## What I Did
1. Created a VPC with public and private subnets
2. Generated an SSH key pair (.pem format) for secure instance access
3. Launched a **public EC2 instance** in the public subnet with SSH allowed from anywhere
4. Launched a **private EC2 instance** in the private subnet with SSH allowed **only from the public server's security group**
5. Used VPC Creation Wizard to rapidly spin up a second VPC matching the same architecture

## Key Concepts

### Security Group Chaining
The private server's security group doesn't allow SSH from `0.0.0.0/0` — it only allows SSH from the **public server's security group ID**. This means:
- Only resources attached to the public SG can reach the private server
- No direct internet access to the private server — even if someone has the key pair

### SSH Key Pairs
- **Public key** → installed on the EC2 instance by AWS
- **Private key (.pem)** → downloaded once, kept secure on your machine
- The instance uses the public key to create an encrypted challenge only the private key can answer

### VPC Resource Map
The VPC wizard's visual resource map shows all subnets, route tables, and gateways at a glance — making it easier to verify architecture before creation.

## What I Learned
- Security group chaining is a powerful pattern for isolating backend servers
- The VPC resource map is extremely useful for visualizing and validating architecture
- Two separate VPCs in the same account can share the same CIDR block (they're isolated by default unless peered)
