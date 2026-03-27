---
name: verification
description: Use before claiming work is complete, fixed, or passing - requires running verification commands and confirming output before making any success claims
---

# Verification Before Completion

Run verification and confirm output before claiming anything works. Evidence before assertions.

<HARD-GATE>
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE.

Never say "tests pass", "build succeeds", "fix works", or "implementation complete" unless you have run the relevant commands IN THIS SESSION and can cite the output. "I believe" and "should work" are not evidence.
</HARD-GATE>

## The Verification Gate

Before any completion claim, run through this gate:

### 1. Run Tests
```bash
# Run the full test suite or relevant subset
./gradlew test  # or mvn test, pytest, npm test, etc.
```
**Required evidence:** Test output showing pass/fail counts and zero failures.

### 2. Run Build
```bash
# Verify the project compiles
./gradlew build  # or mvn compile, etc.
```
**Required evidence:** Build output showing success.

### 3. Check for Regressions
- Did any previously passing tests start failing?
- Did the build introduce new warnings?

### 4. Verify Requirements
- Check each acceptance criterion from the task spec
- Can you demonstrate each one is met with evidence?

### 5. Verify Git State
```bash
git status
git diff --stat
```
- Are all intended changes staged/committed?
- Are there unintended changes?
- No secrets or credentials in the diff?

## Common Failures

| Claim | Required Evidence | Insufficient |
|-------|-------------------|-------------|
| "Tests pass" | Fresh test output from this session | "They passed earlier" |
| "Build succeeds" | Fresh build output | "It compiled when I wrote it" |
| "Bug is fixed" | Failing test now passes + no regressions | "I changed the code" |
| "Feature complete" | All acceptance criteria demonstrated | "I implemented everything" |

## When to Apply

- Before saying "done" or "complete"
- Before committing (verify tests pass first)
- Before claiming a bug is fixed
- After any subagent reports "DONE"
- Before presenting work to the user as finished

## The Bottom Line

If you haven't run it, you don't know it works. Run it. Read the output. Then make your claim.
