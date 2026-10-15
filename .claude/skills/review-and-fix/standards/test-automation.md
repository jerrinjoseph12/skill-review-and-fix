# Test Automation Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers end-to-end and UI automation (Playwright, Cypress, Selenium/WebDriver, Appium, XCUITest, Espresso), unit and integration tests (JUnit, TestNG, NUnit, xUnit, MSTest, pytest, Jest, Vitest, Go `testing`, PHPUnit, XCTest), and BDD (Cucumber, SpecFlow/Reqnroll). Also apply the standards file for the test code's language. Native iOS and Android test tools have extra checks in [mobile-testing.md](mobile-testing.md); benchmarks and load tests are in [performance-testing.md](performance-testing.md).

**Severity in test code:** a defect that makes a test **pass when it should fail** is High, because it hides real bugs. Flakiness is usually Medium, or High if it blocks CI. Style is Low.

## Authorities
- Playwright *Best Practices*; Cypress *Best Practices*; Selenium documentation (waits, Page Object Models); Appium docs
- *xUnit Test Patterns* (Meszaros); the testing guides of each framework

## Tooling to look for and run
- The project's test command, running the changed tests at least
- For flakiness, rerun changed E2E tests a few times (`--repeat-each=3` in Playwright)
- Linters: `eslint-plugin-playwright`, `eslint-plugin-cypress`, `eslint-plugin-jest`

Don't run E2E suites against shared or production environments without asking.

## Tests that can't fail (pass when they shouldn't)
- `[H]` No assertions, or assertions only on mocks the test itself set up.
- `[H]` A missing `await` on an async assertion or action (Playwright, Jest or Vitest with promises). The test finishes before the check runs.
- `[H]` Exceptions caught inside the test and ignored, or `try { ... } catch { pass }` around the code under test.
- `[H]` `expect(await locator.isVisible()).toBe(true)` in Playwright: a one-time check with no retry. Use the auto-retrying assertion `await expect(locator).toBeVisible()`.
- `[H]` Snapshot files updated wholesale along with the code change, without review; assertions loosened (`toBeTruthy`, `contains`) to make a test pass.
- `[H]` `test.only`, `fit`, `fdescribe` or `describe.only` committed, which silently skips the rest of the suite. In Playwright, set `forbidOnly: !!process.env.CI`.
- `[M]` Tests skipped or disabled (`@Disabled`, `@Ignore`, `test.skip`, `xit`, `[Fact(Skip=...)]`) without a reason or ticket.

## Flakiness and waits
- `[H]` Fixed sleeps: `page.waitForTimeout`, `cy.wait(5000)`, `Thread.sleep`, `time.sleep`, `Task.Delay` used as waits. Use auto-waiting locators, web-first assertions or explicit waits on a condition.
- `[H]` Selenium implicit and explicit waits mixed, which makes timeouts unpredictable.
- `[H]` Tests that depend on run order or on state left by another test (shared accounts, records or static fields); Playwright `describe.serial` used to cover up dependencies.
- `[M]` Cypress: treating commands as promises (`const el = cy.get(...)`, `async`/`await` with `cy`); conditional testing on DOM state that's still changing.
- `[M]` Tests that depend on the real clock, time zone, random values, network or third-party services without fakes or seeding.
- `[M]` Retries (`retries: 3`, `@RetryingTest`, `rerunFailingTestsCount`) added to hide a flaky test instead of fixing it.

## Selectors and structure
- `[M]` Brittle selectors: absolute XPath, deep CSS chains, `nth-child`, auto-generated class names. Prefer role or label locators (`getByRole`, `getByLabel`) or dedicated test IDs (`data-testid`, `data-cy`, Appium `accessibilityId`).
- `[M]` Playwright `page.$` / `ElementHandle` in new code. Use locators.
- `[M]` Logic duplicated across E2E specs that belongs in a Page Object, fixture or helper; assertions placed inside Page Objects when the project's convention keeps them in tests.
- `[M]` Logging in through the UI in every test instead of reusing authentication state (Playwright `storageState`, Cypress `cy.session`).
- `[L]` Arguments in the wrong order, such as `assertEquals(actual, expected)`, which produces misleading failure messages; vague test names that don't describe the behavior.

## Security and environment
- `[C]` Real credentials, tokens or production URLs hardcoded in tests or config. Use environment variables or CI secrets.
- `[H]` Tests that can run against **production** and create, change or delete real data, without a guard.
- `[M]` Test data not cleaned up, so environments fill up with junk and later runs collide; secrets printed in test logs or saved traces and screenshots.

## Unit and integration tests
- `[H]` New or changed behavior with no test, especially a bug fix without a regression test.
- `[M]` Over-mocking: mocking the unit under test or its value objects, or checking implementation calls instead of outcomes.
- `[M]` Integration tests sharing a database without isolation (transactions, rollback, Testcontainers or unique data).
- `[M]` Tests that rely on static or global mutable state, such as a singleton or a mutated environment variable, without resetting it.

## BDD (Cucumber, Reqnroll)
- `[M]` Step definitions with complex logic or hidden assertions; UI-level detail ("click button #submit") in feature files instead of business language; steps that depend on state from earlier scenarios.
