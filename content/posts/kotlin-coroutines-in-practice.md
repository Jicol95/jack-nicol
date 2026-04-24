+++
title = "Kotlin Coroutines in Practice"
date = "2024-09-15"
description = "Practical patterns for using Kotlin coroutines in backend services, beyond the basics."
tags = [
    "kotlin",
    "coroutines",
    "backend",
]
+++

Kotlin coroutines are well documented at the introductory level, but production usage surfaces patterns that tutorials tend to gloss over. Here are a few that have been useful.

## Structured Concurrency Is Your Friend

The key mental model is that coroutines are scoped. If a parent scope is cancelled, all its children are cancelled too. This makes resource cleanup predictable.

```kotlin
coroutineScope {
    val result1 = async { fetchUserData(userId) }
    val result2 = async { fetchAccountBalance(userId) }
    process(result1.await(), result2.await())
}
```

If either fetch throws, the scope cancels the other — no dangling background work.

## `supervisorScope` for Independent Tasks

When you want failures to be isolated rather than propagating to siblings, use `supervisorScope`:

```kotlin
supervisorScope {
    val jobs = userIds.map { id ->
        launch {
            runCatching { syncUser(id) }
                .onFailure { logger.error("Failed to sync user $id", it) }
        }
    }
}
```

One failing sync won't kill the others.

## Avoid `GlobalScope`

`GlobalScope` coroutines live as long as the application and aren't tied to any lifecycle. In a service context, prefer injecting a `CoroutineScope` tied to the component's lifecycle. Spring's `@Bean` with a `CoroutineScope` backed by `SupervisorJob` works well.

```kotlin
@Bean
fun applicationScope(): CoroutineScope =
    CoroutineScope(SupervisorJob() + Dispatchers.Default)
```

## Flow for Streaming Data

`Flow` is the coroutines equivalent of a reactive stream — cold, composable, and cancellable. It's particularly useful for processing database result sets without loading everything into memory:

```kotlin
fun findActiveUsers(): Flow<User> = flow {
    database.streamActiveUsers().forEach { emit(it) }
}
```

## Dispatcher Choice

- `Dispatchers.Default` — CPU-bound work (computation, parsing)
- `Dispatchers.IO` — blocking I/O (JDBC, file access)
- `Dispatchers.Unconfined` — rarely needed outside testing

When in doubt, let your framework handle dispatch (Ktor and Spring both have coroutine-aware integrations).

## Testing

Use `runTest` from `kotlinx-coroutines-test` for unit tests — it handles virtual time and makes testing `delay`-based logic straightforward without actually waiting.
