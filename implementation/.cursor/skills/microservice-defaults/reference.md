# Справочник паттернов микросервисов

Эталон: `application/currency-valuables/currencies-crud`.  
Спецификация эталона: `documentation/specifications/microservices/currency-valuables/currencies-crud/`.

**Не открывать файлы эталона** для стандартных CRUD-задач — достаточно этого справочника.

---

## Naming нового сервиса

| Элемент | Шаблон | Пример |
|---------|--------|--------|
| Каталог | `application/<группа>/<имя-сервиса>/` | `application/product-factory/references-crud/` |
| Application-класс | `<Group><Service>Application` | `ProductFactoryReferencesCrudApplication` |
| Базовый пакет | `com.vimpishsnorfle.<group>.<service>` | `com.vimpishsnorfle.productFactory.referencesCrud` |

---

## Структура пакетов (эталон)

```
src/main/kotlin/com/vimpishsnorfle/currencyValuables/currenciesCrud/
├── CurrencyValuablesCurrenciesCrudApplication.kt
├── web/           V1CurrencyController.kt, GlobalExceptionHandler.kt
├── service/       CurrencyQueryService.kt, CurrencyMutationService.kt
├── repository/    CurrencyRepository.kt, CurrencySortFields.kt, CurrencyValidityCondition.kt
├── domain/        CurrencyListFilter.kt, CurrencyExceptions.kt
├── mapping/       CurrencyMapper.kt
└── kafka/         CurrencyEventPublisher.kt, ClassifierArchivedKafkaConsumer.kt
```

---

## Файлы-ориентиры

| Тема | Путь от корня workspace |
|------|-------------------------|
| Build | `application/currency-valuables/currencies-crud/build.gradle.kts` |
| Settings | `application/currency-valuables/currencies-crud/settings.gradle.kts` |
| DB test image | `application/currency-valuables/currencies-crud/gradle.properties` |
| Main + Clock bean | `.../CurrencyValuablesCurrenciesCrudApplication.kt` |
| Controller | `.../web/V1CurrencyController.kt` |
| Exception handler | `.../web/GlobalExceptionHandler.kt` |
| Repository | `.../repository/CurrencyRepository.kt` |
| Sort fields | `.../repository/CurrencySortFields.kt` |
| Query service | `.../service/CurrencyQueryService.kt` |
| Mutation service | `.../service/CurrencyMutationService.kt` |
| Mapper | `.../mapping/CurrencyMapper.kt` |
| Kafka consumer | `.../kafka/ClassifierArchivedKafkaConsumer.kt` |
| Kafka producer | `.../kafka/CurrencyEventPublisher.kt` |
| Testcontainers config | `.../TestcontainersConfiguration.kt` |
| Integration test | `.../integration/CurrencyRepositoryIntegrationTest.kt` |
| Context-load test | `.../CurrencyValuablesCurrenciesCrudApplicationTests.kt` |
| Test profile | `src/test/resources/application-test.yaml` |
| Test lib | `libs/gradle/database-test-support/testkit/src/main/kotlin/com/vimpishsnorfle/test/database/PrebakedPostgresTestConfiguration.kt` |
| Flyway (не в сервисе) | `data-schemas/<группа>/<схема>/src/main/sql/` |

---

## Gradle: плагины и версии

```kotlin
plugins {
    kotlin("jvm") version "2.1.20"
    kotlin("plugin.spring") version "2.1.20"
    id("org.springframework.boot") version "3.5.14"
    id("io.spring.dependency-management") version "1.1.7"
    id("com.vimpishsnorfle.database-test") version "0.1.0"
    id("com.vimpishsnorfle.jib") version "0.1.0"
    jacoco
}
```

Java toolchain 21. `extra["springCloudVersion"] = "2025.0.2"`.

Типичные implementation-зависимости:

- `com.vimpishsnorfle.rest:<artifact>:<version>`
- `com.vimpishsnorfle.jooq:<schema-group>:<version>`
- `com.vimpishsnorfle.messaging:<topic-artifact>:<version>`
- `com.vimpishsnorfle.crud:crud-web-error-support:0.1.0`
- `com.vimpishsnorfle.jooq:jooq-query-support:0.1.0`

---

## Gradle: jacoco (minimum 70%)

```kotlin
tasks.withType<Test> {
    useJUnitPlatform()
    maxParallelForks = 1
    jvmArgs("-XX:+EnableDynamicAgentLoading")
    finalizedBy(tasks.jacocoTestReport)
}

tasks.jacocoTestCoverageVerification {
    dependsOn(tasks.jacocoTestReport)
    violationRules {
        rule {
            limit { minimum = "0.70".toBigDecimal() }
        }
    }
}

tasks.check { dependsOn(tasks.jacocoTestCoverageVerification) }
```

---

## gradle.properties для database-test

```properties
databaseImage=vimpishsnorfle/<schema-group>-postgres
databaseImageTag=<version>
```

Тег образа совпадает с версией jOOQ-артефакта схемы.

---

## Application-класс

```kotlin
@SpringBootApplication
class CurrencyValuablesCurrenciesCrudApplication {
    @Bean
    fun clock(): Clock = Clock.systemUTC()
}
```

При Feign к соседним сервисам — добавить `@EnableFeignClients`.

---

## Controller

```kotlin
@RestController
class V1CurrencyController(
    private val queryService: CurrencyQueryService,
    private val mutationService: CurrencyMutationService,
    private val request: HttpServletRequest,
) : V1CurrencyControllerApi {
    // override методы Api → делегирование в service
}
```

API-интерфейс: `com.vimpishsnorfle.rest.<group>.<service>.api.V1*ControllerApi`  
DTO: `com.vimpishsnorfle.rest.<group>.<service>.model.*`

---

## GlobalExceptionHandler

```kotlin
@RestControllerAdvice
class GlobalExceptionHandler : CrudExceptionHandlerSupport<InternalServerError>() {
    override fun internalServerError(message: String, stackTrace: String): InternalServerError =
        InternalServerError(message, stackTrace)
}
```

`InternalServerError` — generated model того же REST-артефакта.

---

## Repository (jOOQ)

```kotlin
import com.vimpishsnorfle.jooq.currencyValuables.tables.Currencies.CURRENCIES
import com.vimpishsnorfle.jooq.currencyValuables.tables.records.CurrenciesRecord
import com.vimpishsnorfle.jooq.query.applyListSort
import com.vimpishsnorfle.jooq.query.ClockSupport

@Repository
class CurrencyRepository(
    private val dsl: DSLContext,
    private val clock: Clock,
) {
    fun findByUuid(uuid: UUID): CurrenciesRecord? =
        dsl.selectFrom(CURRENCIES)
            .where(CURRENCIES.UUID.eq(uuid))
            .and(CURRENCIES.DELETED_AT.isNull)
            .fetchOne()
}
```

Правила: static imports таблиц/records; `applyListSort` для списков; `ClockSupport.now(clock)` для timestamps; sort map в отдельном файле (`*SortFields.kt`); raw SQL запрещён.

---

## Kafka consumer (generated interface)

```kotlin
@Component
class ClassifierArchivedKafkaConsumer(
    private val archiveService: ClassifierArchiveService,
) : CurrenciesClassifierElementArchivedV2Consumer {

    override fun consume(headers: MessageHeaders, payload: MessagePayload) {
        archiveService.handleClassifierArchived(payload)
    }
}
```

Типы из `com.vimpishsnorfle.messaging.<Group>.<Topic>.*`.

---

## TestcontainersConfiguration

```kotlin
@TestConfiguration(proxyBeanMethods = false)
@EnableTestBinder
@Import(PrebakedPostgresTestConfiguration::class)
class TestcontainersConfiguration
```

---

## Integration test

```kotlin
@SpringBootTest
@Import(TestcontainersConfiguration::class)
@ActiveProfiles("test")
class CurrencyRepositoryIntegrationTest {
    @Autowired
    private lateinit var repository: CurrencyRepository
    // ...
}
```

---

## Feign к соседним сервисам

```kotlin
@FeignClient(name = "...", url = "\${...}")
interface SomeServiceClient : V1SomeControllerApi
```

Для внешних API (не platform REST) — отдельный Feign-интерфейс с `@FeignClient` + `FeignClientConfig` (см. `precious-metals-rates-crud`).

---

## Чеклист нового CRUD-сервиса

1. Spec + OpenAPI в `documentation/specifications/microservices/...`
2. Flyway в `data-schemas/.../src/main/sql/` (если новые таблицы)
3. Сгенерированные артефакты `rest`, `jooq`, `messaging` (версии в `build.gradle.kts`)
4. Скелет сервиса: application, web, service, repository, mapping
5. `GlobalExceptionHandler`, `TestcontainersConfiguration`, integration tests
6. jacoco 70% на `check`
