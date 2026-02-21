# 08 — Cloud Security with AWS IAM

## Overview
Implemented tag-based access control using IAM to restrict a new user's permissions so they can manage development EC2 instances but cannot touch production resources — demonstrating least-privilege access patterns.

## Architecture

```
┌──────────── AWS Account ─────────────────────────┐
│                                                    │
│  ┌─────────────┐    ┌────────────────────┐        │
│  │ IAM User     │    │ IAM User Group      │       │
│  │ (new hire)   │───►│ (developers)        │       │
│  └─────────────┘    │                    │        │
│                      │  Attached Policy:  │        │
│                      │  EC2-Dev-Only      │        │
│                      └────────┬───────────┘        │
│                               │                    │
│                    ┌──────────┴──────────┐         │
│                    │  Custom IAM Policy   │         │
│                    │  (JSON)              │         │
│                    │                      │         │
│                    │  ✅ Allow: EC2 actions│         │
│                    │  IF tag Env=dev      │         │
│                    │  ✅ Allow: Describe   │         │
│                    │  ❌ Deny: Create/     │         │
│                    │     Delete tags       │         │
│                    └──────────────────────┘         │
│                                                    │
│  ┌──────────────┐    ┌──────────────┐             │
│  │ EC2 Instance  │    │ EC2 Instance  │            │
│  │ Tag: Env=     │    │ Tag: Env=     │            │
│  │ production    │    │ development   │            │
│  │               │    │               │            │
│  │ ❌ Stop denied │    │ ✅ Stop allowed│            │
│  └──────────────┘    └──────────────┘             │
└────────────────────────────────────────────────────┘
```

## What I Did
1. Launched two EC2 instances and tagged them: `Env=production` and `Env=development`
2. Created a **custom IAM policy (JSON)** with three statements:
   - Allow all EC2 actions on instances tagged `Env=development`
   - Allow Describe actions on all EC2 resources
   - Deny `CreateTags` and `DeleteTags` on all resources
3. Set up an **account alias** for a friendly login URL
4. Created an **IAM user group** and attached the custom policy
5. Created an **IAM user** and added them to the group
6. Logged in as the IAM user and tested:
   - ❌ Stop production instance → **denied**
   - ✅ Stop development instance → **allowed**
7. Validated with the **IAM Policy Simulator**

## Key Concepts

### Tag-Based Access Control (TBAC)
Instead of listing specific resource ARNs, the policy uses **Condition** blocks to check tags:
```json
"Condition": {
    "StringEquals": {
        "ec2:ResourceTag/Env": "development"
    }
}
```
This scales automatically — any new instance tagged `Env=development` is immediately manageable.

### IAM Policy Structure
```
Effect  →  Allow or Deny
Action  →  What operations (e.g., ec2:StopInstances)
Resource → Which resources (ARN or *)
Condition → Under what conditions (e.g., tag value)
```

### Why Deny Tag Changes?
If the user could change tags, they could re-tag the production instance as `Env=development` and bypass the restriction entirely. Denying `CreateTags`/`DeleteTags` closes this loophole.

### IAM Policy Simulator
Safely test policies without actually performing actions on real resources — critical for production environments where testing on live instances is disruptive.

## What I Learned
- Tag-based policies are more scalable than resource-ARN-based policies
- Explicit denies **always** override allows — useful for guardrails
- The deny-tag-changes pattern is essential to prevent privilege escalation
- IAM Policy Simulator should be used before deploying any policy to production
