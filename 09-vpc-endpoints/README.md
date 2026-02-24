# 09 — VPC Endpoints for Private S3 Access

## Overview

In this project, I designed a secure path from an EC2 instance in a VPC to an S3 bucket using an **S3 Gateway VPC endpoint**, bucket policies, and endpoint policies.
I started with the default pattern (EC2 → Internet Gateway → public S3 endpoint using access keys), then tightened it so that all S3 access is forced through a private VPC endpoint and blocked from everywhere else.

This lab shows how to combine **networking** (routes, endpoints) and **identity** (IAM, bucket policies, endpoint policies) so both the network path and the caller’s identity must line up before S3 will allow access.

---

## Architecture

```text
┌──────────────────────── AWS Cloud ────────────────────────┐
│                                                           │
│  VPC (10.0.0.0/16)                                        │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                    Public Subnet                   │   │
│  │                     10.0.0.0/24                    │   │
│  │                                                     │  │
│  │   ┌────────────────────────────────────────────┐    │  │
│  │   │ EC2 Instance                              │    │  │
│  │   │ - AWS CLI configured                      │    │  │
│  │   │ - Uses access keys                        │    │  │
│  │   └────────────────────────────────────────────┘    │  │
│  │                                                     │  │
│  │  Route Table:                                       │  │
│  │   - 10.0.0.0/16  → local                            │  │
│  │   - 0.0.0.0/0    → Internet Gateway                 │  │
│  │   - com.amazonaws.<region>.s3 (prefix list) →       │  │
│  │         S3 Gateway VPC Endpoint                     │  │
│  └───────────────┬─────────────────────────────────────┘  │
│                  │                                        │
│          ┌───────┴────────┐                               │
│          │ Internet       │                               │
│          │ Gateway (IGW)  │                               │
│          └───────────────┘                               │
│                                                           │
│                   VPC Endpoint (Gateway)                  │
│            ┌────────────────────────────────┐             │
│            │  S3 Gateway VPC Endpoint       │             │
│            │  - Attached to route table     │             │
│            │  - Optional endpoint policy    │             │
│            └────────────────────────────────┘             │
│                             │                            │
│                             ▼                            │
│                    ┌────────────────┐                    │
│                    │  S3 Bucket     │                    │
│                    │  - Strict      │                    │
│                    │    bucket      │                    │
│                    │    policy:     │                    │
│                    │    allow only  │                    │
│                    │    aws:SourceVpce│                  │
│                    └────────────────┘                    │
└───────────────────────────────────────────────────────────┘
```

Initially, the instance reaches S3 via the **Internet Gateway**.
After adding the endpoint and updating the route table and bucket policy, all S3 traffic is forced through the **Gateway VPC endpoint**.

---

## AWS Services Used

- **Amazon VPC** – custom VPC, subnet, route tables, Internet Gateway.
- **EC2** – public instance with AWS CLI used as a jump host to test S3 access.
- **Amazon S3** – bucket for test objects, secured with a strict bucket policy using `aws:SourceVpce`.
- **VPC Gateway Endpoint (S3)** – private path from VPC to S3, integrated with route tables.
- **IAM** – access keys for the lab, plus discussion of IAM roles as best practice for production.

---

## What I Did

1. **Built the baseline architecture**  
   - Created a VPC (`10.0.0.0/16`), public subnet, Internet Gateway, and a public EC2 instance with a basic SSH/EC2 Instance Connect–enabled security group.
   - Created an S3 bucket and uploaded test objects so the instance could interact with real data.

2. **Verified network vs. identity separation**  
   - From EC2, ran `aws s3 ls` and saw it fail due to missing credentials, proving that network reachability alone is not enough to access S3.
   - Created IAM access keys for an admin user, configured them with `aws configure` on the instance, and reran `aws s3 ls` to confirm successful authentication and bucket listing.

3. **Interacted with the S3 bucket over the public internet**  
   - Listed all buckets and then the specific lab bucket with:  
     - `aws s3 ls`  
     - `aws s3 ls s3://vpc-bucket-davidfranco`
   - Created and uploaded a test file from `/tmp` using:  
     - `sudo touch /tmp/nextwork.txt`  
     - `aws s3 cp /tmp/nextwork.txt s3://vpc-bucket-davidfranco` 
   - Confirmed the new object appeared in the bucket listing.

4. **Created an S3 Gateway VPC endpoint**  
   - Added a **Gateway VPC endpoint** for S3 into the VPC.
   - Associated the endpoint with the public subnet’s route table so S3 traffic could be routed privately instead of via the Internet Gateway.

5. **Locked down the S3 bucket with a bucket policy**  
   - Attached a bucket policy that explicitly **denies all S3 actions** unless the request comes from the specific VPC endpoint ID using `aws:SourceVpce`.
   - This made the bucket effectively private to traffic coming through that endpoint, blocking direct console access and any requests still using the public internet path.

6. **Updated the route table to send S3 traffic to the endpoint**  
   - Updated the subnet’s route table to direct the S3 prefix list destination to the S3 Gateway VPC endpoint.
   - Before this change, S3 continued to see requests as coming from the public internet, so the bucket policy returned `AccessDenied` even though the endpoint existed.

7. **Validated private S3 access through the endpoint**  
   - Reran `aws s3 ls s3://vpc-bucket-davidfranco` from the EC2 instance.
   - This time the command succeeded, confirming that routing, endpoint configuration, and the bucket policy were aligned and traffic was flowing over the private endpoint path.

8. **Experimented with endpoint policies**  
   - Edited the VPC endpoint policy JSON and changed the `Effect` from `Allow` to `Deny` for S3 actions. 
   - This immediately blocked the instance from reaching the bucket, even though routing and the bucket policy allowed it, demonstrating that endpoint policies act as an extra authorization layer on the network path.

---

## Key Concepts

### VPC Endpoints and S3 Gateway Endpoints

- A **VPC endpoint** creates a private connection between a VPC and AWS services such as S3 without sending traffic over the public internet. 
- A **Gateway VPC endpoint** for S3 integrates with route tables: traffic to the S3 prefix list is directed to the endpoint target instead of the Internet Gateway.

### Bucket Policies with `aws:SourceVpce`

- Bucket policies are JSON IAM policies attached directly to S3 buckets to control access at the bucket level.
- Using `aws:SourceVpce` in a deny statement allows the bucket to **only** accept requests that come from a specific VPC endpoint, even when callers have otherwise valid IAM permissions.

### Endpoint Policies

- An endpoint policy is a separate JSON policy attached to the VPC endpoint that controls what actions and resources are allowed through that endpoint.
- Even if IAM roles and bucket policies allow an action, the endpoint policy can still block or further scope access, making it a powerful network-level control.

### Access Keys vs. IAM Roles

- **Access keys** give the EC2 instance a long‑lived programmatic identity to call AWS APIs via the CLI; they were used here for learning and clear visibility.
- In production, **IAM roles for EC2** are preferred: they provide temporary, automatically rotated credentials via the instance metadata service, removing the need to store long‑lived secrets on disk.
---

## CLI Commands Used (Examples)

```bash
# Configure AWS CLI credentials
aws configure

# List all buckets
aws s3 ls

# List objects in the lab bucket
aws s3 ls s3://vpc-bucket-davidfranco

# Create a local test file
sudo touch /tmp/nextwork.txt

# Upload file to S3
aws s3 cp /tmp/nextwork.txt s3://vpc-bucket-davidfranco

# Validate S3 access after endpoint + bucket policy
aws s3 ls s3://vpc-bucket-davidfranco
```

---

## What I Learned

- Misaligned route tables and bucket policies can completely lock out access to a bucket even when an endpoint exists, so routing, identity, and resource policies all have to line up.
- S3 Gateway VPC endpoints combined with `aws:SourceVpce`-based bucket policies are a strong pattern for enforcing **private-only** S3 access from a VPC.
- Endpoint policies provide a fast way to shut off or tightly scope what an endpoint can reach without changing IAM roles or bucket policies.
- Having a clear baseline of public-internet access with access keys made it much easier to understand and explain the impact of moving to endpoint‑only private connectivity.
