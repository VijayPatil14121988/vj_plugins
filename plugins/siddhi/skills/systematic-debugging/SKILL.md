---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior - enforces root cause investigation before proposing fixes, with domain-aware diagnostic steps
---

# Systematic Debugging

Investigate before fixing. No fixes without understanding the root cause.

**Announce at start:** "I'm using the Systematic Debugging skill to investigate this issue."

<HARD-GATE>
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST.

Do not propose a fix, write a patch, or suggest a workaround until you have identified the root cause with evidence. This is non-negotiable.
</HARD-GATE>

## The Four Phases

### Phase 1: Root Cause Investigation

1. **Reproduce the issue** — run the failing test/command, see the exact error
2. **Read the error carefully** — stack traces, error messages, log output
3. **Trace the code path** — follow the execution from entry point to failure
4. **Check recent changes** — `git log`, `git diff` to see what changed
5. **Form a hypothesis** — "I believe X is failing because Y"

### Phase 2: Domain-Aware Diagnostics

Based on the error context, apply domain-specific investigation:

**Healthcare/OMOP:**
- Check concept_id mappings — wrong vocabulary_id?
- Check ETL transformation logic — data type mismatch?
- Check OMOP CDM version compatibility

**Java/Spring Boot:**
- Check Spring context — missing beans, circular dependencies?
- Check transaction boundaries — uncommitted data, isolation level?
- Check serialization — Jackson mappings, DTO mismatches?
- Check application.yml / profiles — wrong environment config?

**Data Pipelines:**
- Check message format — schema evolution issue?
- Check consumer offset — reprocessing or skipping messages?
- Check idempotency — duplicate processing side effects?
- Check ordering — out-of-order messages causing logic errors?

**Database:**
- Check migrations — failed or out-of-order migration?
- Check query plans — EXPLAIN ANALYZE for slow queries
- Check constraints — FK violations, unique constraint conflicts?
- Check connection pool — exhausted connections, timeout?

**AWS:**
- Check IAM permissions — AccessDenied?
- Check Lambda logs — CloudWatch for cold start / timeout?
- Check SQS — messages in DLQ? Visibility timeout too short?
- Check S3 — permissions, bucket policy, lifecycle rules?

### Phase 3: Hypothesis Testing

1. **State your hypothesis** clearly before testing it
2. **Design a minimal test** that confirms or disproves it
3. **Run the test** and observe the result
4. **If confirmed** → proceed to Phase 4
5. **If disproved** → return to Phase 1 with new evidence, form a new hypothesis

### Phase 4: Fix Implementation

1. **Write a failing test** that captures the bug (if one doesn't exist)
2. **Implement the minimal fix** — fix the root cause, not symptoms
3. **Run all tests** — the new test passes, no regressions
4. **Commit** with a message explaining the root cause and fix

## Agent Dispatch

When debugging is triggered by a checkpoint failure or a complex investigation:

1. Dispatch the `debugger` specialist agent with:
   - Error context: stack traces, logs, failing test output
   - Path to CLAUDE.md and architecture doc (if exists)
   - Recent git changes relevant to the failure
2. Agent follows the four-phase investigation process
3. Agent reports: `ROOT_CAUSE_FOUND`, `ROOT_CAUSE_FOUND_WITH_FIX`, `NEEDS_MORE_INFO`, or `BLOCKED`
4. If `NEEDS_MORE_INFO`: gather the requested information and re-dispatch or investigate manually
5. If `ROOT_CAUSE_FOUND_WITH_FIX`: run checkpoint verification on the fix

For simple, obvious failures (typo, missing import), fix directly without agent dispatch.

## Red Flags

These mean you're doing it wrong:

| Behavior | Problem |
|----------|---------|
| "Let me try changing X" without investigation | Shotgun debugging — stop and investigate |
| Fixing symptoms instead of root cause | The bug will return in a different form |
| "It works now" without understanding why | You got lucky — find the actual cause |
| Changing multiple things at once | You won't know which change fixed it |
| Ignoring test failures to "fix later" | Test failures are evidence — use them |

## Key Principles

- **Reproduce first** — if you can't see it fail, you can't confirm the fix
- **One hypothesis at a time** — don't shotgun debug
- **Evidence over intuition** — stack traces and logs are your source of truth
- **Minimal fix** — change only what's necessary to fix the root cause
- **Regression test** — every bug fix gets a test that would have caught it
