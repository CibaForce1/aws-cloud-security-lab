# 01 — Secure Three-Tier AWS Architecture

## Goal
Secure a three-tier AWS application using network isolation, least-privilege access, private endpoints and encryption.

## Diagram
![Three-tier application](images/diagram-3-tier-application.svg)

## Build steps

### 1. Build the VPC and subnet tiers
Created `FIN-LAB-VPC` with public, private-app and private-db subnets across two AZs. Only the public route table has the default route to the internet gateway.

![VPC resource map](images/evidence-vpc-resource-map.webp)

### 2. Deploy the private application instance
The application instance has no public IP and is managed through SSM rather than SSH. IMDSv2 is required and the SSM IAM role is attached.

![Private EC2 evidence](images/evidence-private-ec2.webp)

### 3. Make the application reachable without exposing the instance
The internet-facing ALB accepts HTTP/80 and forwards to the private application on port 8080. The Python service was started with systemd and the target became healthy.

![ALB application proof](images/evidence-alb-app-proof.webp)

### 4. Keep database access private
RDS MySQL is not public, is encrypted, uses IAM DB authentication and is reachable from the application security group on port 3306.

![Private RDS evidence](images/evidence-rds-private.webp)

### 5. Add private AWS service access
SSM and Secrets Manager use interface endpoints; S3 uses a gateway endpoint. The application security group controls HTTPS access to the endpoint security group.

![Network validation](images/evidence-network-validation.webp)

### 6. Apply storage controls
The S3 bucket has Block Public Access and bucket-policy controls. The final validation allowed the intended plain upload and denied the tested KMS-encrypted upload.

![S3 policy test](images/evidence-s3-policy-test.webp)

### 7. Validate intended and denied paths
Connectivity testing and Reachability Analyzer were used to prove the security boundaries.

![Reachability validation](images/evidence-reachability-summary.webp)

## What broke & fix

| Issue | Root cause | Fix |
|---|---|---|
| SSM was not connected | App SG had no HTTPS egress to the endpoint SG. | Added TCP/443 egress to the endpoint SG. |
| Secrets Manager failed at network level, then with `AccessDenied` | No endpoint initially; then no IAM permission. | Added the endpoint and a one-ARN `GetSecretValue` policy. |
| ALB target was unhealthy | Nothing was listening on port 8080. | Created and enabled the `finlab-web` systemd service. |
| S3 policy denied normal uploads | Earlier conditions treated a missing encryption header as a denial. | Iterated the condition logic until plain upload was allowed while the tested KMS upload was denied. |

## Proof

- **Blocked:** direct internet access, EC2 API access and DB port `22`.
- **Open:** DB `3306`, SSM `443`, Secrets Manager `443` and S3 `443`.
- Interface endpoints resolved to private `10.0.x.x` addresses.
- Reachability Analyzer showed `app-to-db-3306` **Reachable** and `igw-to-app-22` **Not reachable**.
- The ALB served the application while the application instance remained private.
- Plain S3 upload succeeded; the tested `--sse aws:kms` upload was denied by the bucket policy.

## Lab vs production
| Lab | Production |
|---|---|
| One EC2 instance | Auto Scaling across two AZs |
| Single-AZ RDS | Multi-AZ RDS |
| HTTP only | ACM certificate, HTTPS and WAF |
| No VPC Flow Logs; backups disabled | VPC Flow Logs and backups with PITR |
| Manual console configuration using root | Terraform, IAM Identity Center and no root usage |
