# AWS Cloud Security Lab

**Region:** `us-east-1` · **Built:** 19–25 September 2026

A hands-on AWS security lab built manually in the AWS Console. The lab demonstrates network isolation, private AWS service access, least-privilege access, security validation, CloudTrail monitoring and event-driven threat detection.

## Evidence pack

### Write-ups

| # | Write-up | Focus |
|---|---|---|
| 01 | [Secure Architecture](01-secure-architecture.md) | Three-tier AWS architecture, private endpoints, access controls and network validation |
| 02 | [Threat Detection & Response](02-threat-detection-response.md) | CloudTrail, EventBridge, Lambda, detection findings and incident response |

### Architecture diagrams

#### 1. AWS Lab Overview
Color/icon-rich overview of the secure application architecture and the threat-detection path.

![AWS lab overview](images/diagram-aws-lab-overview.svg)

#### 2. VPC Topology
Color/icon-rich view of the VPC, subnet tiers, route boundaries, security groups and private AWS service access.

![VPC topology](images/diagram-vpc-topology.svg)

#### 3. Three-Tier Application
Color/icon-rich application-flow view showing the public ALB, private application tier, private database tier and endpoint paths.

![Three-tier application](images/diagram-3-tier-application.svg)
