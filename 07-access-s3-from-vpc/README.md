# 07 — Access S3 from a VPC

## Overview
Launched an EC2 instance inside a custom VPC and configured it to interact with Amazon S3 using the AWS CLI — demonstrating the difference between network connectivity and authenticated access, and exploring access key security best practices.

## Architecture

```
┌──────────────── VPC ──────────────────┐
│                                        │
│  ┌──────────────────┐                  │
│  │  Public Subnet    │                  │
│  │  ┌────────────┐  │                  │
│  │  │   EC2       │  │     ┌────────┐  │
│  │  │  AWS CLI    │──┼────►│  S3     │  │
│  │  │  configured │  │     │ Bucket  │  │
│  │  └────────────┘  │     └────────┘  │
│  └────────┬─────────┘                  │
│      ┌────┴──────┐                     │
│      │ Internet  │                     │
│      │ Gateway   │                     │
│      └───────────┘                     │
└────────────────────────────────────────┘

Authentication: Access Keys (configured via `aws configure`)
Best Practice: Use IAM Roles instead of access keys
```

## What I Did
1. Created a VPC with a single public subnet + internet gateway
2. Launched EC2 instance with SSH access
3. Ran `aws s3 ls` — **failed** (no credentials configured)
4. Created IAM access keys and ran `aws configure` on the instance
5. Ran `aws s3 ls` again — **success** (listed S3 buckets)
6. Created an S3 bucket and uploaded files
7. Used `aws s3 cp` to upload a file from EC2 to S3
8. Verified with `aws s3 ls s3://<bucket-name>`

## Key Concepts

### Network Connectivity ≠ Authenticated Access
Even with full internet access, an EC2 instance can't interact with S3 without proper credentials. This is a fundamental AWS security principle — **identity and network are separate layers**.

### Access Keys
- **Access Key ID** — like a username for API calls
- **Secret Access Key** — like a password (shown once, can't be retrieved)
- Stored in `~/.aws/credentials` after running `aws configure`

### Best Practice: IAM Roles > Access Keys
| Method | Security | Management |
|--------|----------|-----------|
| Access Keys | ⚠️ Long-term, static | Must rotate manually |
| **IAM Roles** | ✅ Temporary credentials | Auto-rotated via instance metadata |

For production: attach an IAM role to the EC2 instance — AWS handles credential rotation automatically.

## CLI Commands Used
```bash
aws s3 ls                                    # List all buckets
aws s3 ls s3://vpc-project-david-franco      # List objects in bucket
sudo touch /tmp/test.txt                     # Create test file
aws s3 cp /tmp/test.txt s3://vpc-project-david-franco  # Upload to S3
aws s3 ls s3://vpc-project-david-franco      # Verify upload
```

## What I Learned
- Having network connectivity doesn't mean you have access — credentials are a separate layer
- `aws configure` stores keys locally, which is fine for learning but risky in production
- IAM roles are the production best practice — they provide temporary, auto-rotated credentials
