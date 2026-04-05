---
name: cloud-aws
description: Use when working with AWS services (Lambda, S3, SQS/SNS) - provides domain-specific questions for requirements and review checklists
---

# Cloud/AWS Domain — Pipeline Guidance

Lightweight domain guidance for pipeline stages. Auto-activated when AWS SDK dependencies or infrastructure templates are detected. Deep implementation knowledge lives in the `aws-engineer` agent.

## When This Activates

Triggered when the project contains:
- AWS SDK dependencies (`aws-sdk-*`, `software.amazon.awssdk`)
- SAM templates (`template.yaml`), CDK code, or CloudFormation
- Lambda handler classes
- SQS/SNS/S3 client usage

## Product Owner Questions

When gathering requirements for AWS work:
- Which AWS services are involved?
- Cost implications at expected scale?
- Cold start / latency requirements for Lambda?
- Existing deployment patterns (SAM, CDK, CloudFormation, manual)?
- Any VPC or networking constraints?
- Data encryption requirements (SSE-S3, SSE-KMS)?

## Code Review Checklist

When cloud-aws domain is active, code review adds:
- [ ] IAM follows least privilege (no `*` resources, no `*` actions)
- [ ] No hardcoded AWS credentials or account IDs
- [ ] Lambda cold start impact considered and mitigated
- [ ] SQS visibility timeout greater than expected processing time
- [ ] DLQ configured for async Lambda and SQS consumers
- [ ] S3 server-side encryption enabled
- [ ] Localstack used in integration tests
- [ ] Error handling returns structured responses, not raw exceptions
- [ ] Lambda handlers are thin — business logic in separate testable modules
