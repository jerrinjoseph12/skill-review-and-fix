# Java Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

## Authorities
- Oracle *Secure Coding Guidelines for Java SE*
- SEI CERT Oracle Coding Standard for Java
- *Effective Java*, 3rd edition (Bloch)
- OWASP Java cheat sheets
- Style: the project's Checkstyle or formatter config. If there is none, follow the Google Java Style Guide, but don't report pure style issues that no project rule backs.

## Tooling to look for and run
- Build checks: `mvn -q verify` or `./gradlew check`.
- Static analysis, if configured: Checkstyle, PMD, SpotBugs (with FindSecBugs), Error Prone, SonarQube rules.
- Dependency audit: OWASP Dependency-Check.

## Version awareness
Read `maven.compiler.release`, `<java.version>` or the Gradle `toolchain` setting. Don't suggest features newer than the target version:
- records: 16
- sealed types: 17
- pattern-matching `switch`: 21
- virtual threads: 21
- UTF-8 as the default charset: 18
- Java 25 is the current LTS, and Java 21 is the previous one.

## Security
- `[C]` SQL, JPQL or HQL built by concatenation: `createQuery("... " + x)`, or `Statement.execute(sql + x)`. Use `PreparedStatement` or named parameters.
- `[C]` `Runtime.exec` or `ProcessBuilder` with user input, especially through `sh -c` or `cmd /c`.
- `[C]` Unsafe deserialization:
  - `ObjectInputStream.readObject` on untrusted bytes without an `ObjectInputFilter`
  - Jackson `enableDefaultTyping` or `activateDefaultTyping`, or `@JsonTypeInfo(use = Id.CLASS)`
  - XStream without an allowlist
  - SnakeYAML before 2.0 with `new Yaml()`
- `[C]` XXE in `DocumentBuilderFactory`, `SAXParserFactory`, `XMLInputFactory`, `TransformerFactory` or `SchemaFactory` when it isn't hardened (see [xml.md](xml.md)).
- `[C]` `X509TrustManager` that accepts every certificate, or a `HostnameVerifier` that returns `true`.
- `[H]` Path traversal: `new File(base, userInput)` or `Paths.get(base, userInput)` without a `normalize()` + `startsWith(base)` check. Zip slip in `ZipInputStream` extraction.
- `[H]` `Cipher.getInstance("AES")`, which defaults to ECB; a static IV; MD5 or SHA-1 for passwords; `java.util.Random` or `Math.random` for tokens. Use `SecureRandom`.
- `[H]` Spring:
  - CSRF disabled for a browser session app
  - `@CrossOrigin("*")` with credentials
  - a new controller method missing the `@PreAuthorize` or `@Secured` its siblings have
  - a JPA entity bound directly from the request (mass assignment)
- `[H]` log4j-core older than 2.17.1, or another dependency with known CVEs.
- `[M]` Expression-language injection (SpEL, OGNL, MVEL) when evaluating user strings.

## Correctness
- `[H]` `==` or `!=` on `String`, boxed `Integer` or `Long`, or other objects. The `Integer` cache makes it work only for -128 to 127.
- `[H]` `equals` overridden without `hashCode`, or mutable fields used in `hashCode` for keys in a `HashMap` or `HashSet`.
- `[H]` `InputStream`, `Reader`, `Connection`, `Statement` or `ResultSet` not closed in try-with-resources.
- `[H]` Auto-unboxing a possibly-null `Integer`, `Boolean`, etc. causes an NPE.
- `[H]` `Optional.get()` without a presence check; a method returning `Optional` that returns `null`.
- `[H]` `Collectors.toMap` without a merge function on keys that may repeat, which throws `IllegalStateException`.
- `[H]` Removing from a collection inside a for-each loop. Use `Iterator.remove` or `removeIf`.
- `[H]` Money held in `float` or `double`; `new BigDecimal(0.1)`, which should be `BigDecimal.valueOf` or the string constructor; `BigDecimal.equals` used where `compareTo` was meant, since `equals` is scale-sensitive.
- `[H]` `int` overflow in time or size arithmetic, such as `int millis = days * 86_400_000`.
- `[M]` `catch (InterruptedException e)` without `Thread.currentThread().interrupt()`.
- `[M]` Locale and charset defaults: `toUpperCase()` or `toLowerCase()` without `Locale.ROOT` (the Turkish-i bug); `getBytes()` or `new String(bytes)` without a charset when targeting versions before 18.
- `[M]` `java.util.Date` or `Calendar` in new code; `LocalDateTime` used where an instant in time is meant (use `Instant` or `OffsetDateTime`).

## Concurrency
- `[H]` `HashMap`, `ArrayList` or `SimpleDateFormat` shared across threads, for example in a static field or a singleton bean.
- `[H]` Check-then-act on a `ConcurrentHashMap` (`if (!map.containsKey(k)) map.put(...)`). Use `computeIfAbsent` or `putIfAbsent`.
- `[H]` Double-checked locking without `volatile`.
- `[M]` `synchronized` blocks doing I/O on virtual threads on JDK 21 to 23, which pins the carrier thread. JDK 24 and later fixed this.
- `[M]` A `parallelStream()` or `map` stage that mutates shared state.

## Spring and JPA
- `[H]` `@Transactional` or `@Async` on private methods, or called from the same class. The proxy is bypassed, so there's no transaction or async execution.
- `[H]` N+1 lazy loading inside a loop. Use `JOIN FETCH`, `@EntityGraph` or a batch fetch.
- `[M]` `FetchType.EAGER` on collections; open-session-in-view relied on for lazy loading.
- `[M]` Unbounded `findAll()` on large tables. Use paging.
- `[L]` Field injection (`@Autowired` on fields) in new code. Prefer constructor injection.

## Maintainability
- `[M]` A new public API with no tests; reimplementing a utility that already exists in the project, Apache Commons or Guava, when the project already depends on it.
- `[L]` Raw types, unchecked casts without `@SuppressWarnings` and a justification, wildcard imports (if the style forbids them), dead code.
