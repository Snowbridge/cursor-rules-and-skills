---
name: java-kotlin-typed-dto-messaging
description: Requires type-safe generated DTOs for Kafka, REST, and jOOQ database access in Java/Kotlin/Spring projects. Use when implementing Kafka/REST/Feign, repositories, jOOQ queries, or when the user mentions messaging schemas, OpenAPI, generated artifacts, or database mappings.
---

# Java/Kotlin + Spring: typed contracts (Kafka, REST, jOOQ)

## Rule

Обязательно использовать **типобезопасные** контракты и маппинги — без «сырых» структур и строк там, где платформа даёт generated-типы.

**Единственное исключение** — если пользователь **прямо в тексте задачи** разрешил отступление.

Если задачу **нельзя решить типобезопасно** и явного разрешения нет — **прервать выполнение** и сообщить, что мешает и что нужно (разрешение или доработка артефакта/схемы).

---

## Kafka и REST

Для **всех** случаев работы с Kafka и REST — **типобезопасные DTO** для сериализации и десериализации передаваемых или получаемых объектов.

Источник: заранее сгенерированные артефакты (`com.vimpishsnorfle.messaging:*`, `com.vimpishsnorfle.rest:*`), не ручная сборка «на лету».

**Запрещено** (без явного разрешения в задаче):
- `Map<String, Any>`, `LinkedHashMap`, `Map<String, Object>` как тело сообщения или REST-контракт
- `MessageBuilder.withPayload(mapOf(...))` / `StreamBridge.send` с map вместо DTO
- `Object`, `Any`, сырой JSON (`String`/`JsonNode`) там, где есть generated-тип
- Обход generated REST API (`*Api`) и messaging-схем

### Kafka

1. Consumer: `Consumer<Message<ConcreteDto>>`, тип из `com.vimpishsnorfle.messaging:*`.
2. Producer: payload — generated messaging DTO по схеме топика.
3. При конфликте одноимённых классов в нескольких messaging-артефактах — **не** сваливаться в `Map`; отдельные модули, alias imports, общий артефакт, или остановка с запросом уточнения.

### REST

1. Controllers: `implements` generated `*Api`, DTO из `com.vimpishsnorfle.rest:*.model.*`.
2. Feign: `interface Client : V1*ControllerApi` — те же request/response DTO.
3. Не дублировать OpenAPI-модели ручными data class.

---

## jOOQ (БД)

Для доступа к БД — **типобезопасные jOOQ-маппинги** на основе сгенерированного артефакта (например `com.vimpishsnorfle.jooq:*`), а не SQL текстом.

**Запрещено** (без явного разрешения в задаче):
- SQL-запросы в «сыром виде»: `dsl.fetch("SELECT ...")`, `execute("UPDATE ...")`, `@Query` с нативным SQL, `jdbcTemplate`, строковая конкатенация SQL
- Динамическая сборка SQL кусками строк вместо jOOQ DSL
- Обход generated `tables.*`, `Keys`, `Records`, `Pojos` там, где jOOQ-артефакт доступен

**Требуется:**
1. `DSLContext` + generated table/column references (`CURRENCIES.UUID.eq(...)`).
2. DTO/Record из jOOQ codegen (`*Record`, `tables.pojos.*`) для чтения и записи.
3. Запросы максимально близки к SQL через DSL (`selectFrom`, `where`, `join`), без лишней императивной сборки условий, если это можно выразить декларативно.

Если jOOQ не покрывает кейс (нет таблицы в артефакте, нужна миграция) — остановиться: нужна миграция + пересборка jOOQ, не raw SQL как обход.

---

## Антипримеры

```kotlin
// BAD — Kafka: map вместо messaging DTO
private fun buildEnvelope(...): Map<String, Any> { ... }

// BAD — jOOQ: сырой SQL
dsl.fetch("SELECT * FROM currency_valuables.currencies WHERE uuid = ?", uuid)

// GOOD — jOOQ
dsl.selectFrom(CURRENCIES).where(CURRENCIES.UUID.eq(uuid)).fetchOne()
```

```kotlin
// GOOD — messaging DTO
fun toAddedMessage(record: CurrenciesClassifierRecord): CurrenciesClassifierElementAddedMessage = ...
```

---

## Проверка перед завершением

- [ ] Kafka produce/consume — generated messaging DTO
- [ ] REST/Feign — generated REST DTO и `*Api`
- [ ] БД — только jOOQ DSL + generated tables/records/pojos; нет raw SQL
- [ ] В diff нет `Map<String, Any>` для контрактов и нет строкового SQL
- [ ] Отступления — только при явном разрешении в задаче
