# AWS Cloud Security Lab

**Region:** `us-east-1` · **Built:** 19–25 September 2026

A hands-on AWS security lab built manually in the AWS Console. It secures a three-tier web application (load balancer, application server and database) and then adds monitoring and automated threat detection on top. The lab demonstrates network isolation, private AWS service access, least-privilege access, security validation, CloudTrail monitoring and event-driven threat detection.

## Securing the three-tier application

Only the load balancer is reachable from the internet. Each tier below it accepts traffic solely from the tier above:

- **Public tier:** an internet-facing ALB on HTTP/80 is the single entry point and forwards to the app on port 8080.
- **Application tier:** the EC2 instance sits in private subnets with no public IP and no key pair, and requires IMDSv2. It is managed through SSM instead of SSH. Its security group allows `8080` only from the ALB security group.
- **Database tier:** encrypted RDS MySQL sits in private subnets with no internet route and has IAM DB authentication. Its master secret is managed by Secrets Manager. Its security group allows `3306` only from the application security group.
- **Private AWS access:** SSM, Secrets Manager and S3 are reached through VPC endpoints, so no NAT gateway or internet egress is needed. A least-privilege IAM policy grants `GetSecretValue` on a single secret ARN.
- **Storage:** the S3 bucket has Block Public Access on, and its bucket policy denies non-TLS requests and uploads without the required encryption.
- **Proof:** Reachability Analyzer confirms app→DB `3306` is reachable and internet→app `22` is not. Instance-side tests confirm that direct internet and EC2 API access are blocked.

## Evidence pack

### Write-ups

| # | Write-up | Focus |
|---|---|---|
| 01 | [Secure Architecture](01-secure-architecture.md) | Three-tier AWS architecture, private endpoints, access controls and network validation |
| 02 | [Threat Detection & Response](02-threat-detection-response.md) | CloudTrail, EventBridge, Lambda, detection findings and incident response |

### Architecture diagrams

#### 1. Three-Tier Application Architecture
VPC across two AZs with the public ALB, private application tier, private database tier, route tables, VPC interface endpoints and S3 gateway endpoint.

![Three-tier application architecture](images/diagram-3-tier-architecture.webp)

#### 2. Threat Detection & Response
CloudTrail, EventBridge, Lambda and CloudWatch Logs detection path, with the triage, contain and escalate runbook.

![Threat detection and response](images/diagram-threat-detection-response.webp)
