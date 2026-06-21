---
name: microservice-defaults
description: Задаёт дефолтные требования к микросервисам в application/ (Kotlin, Spring Boot, jOOQ, generated REST/Kafka DTO, Feign, логирование, покрытие тестами 70%). Сначала читай reference.md, не исследуй repo. Применять при создании, доработке или ревью микросервисов в application/, если пользователь явно не указал иное.
---

# Дефолтные требования к микросервисам

## Когда применять

При любой задаче в `application/<группа>/<сервис>/`, если пользователь **не переопределил** требования в промпте.

Перед реализацией: спецификация в `documentation/specifications/microservices/...` — источник истины для контрактов. Детали паттернов реализации — [reference.md](./reference.md); открывать `application/currency-valuables/currencies-crud` только по явной просьбе или при новом паттерне.

## Порядок работы без повторного исследования

1. Прочитать [reference.md](./reference.md) — паттерны уже зафиксированы, не исследовать repo.
2. Прочитать spec целевого сервиса: `documentation/specifications/microservices/<группа>/<сервис>/`.
3. При необходимости — релевантные ADR из `documentation/ADR/index.md` (только по теме).
4. Если сервис уже существует — читать только его код в `application/<группа>/<сервис>/`.
5. Реализовать по требованиям ниже и rule `application/.cursor/rules/microservice-implementation.mdc`.

---

## Общие требования

По умолчанию, если в задаче явно не указано иное, все микросервисы подчиняются общим требованиям:

1. Стэк: kotlin, spring, spring-boot, kafka, postgres
2. Для доступа к БД используй jOOQ:
    1. используй заранее сгенерированные DTO из implementation("com.vimpishsnorfle.jooq:*")
    2. SQL-запросы в "сыром виде" запрещены, всегда используй jOOQ-маппинги;
    3. Запросы на основе маппингов должны быть максимально приближены к SQL и везде, где это это вообще возможно, следует избегать сборки запроса кодом из кусков.
3. Так же используй заранее сгенерированные DTO и REST-интерфейсы вебсервисов из implementation("com.vimpishsnorfle.rest:*")
4. Для работы с топиками Kafka используй точно так же заранее сгенерированые DTO implementation("com.vimpishsnorfle.messaging:*")
5. Для выполнения REST-запросов во вне и в соседние сервисы исползуй OpenFeign на основе DTO и интерфейсов из группы `com.vimpishsnorfle.rest`
6. Для логирования используй logback и SLF4J:
    1. сервисы должны в лог записывать все ключевые решения, которые принимает код и полученные ключевые результаты;
    2. из лога должно быть понятно, что сервис сделал и почему.
7. Каждый сервис должен быть покрыт тестами не менее, чем на 70% по строкам.
8. При реализации соблюдай принципам DRY и SRP

---

## Практические ориентиры

Следующие пункты **не заменяют** требования выше — только конкретизируют принятые в репозитории паттерны.

### REST и Feign

- Контроллер: `implements` generated `*Api`, DTO из `com.vimpishsnorfle.rest:*.model.*`.
- Feign-клиент: `interface XxxClient : V1*ControllerApi` — те же DTO, без ручных дубликатов OpenAPI-моделей.

### Kafka

- Consumer: `Consumer<Message<ConcreteDto>>`, тип из `com.vimpishsnorfle.messaging:*`.
- Producer: payload — generated messaging DTO по схеме топика.

### Покрытие тестами (п. 7)

- Плагин `jacoco`, `jacocoTestCoverageVerification` с `minimum = "0.70".toBigDecimal()`, `check` зависит от верификации — см. [reference.md](./reference.md).
- Интеграционные тесты с БД: плагин `com.vimpishsnorfle.database-test`, Testcontainers — см. [reference.md](./reference.md).

### Логирование (п. 6)

- `private val log = LoggerFactory.getLogger(javaClass)` (или аналог в Kotlin).
- Уровни: `info` — ключевые решения и итоги; `debug` — входные данные и промежуточные шаги; `warn`/`error` — отклонения и сбои с контекстом (идентификаторы сущностей, причина).

### Исключения из дефолтов

Если задачу **нельзя** выполнить в рамках требований (нет артефакта, нет таблицы в jOOQ и т.д.) и пользователь **не разрешил** отступление — остановиться и сообщить, что нужно: доработка схемы/миграции/артефакта или явное разрешение в промпте.
