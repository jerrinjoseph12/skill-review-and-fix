# Android Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers the Android platform. For the code's language, also apply [kotlin.md](kotlin.md) or [java.md](java.md).

## Authorities
- Android Developers: *Security best practices*, *App quality guidelines*, the Jetpack Compose and architecture guides
- Google Play policies, including the target API level requirement
- OWASP MASVS v2, the OWASP Mobile Application Security Testing Guide (MASTG), and the OWASP Mobile Top 10

## Tooling to look for and run
- `./gradlew lint`
- `./gradlew testDebugUnitTest`
- `./gradlew connectedDebugAndroidTest` (needs a device or emulator, so ask first)
- detekt, ktlint

## Version awareness
Read `minSdk`, `targetSdk` and `compileSdk`:
- Calls to newer APIs need `Build.VERSION.SDK_INT` checks.
- Each targetSdk raise changes behavior:
  - 31: explicit `android:exported`; `PendingIntent` mutability flags
  - 33: notification permission
  - 34: foreground service types; restrictions on implicit intents

## Security
- `[C]` `android:debuggable="true"` or `isDebuggable = true` in a release build type. Also flag signing-keystore passwords committed in Gradle files.
- `[C]` WebView `setAllowUniversalAccessFromFileURLs(true)`, or `setJavaScriptEnabled(true)` plus `addJavascriptInterface` on content that isn't fully trusted.
- `[C]` `onReceivedSslError` calling `handler.proceed()`; a custom `TrustManager` or `HostnameVerifier` that accepts everything.
- `[H]` Activities, services, receivers or providers with `exported="true"` and no `android:permission` or caller check, when they perform sensitive actions.
- `[H]` A `PendingIntent` with `FLAG_MUTABLE` and an implicit base intent; sensitive data sent in implicit intents or broadcasts.
- `[H]` `ContentProvider` queries built by concatenating `selection` strings. Use `selectionArgs`. Also flag a `FileProvider` with a `<root-path>` or overly broad paths.
- `[H]` Tokens or personal data in plain `SharedPreferences`, external storage or logs (`Log.d` / `Log.v` in release). Use Android Keystore-backed encryption. Jetpack Security's `EncryptedSharedPreferences` is deprecated, so prefer Keystore with Tink, or DataStore with encryption.
- `[H]` API secrets in `strings.xml`, `BuildConfig` or `gradle.properties` that ship in the APK. They can be extracted.
- `[H]` A network security config with `cleartextTrafficPermitted="true"` or user-installed CAs trusted in release.
- `[H]` Deep link or App Link parameters trusted without validation.
- `[M]` `allowBackup` or `dataExtractionRules` exposing sensitive data; sensitive screens without `FLAG_SECURE`.

## Correctness and lifecycle
- `[H]` Network calls, disk, database or other heavy work on the main thread (crashes with `NetworkOnMainThreadException`, or freezes the app with an ANR).
- `[H]` Context leaks: an `Activity`, `View` or `Fragment` held in a static field, a singleton or a `ViewModel`.
- `[H]` In Fragments, observing `LiveData` or collecting Flows with `this` instead of `viewLifecycleOwner`; collecting Flows in the UI without `repeatOnLifecycle`.
- `[H]` State lost on rotation or process death: UI state not kept in a `ViewModel` or `SavedStateHandle`.
- `[H]` Runtime permissions used without a request, or without handling denial (including `POST_NOTIFICATIONS` on 33+); a foreground service started without the declared `foregroundServiceType` (crashes on 34+).
- `[H]` Fragment transactions committed after `onSaveInstanceState` (`IllegalStateException`).
- `[M]` `GlobalScope` used instead of `viewModelScope` or `lifecycleScope`; `AsyncTask` or other deprecated APIs in new code.

## Jetpack Compose
- `[H]` Side effects such as coroutines, I/O or analytics run directly in a composable body. Use `LaunchedEffect`, `DisposableEffect` or `SideEffect`.
- `[H]` `mutableStateOf` not wrapped in `remember`, or `remember` keys missing so values go stale.
- `[M]` Unstable parameters, such as lambdas recreated every time or mutable collections, causing excess recomposition; `LazyColumn` items without a stable `key`.

## Performance
- `[H]` Bitmaps loaded at full resolution, which can run out of memory. Use Coil or Glide, or `inSampleSize`.
- `[M]` `RecyclerView` updated with `notifyDataSetChanged()` instead of `ListAdapter` / `DiffUtil`; R8 or minify disabled for release without a reason.

## Accessibility
- `[H]` Actionable `ImageView`, `ImageButton` or `Icon` without a `contentDescription`. Use `null` for decorative images.
- `[M]` Touch targets smaller than 48×48 dp; text sized in `dp` instead of `sp`; custom clickables without a semantic role (`Modifier.semantics`, `Role.Button`); low contrast.

## Maintainability
- `[M]` Hardcoded user-facing strings instead of `strings.xml` (it blocks localization); Gradle dependencies with known CVEs, or unpinned versions outside the version catalog.
- `[L]` Lint warnings suppressed without a reason; unused resources.
