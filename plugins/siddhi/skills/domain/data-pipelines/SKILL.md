---
name: data-pipelines
description: Use when working with Kafka, ETL pipelines, S3 data processing, or event-driven architectures - provides domain-specific questions for requirements and review checklists
---

# Data Pipelines Domain — Pipeline Guidance

Lightweight domain guidance for pipeline stages. Auto-activated when Kafka dependencies or pipeline patterns are detected. Deep implementation knowledge lives in the `kafka-engineer` and `data-pipeline-engineer` agents.

## When This Activates

Triggered when the project contains:
- Kafka dependencies (`spring-kafka`, `kafka-clients`)
- S3 SDK dependencies for data processing
- ETL-related code patterns
- Message queue configurations

## Product Owner Questions

When gathering requirements for data pipeline work:
- What ordering guarantees are needed for messages?
- Idempotency requirements for consumers?
- Error handling strategy preference (DLQ, retry, skip)?
- Expected throughput and any backpressure concerns?
- Schema evolution approach (Avro, Protobuf, JSON Schema)?
- Is incremental processing needed or full reprocessing?

## Code Review Checklist

When data-pipelines domain is active, code review adds:
- [ ] Idempotency guaranteed for all consumers
- [ ] Manual offset commit (no auto-commit)
- [ ] DLQ configured for failed messages
- [ ] Backpressure/rate limiting considered
- [ ] Schema evolution is backward-compatible
- [ ] ETL jobs safe to re-run (idempotent)
- [ ] S3 writes are atomic (temp + rename pattern)
- [ ] Record counts validated across pipeline stages
- [ ] No silently dropped messages
