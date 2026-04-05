---
name: java-spring
description: Use when working with Java/Spring Boot projects - provides domain-specific questions for requirements and review checklists
---

# Java/Spring Boot Domain — Pipeline Guidance

Lightweight domain guidance for pipeline stages. Auto-activated when `pom.xml` or `build.gradle` is detected. Deep implementation knowledge lives in the `spring-boot-engineer` agent.

## When This Activates

Triggered when the project contains:
- `pom.xml` (Maven) or `build.gradle` / `build.gradle.kts` (Gradle)
- Spring Boot dependencies
- Java source files under `src/main/java`

## Product Owner Questions

When gathering requirements for Java/Spring Boot work:
- Which Spring modules are involved? (Web, Data JPA, Security, Kafka, etc.)
- What are the transaction boundaries for this feature?
- REST API versioning considerations?
- Existing service patterns to follow in the codebase?
- Any specific Spring profiles or configuration needed?

## Code Review Checklist

When java-spring domain is active, code review adds:
- [ ] Constructor injection used (no field injection with @Autowired)
- [ ] @Transactional at service layer with correct scope and readOnly where applicable
- [ ] REST conventions followed (proper HTTP methods, status codes, error format)
- [ ] No circular dependencies
- [ ] Bean Validation on request DTOs (@Valid + constraints)
- [ ] Testcontainers for database tests (not H2)
- [ ] Configuration externalized (no hardcoded values)
- [ ] Exception handling via @ControllerAdvice with domain-specific exceptions
