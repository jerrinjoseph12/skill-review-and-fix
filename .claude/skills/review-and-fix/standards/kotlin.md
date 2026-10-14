# Kotlin Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers the Kotlin language and server-side Kotlin (Spring Boot, Ktor). For Android apps, also apply [android.md](android.md). The JVM-level security rules in [java.md](java.md) (injection, deserialization, crypto, Spring security) apply to Kotlin too.

## Authorities
- Kotlin coding conventions
- kotlinx.coroutines guide (structured concurrency, cancellation, exceptions)
- Spring Boot Kotlin support docs; Ktor docs

## Tooling to look for and run
- `./gradlew check` / `test`
- detekt
- ktlint (or `ktlintCheck`)

## Null safety
- `[H]` `!!` on values that can really be null: map lookups, Java interop, parsed input.
- `[H]` Platform types from Java (`String!`) treated as non-null without a check.
- `[H]` A `lateinit` property that can be read before it's initialized. Use `::prop.isInitialized`, or a nullable or lazy property.

## Coroutines
- `[H]` `runBlocking` inside request handlers, coroutines or UI code. It blocks threads, and on the main thread it can deadlock.
- `[H]` `catch (e: Exception)` or `catch (e: Throwable)` inside a coroutine that swallows `CancellationException` and breaks cancellation. Rethrow it. `runCatching` around suspend calls has the same problem.
- `[H]` Blocking I/O (JDBC, file, `Thread.sleep`) on `Dispatchers.Default` or on an event loop (Ktor, WebFlux). Use `withContext(Dispatchers.IO)`.
- `[M]` `GlobalScope.launch`, which escapes structured concurrency and leaks work.
- `[M]` `async` without `await`, so errors are lost.
- `[M]` `SupervisorJob` passed as an argument to `launch` or `async`, which has no effect.
- `[M]` A cold `Flow` collected several times where the work should be shared. Use `shareIn` or `stateIn`.

## Server-side (Spring, Ktor)
- `[C]` Ktor routes missing an `authenticate { }` block that sibling routes have; Spring endpoints missing method security (see [java.md](java.md)).
- `[H]` The `kotlin-spring` (all-open) plugin missing. Kotlin classes are `final`, so `@Transactional`, `@Cacheable` and `@Async` proxies silently don't apply.
- `[H]` JPA entities declared as `data class`:
  - `equals`/`hashCode` cover all fields, including lazy relations and mutable IDs
  - `toString` triggers lazy loading
  
  Use a regular class with ID-based equality and the `kotlin-jpa` (no-arg) plugin.
- `[H]` Ktor CORS `anyHost()` with credentials.
- `[M]` kotlinx.serialization or Jackson configured without the Kotlin module, so defaults and nullability are ignored when deserializing.

## Correctness
- `[H]` `object` singletons or companion objects with mutable state shared across threads.
- `[M]` `copy()` on a data class bypassing the validation in `init`. Use a private constructor with a factory if invariants matter.
- `[M]` A public property exposing a mutable collection (`val items: MutableList<T>`). Expose `List<T>`.
- `[M]` `===` used where value equality is meant.
- `[M]` `when` on a non-sealed type with an `else` branch that hides newly added cases. Prefer sealed types.
- `[M]` Money in `Double`. Use `BigDecimal`.

## Maintainability
- `[L]` Not following the Kotlin coding conventions; Java-style getters and setters instead of properties; scope functions (`let`, `apply`, `also`, `run`) nested so deeply they hurt readability.
