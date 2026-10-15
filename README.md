# review-and-fix

A [Claude Code](https://claude.com/claude-code) skill that reviews code, ranks each finding by severity, and fixes them from the most severe down to a threshold you choose. After each severity level it runs your project's tests and linters to check the fixes.

It applies platform standards for:

- **Languages:** Java, Kotlin, C#/.NET, Go, PHP, Python, JavaScript/TypeScript, SQL
- **Mobile:** Apple iOS (Swift, Objective-C), Android
- **Web and low-code:** HTML/CSS and accessibility, XML, Appian (SAIL), Salesforce (Apex, LWC, Flows), OutSystems
- **Testing:** Playwright, Cypress, Selenium, Appium and unit test frameworks; iOS and Android test tools (XCTest, Swift Testing, XCUITest, Espresso, Compose, Robolectric); performance testing (JMH, BenchmarkDotNet, JMeter, Gatling, k6, Locust)
- **Infrastructure and delivery:** Terraform and other infrastructure-as-code, Docker and Kubernetes, CI/CD pipelines, shell scripts

It also checks every review against the **OWASP Top 10:2025** for security.

## Install

The skill lives in `.claude/skills/review-and-fix/`.

**For all your projects:**

```bash
git clone https://github.com/jerrinjoseph12/skill-review-and-fix.git /tmp/skill-review-and-fix
mkdir -p ~/.claude/skills
cp -r /tmp/skill-review-and-fix/.claude/skills/review-and-fix ~/.claude/skills/
rm -rf /tmp/skill-review-and-fix
```

**For one project, shared with your team:** run the same commands from the project's root folder, but copy into `.claude/skills/` instead of `~/.claude/skills/`. Then commit that folder.

Start a new Claude Code session and type `/review-and-fix` to check it's installed. To update, run the commands again.

## Usage

```
/review-and-fix                              # uncommitted changes; fixes Critical and High
/review-and-fix src/api --threshold medium   # a folder; fixes Critical, High and Medium
/review-and-fix feature/login                # changes on a branch
/review-and-fix 123 --dry-run                # review PR #123 without changing files
```

Claude also uses the skill on its own when you ask something like "review and fix my changes".

| Argument | Meaning | Default |
|---|---|---|
| target | A file or folder path, a branch name, or a PR number | Uncommitted changes |
| `--threshold critical\|high\|medium\|low` | Lowest severity to fix. Findings below it are reported only | `high` |
| `--dry-run` | Review and report only; change nothing | off |

PR targets need the [GitHub CLI](https://cli.github.com/) (`gh`). Folders that aren't git repos can be reviewed by path.

## How it works

1. **Scope:** works out which files and changes to review.
2. **Context:** reads the project's rules (`CLAUDE.md`, linter configs, `.editorconfig`), finds its test, lint and build commands, and runs them once to record a baseline.
3. **Standards:** loads the security standards and the standards file for each platform in scope.
4. **Review:** checks for bugs, security, data integrity, performance, interface or contract changes, and maintainability. A finding is kept only if it comes with a concrete failure scenario.
5. **Report:** shows a table sorted by severity **before** changing anything. Each finding is marked **Fix**, **Report** (below the threshold) or **Needs decision**.
6. **Fix:** works one severity level at a time, making the smallest change that resolves each finding. It reruns the checks after each level and reverts any fix that breaks them.
7. **Summary:** gives each finding's final status, the files changed, check results before and after, and any open items.

### Severity levels

| Level | Meaning | Examples |
|---|---|---|
| **Critical** | Blocks shipping | Injection reachable from user input, an authorization bypass, committed secrets, data loss |
| **High** | Fix before merging | Wrong results for realistic inputs, crashes on reachable paths, race conditions, N+1 queries on hot paths |
| **Medium** | Should fix | Edge cases, missing validation, swallowed errors, missing tests |
| **Low** | Nice to fix | Naming, dead code, minor deviations from convention |

Severity is judged by **impact × likelihood**, not by category. The full rubric is in [severity.md](.claude/skills/review-and-fix/severity.md).

### Safety rules

- It never commits, pushes or posts comments unless you ask.
- It never weakens tests, deletes assertions or adds lint suppressions to make checks pass.
- It doesn't fix these automatically, and marks them **Needs decision** instead:
  - public interface or database schema changes
  - new dependencies
  - fixes touching more than about 3 files
  - hand edits to generated files, such as Appian export XML
- It asks for confirmation before changing anything in a live environment, for example Appian objects through an MCP server.

## Platform standards

| Files | Standards | Main sources |
|---|---|---|
| All | [security-owasp.md](.claude/skills/review-and-fix/standards/security-owasp.md) | OWASP Top 10:2025, CWE Top 25 |
| `*.java`, `pom.xml`, Gradle, Appian plug-ins | [java.md](.claude/skills/review-and-fix/standards/java.md) | Oracle Secure Coding Guidelines, SEI CERT Java, *Effective Java* |
| `*.cs`, `*.csproj`, Razor, `web.config` | [csharp-dotnet.md](.claude/skills/review-and-fix/standards/csharp-dotnet.md) | Framework Design Guidelines, .NET analyzers (CA/IDE rules), ASP.NET Core security |
| SAIL, Appian exports, Appian MCP objects | [appian.md](.claude/skills/review-and-fix/standards/appian.md) | Appian docs, Health Check, Appian accessibility checklist |
| HTML, CSS, templates (JSP, Razor, JSX, Vue, Angular) | [html-css.md](.claude/skills/review-and-fix/standards/html-css.md) | WCAG 2.2 AA, WAI-ARIA 1.2, WHATWG HTML, OWASP XSS guidance |
| `*.xml`, XSD, XSLT, config, XML-parsing code | [xml.md](.claude/skills/review-and-fix/standards/xml.md) | W3C specs, OWASP XXE Prevention |
| `*.js`, `*.ts`, `*.jsx`, `*.tsx`, `*.vue` | [javascript-typescript.md](.claude/skills/review-and-fix/standards/javascript-typescript.md) | OWASP Node.js guidance, typescript-eslint |
| `*.sql`, migrations, stored procedures | [sql.md](.claude/skills/review-and-fix/standards/sql.md) | Vendor docs, OWASP SQL Injection Prevention |
| `*.py` | [python.md](.claude/skills/review-and-fix/standards/python.md) | PEP 8, PEP 484, Bandit, Django deployment checklist |
| `*.kt`, `*.kts` | [kotlin.md](.claude/skills/review-and-fix/standards/kotlin.md) | Kotlin coding conventions, kotlinx.coroutines guide, Spring Boot and Ktor docs |
| `*.go` | [go.md](.claude/skills/review-and-fix/standards/go.md) | *Effective Go*, Go Code Review Comments, `go vet`, `govulncheck` |
| `*.php`, Blade, Twig | [php.md](.claude/skills/review-and-fix/standards/php.md) | PHP manual security section, PSR-12 / PER, Laravel, Symfony and WordPress security docs |
| `*.swift`, `*.m`, `Info.plist`, privacy manifests | [ios-swift.md](.claude/skills/review-and-fix/standards/ios-swift.md) | Swift API Design Guidelines, Apple Platform Security, App Store Review Guidelines, OWASP MASVS |
| Android modules, `AndroidManifest.xml` | [android.md](.claude/skills/review-and-fix/standards/android.md) | Android security best practices and app quality guidelines, OWASP MASVS / MASTG |
| Test code and E2E suites | [test-automation.md](.claude/skills/review-and-fix/standards/test-automation.md) | Playwright, Cypress, Selenium and Appium best-practice guides, *xUnit Test Patterns* |
| Terraform, CloudFormation, Bicep, Pulumi | [infrastructure-as-code.md](.claude/skills/review-and-fix/standards/infrastructure-as-code.md) | Terraform recommended practices, CIS cloud benchmarks |
| `Dockerfile`, Compose, Kubernetes, Helm | [containers-kubernetes.md](.claude/skills/review-and-fix/standards/containers-kubernetes.md) | Dockerfile best practices, CIS Docker and Kubernetes benchmarks, Pod Security Standards |
| GitHub Actions, Jenkins, Bitbucket, GitLab, Azure Pipelines | [ci-cd.md](.claude/skills/review-and-fix/standards/ci-cd.md) | GitHub Actions security hardening, OWASP Top 10 CI/CD Security Risks, SLSA |
| `*.sh`, `*.bash`, `*.zsh`, `*.ps1` | [shell-scripts.md](.claude/skills/review-and-fix/standards/shell-scripts.md) | ShellCheck, Google Shell Style Guide, PSScriptAnalyzer |
| Apex, LWC, Aura, Visualforce, Flows | [salesforce.md](.claude/skills/review-and-fix/standards/salesforce.md) | Apex Developer Guide, Salesforce Secure Coding Guide, Salesforce Code Analyzer |
| OutSystems extensions, exported code, Mentor findings | [outsystems.md](.claude/skills/review-and-fix/standards/outsystems.md) | OutSystems Architecture Canvas, security and performance best practices, OutSystems Mentor |
| iOS and Android test suites | [mobile-testing.md](.claude/skills/review-and-fix/standards/mobile-testing.md) | XCTest, Swift Testing, Espresso, Compose testing and coroutine testing guides |
| Benchmarks and load tests | [performance-testing.md](.claude/skills/review-and-fix/standards/performance-testing.md) | JMH, BenchmarkDotNet, Go benchmarks, JMeter, Gatling, k6 and Locust docs |

A file can match more than one row, and the skill loads every matching file. For example, an Android Kotlin test gets the Kotlin, Android and test-automation checklists.

Each checklist item has a default severity tag: `[C]`, `[H]`, `[M]` or `[L]`. The review adjusts it for the actual impact. For other languages, the skill uses the security standards, the language's official style guide and the project's own linters.

**Your project's rules come first.** If the project's linter config, `.editorconfig` or `CLAUDE.md` intentionally differs from these standards, the skill follows the project.

For Appian, install the `appian` skill as well if you have it. This skill uses it for exact function and component details.

## Customizing

- **Add a platform:** create `standards/<platform>.md` following the format of the existing files, and add a row for it to the table in step 2 of [SKILL.md](.claude/skills/review-and-fix/SKILL.md).
- **Add your organization's rules:** put them in the relevant standards file, or in a new file listed in the same table.
- **Change severity rules:** edit [severity.md](.claude/skills/review-and-fix/severity.md).
- **Change the default threshold:** edit the argument table in step 0 of [SKILL.md](.claude/skills/review-and-fix/SKILL.md).

## Repository layout

```
.claude/skills/review-and-fix/
├── SKILL.md            # Workflow instructions Claude follows
├── severity.md         # Severity rubric
└── standards/          # Per-platform review checklists
    ├── security-owasp.md
    ├── java.md
    ├── csharp-dotnet.md
    ├── appian.md
    ├── html-css.md
    ├── xml.md
    ├── javascript-typescript.md
    ├── sql.md
    ├── python.md
    ├── kotlin.md
    ├── go.md
    ├── php.md
    ├── ios-swift.md
    ├── android.md
    ├── test-automation.md
    ├── infrastructure-as-code.md
    ├── containers-kubernetes.md
    ├── ci-cd.md
    ├── shell-scripts.md
    ├── salesforce.md
    ├── outsystems.md
    ├── mobile-testing.md
    └── performance-testing.md
```
