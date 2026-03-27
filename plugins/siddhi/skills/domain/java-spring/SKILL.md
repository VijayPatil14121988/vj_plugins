---
name: java-spring
description: Use when working with Java/Spring Boot projects - provides conventions for Spring patterns, Gradle/Maven builds, REST APIs, transaction management, and testing with Spring Boot Test
---

# Java/Spring Boot Domain

Domain-specific guidance for Java and Spring Boot development. Auto-activated when `pom.xml` or `build.gradle` is detected.

## When This Activates

Triggered when the project contains:
- `pom.xml` (Maven) or `build.gradle` / `build.gradle.kts` (Gradle)
- Spring Boot dependencies
- Java source files under `src/main/java`

## Spring Boot Patterns

### Dependency Injection
- Use constructor injection (not field injection with `@Autowired`)
- Mark dependencies as `final`
- Use `@RequiredArgsConstructor` (Lombok) or explicit constructor
- Avoid circular dependencies — if detected, refactor the design

```java
// Good
@Service
@RequiredArgsConstructor
public class PatientService {
    private final PatientRepository patientRepository;
    private final ConceptValidator conceptValidator;
}

// Bad
@Service
public class PatientService {
    @Autowired
    private PatientRepository patientRepository;
}
```

### Configuration
- Use `@ConfigurationProperties` for typed config (not raw `@Value`)
- Externalize all environment-specific values
- Use Spring profiles for environment separation (`application-dev.yml`, `application-prod.yml`)

### REST Conventions
- Use proper HTTP methods: GET (read), POST (create), PUT (full update), PATCH (partial update), DELETE (remove)
- Return appropriate status codes: 200 (OK), 201 (Created), 204 (No Content), 400 (Bad Request), 404 (Not Found), 409 (Conflict), 500 (Internal Server Error)
- Use `ResponseEntity<>` for explicit status control
- Validate request bodies with `@Valid` and Bean Validation annotations
- Use consistent error response format across all endpoints

### Transaction Management
- Use `@Transactional` at the service layer, not the controller or repository
- Keep transaction scope narrow — only what needs to be atomic
- Use `@Transactional(readOnly = true)` for read operations
- Be explicit about propagation and isolation when needed

### Exception Handling
- Use `@ControllerAdvice` with `@ExceptionHandler` for global error handling
- Define domain-specific exceptions (not generic RuntimeException)
- Never expose stack traces in API responses
- Log exceptions at the service layer, return clean error responses

## Build Conventions

### Gradle
```groovy
// Use consistent dependency versions via platform/BOM
implementation platform('org.springframework.boot:spring-boot-dependencies:3.x.x')
```

### Maven
```xml
<!-- Use Spring Boot parent for dependency management -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
</parent>
```

## Testing Patterns

### Integration Tests with Spring Boot Test
```java
@SpringBootTest
@Testcontainers
class PatientServiceIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
}
```

### Slice Tests
- `@WebMvcTest` for controller tests (no full context)
- `@DataJpaTest` for repository tests
- `@JsonTest` for serialization tests

### Key Testing Rules
- Testcontainers over H2 — test against real PostgreSQL/MySQL
- No mocking internal services — mock only external boundaries
- Use `@Sql` or test fixtures for database state setup

## Code Review Additions (Tier 3)

When java-spring domain is active, code review adds:
- [ ] Constructor injection used (no field injection)
- [ ] @Transactional at service layer with correct scope
- [ ] REST conventions followed (methods, status codes, error format)
- [ ] No circular dependencies
- [ ] Bean Validation on request DTOs
- [ ] Testcontainers for database tests (not H2)
- [ ] Configuration externalized (no hardcoded values)
