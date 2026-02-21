# 06 — VPC Monitoring with Flow Logs

## Overview
Built a two-VPC peered network and implemented VPC Flow Logs with CloudWatch Logs Insights to capture, store, and analyze network traffic — identifying top talkers, validating connectivity, and spotting accepted/rejected traffic.

## Architecture

```
┌── VPC-1 (10.1.0.0/16) ──┐     ┌── VPC-2 (10.2.0.0/16) ──┐
│  ┌─────────────────┐     │     │     ┌─────────────────┐  │
│  │  Instance-1      │─────┼─pcx─┼─────│  Instance-2      │  │
│  └─────────────────┘     │     │     └─────────────────┘  │
│           │               │     │                          │
│     VPC Flow Logs ───────►│     │                          │
│           │               │     │                          │
└───────────┼───────────────┘     └──────────────────────────┘
            │
            ▼
   ┌─────────────────┐      ┌──────────────────┐
   │  CloudWatch      │      │   IAM Role        │
   │  Log Group       │◄─────│   (trust policy:  │
   │                  │      │   vpc-flow-logs   │
   │  Logs Insights   │      │   .amazonaws.com) │
   │  ┌────────────┐ │      └──────────────────┘
   │  │ Top 10     │ │
   │  │ Talkers    │ │
   │  │ Query      │ │
   │  └────────────┘ │
   └─────────────────┘
```

## What I Did
1. Created two VPCs (10.1.0.0/16 and 10.2.0.0/16) with public subnets
2. Launched EC2 instances with ICMP-allowing security groups
3. Created a **CloudWatch Log Group** to store flow log data
4. Created an **IAM policy** granting `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents`
5. Created an **IAM role** with a custom trust policy allowing only `vpc-flow-logs.amazonaws.com`
6. Enabled **VPC Flow Logs** on the public subnet (1-minute intervals, all traffic)
7. Set up VPC peering + updated routes for private IP communication
8. Ran ping tests (public and private IPs) to generate traffic
9. Analyzed logs using **CloudWatch Logs Insights** — "Top 10 byte transfers by source/destination IP"

## Key Concepts

### Flow Log Entry Anatomy
```
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
```
Key fields: `srcaddr`, `dstaddr`, `bytes`, `action` (ACCEPT/REJECT)

### IAM for Flow Logs
Flow Logs need an IAM role to write to CloudWatch. The setup requires:
1. **IAM Policy** — permissions to create log streams and put log events
2. **IAM Role** — assumed by `vpc-flow-logs.amazonaws.com` (custom trust policy)
3. Without this, Flow Logs silently fail to deliver data

### Logs Insights Query Example
```
stats sum(bytes) as totalBytes by srcAddr, dstAddr
| sort totalBytes desc
| limit 10
```
This instantly reveals the busiest traffic paths in the network.

## What I Learned
- A simple Logs Insights query can instantly highlight top talkers and potential hotspots
- IAM trust policies are critical — they control which AWS service can assume a role
- Flow Logs capture accepted AND rejected traffic, making them valuable for security auditing
- Public IP pings work via IGW; private IP pings require peering + proper routes
