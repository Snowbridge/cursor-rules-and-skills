---
name: java-kotlin-no-suppress-warnings
description: Enforces fixing Java/Kotlin/Spring warnings at the source instead of suppressing them. Use when writing or editing Java, Kotlin, Spring Boot, or Spring code; when the user mentions warnings, lint, or @Suppress; or when working in any JVM/Spring project.
---

# Java/Kotlin + Spring: no warning suppression

## Rule

Строго запрещено гасить варнинги аннотациями вроде `@Suppress` (и эквивалентами: `@SuppressWarnings`, `@file:Suppress`, `@kotlin.Suppress`, IDE/local `//noinspection`, `@SuppressLint` и т.п.).

**Единственное исключение** — если пользователь **прямо в тексте задачи** разрешил подавлять варнинги.

Если задачу **нельзя решить без подавления** и явного разрешения нет — **прервать выполнение** и сообщить пользователю:
- какой варнинг мешает;
- почему исправление «в корне» не удалось в рамках задачи;
- что нужно: разрешение на suppress **или** уточнение требований.

Не подбирать обходные suppress «тихо» и не предлагать их по умолчанию.

## Что делать вместо suppress

1. Исправить причину: типы, null-safety, неиспользуемые символы, deprecated API → замена, generics, рефакторинг.
2. Если варнинг ложный — перестроить код так, чтобы анализатор перестал ругаться (без suppress).
3. Если без suppress нужны изменения вне scope задачи — остановиться и спросить.

## Проверка перед завершением

- [ ] В diff нет новых `@Suppress*` / `//noinspection` / `@SuppressLint`
- [ ] Если suppress есть — в задаче было явное разрешение, процитировано в ответе
