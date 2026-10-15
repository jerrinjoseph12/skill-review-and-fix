# Mobile Testing Standards (iOS and Android)

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers native mobile test code. The general rules in [test-automation.md](test-automation.md) (tests that can't fail, flakiness, selectors, secrets) also apply. This file adds the tool-specific checks.

| Platform | Tools covered |
|---|---|
| iOS | XCTest, **Swift Testing** (`@Test`, `#expect`, `#require`), XCUITest, swift-snapshot-testing, EarlGrey, Quick/Nimble |
| Android | JUnit 4/5, **Espresso**, **UI Automator**, **Compose UI testing**, Robolectric, MockK/Mockito, Turbine, `kotlinx-coroutines-test`, Hilt testing, Paparazzi/Roborazzi |
| Cross-platform | Appium, Maestro, Detox (React Native), Flutter `integration_test` |

## Tooling to look for and run
- iOS: `xcodebuild test -scheme <scheme> -destination 'platform=iOS Simulator,name=<device>'`, or `swift test` for packages.
- Android: `./gradlew testDebugUnitTest` for local tests. `./gradlew connectedDebugAndroidTest` needs a device or emulator, so ask first.
- Maestro: `maestro test <flow>.yaml`. Device clouds such as Firebase Test Lab and BrowserStack need credentials; don't start them without asking.

## iOS: XCTest and Swift Testing
- `[H]` `XCTestExpectation` created but never waited on (`wait(for:timeout:)`, or `await fulfillment(of:)` in async tests), so the test passes before the callback runs.
- `[H]` In Swift Testing, `#expect` used where later lines depend on the value (they crash or report misleading failures). Use `try #require`, which stops the test and unwraps optionals.
- `[H]` Async code tested synchronously: the test returns before the `Task` completes. Mark the test `async` and `await` the work.
- `[M]` Shared mutable state between tests. Swift Testing runs tests **in parallel by default**. Use `.serialized` only when needed, and give each test its own fixtures.
- `[M]` Performance tests (`measure {}`, `XCTMetric`) without a baseline, or with high variance. Keep performance tests in XCTest; Swift Testing doesn't replace them.

## iOS: XCUITest
- `[H]` `sleep()` or `Thread.sleep` used for waiting. Use `waitForExistence(timeout:)` or `XCTNSPredicateExpectation`.
- `[M]` Elements found by label or visible text, which breaks with localization. Set `accessibilityIdentifier` in the app and query by it.
- `[M]` No launch arguments or environment to reset state or stub the network (`app.launchArguments += ["-UITests"]`), so tests depend on backend data and earlier runs.
- `[M]` `continueAfterFailure` left `true` in UI tests, so failures cascade into confusing follow-on errors; system alerts (permissions) not handled with `addUIInterruptionMonitor`.

## Android: local tests (JUnit, Robolectric, coroutines)
- `[H]` Coroutines tested with `runBlocking` and real delays instead of `runTest`. No `Dispatchers.setMain(StandardTestDispatcher())` or `MainDispatcherRule`, which crashes with "Module with the Main dispatcher is missing".
- `[H]` Hardcoded `Dispatchers.IO` in production code, so tests can't control timing. Inject dispatchers.
- `[H]` `Flow` assertions that collect forever or miss emissions. Use Turbine (`test { awaitItem() }`) or `first()` / `toList()` with `take`.
- `[M]` `LiveData` tests without `InstantTaskExecutorRule`; Room tests against the on-disk database instead of `Room.inMemoryDatabaseBuilder`.
- `[M]` Mockito used on Kotlin `final` classes without `mockito-inline`, or MockK `relaxed = true` hiding missing stubs.

## Android: Espresso, UI Automator and Compose
- `[H]` `Thread.sleep` or `SystemClock.sleep` for synchronization. Use Espresso `IdlingResource`, or Compose `waitUntil` / `waitUntilExactlyOneExists`.
- `[H]` Animations left on for instrumented tests (set `testOptions.animationsDisabled = true`, or turn off the developer animation scales).
- `[M]` `onView(withText("..."))` with hardcoded strings. Use resource IDs, `getString(R.string...)`, or a Compose `Modifier.testTag`.
- `[M]` Compose assertions on merged trees that can't find child nodes. Use `useUnmergedTree = true` where it's intended.
- `[M]` `ActivityScenario` not closed (use `.use {}` or `ActivityScenarioRule`); no Android Test Orchestrator where tests share app state.
- `[M]` Hilt tests missing `HiltAndroidRule` or `@UninstallModules`, so real network or database modules run.

## Snapshot and screenshot tests
- `[H]` Reference images re-recorded wholesale in the same change as UI code, without review.
- `[M]` Snapshots taken without fixing the device, locale, font scale and dark/light mode, so they differ between machines.

## Cross-platform (Appium, Maestro, Detox)
- `[M]` XPath locators in Appium. They're slow and brittle; use accessibility IDs. Implicit waits mixed with explicit waits.
- `[M]` A new app session per test where reset capabilities (`noReset`, `fullReset`) or deep links would isolate state faster.
- `[M]` Maestro flows that use `- waitForAnimationToEnd` or fixed `- extendedWaitUntil` timeouts as a crutch instead of stable IDs.
