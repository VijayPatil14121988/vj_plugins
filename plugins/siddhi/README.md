# Siddhi (सिद्धि)

Siddhi is a disciplined software development workflow for your coding agents. It provides an adaptive pipeline that scales from trivial config fixes to large multi-component features, with deep domain expertise in healthcare data engineering, Java/Spring Boot, data pipelines, and AWS cloud services.

The name comes from Sanskrit — सिद्धि means "mastery through disciplined practice."

## How It Works

When you start building something, Siddhi doesn't jump into writing code. It steps back and discovers what you actually need.

**The Pipeline:**

```
Product Owner → Architecture Doc → Architecture Review → Logical Tasks → Implementation Phase → Code Review
```

**Adaptive Scaling:** Not every task needs the full pipeline.

| Task Size | What Happens |
|-----------|-------------|
| Trivial (config, env var) | Option Zero resolves it → fix → commit |
| Small (bug fix, single file) | Requirements → implement → quick review |
| Medium (new endpoint) | Requirements → architecture doc → implement → standard review |
| Large (new service) | Full pipeline with formal review gates |

## Key Features

- **Option Zero Gate** — before proposing solutions, checks if a simple config change suffices
- **Architecture Docs with Mermaid** — every medium+ task gets a visual design
- **Tiered Code Review** — quick for small, domain-aware for large
- **Integration-First Testing** — Testcontainers over mocks, Localstack for AWS
- **Domain Expertise** — healthcare/OMOP, Java/Spring, Kafka pipelines, AWS
- **No Auto-Push** — stops after commit, you decide when to push
- **Jira Integration** — pulls ticket context for requirements discovery
- **Adaptive Subagents** — parallel for independent tasks, sequential for dependent

## Slash Commands

| Command | Purpose |
|---------|---------|
| `/discover` | Start requirements discovery (Product Owner) |
| `/architect` | Create architecture doc with mermaid diagrams |
| `/implement` | Begin implementation phase |

## Skills

### Pipeline Skills
- `product-owner` — Requirements discovery with Option Zero gate
- `architecture-doc` — System design with mermaid diagrams
- `architecture-review` — Self-review + user approval gate
- `logical-tasks` — Contract-based task decomposition
- `implementation` — Adaptive subagent dispatch
- `code-review` — Tiered review (quick/standard/domain-aware)

### Support Skills
- `using-siddhi` — Session start, skill routing
- `systematic-debugging` — Root cause investigation with domain diagnostics
- `verification` — Evidence-based completion checks

### Domain Skills
- `healthcare` — OMOP CDM, HIPAA/PHI, clinical data quality
- `java-spring` — Spring Boot patterns, Gradle/Maven, REST conventions
- `data-pipelines` — Kafka, ETL idempotency, S3 data processing
- `cloud-aws` — Lambda, SQS/SNS, S3, IAM best practices

## Installation

### Claude Code (Plugin Marketplace)

```bash
/plugin marketplace add <your-marketplace>
/plugin install siddhi@<your-marketplace>
```

## Git Workflow

- Auto-pulls latest on develop/master and creates feature branches
- Logical, atomic commits — one per task
- Clean commit messages (no Co-Authored-By)
- Never pushes — stops after commit, you push when ready

## License

MIT
