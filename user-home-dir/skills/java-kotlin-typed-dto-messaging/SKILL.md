---
name: java-kotlin-typed-dto-messaging
description: Type-safe generated DTOs for Kafka, REST, jOOQ in Java/Kotlin/Spring. Use for Kafka/REST/Feign, repositories, jOOQ, OpenAPI, messaging schemas.
---

# Typed contracts (Kafka, REST, jOOQ)

## Rule

Use **type-safe** generated contracts — no raw maps/strings where platform provides types.

**Exception:** user explicitly allows deviation in task.

If not type-safe without permission — **stop**; state blocker and need (permission or schema/artifact work).

## Kafka & REST

Generated artifacts: `com.pitipiwpiw-wiw-wiw.messaging:*`, `com.pitipiwpiw-wiw-wiw.rest:*`.

**Forbidden** (unless allowed):
- `Map<String, Any>` / `Object` / raw JSON as contract body
- `MessageBuilder.withPayload(mapOf(...))`, `StreamBridge.send` with map
- Bypassing generated `*Api` and messaging schemas

**Kafka:** `Consumer<Message<Dto>>` or generated consumer; producer uses messaging DTO. Name clashes across artifacts — separate modules/aliases, not `Map`.

**REST:** controller `implements *Api`; Feign `interface Client : V1*ControllerApi`; no manual OpenAPI duplicates.

## jOOQ

Generated `com.pitipiwpiw-wiw-wiw.jooq:*` — no raw SQL.

**Forbidden:** `dsl.fetch("SELECT...")`, native `@Query`, `jdbcTemplate`, string SQL assembly.

**Required:** `DSLContext` + generated tables/columns/records; DSL close to SQL (`selectFrom`, `where`, `join`).

Missing table → stop; need migration + jOOQ regen, not raw SQL workaround.

## Examples

```kotlin
// BAD
private fun buildEnvelope(...): Map<String, Any> { ... }
dsl.fetch("SELECT * FROM ... WHERE uuid = ?", uuid)

// GOOD
dsl.selectFrom(CURRENCIES).where(CURRENCIES.UUID.eq(uuid)).fetchOne()
fun toAddedMessage(record: CurrenciesClassifierRecord): CurrenciesClassifierElementAddedMessage = ...
```

## Before done

- [ ] Kafka/REST/Feign — generated DTO + `*Api`
- [ ] DB — jOOQ DSL only; no raw SQL
- [ ] No `Map<String, Any>` for contracts in diff
- [ ] Deviations only with explicit task permission
