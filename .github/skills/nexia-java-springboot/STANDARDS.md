# Java 21 + Spring Boot 3.5 — Standards

> **Tier 2 (language-specific) — Skill-local standards.** Extends [Core Standards (Tier 1)](../../standards/core.md) and [Backend Development Standards (Tier 2)](../nexia-backend-development/STANDARDS.md). Core and backend-agnostic standards always take precedence; this file adds Java/Spring Boot–specific rules only.

Java and Spring Boot specific standards for backend development.
These accompany the language-agnostic [backend-development/STANDARDS.md](../nexia-backend-development/STANDARDS.md) and [SKILL.md](../nexia-backend-development/SKILL.md).

---

## Java / Spring Boot Architecture Rules

- **No `@Autowired` on fields** — constructor injection only (`@RequiredArgsConstructor` with `final` fields)
- **`@PreAuthorize`** for authorization — never inline role checks in business logic
- **Lombok**: Use minimally — `@Getter`, `@Builder`, `@Slf4j` preferred over `@Data`
- **No Spring annotations in `domain/` package** — domain layer must be framework-free
- **`@Transactional` at service layer only** — never in controllers or repositories
- **Jakarta Validation** — use `@Valid` / `@NotNull` / `@Size` on request DTOs
- **MapStruct** for DTO mapping — no manual mapping in service layer

---

## Application Configuration Structure

```
src/main/resources/
├── application.yml         ← Common config (no secrets, no env-specific values)
├── application-dev.yml     ← Local dev overrides (H2 / local DB, debug logging)
├── application-prod.yml    ← Production (all sensitive values via env vars)
└── logback-spring.xml      ← Structured JSON logging for prod, console for dev
```

Required MDC fields in `logback-spring.xml`: `traceId`, `spanId`, `userId`, `requestId`

---

## Security Configuration Template

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(AbstractHttpConfigurer::disable)        // stateless API
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**", "/actuator/health").permitAll()
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }
}
```

---

## Maven Project Folder Structure

```
src/main/java/com/{company}/{project}/
├── {module}/
│   ├── domain/         ← Entities, Value Objects, Repository interfaces
│   ├── application/    ← Service interfaces, Use cases, DTOs
│   ├── infrastructure/ ← JPA Repositories, external clients, config
│   └── api/            ← REST Controllers, request/response mappers
├── shared/             ← Cross-cutting: exceptions, utils, audit
└── Application.java
```

---

## Docker Image Template

```dockerfile
FROM eclipse-temurin:21-jre-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
COPY target/app.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Rules:
- Non-root user (`appuser`) — mandatory for security
- Alpine base — minimal attack surface
- No `latest` tags — always pin base image version
