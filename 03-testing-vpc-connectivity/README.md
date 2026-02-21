# 03 — Testing VPC Connectivity

## Overview
Systematically tested connectivity between public and private EC2 instances and out to the internet using SSH, ping (ICMP), and curl (HTTP) — and debugged failures layer by layer across security groups, NACLs, and route tables.

## Architecture

```
┌──────────────── VPC ──────────────────────────────┐
│                                                    │
│  ┌─────────────────┐    ┌──────────────────┐      │
│  │  Public Subnet   │    │  Private Subnet   │     │
│  │                  │    │                   │     │
│  │  ┌────────────┐ │    │  ┌────────────┐  │     │
│  │  │ Public EC2  │─┼─ping──►│ Private EC2 │ │     │
│  │  │             │─┼─SSH───►│            │  │     │
│  │  └──────┬─────┘ │    │  └────────────┘  │     │
│  │         │        │    │                   │     │
│  │     curl │       │    │  NACL: must allow │     │
│  │     ▼    │       │    │  ICMP from public │     │
│  │  Internet│       │    │  subnet CIDR      │     │
│  └──────┬───┘───────┘    └──────────────────┘     │
│    ┌────┴──────┐                                   │
│    │ Internet  │                                   │
│    │ Gateway   │                                   │
│    └───────────┘                                   │
└────────────────────────────────────────────────────┘
```

## What I Did
1. Connected to Public Server via EC2 Instance Connect
2. Ran `ping <private-server-private-ip>` — **failed** (no ICMP reply)
3. Debugged: discovered private subnet's NACL was blocking ICMP inbound
4. Fixed: added NACL inbound + outbound rules for "All ICMP – IPv4" from public subnet CIDR
5. Fixed: added security group inbound rule for ICMP from Public Server's SG
6. Re-ran ping — **success** (continuous replies)
7. Ran `curl https://learn.nextwork.org/...` from Public Server — **success** (confirmed internet connectivity via IGW)

## Troubleshooting Walkthrough

| Step | Test | Result | Root Cause | Fix |
|------|------|--------|-----------|-----|
| 1 | EC2 Instance Connect to Public Server | ❌ Failed | SG only allowed HTTP, not SSH | Added SSH (port 22) inbound rule |
| 2 | `ping` Private Server (private IP) | ❌ Timeout | Private NACL blocked ICMP | Added ICMP allow rules to NACL + SG |
| 3 | `ping` Private Server (after fix) | ✅ Replies | — | — |
| 4 | `curl example.com` from Public Server | ✅ HTML returned | — | — |

## Key Concepts

### Ping vs. Curl
- **Ping** = ICMP (layer 3) — checks if a machine is reachable
- **Curl** = HTTP/HTTPS (layer 7) — checks if a web application responds
- A successful ping doesn't guarantee curl will work (different protocols, ports, and firewall rules)

### Layered Troubleshooting Order
When connectivity fails, check in this order:
1. **Route tables** — does traffic have a path?
2. **NACLs** — is the subnet allowing the protocol?
3. **Security groups** — is the instance allowing the protocol?
4. **Application** — is the service running and listening?

## What I Learned
- A single misconfigured NACL rule can silently break connectivity even when everything else looks correct
- NACLs are **stateless** — you must allow traffic in both directions (inbound + outbound)
- Always test connectivity at multiple layers (ICMP, HTTP) to isolate where the block is happening
