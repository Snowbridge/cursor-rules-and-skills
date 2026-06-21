# Microservice pattern reference

Reference: `application/currency-valuables/currencies-crud`. Spec: `documentation/specifications/microservices/currency-valuables/currencies-crud/`.

**Do not open reference files** for standard CRUD — this doc is enough.

## Naming

| Item | Pattern | Example |
|------|---------|---------|
| Dir | `application/<group>/<service>/` | `application/product-factory/references-crud/` |
| Application | `<Group><Service>Application` | `ProductFactoryReferencesCrudApplication` |
| Package | `com.pitipiwpiw-wiw-wiw.<group>.<service>` | `com.pitipiwpiw-wiw-wiw.productFactory.referencesCrud` |

## Package layout (reference)

```
src/main/kotlin/.../currenciesCrud/
├── CurrencyValuablesCurrenciesCrudApplication.kt
├── web/       V1CurrencyController.kt, GlobalExceptionHandler.kt
├── service/   CurrencyQueryService.kt, CurrencyMutationService.kt
├── repository/ CurrencyRepository.kt, CurrencySortFields.kt, ...
├── domain/    CurrencyListFilter.kt, ...
├── mapping/   CurrencyMapper.kt
└── kafka/     CurrencyEventPublisher.kt, ClassifierArchivedKafkaConsumer.kt
```

## File pointers

| Topic | Path |
|-------|------|
| Build | `application/currency-valuables/currencies-crud/build.gradle.kts` |
| Settings | `.../settings.gradle.kts` |
| DB test image | `.../gradle.properties` |
| Main + Clock | `.../CurrencyValuablesCurrenciesCrudApplication.kt` |
| Controller | `.../web/V1CurrencyController.kt` |
| Exception handler | `.../web/GlobalExceptionHandler.kt` |
| Repository | `.../repository/CurrencyRepository.kt` |
| Kafka consumer/producer | `.../kafka/ClassifierArchivedKafkaConsumer.kt`, `CurrencyEventPublisher.kt` |
| Testcontainers | `.../TestcontainersConfiguration.kt` |
| Integration test | `.../integration/CurrencyRepositoryIntegrationTest.kt` |
| Test lib | `libs/gradle/database-test-support/.../PrebakedPostgresTestConfiguration.kt` |
| Flyway | `data-schemas/<group>/<schema>/src/main/sql/` |

## Gradle plugins

```kotlin
plugins {
    kotlin("jvm") version "2.1.20"
    kotlin("plugin.spring") version "2.1.20"
    id("org.springframework.boot") version "3.5.14"
    id("io.spring.dependency-management") version "1.1.7"
    id("com.pitipiwpiw-wiw-wiw.database-test") version "0.1.0"
    id("com.pitipiwpiw-wiw-wiw.jib") version "0.1.0"
    jacoco
}
```

Java 21. `extra["springCloudVersion"] = "2025.0.2"`.

Typical deps: `com.pitipiwpiw-wiw-wiw.rest:*`, `jooq:*`, `messaging:*`, `crud-web-error-support:0.1.0`, `jooq-query-support:0.1.0`.

## jacoco (70%)

```kotlin
tasks.withType<Test> {
    useJUnitPlatform()
    maxParallelForks = 1
    jvmArgs("-XX:+EnableDynamicAgentLoading")
    finalizedBy(tasks.jacocoTestReport)
}
tasks.jacocoTestCoverageVerification {
    dependsOn(tasks.jacocoTestReport)
    violationRules { rule { limit { minimum = "0.70".toBigDecimal() } } }
}
tasks.check { dependsOn(tasks.jacocoTestCoverageVerification) }
```

## gradle.properties (database-test)

```properties
databaseImage=pitipiwpiwwiwwiw/<schema-group>-postgres
databaseImageTag=<version>
```

Tag matches jOOQ schema artifact version.

## Application

```kotlin
@SpringBootApplication
class CurrencyValuablesCurrenciesCrudApplication {
    @Bean fun clock(): Clock = Clock.systemUTC()
}
```

Add `@EnableFeignClients` when calling neighbors.

## Controller

```kotlin
@RestController
class V1CurrencyController(...) : V1CurrencyControllerApi {
    // delegate to services
}
```

API: `com.pitipiwpiw-wiw-wiw.rest.<group>.<service>.api.V1*ControllerApi`  
DTO: `...model.*`

## GlobalExceptionHandler

```kotlin
@RestControllerAdvice
class GlobalExceptionHandler : CrudExceptionHandlerSupport<InternalServerError>() {
    override fun internalServerError(message: String, stackTrace: String) =
        InternalServerError(message, stackTrace)
}
```

## Repository (jOOQ)

```kotlin
@Repository
class CurrencyRepository(private val dsl: DSLContext, private val clock: Clock) {
    fun findByUuid(uuid: UUID) =
        dsl.selectFrom(CURRENCIES)
            .where(CURRENCIES.UUID.eq(uuid).and(CURRENCIES.DELETED_AT.isNull))
            .fetchOne()
}
```

Static table/record imports; `applyListSort`, `ClockSupport.now(clock)`; sort map in `*SortFields.kt`; no raw SQL.

## Kafka consumer

```kotlin
@Component
class ClassifierArchivedKafkaConsumer(...) : CurrenciesClassifierElementArchivedV2Consumer {
    override fun consume(headers: MessageHeaders, payload: MessagePayload) { ... }
}
```

Types: `com.pitipiwpiw-wiw-wiw.messaging.<Group>.<Topic>.*`

## TestcontainersConfiguration

```kotlin
@TestConfiguration(proxyBeanMethods = false)
@EnableTestBinder
@Import(PrebakedPostgresTestConfiguration::class)
class TestcontainersConfiguration
```

## Integration test

```kotlin
@SpringBootTest
@Import(TestcontainersConfiguration::class)
@ActiveProfiles("test")
class CurrencyRepositoryIntegrationTest { ... }
```

## Feign (neighbors)

```kotlin
@FeignClient(name = "...", url = "\${...}")
interface SomeServiceClient : V1SomeControllerApi
```

External APIs: custom `@FeignClient` + `FeignClientConfig` (see `precious-metals-rates-crud`).

## New CRUD checklist

1. Spec + OpenAPI
2. Flyway in `data-schemas/...` if new tables
3. Generated `rest`, `jooq`, `messaging` artifacts in `build.gradle.kts`
4. Skeleton: application, web, service, repository, mapping
5. `GlobalExceptionHandler`, `TestcontainersConfiguration`, integration tests
6. jacoco 70% on `check`
