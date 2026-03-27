---
name: cloud-aws
description: Use when working with AWS services (Lambda, S3, SQS/SNS) - provides conventions for serverless patterns, IAM security, error handling, and testing with Localstack
---

# Cloud/AWS Domain

Domain-specific guidance for AWS cloud services. Auto-activated when AWS SDK dependencies or infrastructure templates are detected.

## When This Activates

Triggered when the project contains:
- AWS SDK dependencies (`aws-sdk-*`, `software.amazon.awssdk`)
- SAM templates (`template.yaml`), CDK code, or CloudFormation
- Lambda handler classes
- SQS/SNS/S3 client usage

## Lambda Patterns

### Handler Design
- Keep handlers thin — extract business logic into testable service classes
- Handle cold starts: minimize initialization code, use lazy loading for heavy resources
- Set appropriate timeout (default 3s is too low for most use cases)
- Set memory based on workload (more memory = more CPU)

### Cold Start Mitigation
- Initialize SDK clients and DB connections outside the handler method (class level)
- Use provisioned concurrency for latency-sensitive functions
- Keep deployment packages small — minimize dependencies
- Prefer ARM64 (Graviton2) for better price/performance

### Error Handling
- Return structured error responses, not raw exceptions
- Use dead letter queues for async invocations
- Log errors with request context (requestId, traceId)
- Set up CloudWatch alarms for error rates

## S3 Patterns

### Data Operations
- Use multipart upload for files > 100MB
- Use S3 Select for querying large files without full download
- Set appropriate storage class (Standard, IA, Glacier)
- Enable versioning for critical data buckets

### Security
- Never use hardcoded credentials — use IAM roles
- Use bucket policies for access control
- Enable server-side encryption (SSE-S3 or SSE-KMS)
- Block public access unless explicitly required

### Lifecycle
- Set lifecycle rules for temp/intermediate data
- Archive old data to Glacier/Deep Archive
- Delete incomplete multipart uploads automatically

## SQS/SNS Patterns

### SQS Consumer
- Set visibility timeout > expected processing time
- Use long polling (`WaitTimeSeconds: 20`) to reduce empty receives
- Configure DLQ with `maxReceiveCount` (typically 3-5)
- Delete messages only after successful processing

### SQS Producer
- Use `MessageGroupId` for FIFO ordering
- Use `MessageDeduplicationId` for exactly-once in FIFO queues
- Handle `SendMessage` failures with retry

### SNS Publisher
- Use message attributes for filtering
- Use FIFO topics when ordering matters
- Set up DLQ for failed deliveries

## IAM Best Practices

- **Least privilege** — grant only the permissions needed
- **Resource-specific** — use specific ARNs, not `*`
- **No inline policies** — use managed policies
- **No hardcoded credentials** — use IAM roles, environment variables, or Secrets Manager

## Testing with Localstack

### Setup
```java
@Testcontainers
class S3ServiceIntegrationTest {

    @Container
    static LocalStackContainer localstack = new LocalStackContainer(
        DockerImageName.parse("localstack/localstack:3.0")
    )
    .withServices(LocalStackContainer.Service.S3, LocalStackContainer.Service.SQS);

    @BeforeAll
    static void setup() {
        // Create test buckets, queues, etc.
    }
}
```

### Key Testing Rules
- Use Localstack for S3, SQS, SNS, Lambda integration tests
- Test Lambda handlers as plain Java methods (unit) + with Localstack (integration)
- Test SQS consumer error paths — verify DLQ routing
- Test S3 operations with real Localstack S3, not mocks
- Verify IAM-like behavior where Localstack supports it

## Code Review Additions (Tier 3)

When cloud-aws domain is active, code review adds:
- [ ] IAM follows least privilege (no `*` resources)
- [ ] No hardcoded AWS credentials or account IDs
- [ ] Lambda cold start impact considered
- [ ] SQS visibility timeout > processing time
- [ ] DLQ configured for async Lambda and SQS
- [ ] S3 server-side encryption enabled
- [ ] Localstack used in integration tests
- [ ] Error handling returns structured responses, not raw exceptions
