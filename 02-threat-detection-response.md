# 02 — Automated Threat Detection & Response

## Goal
Detect security-group ingress changes that expose a service to the internet, create an actionable finding and define a response path.

## Diagram
![Threat detection pipeline](images/diagram-aws-lab-overview.svg)

## Build steps

### 1. Enable CloudTrail logging
CloudTrail records API activity and delivers the trail logs to the versioned S3 log bucket with log-file validation enabled.

![CloudTrail evidence](images/evidence-cloudtrail-s3-logs.webp)

### 2. Match the security-group API event
The EventBridge rule matches the EC2 CloudTrail event `AuthorizeSecurityGroupIngress`.

![EventBridge rule](images/evidence-eventbridge-rule.webp)

### 3. Send the event to Lambda
The matched event invokes `FIN-LAB-Lambda`, which analyzes the security-group rule and writes a structured finding.

![Detection build](images/evidence-detection-build.webp)

### 4. Trigger the detection with an exposed test rule
The test security group was opened to HTTP/80 from `0.0.0.0/0`.

![Test security group](images/evidence-test-security-group.webp)

### 5. Verify the finding and attribution
The Lambda produced a HIGH-severity finding. CloudTrail was then used to investigate actor and endpoint context.

![Finding](images/evidence-detection-finding.webp)

![CloudTrail attribution](images/evidence-cloudtrail-attribution.webp)

## What broke & fix

| Issue | Root cause | Fix / lesson |
|---|---|---|
| Initial SSH/22 test produced no finding | The EventBridge Enhanced builder silently failed to save the intended rule. | Rebuilt the rule with the Advanced builder and retested. A detection that has never fired is a hypothesis. |

## Proof

```json
{
  "finding": "SECURITY_GROUP_OPENED_TO_INTERNET",
  "severity": "HIGH",
  "group_id": "sg-00ee03ff9ea6eda40",
  "actor": "arn:aws:iam::[REDACTED]:root",
  "rules": [{"protocol":"tcp","from_port":80,"to_port":80}]
}
```

**Response runbook**

1. **Triage:** Identify the affected resource, actor and approval status.
2. **Contain:** Revoke unauthorized ingress and preserve the CloudTrail record.
3. **Escalate:** Investigate root-user activity as a separate finding.

## Lab vs production
| Lab | Production / next step |
|---|---|
| Finding recorded in CloudWatch Logs | SNS/Slack notification |
| Manual containment | Auto-revoke unapproved rules; dry run first |
| Basic internet-exposure detection | Tag-based allowlist for legitimate ALB rules |
| `AuthorizeSecurityGroupIngress` coverage | Add `ModifySecurityGroupRules` and IPv6 `::/0` |
| CloudTrail + custom Lambda detection | Add GuardDuty, AWS Config and Security Hub |
| Security-group test scenario | Extend testing to PDF uploads with GuardDuty Malware Protection, quarantine and alerting |
