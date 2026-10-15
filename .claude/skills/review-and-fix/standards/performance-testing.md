# Performance Testing Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers code that **measures** performance. This file doesn't cover code that is slow; that's handled in each platform file's Performance section. Unit test frameworks like JUnit aren't performance tools; their rules are in [test-automation.md](test-automation.md).

| Kind | Tools covered |
|---|---|
| Microbenchmarks | **JMH** (Java/Kotlin), **BenchmarkDotNet** (.NET), Go `testing.B`, pytest-benchmark, Jetpack Microbenchmark, XCTest `measure` |
| App startup and rendering | Jetpack **Macrobenchmark**, XCTest metrics (`XCTApplicationLaunchMetric`), Lighthouse CI |
| Load, stress and soak tests | **JMeter**, **Gatling**, **k6**, **Locust**, Artillery, LoadRunner, NeoLoad, Azure/AWS load testing |

## Tooling to look for and run
- Run microbenchmarks only on request; they take a long time.
- Validate load scripts without generating load:
  - `k6 inspect script.js`
  - Gatling compile (`./mvnw gatling:test` with a smoke profile)
  - `jmeter -n -t plan.jmx` against a **local** target only
- **Never point a load test at a shared or production environment without explicit permission.**

## Safety
- `[C]` A load test whose default target is production, or a shared environment, with no guard or approval step.
- `[H]` Real credentials or personal data in scripts or CSV data files. Use generated test users and secrets from the environment.

## Microbenchmarks: results that are meaningless
- `[H]` **Dead-code elimination**: the benchmark result is discarded, so the compiler can remove the work.
  - JMH: return the value or use a `Blackhole`.
  - BenchmarkDotNet: return the value.
  - Go: use `b.Loop()` (Go 1.24+), or keep the result.
- `[H]` **Constant folding**: inputs are compile-time constants or `final` fields. In JMH, use `@State` fields.
- `[H]` "Performance tests" written as JUnit, xUnit or pytest tests timing a single run with `System.currentTimeMillis`, `Stopwatch` or `time.time`. There's no warm-up and no statistics, so the result is noise. Use the benchmark harness.
- `[H]` Benchmarks run in Debug builds or with a debugger attached. BenchmarkDotNet warns about this; Android Microbenchmark needs a release-like, non-debuggable build.
- `[M]` No warm-up or iteration settings (JMH `@Warmup` / `@Measurement` / `@Fork(0)`, which disables forking); Go setup not excluded from timing (missing `b.ResetTimer` in `b.N` loops; `b.Loop` handles this itself).
- `[M]` Comparisons based on one run with no confidence interval or allocation data (BenchmarkDotNet `[MemoryDiagnoser]`, JMH `-prof gc`).

## Load tests: results that look good but are wrong
- `[H]` **No pass/fail criteria**, so the test never fails. Add:
  - k6 `thresholds`
  - Gatling `assertions`
  - JMeter assertions plus a results check in CI
  - Locust exit-code checks
- `[H]` **Errors counted as successes**: HTTP 4xx/5xx or error pages not checked. A failing system then looks fast.
- `[H]` **Missing correlation**: session tokens, CSRF tokens or IDs captured once or hardcoded, so most requests fail quickly or hit the cache.
- `[H]` **Load generator is the bottleneck**:
  - JMeter run in GUI mode for load (use `-n`)
  - JMeter listeners such as View Results Tree enabled during the run
  - a single generator machine for high throughput
- `[H]` **Coordinated omission**: a closed workload model (fixed virtual users waiting on responses) used to state a requests-per-second capacity. Use an open model: k6 `constant-arrival-rate` / `ramping-arrival-rate`, Gatling `injectOpen`, JMeter Throughput Shaping or Arrivals threads.
- `[M]` Only averages reported. Use percentiles (p95, p99) and error rate.
- `[M]` No ramp-up; no think time or pacing; one user or one data row reused, so caches give unrealistic results.
- `[M]` Test too short to reveal leaks or degradation (soak tests need hours); environment not like production (data volume, instance sizes), and the results not caveated.
- `[L]` Scenarios hardcoded instead of parameterized (base URL, users, duration from environment or properties); no tags or groups for reporting by transaction.

## Frontend performance budgets
- `[M]` Lighthouse CI or bundle-size budgets loosened in the same change that regresses them; budgets with no assertions (`lighthouserc` without `assert`).
