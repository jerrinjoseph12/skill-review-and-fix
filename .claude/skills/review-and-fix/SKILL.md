---
name: review-and-fix
description: Review code for bugs, security issues, performance problems and maintainability concerns, rank every finding by severity (Critical, High, Medium, Low), then fix them in severity order, highest first, down to a chosen threshold, checking each fix with the project's tests and linters. Applies platform standards for Java, Kotlin, C#/.NET, Go, PHP, Python, JavaScript/TypeScript, Apple iOS/Swift, Android, Salesforce (Apex, LWC, Flows), OutSystems, Appian (SAIL), HTML/CSS/accessibility, XML, SQL, test automation (Playwright, Cypress, Selenium, Appium, unit tests), iOS and Android testing tools (XCTest, XCUITest, Espresso, Compose), performance testing (JMH, BenchmarkDotNet, JMeter, Gatling, k6, Locust), Terraform/IaC, Docker/Kubernetes, CI/CD pipelines and shell scripts, plus OWASP Top 10:2025. Use when the user asks to review and fix code, clean up a diff before merging, harden a file or module, or "find and fix issues". Targets the uncommitted diff by default, or a path, branch or PR number.
argument-hint: "[path | branch | PR#] [--threshold critical|high|medium|low] [--dry-run]"
---

# Review and Fix

Review the target code, rank each finding by severity, and fix findings from the most severe down to the threshold. Every fix is checked before moving on.

Arguments: `$ARGUMENTS`

## 0. Parse arguments

| Argument | Meaning | Default |
|---|---|---|
| target (positional) | A file or directory path, a branch name, or a PR number | The uncommitted diff (staged + unstaged); if there is no diff, ask what to review |
| `--threshold <level>` | Lowest severity to fix automatically. Findings below it are reported, not fixed | `high` (fixes Critical and High) |
| `--dry-run` | Review and report only. Change no files | off |

## 1. Determine scope

- **Diff (default):** `git diff HEAD` plus untracked files from `git status --porcelain`. Review the changed lines, and read enough of the surrounding code to judge them. Report problems in unchanged code only if a change makes them reachable or worse.
- **Path:** review every source file under it. Skip vendored, generated, lock and build-output files (`node_modules/`, `dist/`, `vendor/`, `*.lock`, `*.min.js`, and so on).
- **Branch:** `git diff $(git merge-base HEAD <branch>)...<branch>`.
- **PR number:** `gh pr diff <n>` and `gh pr view <n>` for intent. If you will apply fixes, check out the PR branch first and confirm with the user before doing so.
- **Not a git repo:** treat the target as a path. If none was given, ask.

If the scope is larger than about 40 files or 3,000 changed lines, tell the user and suggest narrowing it before continuing.

## 2. Gather context

Before judging anything, learn the project's own rules:

1. Read `CLAUDE.md`, `CONTRIBUTING.md`, and linter or formatter configs if present.
2. Find the commands that check correctness: the test runner, type checker, linter and build (from `package.json` scripts, `Makefile`, `pyproject.toml`, `Cargo.toml`, CI workflow files, and so on). Record them; step 5 needs them.
3. Run those checks once **before** changing anything, and record the baseline. Failures that already exist are not caused by your fixes. Note them separately and don't fix them unless they fall within the scope.
4. For each file under review, read the callers and callees of the changed code that matter, not just the diff hunk.
5. **Load the platform standards** that match the files in scope. Always load [standards/security-owasp.md](standards/security-owasp.md). Then load only the rows below that apply:

| Files in scope | Standards file |
|---|---|
| `*.java`, `pom.xml`, `build.gradle(.kts)`, `appian-plugin.xml` | [standards/java.md](standards/java.md) |
| `*.cs`, `*.csproj`, `*.cshtml`, `*.razor`, `appsettings*.json`, `web.config` | [standards/csharp-dotnet.md](standards/csharp-dotnet.md) |
| SAIL (`*.sail`), Appian export XML (`content/*.xml`), Appian objects read via an Appian MCP server | [standards/appian.md](standards/appian.md) |
| `*.html`, `*.css`, `*.scss`, and markup in templates (`*.jsp`, `*.cshtml`, `*.razor`, `*.jsx`, `*.tsx`, `*.vue`, Angular templates) | [standards/html-css.md](standards/html-css.md) |
| `*.xml`, `*.xsd`, `*.xsl(t)`, `*.config`, `AndroidManifest.xml`, and any code that parses or builds XML | [standards/xml.md](standards/xml.md) |
| `*.js`, `*.mjs`, `*.cjs`, `*.ts`, `*.tsx`, `*.jsx`, `*.vue` | [standards/javascript-typescript.md](standards/javascript-typescript.md) |
| `*.sql`, migrations, stored procedures, and SQL strings embedded in code | [standards/sql.md](standards/sql.md) |
| `*.py` | [standards/python.md](standards/python.md) |
| `*.kt`, `*.kts` | [standards/kotlin.md](standards/kotlin.md) |
| `*.go`, `go.mod` | [standards/go.md](standards/go.md) |
| `*.php`, `composer.json`, `*.blade.php`, `*.twig` | [standards/php.md](standards/php.md) |
| `*.swift`, `*.m`, `*.mm`, `Package.swift`, `Info.plist`, `*.entitlements`, `*.xcprivacy` | [standards/ios-swift.md](standards/ios-swift.md) |
| Android modules: `AndroidManifest.xml`, Gradle files using `com.android.*` plugins, `res/` XML, and Kotlin/Java code that uses Android APIs | [standards/android.md](standards/android.md), plus kotlin.md or java.md |
| Test code: `*.spec.*`, `*.test.*`, `*.cy.*`, `test_*.py`, `*Test.java`, `*Tests.cs`, `*_test.go`, `e2e/`, `tests/`, `playwright.config.*`, `cypress.config.*`, `*.feature` | [standards/test-automation.md](standards/test-automation.md), plus the language file |
| `*.tf`, `*.tfvars`, `*.hcl`, CloudFormation or CDK templates, `*.bicep`, Pulumi programs | [standards/infrastructure-as-code.md](standards/infrastructure-as-code.md) |
| `Dockerfile`, `Containerfile`, `compose.yaml` / `docker-compose*.yml`, Kubernetes manifests, Helm charts, Kustomize | [standards/containers-kubernetes.md](standards/containers-kubernetes.md) |
| `.github/workflows/*.yml`, `Jenkinsfile`, `bitbucket-pipelines.yml`, `.gitlab-ci.yml`, `azure-pipelines.yml` | [standards/ci-cd.md](standards/ci-cd.md) |
| `*.sh`, `*.bash`, `*.zsh`, files with a shell shebang, `*.ps1`, `*.psm1` | [standards/shell-scripts.md](standards/shell-scripts.md) |
| Salesforce: `sfdx-project.json`, `*.cls`, `*.trigger`, `lwc/`, `aura/`, `*.page`, `*.component`, `*.flow-meta.xml` | [standards/salesforce.md](standards/salesforce.md) |
| OutSystems: Integration Studio extensions, ODC external libraries, and exported custom JS/CSS/SQL or Mentor findings | [standards/outsystems.md](standards/outsystems.md) |
| iOS tests (`*Tests.swift`, `*UITests/`), Android tests (`src/test/`, `src/androidTest/`), Appium, Maestro or Detox suites | [standards/mobile-testing.md](standards/mobile-testing.md), plus test-automation.md |
| Benchmarks and load tests: JMH (`@Benchmark`), BenchmarkDotNet, Go `Benchmark*`, `*.jmx`, Gatling simulations, k6 scripts, `locustfile.py`, Artillery YAML, Lighthouse CI config | [standards/performance-testing.md](standards/performance-testing.md) |

A file can match more than one row; load each file that matches. For example, an Android Kotlin test loads kotlin.md, android.md and test-automation.md.

For a language with no standards file, use security-owasp.md, that language's official style guide, and the linters the project configures.

**The project's own rules take precedence** over these general standards. If the project's linter config, `.editorconfig`, `CLAUDE.md` or existing conventions intentionally differ, follow the project and don't report the difference.

## 3. Review

Go through the code looking for each category below, and apply the checklist in each loaded standards file. The `[C]`, `[H]`, `[M]` and `[L]` tags in those files are default severities. Adjust them for actual impact and likelihood using [severity.md](severity.md), which has the full rubric and examples.

- **Correctness:** logic errors, off-by-one, null or undefined handling, wrong conditions, unhandled error paths, race conditions, resource leaks, broken edge cases (empty, zero, max, unicode, timezone).
- **Security:** injection (SQL, shell, path, template), missing authentication or authorization checks, secrets in code, unsafe deserialization, SSRF, XSS, weak crypto, sensitive data in logs.
- **Data integrity:** missing transactions, partial writes, lost updates, unsafe migrations.
- **Performance:** N+1 queries, unbounded loops or memory, blocking I/O on hot paths, accidentally quadratic work.
- **API and contract:** breaking changes to public interfaces, mismatches between schema and code, wrong status codes or error shapes.
- **Maintainability:** duplication of existing helpers, dead code, misleading names, missing tests for new behavior, divergence from project conventions.

For each candidate finding, **verify it before you record it**. Re-read the exact lines, trace how the bad input or state arises, and confirm the path is reachable. Drop anything you cannot back with a concrete failure scenario. A short list of real findings beats a long list of maybes.

Record each verified finding as:

```
ID:        F1, F2, ... (numbered after sorting by severity)
Severity:  Critical | High | Medium | Low
Category:  correctness | security | data-integrity | performance | api | maintainability
Location:  path/to/file.ext:LINE
Problem:   one sentence stating the defect
Scenario:  concrete input or state → the wrong result, crash or exposure
Fix:       the minimal change that resolves it
Auto-fix:  yes | needs-decision (see below)
```

Mark a finding **needs-decision** instead of fixing it if the fix would:
- change a public API, a database schema, or a persisted data format,
- require choosing between product behaviors the code doesn't settle,
- touch more than about 3 files or need a large refactor,
- add a new dependency,
- require hand-editing generated files, such as Appian export XML with its UUIDs and version info.

Fixes to objects in a **live environment** (for example Appian objects edited through an MCP server) change shared state. List the exact changes and get the user's confirmation before you make them, even for findings above the threshold.

## 4. Report findings

Sort by severity (Critical → Low), then by file. Show the user a table **before** editing anything:

```
| ID | Sev | Category | Location | Problem | Action |
|----|-----|----------|----------|---------|--------|
| F1 | Critical | security | api/users.py:42 | SQL built by string concatenation from request input | Fix |
| F2 | High | correctness | lib/cart.ts:88 | Discount applied twice when coupon is re-entered | Fix |
| F3 | Medium | performance | lib/feed.ts:120 | N+1 query per post author | Report (below threshold) |
| F4 | High | api | routes/v1.ts:30 | Response field renamed; breaks clients | Needs decision |
```

`Action` is one of: **Fix** (at or above the threshold and auto-fixable), **Report (below threshold)**, or **Needs decision**.

If `--dry-run` is set, stop here and end with the summary from step 6.

If there are no findings, say so plainly, list what you checked, and stop.

## 5. Fix in severity order

Work one severity tier at a time: all Critical, then all High, then down to the threshold. Within a tier, fix one finding at a time:

1. Re-read the current code at the location, since earlier fixes may have moved it.
2. Make the **smallest change** that resolves the finding. Match the surrounding style. Don't refactor nearby code, rename things, or reformat untouched lines.
3. For correctness and security fixes, add or update a test that fails without the fix, if the project has a test suite for that area.
4. Keep a list of the files changed for each finding.

After finishing each tier, run the checks recorded in step 2:

- **Checks pass** (or match the baseline): move on to the next tier.
- **A new failure appears:** find the fix that caused it. Correct it if the cause is clear. Otherwise revert that fix only, and mark the finding **Fix failed** with the error output. Don't let one bad fix block the rest.
- **No checks available:** say so in the report, and re-read each changed region to confirm it parses and still does what it did, plus the fix.

Never weaken a test, delete an assertion, add a lint-suppression comment, or catch-and-ignore an error just to make a check pass.

## 6. Final summary

End with:

1. **Result table:** each finding with its final status: Fixed, Fix failed, Needs decision, or Reported.
2. **Changes:** files modified for each fixed finding, in one line each.
3. **Verification:** which checks ran and their before and after results. Quote failures exactly.
4. **Open items:** findings that need a decision, with the trade-off in one or two sentences each, and findings below the threshold, so the user can rerun with a lower `--threshold` if they want those fixed.

Don't commit, push or post comments unless the user asks.
