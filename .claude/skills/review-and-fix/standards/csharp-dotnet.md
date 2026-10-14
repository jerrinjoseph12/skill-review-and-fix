# C# / .NET Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

## Authorities
- Microsoft *Framework Design Guidelines*
- C# coding conventions
- .NET code-analysis rules (`CAxxxx` for quality, `IDExxxx` for style)
- ASP.NET Core security documentation
- OWASP .NET Security Cheat Sheet
- The project's `.editorconfig` overrides general conventions.

## Tooling to look for and run
- `dotnet build` (respect `<TreatWarningsAsErrors>` and `<AnalysisLevel>`)
- `dotnet test`
- `dotnet format --verify-no-changes`
- `dotnet list package --vulnerable --include-transitive`
- Analyzers, if referenced: StyleCop.Analyzers, SonarAnalyzer.CSharp, Roslynator, Microsoft.CodeAnalysis.NetAnalyzers

## Version awareness
Read `<TargetFramework>` and `<LangVersion>`:
- `net48` and other `net4x` targets are **.NET Framework**. Its APIs and threading model differ, for example `SynchronizationContext` deadlocks.
- `net8.0` and `net10.0` are the current LTS releases.
- If `<Nullable>enable</Nullable>` is set, treat new nullable warnings as findings.

## Security
- `[C]` SQL by concatenation: `new SqlCommand("..." + x)`, or `FromSqlRaw` or `ExecuteSqlRaw` with concatenated or interpolated input. Use parameters, or `FromSql` / `FromSqlInterpolated`, which turn an interpolated string into parameters.
- `[C]` Unsafe deserialization:
  - `BinaryFormatter`, `NetDataContractSerializer`, `SoapFormatter`, `LosFormatter`
  - Newtonsoft `TypeNameHandling` set to anything other than `None` on untrusted input
  - `JavaScriptSerializer` with a type resolver
- `[C]` `ServerCertificateCustomValidationCallback` or `ServicePointManager.ServerCertificateValidationCallback` returning `true`.
- `[C]` Secrets in `appsettings*.json` or `web.config` that are committed. Use user-secrets, environment variables or a key vault.
- `[H]` A new controller or action missing `[Authorize]` (or covered by a global policy that its siblings rely on); `[AllowAnonymous]` added without a reason; an object fetched by ID with no ownership check.
- `[H]` `Html.Raw`, `MarkupString` or `@((MarkupString)x)` with user data (XSS).
- `[H]` `Path.Combine(base, userInput)`: if `userInput` is an absolute path, `base` is **discarded**. Check `Path.GetFullPath(result).StartsWith(base)`.
- `[H]` `Process.Start` with user-controlled file names or arguments.
- `[H]` XML: `XmlReaderSettings.DtdProcessing = Parse`, or a non-null `XmlResolver` on `XmlDocument` or `XmlTextReader`, especially when targeting .NET Framework (see [xml.md](xml.md)).
- `[H]` `System.Random` for tokens (use `RandomNumberGenerator`); MD5 or SHA-1 for passwords (use `Rfc2898DeriveBytes.Pbkdf2` or ASP.NET Core Identity's hasher).
- `[H]` CORS `SetIsOriginAllowed(_ => true)` together with `AllowCredentials()`.
- `[M]` `[IgnoreAntiforgeryToken]` on state-changing endpoints in cookie-auth apps; `UseDeveloperExceptionPage` not gated to Development.

## Async and resources
- `[H]` `async void` methods other than event handlers. Exceptions from them crash the process.
- `[H]` Sync-over-async with `.Result`, `.Wait()` or `.GetAwaiter().GetResult()`. This deadlocks in .NET Framework, UI or classic ASP.NET contexts, and starves the thread pool in ASP.NET Core.
- `[H]` A task not awaited (warning CS4014, or a `Task` discarded silently), so exceptions are lost.
- `[H]` `new HttpClient()` per request, which exhausts sockets. Use `IHttpClientFactory` or a shared client.
- `[H]` `DbContext` registered as a singleton or used concurrently across threads; a scoped service captured by a singleton (captive dependency).
- `[H]` `IDisposable` or `IAsyncDisposable` not disposed. Use `using` or `await using`.
- `[M]` `CancellationToken` accepted but not passed to downstream async calls.
- `[M]` `lock` used around `await`, which doesn't compile; working around it badly. Use `SemaphoreSlim`. Also flag `lock(this)`, `lock(typeof(...))` and locks on strings.
- `[L]` Missing `ConfigureAwait(false)` in library code (not app code), if the project uses that convention.

## Correctness
- `[H]` `throw ex;` loses the stack trace. Use `throw;`.
- `[H]` `catch (Exception) { }` that swallows errors on business paths.
- `[H]` Money in `double` or `float`. Use `decimal`.
- `[H]` `DateTime.Now` stored or compared across machines. Use `DateTime.UtcNow` or `DateTimeOffset`.
- `[M]` `DateTime.Parse`, `ToString()` or `double.Parse` without a culture, where the format matters. Use `CultureInfo.InvariantCulture`.
- `[M]` String comparisons without a `StringComparison` (CA1307, CA1309, CA1310).
- `[M]` The null-forgiving `!` used to silence real nullability warnings.
- `[M]` Mutable structs; `GetHashCode` based on mutable fields.
- `[M]` Static mutable state in web apps, which is shared across requests.

## LINQ and EF Core performance
- `[H]` `.ToList()`, `.AsEnumerable()` or client evaluation before `.Where()` on an `IQueryable`, which loads the whole table into memory.
- `[H]` N+1 from lazy loading or queries inside a loop. Use `Include`, projection or batching.
- `[M]` Enumerating an `IEnumerable` more than once (CA1851); `Count() > 0` instead of `Any()`.
- `[L]` No `AsNoTracking()` on read-only queries in hot paths.

## Maintainability
- `[M]` Public API changes without XML doc comments where the project documents its public surface; new behavior without tests.
- `[L]` Naming that breaks .NET conventions: PascalCase for types and members, `_camelCase` for private fields, `I` prefix for interfaces, `Async` suffix for async methods. Dead code; `#pragma warning disable` without a justification comment.
