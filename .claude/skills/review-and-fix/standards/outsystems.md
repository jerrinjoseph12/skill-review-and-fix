# OutSystems Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers OutSystems 11 (O11) and OutSystems Developer Cloud (ODC): Reactive Web, Mobile and Traditional Web apps.

## What can be reviewed and fixed
OutSystems modules (`.oml`, `.oap`, `.oapx`) are **binary models edited in Service Studio**. This skill can't open or edit them directly.

**Can be reviewed and fixed as text:**
- Integration Studio extensions (C#, so also apply [csharp-dotnet.md](csharp-dotnet.md))
- ODC external libraries (C#/.NET)
- Custom JavaScript, CSS and SQL that the user pastes or exports
- Forge or theme assets kept in the repo

**Can be reviewed but not edited:**
- Module logic the user describes, screenshots, or exports, such as AI Mentor Studio or ODC Code Quality findings (CSV/Excel).
- Report these findings with exact fix steps for Service Studio, and mark them **needs-decision**. Never claim a module-level fix was applied.

## Authorities
- OutSystems documentation: the Architecture Canvas, security best practices, performance best practices, and the Reactive Web security guide
- **OutSystems Mentor**:
  - AI Mentor Studio in O11, which replaced the Architecture Dashboard
  - the Code Quality console in the ODC Portal (Maintainability, Performance and Security mentors)
- Service Studio TrueChange warnings

If the user has Mentor findings, use them as the primary input and verify each one, rather than rediscovering the same issues.

## Security
- `[C]` SQL elements using **Expand Inline = Yes** on parameters built from user input (SQL injection). Use normal parameters, or `EncodeSql` and built-in list functions for IN clauses.
- `[C]` Server-side checks placed only in **client actions** in Reactive or Mobile apps. Client logic runs in the browser and can be bypassed. Authorization (`Check<Role>Role`) and validation must happen again in server actions or data actions.
- `[C]` Exposed REST or SOAP APIs with no authentication, or with Basic auth over credentials stored in plain text; secrets in site properties or settings not marked secret (ODC: secret settings).
- `[H]` Screens or server actions open to **Anonymous** or **Registered** roles when they need a specific role; `GetUserId()` passed from client to server as a parameter instead of being read on the server (Mentor flags this pattern).
- `[H]` Expressions with **Escape Content = No** showing user data (XSS); `JavaScript` nodes or `innerHTML` built from user data.
- `[H]` Sensitive data sent to the client: aggregates in screens fetching entities with sensitive attributes, because client-side data is visible in network traffic. Fetch only what the screen needs, through server-side data actions.
- `[M]` HTTP Security settings (HTTPS, HSTS, secure cookies, Content Security Policy) not enabled in Service Center or ODC Portal, where the app handles sensitive data.

## Performance
- `[H]` Aggregates or SQL queries **inside a For Each** loop (N+1). Join them, or fetch once and filter in memory.
- `[H]` Aggregates without a **Max Records** limit on growing tables in screens or lists.
- `[H]` Reactive screens with many **At start** aggregates or data actions that block rendering. Use **Only on demand** fetching, or combine them into one data action.
- `[H]` Large session variables (Traditional Web); big lists kept in client variables or local storage (Mobile).
- `[M]` SQL elements selecting `*` or unused columns (unlike aggregates, they aren't trimmed automatically); missing indexes on filtered or joined entity attributes.
- `[M]` Timers without a timeout or checkpointing that process everything in one run; long synchronous integrations in screen actions.

## Architecture (Architecture Canvas / 4-Layer Canvas)
- `[H]` **Upward references**: a Foundation or Core module referencing an End-user module.
- `[H]` **Side references** between End-user modules; **cycles** between Core modules.
- `[M]` Business logic in End-user modules that should be in Core Services; entities exposed as writeable to consumer modules (`Public = Yes`, `Expose Read Only = No`) instead of offering wrapper actions.
- `[M]` ODC: apps and libraries split incorrectly. Libraries must not depend on apps, and reuse across apps should go through service actions, events or libraries.

## Correctness
- `[H]` Server actions writing to several entities with no transaction handling: no **Abort Transaction** on exceptions, and commits that leave state partly written.
- `[H]` No exception handlers on integrations and timers; a missing or empty global exception handler on screens.
- `[M]` `CommitTransaction` used inside loops without a reason; `Ignore Errors` used without logging.
- `[M]` Local storage sync (Mobile) without conflict handling, and offline data not cleared at logout.

## Maintainability
- `[M]` Very large actions or screens that should be split into reusable actions or blocks; duplicated logic across modules; unused elements (TrueChange and Mentor flag them).
- `[L]` Elements without descriptions, names that don't follow the team's convention, hardcoded literals that should be static entities or site properties/settings.
