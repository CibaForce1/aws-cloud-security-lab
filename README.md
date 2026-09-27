# AWS Cloud Security Lab

**Region:** `us-east-1` · **Built:** 19–25 September 2026

A hands-on AWS security lab built manually in the AWS Console. The lab demonstrates network isolation, private AWS service access, least-privilege IAM, security validation, CloudTrail monitoring and event-driven threat detection.

## Evidence pack

### Architecture diagrams

#### 1. AWS Lab Overview
A single view of the secure three-tier architecture and the security-group detection pipeline.

![AWS lab overview](images/diagram-aws-lab-overview.svg)

#### 2. VPC Topology
Shows the six subnets, route-table boundaries, private endpoints, ALB, EC2 and RDS placement.

![VPC topology](images/diagram-vpc-topology.svg)

#### 3. Three-Tier Application
Shows the public ALB, private application tier and private database tier with the key traffic paths.

![Three-tier application](images/diagram-3-tier-application.svg)

### Write-ups

| # | Write-up | Focus |
|---|---|---|
| 01 | [Secure Architecture](01-secure-architecture.md) | Three-tier AWS architecture, private endpoints, access controls and validation |
| 02 | [Threat Detection & Response](02-threat-detection-response.md) | CloudTrail, EventBridge, Lambda, security findings and incident response |

The write-ups place the build evidence directly beside the step it proves.
