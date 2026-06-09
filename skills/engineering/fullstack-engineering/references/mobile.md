# Mobile Best Practices

## React Native

### Architecture

Use the latest React Native with the **New Architecture** (Fabric renderer, TurboModules) where
the project supports it. Expo is preferred for new projects unless native modules require bare
workflow.

```
src/
├── app/                      # Expo Router file-based routing (if Expo)
│   ├── (tabs)/
│   │   ├── _layout.tsx
│   │   ├── home.tsx
│   │   └── profile.tsx
│   └── _layout.tsx
├── components/
│   ├── ui/                   # design system primitives
│   └── features/
├── hooks/
├── services/                 # API clients, storage wrappers
├── stores/                   # Zustand stores
├── types/
├── utils/
└── constants/
```

### Key Patterns

- **Navigation**: Expo Router (file-based, recommended) or React Navigation. Type all route params.
- **State**: Zustand for global state, TanStack Query for server state. Same patterns as web React.
- **Styling**: StyleSheet.create for performance. NativeWind (Tailwind for RN) is acceptable for
  rapid development. Avoid inline styles in production code.
- **Platform-specific code**: use `.ios.tsx` / `.android.tsx` suffixes or `Platform.select()` for
  platform branching. Keep platform-specific logic in dedicated files.
- **TypeScript**: strict mode, same standards as web TypeScript.
- **UUIDv7 for entity IDs**: use the `uuidv7` npm package for generating IDs client-side before
  persisting to any backend or local storage. Never rely on server-generated auto-increment IDs
  for offline-first or optimistic update patterns.
- **Testing**: Jest + React Native Testing Library for component tests. Detox for E2E.
- **Performance**: use `React.memo`, `FlatList` (not ScrollView for long lists), avoid anonymous
  functions in render, use `useCallback` for event handlers passed to lists.

## Kotlin (Android)

### Architecture

Follow **MVVM + Clean Architecture** layered approach:

```
app/
└── src/main/java/com/example/app/
    ├── data/
    │   ├── remote/           # Retrofit/Ktor API definitions
    │   ├── local/            # Room DAOs, DataStore
    │   ├── repository/       # Repository implementations
    │   └── model/            # data layer DTOs
    ├── domain/
    │   ├── model/            # domain entities
    │   ├── repository/       # repository interfaces
    │   └── usecase/          # use case classes
    ├── presentation/
    │   ├── ui/
    │   │   ├── screens/      # Compose screens
    │   │   ├── components/   # reusable Compose components
    │   │   └── theme/
    │   └── viewmodel/
    ├── di/                   # Hilt/Koin modules
    └── util/
```

### Key Patterns

- **UI**: Jetpack Compose exclusively. No XML layouts for new code.
- **DI**: Hilt (preferred, compile-time safe) or Koin. All dependencies injected — no service
  locators or manual instantiation in ViewModels.
- **Networking**: Retrofit with Kotlin Serialization (preferred) or Moshi. Ktor-client is acceptable
  for KMP projects.
- **Local storage**: Room for structured data, DataStore (Preferences or Proto) for key-value.
  Never SharedPreferences in new code.
- **Coroutines & Flow**: all async work uses Kotlin Coroutines. Expose data from repositories as
  `Flow<T>`. ViewModels use `StateFlow` for UI state and collect in Compose with
  `collectAsStateWithLifecycle()`.
- **Use Cases**: single-responsibility classes with `operator fun invoke()`. Each use case represents
  one business action.
- **UUIDv7 for all entity IDs**: use `com.fasterxml.uuid:java-uuid-generator` (JUG) library for
  UUIDv7 generation. In domain models, IDs are always `UUID` type generated application-side:
  ```kotlin
  import com.fasterxml.uuid.Generators
  val id: UUID = Generators.timeBasedEpochGenerator().generate() // UUIDv7
  ```
  Room entities use `UUID` as the primary key type with a TypeConverter.
- **Error handling**: use sealed classes or `kotlin.Result` for operation outcomes. Never catch
  `Exception` broadly without rethrowing `CancellationException`.

### Testing

- **Unit tests**: JUnit 5, MockK for mocking, Turbine for testing Flows, Kotlin Coroutines Test
  for coroutine testing.
- **UI tests**: Compose Testing with `createComposeRule()`. Test by semantics, not by implementation.
- **Integration**: Hilt testing with `@HiltAndroidTest`. Use in-memory Room databases.
- Convention: test classes mirror source structure in `src/test/` (unit) and `src/androidTest/`
  (instrumented).

### Build

- Gradle Kotlin DSL (`build.gradle.kts`).
- Version catalogs (`libs.versions.toml`) for dependency management.
- ProGuard/R8 rules for release builds.

## Swift / iOS

### Architecture

Use **MVVM + Clean** with the same domain/application/infrastructure split as
Kotlin Android. SwiftUI for new UIs, UIKit only for components that need it
(camera, advanced animations, third-party SDKs).

```
App/
├── Domain/                 # entities, value objects, domain errors
│   ├── Models/
│   └── Errors/
├── Application/            # use cases + port protocols
│   ├── UseCases/
│   └── Repositories/       # protocols only
├── Infrastructure/
│   ├── Persistence/        # SwiftData / Core Data / GRDB
│   ├── Network/            # URLSession-based API clients
│   └── Storage/            # Keychain wrappers, UserDefaults
├── Presentation/
│   ├── Screens/            # SwiftUI views
│   ├── Components/
│   ├── ViewModels/         # @Observable / ObservableObject
│   └── Theme/
└── Composition/            # DI wiring
```

### Key Patterns

- **Concurrency**: Swift Concurrency (`async/await`, `Task`, actors). Avoid
  Combine for new code unless you have a strong reason; structured concurrency
  composes better. Use `@MainActor` to mark UI-bound types.
- **State**: `@Observable` macro (iOS 17+) for view models. SwiftData for
  persistence on iOS 17+; Core Data with `NSPersistentContainer` on older
  targets. GRDB.swift is acceptable when you want SQL control.
- **Networking**: `URLSession` + `Codable`. Wrap in a repository protocol —
  view models depend on the protocol, not on `URLSession`. Use Alamofire
  only if it provides specific value (multipart, retry policy, network
  reachability).
- **Dependency injection**: constructor injection via initializers. Compose
  the graph in `App` struct or a dedicated `Composition.swift`. Avoid
  Resolver/Swinject unless the project is large.
- **UUIDv7 for entity IDs**: Swift's `UUID` is v4 only. Use the
  [swift-uuid](https://github.com/0xLeif/swift-uuid) package or generate via
  `Foundation`'s bytes manually with the v7 layout (RFC 9562). Generate
  client-side for any persisted entity, including offline-created records.
- **Error handling**: `Result<Success, Failure>` or `throws` with typed errors
  via `enum DomainError: Error`. Avoid `NSError` in new Swift code.

### Testing

- **XCTest** for unit and UI tests. **Swift Testing** (Apple's new framework,
  Xcode 16+) is the future — adopt it for new test targets when you can require
  Xcode 16.
- Mock protocols by hand or with **Mockable** / **Sourcery**. Don't use
  reflection-based mocking — Swift type system makes hand-rolled mocks pleasant.
- UI tests: `XCUIApplication` with accessibility identifiers on every
  testable element.

## Cross-Platform Mobile Concerns (iOS + Android + RN)

These apply to every mobile project regardless of the stack above.

### Secure Storage

- **iOS**: Keychain Services for tokens, refresh tokens, encryption keys.
  Never `UserDefaults` for anything sensitive — it's a plist on disk.
- **Android**: `EncryptedSharedPreferences` (deprecated in API 36, but still
  the easiest path) or **Tink** + `Keystore`-backed master key for new code.
  Never `SharedPreferences` raw for secrets.
- **React Native**: `expo-secure-store` (Expo) or `react-native-keychain`
  (bare). Both wrap the platform-native APIs above.
- **Pin certificates** for any API holding user data — use `URLSessionDelegate`
  on iOS, `NetworkSecurityConfig` on Android, or
  `react-native-cert-pinner` for RN. Rotate via remote config; failing pin →
  hard fail with a "your app is out of date" prompt.

### Crash & Error Reporting

- **Sentry** (preferred — one SDK across iOS, Android, RN, and your backend
  per `references/observability.md`) or **Firebase Crashlytics**. Wire up in
  the `App` initializer / `Application.onCreate()` / `App.tsx` before any
  business logic runs.
- Tag every event with `request_id` from the API response so client errors
  correlate with backend traces.
- Strip PII before send: configure the SDK's `beforeSend` hook to drop
  `email`, raw JWTs, and any user-typed input that could leak data.

### Offline-First Patterns

- **Local-first source of truth**: write to local DB first (SwiftData / Room /
  WatermelonDB / SQLite), then push to the server. Reads always come from
  local. UI never blocks on the network.
- **UUIDv7 for client-generated IDs**: client creates the entity and its ID
  immediately. Server accepts the client's ID on insert. UUIDv7's
  time-sortability gives natural ordering even before sync.
- **Sync engine**: implement a queue of mutations with `client_op_id`
  (UUIDv7) so the server can dedupe. Conflict resolution is
  domain-specific — last-write-wins is the default; CRDTs only when you
  genuinely have concurrent multi-device editing.
- **Connection state**: surface online/offline in the UI. Distinguish "syncing"
  from "synced" from "sync failed (will retry)" — users hate silent failure.

### App Size & Performance

- **App Store / Play Store budget**: target IPA / AAB ≤ 50 MB compressed.
  Audit with Xcode's "App Size" report and Android's APK Analyzer.
- **Asset optimization**: AVIF/WebP for images, AAC/Opus for audio,
  HEVC for video. Don't ship PNG when WebP exists.
- **Cold start**: target ≤ 2 seconds on mid-range devices. Lazy-load anything
  not needed on the first screen. Pre-render the splash with platform tools
  (LaunchScreen.storyboard, splash drawable) — not your React tree.
- **Bundle splitting**: RN's Hermes + bundle splitting for non-critical
  routes. Native code: don't ship debug symbols in release.
