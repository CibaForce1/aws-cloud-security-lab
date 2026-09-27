# 01 — Secure Three-Tier AWS Architecture

## Goal

Build a three-tier AWS application with public ingress at the load balancer while keeping the application and database layers private. The design uses security-group-to-security-group rules, VPC endpoints for private AWS service access, least-privilege IAM and explicit connectivity validation.

## Build steps

### 1. Build the VPC and subnet tiers

Created `FIN-LAB-VPC` (`10.0.0.0/16`) across `us-east-1a` and `us-east-1b` with two public, two private-application and two private-database subnets. Only the public route table has `0.0.0.0/0` to `FIN-LAB-igw`; no NAT gateway is used.

![VPC resource map](images/evidence-vpc-resource-map.webp)

### 2. Keep the application instance private

`FIN-LAB-APP-Private` has no public IPv4 address, no key pair, IMDSv2 required and the `FIN-LAB-SSM-ROLE` role. It is managed through Systems Manager rather than SSH.

![Private EC2 configuration](images/evidence-private-ec2.webp)

### 3. Provide private AWS service access

VPC interface endpoints were used for SSM and Secrets Manager, with endpoint access restricted to HTTPS from the application security group. S3 uses a gateway endpoint. Validation showed direct internet and EC2 API access blocked while the intended private service paths were open.

![Private endpoint and network validation](images/evidence-network-validation.webp)

### 4. Expose only the load balancer publicly

The internet-facing ALB listens on HTTP/80 and forwards to `fin-lab-app-tg` on port `8080`. The application security group allows `8080` only from the ALB security group. The backend Python service is kept on the private instance.

![ALB successfully serving the private application](images/evidence-alb-app-proof.webp)

### 5. Keep the database private

`fin-lab-db` is encrypted RDS MySQL in the private DB subnet group, not public, with IAM DB authentication enabled and the master secret managed by Secrets Manager. The DB security group allows `3306` only from the application security group.

![Private RDS configuration](images/evidence-rds-private.webp)

### 6. Apply storage controls

The S3 bucket has Block Public Access and bucket-policy controls for TLS and upload encryption. The final test allowed the normal upload path and denied the tested `--sse aws:kms` request.

![S3 policy enforcement test](images/evidence-s3-policy-test.webp)

### 7. Validate the boundaries

Reachability Analyzer showed `app-to-db-3306` **Reachable** and `igw-to-app-22` **Not reachable**. Instance-side validation showed the intended blocked and allowed paths.

![Reachability Analyzer validation](images/evidence-reachability-summary.webp)

## What broke & fix

**SSM:** missing HTTPS egress from the app SG to the endpoint SG; TCP/443 egress fixed it.

**Secrets Manager:** first missing the endpoint, then missing IAM permission; endpoint plus a one-ARN `GetSecretValue` policy fixed it.

**ALB:** target was unhealthy because nothing listened on 8080; a `systemd` Python service fixed it.

**S3:** early policy conditions treated a missing encryption header as a denial; the final `Null: false` + `StringNotEquals` logic produced the intended test behavior.

## Security proof

- **Blocked:** direct internet access, EC2 API access and DB `22`.
- **Allowed:** DB `3306`, SSM `443`, Secrets Manager `443` and S3 `443`.
- Endpoint DNS resolved to private `10.0.x.x` addresses.
- The ALB served the application without a public IP on the EC2 instance.

## Lab vs production

| Lab | Production hardening |
|---|---|
| One EC2 instance | Auto Scaling across two AZs |
| RDS lab topology | Multi-AZ database configuration |
| HTTP-only ALB | ACM TLS, HTTPS and WAF |
| No VPC Flow Logs in the lab | VPC Flow Logs, backups and PITR |
| Manual console configuration | Infrastructure as code and centralized identity |
