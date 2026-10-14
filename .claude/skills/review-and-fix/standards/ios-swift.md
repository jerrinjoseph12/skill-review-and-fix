# Apple iOS / Swift Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers Swift, Objective-C, SwiftUI, UIKit, and Apple platform config (`Info.plist`, entitlements, privacy manifests). The same rules apply to iPadOS, macOS, watchOS and visionOS where relevant.

## Authorities
- Apple *Swift API Design Guidelines*
- *The Swift Programming Language* (concurrency chapter)
- Apple Platform Security guide
- App Store Review Guidelines
- Human Interface Guidelines (accessibility)
- OWASP MASVS v2 and the OWASP Mobile Top 10

## Tooling to look for and run
- `xcodebuild test -scheme <scheme> -destination '<simulator>'`, or `swift build` / `swift test` for packages
- SwiftLint, SwiftFormat
- The Xcode static analyzer (`xcodebuild analyze`)

## Version awareness
Check the deployment target (`IPHONEOS_DEPLOYMENT_TARGET`, the `platforms:` in `Package.swift`) and the Swift language mode:
- APIs newer than the deployment target need `if #available` or `@available`.
- In Swift 6 language mode, data-race safety is enforced at compile time. In Swift 5 mode, treat concurrency warnings as findings when the project has strict concurrency checking turned on.

## Security
- `[C]` A `URLSessionDelegate` that accepts any server trust, for example by calling `completionHandler(.useCredential, ...)` without evaluating it.
- `[C]` `WKWebView` that loads untrusted content and exposes a `WKScriptMessageHandler` bridge that doesn't check the message's origin.
- `[H]` Tokens, passwords or personal data stored in `UserDefaults`, plist files or plain files instead of the **Keychain**. Also flag Keychain items using `kSecAttrAccessibleAlways` (deprecated); prefer the `...ThisDeviceOnly` classes.
- `[H]` API secrets in source, `Info.plist` or bundled config. They can be extracted from the app package, so move secret operations to a backend.
- `[H]` App Transport Security weakened: `NSAllowsArbitraryLoads` set to `true`, or broad `NSExceptionDomains` entries, in release builds.
- `[H]` Parameters from deep links or universal links (URL schemes, `onOpenURL`) trusted without validation, such as navigating to admin screens or loading the URL into a web view.
- `[H]` Objective-C: user-controlled format strings (`[NSString stringWithFormat:userInput]`, `NSLog(userInput)`).
- `[M]` Personal data logged with `print` or `NSLog`, or with `os_log` / `Logger` without a `.private` privacy level.
- `[M]` Sensitive screens not hidden in the app switcher snapshot; sensitive data copied to the general pasteboard.

## Privacy and App Store
- `[H]` A missing usage-description string (`NSCameraUsageDescription`, `NSLocationWhenInUseUsageDescription`, and so on) for a protected API the code uses. The app **crashes** when it calls that API.
- `[M]` A missing or out-of-date `PrivacyInfo.xcprivacy` declaration for required-reason APIs (`UserDefaults`, file timestamps, system boot time). App Store Connect rejects the upload.

## Correctness
- `[H]` Force unwrap `!`, `try!` or `as!` on data from the network, disk, user input or other external sources.
- `[H]` Retain cycles:
  - closures stored on `self` (callbacks, timers, `NotificationCenter` blocks, Combine `sink`) that capture `self` strongly. Use `[weak self]`.
  - delegate properties that aren't `weak`
- `[H]` UI updated off the main thread. Use `@MainActor` or `MainActor.run`.
- `[H]` `DispatchQueue.main.sync` called from the main thread, which deadlocks.
- `[H]` Actor reentrancy: assuming state is unchanged across an `await` inside an actor or `@MainActor` method.
- `[H]` Core Data managed objects or contexts used across threads. Use `perform` and pass `NSManagedObjectID`.
- `[H]` `DateFormatter` for fixed machine formats without `locale = Locale(identifier: "en_US_POSIX")` and an explicit time zone.
- `[M]` Combine `AnyCancellable` not stored, so the subscription is cancelled immediately; `Task {}` started without handling cancellation for long-running work.
- `[M]` Sendable or data-race warnings silenced with `@unchecked Sendable` or `nonisolated(unsafe)` without a justification.
- `[M]` Implicitly unwrapped optionals (`var x: T!`) outside `@IBOutlet`.

## SwiftUI
- `[H]` `@ObservedObject` used for an object the view creates. It's recreated on every re-render and loses its state; use `@StateObject`, or `@State` with `@Observable`.
- `[H]` Side effects (network calls, writes) run directly in `body`. Use `.task` or `.onAppear`, and note that `onAppear` can fire more than once.
- `[M]` `ForEach` or `List` with unstable identities, such as `id: \.self` on non-unique values or indices; heavy computation in `body`.

## Performance
- `[M]` Large images decoded at full size for thumbnails. Downsample them with ImageIO.
- `[M]` Formatters (`DateFormatter`, `NumberFormatter`) created repeatedly in cells or `body`.
- `[M]` Synchronous I/O or networking on the main thread.

## Accessibility
- `[H]` Icon-only buttons or images that act as controls, without an `accessibilityLabel`.
- `[M]` Fixed font sizes that ignore Dynamic Type. Use text styles or `UIFontMetrics`.
- `[M]` Decorative images not hidden from VoiceOver; custom controls without accessibility traits; touch targets under 44×44 pt.

## Maintainability
- `[L]` Naming not in line with the Swift API Design Guidelines (clarity at the point of use, argument labels); `print` debugging left in; massive view controllers that should be split.
