---
name: code-review
description: Use after implementation phase completes or when reviewing any code changes - performs tiered review (quick for small, standard for medium, domain-aware for large tasks) with severity-based findings
---

# Code Review — Tiered

Review implemented code with depth proportional to task size. Auto-selects the review tier based on change scope and domain involvement.

**Announce at start:** "I'm using the Code Review skill — Tier [N] review."

## Tier Selection

| Change Scope | Domain Files Involved? | Tier |
|-------------|----------------------|------|
| Single file, config, bug fix | No | **Tier 1: Quick** |
| Multiple files, new endpoint/method | No | **Tier 2: Standard** |
| Multiple files, any scope | Yes (healthcare, pipeline, AWS) | **Tier 3: Domain-Aware** |
| New service, multi-component | Any | **Tier 3: Domain-Aware** |

## Tier 1: Quick Review

**Checks:**
- [ ] Implementation matches the task spec
- [ ] No obvious bugs, regressions, or logic errors
- [ ] All tests pass (run them, don't assume)
- [ ] No secrets, credentials, or API keys committed
- [ ] No unintended file changes outside task scope

## Tier 2: Standard Review

Everything in Tier 1, plus:
- [ ] Code follows existing project patterns and conventions (check CLAUDE.md)
- [ ] Proper error handling — no swallowed exceptions, meaningful error messages
- [ ] API contract consistency — request/response shapes match spec
- [ ] Appropriate test coverage per the testing matrix
- [ ] YAGNI check — no over-engineering, no unnecessary abstractions
- [ ] No N+1 queries or obvious performance issues
- [ ] Clean commit history — logical, atomic commits

## Tier 3: Domain-Aware Review

Everything in Tier 2, plus domain-specific checks based on detected domains:

### Healthcare Domain Checks
- [ ] PHI is never logged, never in error messages, never in URLs
- [ ] OMOP CDM conventions followed (correct vocabulary_id, concept_id usage)
- [ ] Clinical data quality assertions present in tests
- [ ] HIPAA-safe error messages (no patient data in stack traces)
- [ ] Audit trail for data modifications if applicable

### Java/Spring Boot Checks
- [ ] Spring patterns followed (proper DI, @Configuration usage)
- [ ] Annotations correct and minimal (@Transactional boundaries, @Valid)
- [ ] REST conventions (proper HTTP methods, status codes, error responses)
- [ ] No circular dependencies
- [ ] Connection pool and thread pool settings reasonable

### Data Pipeline Checks
- [ ] Idempotency guaranteed for all consumers
- [ ] Kafka consumer/producer patterns correct (proper deserialization, ack modes)
- [ ] Dead letter queue configured for failed messages
- [ ] Backpressure handling if applicable
- [ ] Ordering guarantees maintained where required

### Cloud/AWS Checks
- [ ] IAM follows least privilege principle
- [ ] Lambda cold start impact considered and mitigated
- [ ] SQS retry policy and DLQ configured
- [ ] S3 lifecycle policies set if applicable
- [ ] No hardcoded AWS credentials or account IDs

### Database Checks
- [ ] Migrations are reversible (UP and DOWN)
- [ ] Indexes exist for query patterns
- [ ] N+1 queries avoided (batch fetching or joins)
- [ ] Connection pool settings appropriate
- [ ] Transactions scoped correctly (not too broad, not too narrow)

## Review Output Format

```markdown
## Review: <Feature/Task Name>
**Tier:** 1 | 2 | 3
**Verdict:** Approved | Changes Requested

### Findings
- [CRITICAL] <Must fix before commit> — <description with file:line reference>
- [IMPORTANT] <Should fix> — <description with file:line reference>
- [SUGGESTION] <Nice to have> — <description>

### Domain Checks (Tier 3 only)
- [PASS] <Check description>
- [FAIL] <Check description> — <what's wrong and how to fix>

### Summary
<1-2 sentence overall assessment>
```

## After Review

### If Approved
```
Code review passed (Tier N). All checks clear.

Changes committed to feature/<branch-name> (N commits).
Ready for your review. Please push when satisfied:
  git push -u origin feature/<branch-name>
```

### If Changes Requested
Fix all CRITICAL findings. IMPORTANT findings should be fixed unless user explicitly defers. Re-run the review after fixes.

## Key Principles

- **Run tests, don't assume** — always execute the test suite during review
- **Be specific** — "Line 42 in UserService.java catches Exception but swallows it" not "error handling could be improved"
- **Severity matters** — CRITICAL blocks, IMPORTANT should fix, SUGGESTION is optional
- **Domain checks are real** — PHI leaks and missing idempotency are CRITICAL, not suggestions
- **No push** — stop after commit, request user to push
