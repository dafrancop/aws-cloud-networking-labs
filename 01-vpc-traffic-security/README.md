# 01 — VPC Traffic Flow & Security

## Overview
Built a custom VPC from scratch with subnets, route tables, an internet gateway, security groups, and network ACLs to understand how traffic flows through AWS networking layers and how each security control affects connectivity.

## Architecture

```
┌──────────────── VPC (10.0.0.0/16) ────────────────┐
│                                                     │
│  ┌──────────────────┐                               │
│  │   Public Subnet   │                              │
│  │   10.0.0.0/24     │                              │
│  │                   │                              │
│  │  Security Group ──► Allow HTTP (80) inbound      │
│  │  NACL ────────────► Allow/Deny rules at subnet   │
│  └────────┬──────────┘                              │
│           │                                         │
│      ┌────┴──────┐                                  │
│      │  Route    │  0.0.0.0/0 → Internet Gateway    │
│      │  Table    │  10.0.0.0/16 → local             │
│      └────┬──────┘                                  │
│           │                                         │
│    ┌──────┴────────┐                                │
│    │   Internet    │                                │
│    │   Gateway     │                                │
│    └───────────────┘                                │
└─────────────────────────────────────────────────────┘
```

## AWS Services Used
- Amazon VPC
- Subnets (public)
- Route Tables
- Internet Gateway
- Security Groups (resource-level firewall)
- Network ACLs (subnet-level firewall)
- EC2 Global View

## What I Did
1. Created a custom VPC with a public subnet
2. Attached an internet gateway and configured route table with `0.0.0.0/0` → IGW
3. Configured security group to allow HTTP (port 80) inbound
4. Created a custom Network ACL and observed deny-all default behavior
5. Compared default vs. custom NACL behavior
6. Used EC2 Global View to track resources across regions (deployed test resources in sa-east-1)

## Key Concepts

### Route Tables
- Route tables act like GPS for network traffic — they determine where packets go
- A subnet becomes "public" only when its route table has a route to an internet gateway (`0.0.0.0/0 → igw-xxx`)

### Security Groups vs. Network ACLs

| Feature | Security Groups | Network ACLs |
|---------|----------------|-------------|
| Level | Resource (EC2 instance) | Subnet |
| State | Stateful (return traffic auto-allowed) | Stateless (must allow both directions) |
| Default | Deny all inbound, allow all outbound | Allow all (default) or deny all (custom) |
| Rules | Allow rules only | Allow AND deny rules |

### Key Insight
Custom NACLs deny all traffic by default — this caught me off guard. Even if security groups allow traffic, a restrictive NACL at the subnet level silently blocks it. This reinforced the importance of **layered security thinking**.

## What I Learned
- Traffic must pass through both NACLs and security groups — both need correct rules
- Custom NACLs start with deny-all, unlike default NACLs which allow everything
- EC2 Global View is essential for auditing resources across regions and spotting orphaned resources
