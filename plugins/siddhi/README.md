# Siddhi (सिद्धि)

Siddhi is a disciplined software development workflow with 28 specialized agents. It provides an adaptive pipeline that scales from trivial config fixes to large multi-component features, dispatching the right agent at every stage.

The name comes from Sanskrit — सिद्धि means "mastery through disciplined practice."

## How It Works

When you start building something, Siddhi doesn't jump into writing code. It steps back and discovers what you actually need — then dispatches specialized agents to do the work.

**The Pipeline:**

```
Product Owner → Architecture Doc → Architecture Review → Logical Tasks → Implementation Phase → Code Review
                     ↓                   ↓                                      ↓                    ↓
                 architect           arch-reviewer                       agent-selector         multi-reviewer
                  agent                agent                          picks specialized          dispatch
                                                                    implementer agents
```

**Adaptive Scaling:** Not every task needs the full pipeline.

| Task Size | What Happens |
|-----------|-------------|
| **Trivial** (config, env var, typo) | Option Zero resolves it → fix → commit |
| **Small** (bug fix, single file) | Product Owner → implement (with agent) → quick review → commit |
| **Medium** (new endpoint, method) | Product Owner → Architecture Doc (architect agent) → implement (with agents) → standard review → commit |
| **Large** (new service, multi-component) | Full pipeline with formal review gates and multi-reviewer dispatch |

## Agent Catalog

### Implementers (19)

Implementers build things. The agent selector picks based on task file patterns.

**Backend**
- `spring-boot-engineer` — Java/Spring Boot services, REST APIs, Gradle/Maven
- `python-fastapi-developer` — FastAPI, async Python, Pydantic, SQLAlchemy
- `golang-developer` — Go services, gRPC, concurrency patterns
- `nodejs-developer` — Node.js, Express, NestJS, TypeScript backend
- `dotnet-developer` — .NET/C#, ASP.NET Core, Entity Framework

**Frontend**
- `react-developer` — React, hooks, state management, component libraries
- `typescript-developer` — TypeScript-first development, strict typing, generics
- `nextjs-developer` — Next.js, SSR/SSG, App Router, server components

**Data**
- `kafka-engineer` — Kafka producers/consumers, stream processing, Kafka Streams
- `sql-developer` — SQL queries, schema design, migrations, query optimization
- `data-pipeline-engineer` — ETL/ELT pipelines, idempotency, data quality
- `ml-engineer` — ML model integration, feature engineering, model serving

**Infrastructure**
- `terraform-engineer` — Terraform modules, state management, provider patterns
- `docker-engineer` — Dockerfiles, multi-stage builds, Compose, container optimization
- `kubernetes-engineer` — K8s manifests, Helm charts, operators, resource management
- `aws-engineer` — AWS services, Lambda, SQS/SNS, S3, IAM, CDK/SAM

**Mobile**
- `react-native-developer` — React Native, Expo, cross-platform mobile
- `flutter-developer` — Flutter, Dart, platform channels, widget composition

**Domain**
- `healthcare-engineer` — OMOP CDM, FHIR, HIPAA/PHI, clinical data quality

### Reviewers (4)

Reviewers are dispatched based on task size and risk tier.

| Agent | When Dispatched |
|-------|----------------|
| `code-reviewer` | All reviews (Tier 1+) — correctness, style, tests |
| `architecture-reviewer` | After architecture doc (Tier 2+) — design quality, trade-offs |
| `security-auditor` | High-risk changes (Tier 3) — auth, data handling, secrets |
| `performance-reviewer` | Performance-sensitive paths (Tier 3) — hot paths, query plans |

### Specialists (5)

Specialists are dispatched on-demand within the pipeline.

| Agent | Role |
|-------|------|
| `architect` | Creates architecture docs for medium/large tasks |
| `debugger` | Root cause investigation with systematic hypothesis testing |
| `test-automator` | Integration test generation, Testcontainers, coverage analysis |
| `api-designer` | API contract design, OpenAPI specs, REST/gRPC conventions |
| `database-architect` | Schema design, normalization, indexing strategy, migrations |

## Key Features

- **Option Zero Gate** — before proposing solutions, checks if a simple config change suffices
- **Agent Selector** — picks the right implementer agent(s) based on task file patterns and language/framework signals
- **Pipeline Protocol** — structured stages prevent jumping to code before requirements and design are clear
- **Tiered Code Review** — quick for small changes, domain-aware multi-reviewer dispatch for large ones
- **Integration-First Testing** — Testcontainers over mocks, Localstack for AWS services
- **Domain Expertise via Agents** — healthcare/OMOP, Java/Spring, Kafka pipelines, AWS infrastructure
- **No Auto-Push** — stops after commit; you decide when to push
- **Jira Integration** — pulls ticket context for requirements discovery

## Slash Commands

| Command | Purpose |
|---------|---------|
| `/discover` | Start requirements discovery (Product Owner skill) |
| `/architect` | Create architecture doc with Mermaid diagrams (architect agent) |

## Skills

### Pipeline Skills
- `product-owner` — Requirements discovery with Option Zero gate
- `architecture-doc` — System design with Mermaid diagrams (dispatches architect agent)
- `architecture-review` — Automated review before user approval gate (dispatches architecture-reviewer agent)
- `logical-tasks` — Contract-based task decomposition
- `implementation` — Agent selector dispatch to specialized implementers
- `code-review` — Tiered review with multi-reviewer dispatch for large tasks

### Support Skills
- `using-siddhi` — Session start, skill routing, agent catalog overview
- `systematic-debugging` — Root cause investigation (dispatches debugger agent)
- `verification` — Evidence-based completion checks (dispatches test-automator agent)

### Domain Checklists
- `healthcare` — OMOP CDM, HIPAA/PHI, clinical data quality checklists
- `java-spring` — Spring Boot patterns, Gradle/Maven, REST conventions
- `data-pipelines` — Kafka, ETL idempotency, S3 data processing
- `cloud-aws` — Lambda, SQS/SNS, S3, IAM best practices

## Git Workflow

- Pulls latest on develop/master before starting; creates feature branches (`feature/<jira-ticket>-<short-description>`)
- Logical, atomic commits — one per task, focused on the "why"
- Clean commit messages (no Co-Authored-By lines)
- Never pushes — stops after commit and prompts you to push when ready

## License

MIT
