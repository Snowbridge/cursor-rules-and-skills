---
name: microservice-defaults
description: Default microservice requirements in application/ (Kotlin, Spring Boot, jOOQ, generated REST/Kafka DTO, Feign, logging, 70% tests). Read reference.md first; do not explore repo. Use for create/update/review in application/ unless user overrides.
---

# Microservice defaults

## When to apply

Tasks in `application/<group>/<service>/` unless user overrides in prompt.

Contracts: `documentation/specifications/microservices/...`. Patterns: [reference.md](./reference.md). Open `currencies-crud` only if asked or new pattern.

## Workflow (no re-exploration)

1. Read [reference.md](./reference.md) — do not explore repo.
2. Read target spec: `documentation/specifications/microservices/<group>/<service>/`.
3. ADR from `documentation/ADR/index.md` if needed (topic only).
4. If service exists — read only `application/<group>/<service>/`.
5. Implement per below + rule `microservice-implementation.mdc`.

## Requirements

1. Stack: kotlin, spring-boot, kafka, postgres
2. jOOQ: generated `com.vimpishsnorfle.jooq:*`; no raw SQL; prefer DSL close to SQL, avoid imperative query assembly
3. REST: generated `com.vimpishsnorfle.rest:*` DTO and `*Api`
4. Kafka: generated `com.vimpishsnorfle.messaging:*`
5. Outbound REST: OpenFeign + `com.vimpishsnorfle.rest` DTO/interfaces
6. Logging (SLF4J/logback): log key decisions and outcomes; traceable what/why
7. Tests: ≥70% line coverage (jacoco)
8. DRY, SRP

## Patterns

**REST/Feign:** controller `implements *Api`; Feign `interface Client : V1*ControllerApi` — no manual OpenAPI duplicates.

**Kafka:** `Consumer<Message<Dto>>` or generated consumer; producer uses messaging DTO.

**Tests:** jacoco 0.70 on `check`; `database-test` + Testcontainers — see [reference.md](./reference.md).

**Logging:** `LoggerFactory.getLogger(javaClass)`; `info` decisions/outcomes, `debug` inputs/steps, `warn`/`error` with entity ids + cause.

## Deviations

If blocked (missing artifact, jOOQ table, etc.) and user did not allow — stop; state need: schema/migration/artifact or explicit prompt permission.
