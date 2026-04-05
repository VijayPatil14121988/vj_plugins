---
name: implementation
description: Use when logical tasks are defined and approved - dispatches subagents adaptively (parallel for independent tasks, sequential for dependent), with checkpoints and guardrails
---

# Implementation Phase — Adaptive Subagent Dispatch

Execute logical tasks through subagents. Analyzes dependencies to dispatch work in parallel where possible, sequential where required. Never auto-starts — waits for user confirmation.

**Announce at start:** "I'm using the Implementation Phase skill to execute the task plan."

<HARD-GATE>
NEVER auto-start subagents. Always present the dispatch plan and wait for explicit user confirmation before executing ANY task.
</HARD-GATE>

## The Process

```dot
digraph implementation {
    "Load task list" [shape=doublecircle];
    "Branch setup" [shape=box];
    "Analyze dependencies" [shape=box];
    "Build dispatch plan" [shape=box];
    "Present plan to user" [shape=box];
    "User approves?" [shape=diamond];
    "Execute batch" [shape=box];
    "Checkpoint" [shape=diamond];
    "All batches done?" [shape=diamond];
    "Invoke code-review" [shape=doublecircle];
    "Fix issues" [shape=box];
    "Escalate to user" [shape=box];

    "Load task list" -> "Branch setup";
    "Branch setup" -> "Analyze dependencies";
    "Analyze dependencies" -> "Build dispatch plan";
    "Build dispatch plan" -> "Present plan to user";
    "Present plan to user" -> "User approves?";
    "User approves?" -> "Present plan to user" [label="no, revise"];
    "User approves?" -> "Execute batch" [label="yes"];
    "Execute batch" -> "Checkpoint";
    "Checkpoint" -> "All batches done?" [label="pass"];
    "Checkpoint" -> "Fix issues" [label="fail"];
    "Fix issues" -> "Checkpoint" [label="retry"];
    "Fix issues" -> "Escalate to user" [label="can't fix"];
    "All batches done?" -> "Execute batch" [label="no, next batch"];
    "All batches done?" -> "Invoke code-review" [label="yes"];
}
```

## Step 1: Branch Setup (BEFORE any code work)

<HARD-GATE>
Branch setup MUST happen before any subagent is dispatched or any code is written. This is the first action after loading the task list.
</HARD-GATE>

1. **Check current branch**: `git branch --show-current`
2. **If on `develop`, `master`, or `main`**:
   - Pull latest: `git pull origin <branch>`
   - Create feature branch: `git checkout -b feature/<jira-ticket>-<short-description>`
3. **If already on a feature branch**: Verify it's up to date, continue working on it
4. **Confirm to user**: "Created branch `feature/<name>` from latest `develop`. Ready to proceed."

```
Branch setup:
  Current branch: develop
  Action: Pulled latest → Created feature/PROJ-123-add-patient-api

Ready to present dispatch plan.
```

## Step 2: Dispatch Plan Presentation

Before any work begins, present the plan with agent assignments:

```
Ready to execute:
  Batch 1 (parallel):
    Task 1 (DB Migration)    → sql-developer
    Task 4 (Kafka Consumer)  → kafka-engineer
    Task 5 (S3 Service)      → aws-engineer
  Batch 2 (sequential):
    Task 2 (Entity Classes)  → spring-boot-engineer
    Task 3 (Repository Layer) → spring-boot-engineer
  Batch 3 (after all):
    Task 6 (Integration Tests) → test-automator

Each agent will:
- Read CLAUDE.md, architecture doc, and their task spec
- Follow the Siddhi protocol (report DONE/BLOCKED/ARCHITECTURE_ISSUE)
- Commit after completing their task

Override any agent assignment? Proceed? [y/n]
```

## Agent Selection

After analyzing dependencies, select the right agent for each task:

### Selection Rules

| File Pattern / Signal | Agent |
|---|---|
| `*.java` + Spring annotations | `spring-boot-engineer` |
| `*.py` + FastAPI/Pydantic | `python-fastapi-developer` |
| `*.go` / `go.mod` | `golang-developer` |
| `*.js/*.ts` + Express/Nest/Node | `nodejs-developer` |
| `*.cs` / `*.csproj` | `dotnet-developer` |
| `*.tsx/*.jsx` + React | `react-developer` |
| `*.ts` (non-React) | `typescript-developer` |
| `next.config.*` / app router | `nextjs-developer` |
| Kafka references | `kafka-engineer` |
| SQL migrations / schema | `sql-developer` |
| ETL / Spark / Airflow | `data-pipeline-engineer` |
| ML model / training | `ml-engineer` |
| `*.tf` / `*.tfvars` | `terraform-engineer` |
| `Dockerfile` / `compose` | `docker-engineer` |
| K8s YAML / Helm | `kubernetes-engineer` |
| AWS SDK / SAM / CDK | `aws-engineer` |
| React Native structure | `react-native-developer` |
| `*.dart` / `pubspec.yaml` | `flutter-developer` |
| OMOP / PHI / clinical | `healthcare-engineer` |
| No strong signal | Language-specific fallback or prompt user to choose |

### Selection Process

1. Read the task spec: file paths, contract, constraints, domain tags
2. Match file patterns and domain signals against the table above
3. If multiple agents match, prefer the more specific one (e.g., `nextjs-developer` over `react-developer` for Next.js projects)
4. If no strong signal, use the dominant language's agent as fallback
5. Include the agent assignment in the dispatch plan for user review

## Subagent Dispatch

Since agents already embed the Siddhi protocol, the dispatch context is minimal:

```
ARCHITECTURE DOC: [path to the architecture doc]
YOUR TASK: [full task spec from logical-tasks — contract, acceptance criteria, constraints, file paths]
TESTING APPROACH: [testing strategy for this task type from the testing matrix]
```

The agent already knows to read CLAUDE.md, follow git rules, and report status. Do not duplicate those instructions.

## Dependency Analysis Rules

- Tasks with no unresolved dependencies → **parallel** (dispatch as concurrent subagents)
- Tasks with dependencies → **sequential** (wait for blockers to complete)
- If a task's dependency fails → **skip** that task and report

## Checkpoints

After each task or parallel batch completes:

1. **Compilation check**: Does the code compile/build?
2. **Test check**: Do all tests pass (existing + new)?
3. **Scope check**: Did the subagent only modify files in its task scope?
4. **Convention check**: Does the code follow CLAUDE.md conventions?

### On Checkpoint Failure
- Agent attempts to fix the issue (one attempt)
- If fix succeeds → continue to next batch
- If fix fails → **stop and escalate to user** with clear error description

## Handling Subagent Reports

| Status | Action |
|--------|--------|
| **DONE** | Run checkpoint, proceed if pass |
| **DONE_WITH_CONCERNS** | Review concerns, run checkpoint, flag concerns to user if significant |
| **BLOCKED** | Stop pipeline, escalate to user with blocker details |
| **ARCHITECTURE_ISSUE** | Stop pipeline, present the issue, ask user if architecture doc needs revision |

## Completion

After all batches complete and checkpoints pass:

```
Implementation Phase complete — N tasks executed successfully.

Summary:
- [Task 1]: Completed — [brief description]
- [Task 2]: Completed — [brief description]
...

Invoking Code Review skill.
```

Then invoke the `siddhi:code-review` skill.

## Key Principles

- **Never auto-start** — always get user confirmation
- **CLAUDE.md first** — every subagent reads project conventions
- **Architecture doc is law** — subagents follow it, don't improvise
- **Escalate, don't improvise** — when blocked or confused, stop and ask
- **Checkpoints catch drift** — verify after every batch
