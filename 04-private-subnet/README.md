# 04 — Creating a Private Subnet

## Overview
Extended an existing VPC by adding a private subnet with its own dedicated route table and a custom deny-all Network ACL, demonstrating network segmentation and least-privilege network design.

## Architecture

```
┌──────────────── VPC (10.0.0.0/16) ────────────────┐
│                                                     │
│  ┌──────────────────┐    ┌──────────────────┐      │
│  │  Public Subnet    │    │  Private Subnet   │     │
│  │  10.0.0.0/24      │    │  10.0.1.0/24      │    │
│  │                   │    │                   │     │
│  │  Route Table:     │    │  Route Table:     │     │
│  │  0.0.0.0/0 → IGW │    │  10.0.0.0/16 →    │    │
│  │  10.0.0.0/16 →   │    │  local (ONLY)     │     │
│  │  local            │    │  ❌ No IGW route   │    │
│  │                   │    │                   │     │
│  │  NACL: Default    │    │  NACL: Custom     │     │
│  │  (allow all)      │    │  (DENY ALL)       │     │
│  └────────┬──────────┘    └──────────────────┘     │
│      ┌────┴──────┐                                  │
│      │ Internet  │                                  │
│      │ Gateway   │                                  │
│      └───────────┘                                  │
└─────────────────────────────────────────────────────┘
```

## What I Did
1. Created a new subnet (10.0.1.0/24) — non-overlapping CIDR with the public subnet
2. Created a **dedicated private route table** with only the local route (no IGW route)
3. Explicitly associated the private subnet with this new route table
4. Created a **custom Network ACL** that denies all inbound and outbound traffic by default
5. Associated the custom NACL with the private subnet

## Key Concepts

### What Makes a Subnet "Private"
Two things must be true:
1. **No route to an internet gateway** in its route table
2. Optionally, a **restrictive NACL** that only allows explicitly defined traffic

Simply removing the IGW route isn't enough if the default NACL still allows all traffic — resources could still be exposed to overly permissive internal traffic.

### Route Table Isolation
| Route Table | Destination | Target |
|-------------|------------|--------|
| Public | 0.0.0.0/0 | igw-xxx |
| Public | 10.0.0.0/16 | local |
| **Private** | **10.0.0.0/16** | **local** |
| **Private** | **No 0.0.0.0/0** | **—** |

### Default vs. Custom NACLs
- **Default NACL:** allows all inbound/outbound → permissive
- **Custom NACL:** denies all inbound/outbound → locked down, add rules as needed

## What I Learned
- Removing the IGW route alone doesn't fully secure a subnet — the default NACL still allows all internal traffic
- Both **routing** and **ACLs** must be configured together for proper isolation
- This pattern (private subnet + restrictive NACL) is the foundation for multi-tier architectures (web → app → database)
