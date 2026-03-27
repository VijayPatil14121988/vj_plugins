---
name: data-pipelines
description: Use when working with Kafka, ETL pipelines, S3 data processing, or event-driven architectures - provides conventions for idempotency, message handling, error strategies, and pipeline testing
---

# Data Pipelines Domain

Domain-specific guidance for data pipelines and event-driven systems. Auto-activated when Kafka dependencies or pipeline patterns are detected.

## When This Activates

Triggered when the project contains:
- Kafka dependencies (`spring-kafka`, `kafka-clients`)
- S3 SDK dependencies for data processing
- ETL-related code patterns
- Message queue configurations

## Kafka Conventions

### Consumer Patterns
- **Idempotency is mandatory** — processing the same message twice must produce the same result
- Use `enable.auto.commit=false` — commit offsets only after successful processing
- Configure `max.poll.records` based on processing time per record
- Set reasonable `session.timeout.ms` and `max.poll.interval.ms`

### Idempotency Strategies
1. **Deduplication table** — track processed message IDs, skip duplicates
2. **Upsert/merge** — use INSERT ON CONFLICT UPDATE for database writes
3. **Natural idempotency** — design operations to be naturally safe to repeat (e.g., SET value = X, not INCREMENT value)

### Producer Patterns
- Use `acks=all` for critical data
- Set `enable.idempotence=true` for exactly-once semantics
- Use a consistent partitioning key for ordering guarantees within a partition
- Handle `ProducerRecord` callbacks for error detection

### Error Handling
- **Dead Letter Queue (DLQ)**: Route failed messages to a DLQ topic after N retries
- **Retry with backoff**: Use exponential backoff for transient failures
- **Skip and log**: For poison messages that can never be processed, log and skip after DLQ
- **Never silently drop messages** — always log and track failures

### Schema Management
- Use a schema registry (Avro, Protobuf, or JSON Schema) for contract enforcement
- Backward-compatible schema evolution — add optional fields, don't remove/rename
- Version schemas explicitly

## ETL Patterns

### Idempotent Transformations
- Every ETL job should be safe to re-run
- Use INSERT ON CONFLICT or MERGE for target writes
- Track ETL run metadata (run_id, timestamp, source, record counts)
- Support incremental loads with watermarks (last modified timestamp)

### Data Quality
- Validate source data before transformation
- Count records at each stage (source count, transformed count, loaded count)
- Alert on significant count mismatches
- Log rejected/invalid records with reasons

### S3 Data Processing
- Use consistent key naming: `s3://bucket/dataset/year=YYYY/month=MM/day=DD/`
- Process files atomically — write to temp location, then rename/move
- Handle partial files and retries
- Set lifecycle policies for intermediate data

## Testing Pipelines

### Kafka Integration Tests
```java
@SpringBootTest
@EmbeddedKafka(partitions = 1, topics = {"test-topic"})
class KafkaConsumerIntegrationTest {
    // Or use Testcontainers for a real Kafka broker:
    @Container
    static KafkaContainer kafka = new KafkaContainer(
        DockerImageName.parse("confluentinc/cp-kafka:7.5.0")
    );
}
```

### Key Testing Rules
- Test with Testcontainers (real Kafka) for integration tests
- Test idempotency explicitly — send the same message twice, verify single effect
- Test DLQ routing — send a poison message, verify it lands in DLQ
- Test ordering — send ordered messages, verify processing order
- Use representative data fixtures (anonymized if clinical)

## Code Review Additions (Tier 3)

When data-pipelines domain is active, code review adds:
- [ ] Idempotency guaranteed for all consumers
- [ ] Manual offset commit (no auto-commit)
- [ ] DLQ configured for failed messages
- [ ] Backpressure/rate limiting considered
- [ ] Schema evolution is backward-compatible
- [ ] ETL jobs safe to re-run
- [ ] S3 writes are atomic (temp + rename pattern)
- [ ] Record counts validated across pipeline stages
