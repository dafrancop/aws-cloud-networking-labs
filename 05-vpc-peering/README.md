# 05 — VPC Peering

## Overview
Established a VPC peering connection between two isolated VPCs and validated cross-VPC communication using private IP addresses, demonstrating multi-VPC architecture patterns.

## Architecture

```
┌─── VPC-1 (10.1.0.0/16) ───┐     ┌─── VPC-2 (10.2.0.0/16) ───┐
│                             │     │                             │
│  ┌───────────────────┐     │     │     ┌───────────────────┐  │
│  │  Public Subnet     │     │     │     │  Public Subnet     │  │
│  │  ┌──────────────┐ │     │     │     │ ┌──────────────┐  │  │
│  │  │  Instance-1   │ │     │     │     │ │  Instance-2   │  │  │
│  │  │  10.1.x.x    │─┼─────┼─────┼─────┼─│  10.2.x.x    │  │  │
│  │  └──────────────┘ │     │     │     │ └──────────────┘  │  │
│  └───────────────────┘     │     │     └───────────────────┘  │
│                             │     │                             │
│  Route: 10.2.0.0/16 →     │     │  Route: 10.1.0.0/16 →     │
│         pcx-peering         │     │         pcx-peering         │
└─────────────────────────────┘     └─────────────────────────────┘
                    │                         │
                    └──── VPC Peering ────────┘
                         Connection
```

## What I Did
1. Created two VPCs with unique CIDRs (10.1.0.0/16 and 10.2.0.0/16)
2. Launched EC2 instances in each VPC's public subnet
3. Created a VPC peering connection (requester → accepter model)
4. Updated **both** route tables to direct cross-VPC traffic through the peering connection
5. Configured security groups to allow SSH and ICMP
6. Tested with `ping` using private IPs — initial failure, then success after troubleshooting

## Troubleshooting Walkthrough

| Step | Test | Result | Root Cause | Fix |
|------|------|--------|-----------|-----|
| 1 | EC2 Instance Connect | ❌ Failed | No public IP assigned | Allocated Elastic IP |
| 2 | `ping` Instance-2 private IP | ❌ Timeout | No route to peering connection | Added peering routes to both route tables |
| 3 | `ping` Instance-2 private IP | ❌ Still timing out | SG blocking ICMP | Added ICMP inbound rule to Instance-2's SG |
| 4 | `ping` Instance-2 private IP | ✅ Replies received | — | — |

## Key Concepts

### VPC Peering
- A **direct network link** between two VPCs — traffic stays on AWS backbone (no public internet)
- Requires **both sides** to accept: requester initiates, accepter approves
- Each VPC needs **explicit routes** pointing the other's CIDR to the peering connection

### Why Unique CIDRs Matter
Overlapping CIDRs cause routing conflicts — the network can't determine which VPC a packet belongs to. Always plan CIDR blocks ahead of time.

### Elastic IP Addresses
Static public IPs that persist across instance stop/start cycles — unlike auto-assigned public IPs that change on reboot.

## What I Learned
- Peering connections alone don't route traffic — route table entries are mandatory on both sides
- Troubleshooting requires checking multiple layers: public IP → routes → NACLs → security groups → ICMP rules
- Elastic IPs solve the "instance has no public IP" problem for Instance Connect access
