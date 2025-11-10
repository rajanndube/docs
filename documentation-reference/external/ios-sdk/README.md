# Stringboot iOS SDK - Integration Guide

**Verified for:** v2.0 (String-Sync v2 Protocol)
**Last verified on:** 2025-11-07
**Audience:** iOS developers integrating the SDK
**Source:** Based on verified demo app implementation

---

## Table of Contents

1. [Overview](#overview)
2. [Installation & Setup](#installation--setup)
3. [Initialization](#initialization)
4. [Usage Examples](#usage-examples)
5. [Error Handling](#error-handling)
6. [Caching & Offline Behavior](#caching--offline-behavior)
7. [Best Practices](#best-practices)
8. [FAQ / Troubleshooting](#faq--troubleshooting)
9. [Changelog](#changelog)

---

## Overview

The **Stringboot iOS SDK** provides high-performance internationalization (i18n) for iOS applications. It implements a sophisticated multi-layered caching system with offline support and smart memory management to deliver fast string lookups while minimizing network usage.

### Key Features

- **Offline-First:** Works seamlessly without network connectivity
- **Smart Caching:** LRU memory cache with frequency-based prioritization
- **Delta Sync:** Only downloads changed strings to save bandwidth
- **Fast Lookups:** Target <300ms on 4G networks
- **Memory Efficient:** Automatic cache management based on memory pressure
- **Reactive Updates:** Combine publishers and SwiftUI @ObservedObject integration
- **Async/Await Support:** Modern concurrency patterns with Swift 5.9+
- **SwiftUI Native:** Auto-updating SBText components
- **Language Change:** Seamless language switching with loading states

### Platform Support

| Feature | Requirement |
|---------|------------|
| **Min iOS** | 14.0+ |
| **Min macOS** | 11.0+ |
| **Swift** | 5.9+ |
| **Xcode** | 15.0+ |
| **SDK Version** | 2.0 |
| **License** | Apache 2.0 |

---

## Installation & Setup

### Step 1: Add Swift Package Dependency

In Xcode: **File → Add Packages → Enter repository URL**

```
https://github.com/Stringboot-SDK/stringboot-ios-sdk.git
```

Or add to `Package.swift`:

```swift
dependencies: [
    .package(
        url: "https://github.com/Stringboot-SDK/stringboot-ios-sdk.git",
        from: "2.0.0"
    )
]
```

### Step 2: Add Target Dependency

In Xcode: Select your app target → **Build Phases → Link Binary With Libraries → Add StringbootSDK**

Or in `Package.swift`:

```swift
targets: [
    .target(
        name: "YourApp",
        dependencies: ["StringbootSDK"]
    )
]
```

### Step 3: Import in Your Code

```swift
import StringbootSDK
```

---

## Initialization

### SwiftUI App Initialization (Verified Pattern)

This is the **exact initialization pattern** from the working demo app:

```swift
import SwiftUI
import StringbootSDK

@main
struct StringbootDemoApp: App {

    init() {
        // Configure SDK logging
        StringbootLogger.isLoggingEnabled = true  // Set to false to disable all SDK logs
        StringbootLogger.logLevel = .debug        // Verbosity level when logging is enabled

        // Initialize Stringboot SDK
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "9bc7d015-c9e6-42ff-aec2-8c33b13284c0",
            baseURL: "https://api.stringboot.com",
            autoSync: true  // Auto-fetch strings on startup
        )

        // Set saved language or use device locale
        let currentLang = UserDefaults.standard.string(forKey: "com.stringboot.currentLanguage")
            ?? StringProvider.shared.deviceLocale()
        StringProvider.shared.setLocale(currentLang)
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

### Initialization Parameters Explained

| Parameter | Type | Description |
|-----------|------|-------------|
| `cacheSize` | Int | Maximum number of strings in memory cache (recommended: 1000) |
| `apiToken` | String | Your Stringboot API token from the dashboard |
| `baseURL` | String | API endpoint (default: `https://api.stringboot.com`) |
| `autoSync` | Bool | Automatically sync strings on app launch (recommended: true) |

### Logger Configuration

```swift
// Enable detailed logging for debugging
StringbootLogger.isLoggingEnabled = true
StringbootLogger.logLevel = .debug  // Options: .debug, .info, .warning, .error

// Disable logging for production
StringbootLogger.isLoggingEnabled = false
```

---

## Usage Examples

### Example 1: Complete ContentView with Loading States (Production-Ready)

This is the **exact pattern** from the demo app showing all three states:

```swift
import SwiftUI
import Combine
import StringbootSDK

struct ContentView: View {

    @ObservedObject var stringProvider = StringProvider.shared
    @State private var selectedLanguage: String = ""

    var body: some View {
        NavigationView {
            Group {
                if stringProvider.isReady {
                    mainContent
                } else if let error = stringProvider.initializationError {
                    errorView(error)
                } else {
                    loadingView
                }
            }
            .navigationTitle("Stringboot Demo")
            .onAppear {
                selectedLanguage = stringProvider.currentLanguage ?? stringProvider.deviceLocale()
            }
            .onChange(of: stringProvider.currentLanguage) { _, newLang in
                if let newLang = newLang {
                    selectedLanguage = newLang
                }
            }
        }
    }

    private var loadingView: some View {
        VStack(spacing: 20) {
            ProgressView()
                .scaleEffect(2.0)
                .padding()

            Text("Loading strings...")
                .font(.headline)
                .foregroundColor(.secondary)
        }
    }

    private func errorView(_ message: String) -> some View {
        VStack(spacing: 20) {
            Image(systemName: "exclamationmark.triangle.fill")
                .font(.system(size: 60))
                .foregroundColor(.orange)
                .padding()

            Text("Initialization Failed")
                .font(.title2)
                .fontWeight(.bold)

            Text(message)
                .font(.body)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
                .padding(.horizontal, 32)

            Button {
                Task {
                    await stringProvider.retryInitialization()
                }
            } label: {
                HStack {
                    Image(systemName: "arrow.clockwise")
                    Text("Retry")
                }
                .font(.headline)
            }
            .buttonStyle(.borderedProminent)
            .padding(.top)
        }
    }

    private var mainContent: some View {
        VStack(spacing: 20) {
            // Your app content here
            Text("SDK Ready!")
        }
    }
}
```

### Example 2: Language Change with Full-Screen Overlay (Production Pattern)

This shows the **exact language change handling** from the demo app:

```swift
struct ContentView: View {

    @ObservedObject var stringProvider = StringProvider.shared
    @State private var selectedLanguage: String = ""

    var body: some View {
        NavigationView {
            VStack {
                // Your content here
            }
            .alert("Language Change Failed", isPresented: .constant(stringProvider.languageChangeError != nil)) {
                Button("OK") {
                    selectedLanguage = stringProvider.currentLanguage ?? stringProvider.deviceLocale()
                }
            } message: {
                Text(stringProvider.languageChangeError ?? "Unknown error")
            }
            .overlay {
                if stringProvider.isChangingLanguage {
                    ZStack {
                        Color.black.opacity(0.4)
                            .ignoresSafeArea()

                        VStack(spacing: 16) {
                            ProgressView()
                                .scaleEffect(1.5)
                                .tint(.white)

                            Text("Changing language...")
                                .foregroundColor(.white)
                                .font(.headline)
                        }
                        .padding(32)
                        .background(Color(.systemBackground))
                        .cornerRadius(16)
                        .shadow(radius: 20)
                    }
                }
            }
        }
    }
}
```

### Example 3: Language Picker with Disabled State (Production Pattern)

This is the **exact Picker integration** from the demo app:

```swift
struct ContentView: View {

    @ObservedObject var stringProvider = StringProvider.shared
    @State private var selectedLanguage: String = ""

    var body: some View {
        VStack {
            // Language Picker - Just observe and call SDK
            Picker("Language", selection: $selectedLanguage) {
                ForEach(stringProvider.availableLanguages, id: \.code) { language in
                    Text(language.name).tag(language.code)
                }
            }
            .pickerStyle(.segmented)
            .padding()
            .disabled(stringProvider.isChangingLanguage)
            .onChange(of: selectedLanguage) { _, newLanguage in
                Task {
                    await stringProvider.changeLanguage(to: newLanguage)
                }
            }
        }
        .onAppear {
            selectedLanguage = stringProvider.currentLanguage ?? stringProvider.deviceLocale()
        }
        .onChange(of: stringProvider.currentLanguage) { _, newLang in
            if let newLang = newLang {
                selectedLanguage = newLang
            }
        }
    }
}
```

### Example 4: SBText Usage (Auto-Updating Strings)

This shows the **actual SBText usage** from the demo app:

```swift
import SwiftUI
import StringbootSDK

struct ContentView: View {
    var body: some View {
        VStack(alignment: .leading, spacing: 15) {

            // Using SBText - SDK handles everything automatically:
            // - Language detection (follows picker selection)
            // - Auto-updates when strings refresh from network
            // - Falls back to Localizable.xcstrings if backend offline
            SBText("welcome_message")
                .font(.title)
                .fontWeight(.bold)

            SBText("title_activity_main")
                .font(.body)
                .foregroundColor(.secondary)

            SBText("app_description")
                .font(.subheadline)
                .foregroundColor(.gray)
        }
        .padding()
    }
}
```

### Example 5: Refresh from Network Button

This is the **exact refresh button** from the demo app:

```swift
struct ContentView: View {

    @ObservedObject var stringProvider = StringProvider.shared
    @State private var selectedLanguage: String = ""

    var body: some View {
        VStack {
            Button("Refresh from Network") {
                Task {
                    let lang = selectedLanguage.isEmpty ? nil : selectedLanguage
                    StringbootLogger.i("Refresh button: fetching \(lang ?? "device locale")")
                    let success = await stringProvider.refreshFromNetwork(lang: lang, forceRefresh: true)
                    StringbootLogger.i("Refresh button: result = \(success)")
                    // No need to toggle refreshTrigger - SDK auto-updates via lastUpdate
                }
            }
            .buttonStyle(.borderedProminent)

            Button("Clear Cache") {
                stringProvider.clearCache(clearDatabase: false)
                // No need to toggle refreshTrigger - SDK auto-updates
            }
            .buttonStyle(.bordered)
        }
        .padding()
    }
}
```

### Example 6: Cache Statistics View (Production Component)

This is the **complete CacheStatsView** from the demo app:

```swift
import SwiftUI
import StringbootSDK

struct CacheStatsView: View {

    @State private var stats: CacheStats?

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text("Cache Statistics")
                .font(.headline)

            if let stats = stats {
                HStack {
                    Text("Size:")
                    Spacer()
                    Text("\(stats.memorySize) / \(stats.memoryMaxSize)")
                }

                HStack {
                    Text("Hit Rate:")
                    Spacer()
                    Text(String(format: "%.2f%%", stats.hitRate * 100))
                }

                HStack {
                    Text("Hits:")
                    Spacer()
                    Text("\(stats.memoryHitCount)")
                }

                HStack {
                    Text("Misses:")
                    Spacer()
                    Text("\(stats.memoryMissCount)")
                }

                HStack {
                    Text("Evictions:")
                    Spacer()
                    Text("\(stats.memoryEvictionCount)")
                }
            }
        }
        .font(.caption)
        .padding()
        .background(Color.gray.opacity(0.1))
        .cornerRadius(8)
        .padding(.horizontal)
        .onAppear {
            updateStats()
        }
        .onReceive(Timer.publish(every: 2.0, on: .main, in: .common).autoconnect()) { _ in
            updateStats()
        }
    }

    private func updateStats() {
        stats = StringProvider.shared.getCacheStats()
    }
}
```

### Example 7: Complete App Structure (All Patterns Combined)

Here's a **production-ready example** combining all the verified patterns:

```swift
import SwiftUI
import StringbootSDK

@main
struct MyApp: App {

    init() {
        // Configure logging
        StringbootLogger.isLoggingEnabled = true
        StringbootLogger.logLevel = .debug

        // Initialize SDK
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "YOUR_API_TOKEN_HERE",
            baseURL: "https://api.stringboot.com",
            autoSync: true
        )

        // Restore saved language
        let savedLang = UserDefaults.standard.string(forKey: "com.stringboot.currentLanguage")
            ?? StringProvider.shared.deviceLocale()
        StringProvider.shared.setLocale(savedLang)
    }

    var body: some Scene {
        WindowGroup {
            MainView()
        }
    }
}

struct MainView: View {

    @ObservedObject var stringProvider = StringProvider.shared
    @State private var selectedLanguage: String = ""

    var body: some View {
        NavigationView {
            Group {
                if stringProvider.isReady {
                    contentView
                } else if let error = stringProvider.initializationError {
                    errorView(error)
                } else {
                    loadingView
                }
            }
            .navigationTitle("My App")
            .onAppear {
                selectedLanguage = stringProvider.currentLanguage ?? stringProvider.deviceLocale()
            }
            .onChange(of: stringProvider.currentLanguage) { _, newLang in
                if let newLang = newLang {
                    selectedLanguage = newLang
                    UserDefaults.standard.set(newLang, forKey: "com.stringboot.currentLanguage")
                }
            }
            .alert("Language Change Failed", isPresented: .constant(stringProvider.languageChangeError != nil)) {
                Button("OK") {
                    selectedLanguage = stringProvider.currentLanguage ?? stringProvider.deviceLocale()
                }
            } message: {
                Text(stringProvider.languageChangeError ?? "Unknown error")
            }
            .overlay {
                if stringProvider.isChangingLanguage {
                    languageChangeOverlay
                }
            }
        }
    }

    private var loadingView: some View {
        VStack(spacing: 20) {
            ProgressView()
                .scaleEffect(2.0)
                .padding()

            Text("Loading strings...")
                .font(.headline)
                .foregroundColor(.secondary)
        }
    }

    private func errorView(_ message: String) -> some View {
        VStack(spacing: 20) {
            Image(systemName: "exclamationmark.triangle.fill")
                .font(.system(size: 60))
                .foregroundColor(.orange)
                .padding()

            Text("Initialization Failed")
                .font(.title2)
                .fontWeight(.bold)

            Text(message)
                .font(.body)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
                .padding(.horizontal, 32)

            Button {
                Task {
                    await stringProvider.retryInitialization()
                }
            } label: {
                HStack {
                    Image(systemName: "arrow.clockwise")
                    Text("Retry")
                }
                .font(.headline)
            }
            .buttonStyle(.borderedProminent)
            .padding(.top)
        }
    }

    private var contentView: some View {
        VStack(spacing: 20) {

            // Language Picker
            Picker("Language", selection: $selectedLanguage) {
                ForEach(stringProvider.availableLanguages, id: \.code) { language in
                    Text(language.name).tag(language.code)
                }
            }
            .pickerStyle(.segmented)
            .padding()
            .disabled(stringProvider.isChangingLanguage)
            .onChange(of: selectedLanguage) { _, newLanguage in
                Task {
                    await stringProvider.changeLanguage(to: newLanguage)
                }
            }

            Divider()

            // Auto-updating string views
            VStack(alignment: .leading, spacing: 15) {
                SBText("welcome_message")
                    .font(.title)
                    .fontWeight(.bold)

                SBText("app_description")
                    .font(.body)
                    .foregroundColor(.secondary)
            }
            .padding()

            Spacer()

            // Cache Statistics
            CacheStatsView()

            // Actions
            VStack(spacing: 10) {
                Button("Refresh from Network") {
                    Task {
                        let lang = selectedLanguage.isEmpty ? nil : selectedLanguage
                        _ = await stringProvider.refreshFromNetwork(lang: lang, forceRefresh: true)
                    }
                }
                .buttonStyle(.borderedProminent)

                Button("Clear Cache") {
                    stringProvider.clearCache(clearDatabase: false)
                }
                .buttonStyle(.bordered)
            }
            .padding()
        }
    }

    private var languageChangeOverlay: some View {
        ZStack {
            Color.black.opacity(0.4)
                .ignoresSafeArea()

            VStack(spacing: 16) {
                ProgressView()
                    .scaleEffect(1.5)
                    .tint(.white)

                Text("Changing language...")
                    .foregroundColor(.white)
                    .font(.headline)
            }
            .padding(32)
            .background(Color(.systemBackground))
            .cornerRadius(16)
            .shadow(radius: 20)
        }
    }
}
```

### Example 8: Async/Await String Retrieval

For cases where you need direct string access:

```swift
struct MyView: View {
    @State private var welcomeText = ""

    var body: some View {
        Text(welcomeText)
            .task {
                // Get string with async/await
                welcomeText = await StringProvider.shared.get(
                    "welcome_message",
                    lang: "en"
                )
            }
    }
}
```

---

## Error Handling

### Three States Pattern (From Demo App)

The SDK provides three distinct states:

1. **Loading State** - SDK is initializing
2. **Error State** - Initialization failed
3. **Ready State** - SDK is ready to use

```swift
@ObservedObject var stringProvider = StringProvider.shared

var body: some View {
    Group {
        if stringProvider.isReady {
            // SDK ready - show main content
            mainContent
        } else if let error = stringProvider.initializationError {
            // SDK initialization failed
            errorView(error)
        } else {
            // SDK still loading
            loadingView
        }
    }
}
```

### Retry Failed Initialization

```swift
Button("Retry") {
    Task {
        await stringProvider.retryInitialization()
    }
}
```

### Handle Language Change Errors

```swift
.alert("Language Change Failed", isPresented: .constant(stringProvider.languageChangeError != nil)) {
    Button("OK") {
        // Reset to current language
        selectedLanguage = stringProvider.currentLanguage ?? stringProvider.deviceLocale()
    }
} message: {
    Text(stringProvider.languageChangeError ?? "Unknown error")
}
```

### Disable UI During Language Change

```swift
.disabled(stringProvider.isChangingLanguage)
```

### Observable Properties

The StringProvider exposes these @Published properties for reactive UI:

| Property | Type | Description |
|----------|------|-------------|
| `isReady` | Bool | SDK successfully initialized |
| `initializationError` | String? | Initialization error message |
| `isChangingLanguage` | Bool | Language change in progress |
| `languageChangeError` | String? | Language change error message |
| `currentLanguage` | String? | Current active language code |
| `availableLanguages` | [ActiveLanguage] | List of available languages |

---

## Caching & Offline Behavior

### Three-Layer Cache System

```
┌─────────────────────────────────┐
│  Layer 1: In-Memory Cache       │  Access: <1ms
│  (LRU, 1000 entries by default) │
└─────────────────────────────────┘
         ↓ (miss)
┌─────────────────────────────────┐
│  Layer 2: Core Data             │  Access: 5-20ms
│  (Persistent, SQLite-based)     │
└─────────────────────────────────┘
         ↓ (miss)
┌─────────────────────────────────┐
│  Layer 3: Network Sync          │  Access: 100-500ms
│  (String-Sync v2 with ETag)     │
└─────────────────────────────────┘
```

### Cache Management (From Demo App)

**Get cache statistics:**

```swift
let stats = StringProvider.shared.getCacheStats()
print("Memory: \(stats.memorySize) / \(stats.memoryMaxSize)")
print("Hit rate: \(String(format: "%.2f%%", stats.hitRate * 100))")
print("Hits: \(stats.memoryHitCount)")
print("Misses: \(stats.memoryMissCount)")
print("Evictions: \(stats.memoryEvictionCount)")
```

**Clear cache:**

```swift
// Clear memory cache only (database intact)
stringProvider.clearCache(clearDatabase: false)

// Clear everything (memory + database)
stringProvider.clearCache(clearDatabase: true)
```

**Refresh from network:**

```swift
Task {
    let success = await stringProvider.refreshFromNetwork(
        lang: "en",
        forceRefresh: true  // Bypass ETag check
    )
    if success {
        print("Strings refreshed")
    }
}
```

### Offline Behavior

**Complete offline functionality:**

1. **No Network Required:** SDK works entirely offline if data is cached
2. **Persistent Storage:** Core Data survives app restarts
3. **Graceful Degradation:** Network errors automatically fall back to cache
4. **Auto-sync on Launch:** When `autoSync: true`, attempts sync but never blocks

**Offline example:**

```swift
// After initial sync, strings are cached permanently
// Turn off network → Open app → All strings load from cache instantly

let text = await StringProvider.shared.get("welcome_message", lang: "en")
// Returns cached value, no error thrown, no network request
```

---

## Best Practices

### 1. Initialize in App Init (Not in View)

**✅ Good:**

```swift
@main
struct MyApp: App {
    init() {
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "token",
            baseURL: "https://api.stringboot.com",
            autoSync: true
        )
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

**❌ Bad:**

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Text("Loading...")
                .onAppear {
                    // Too late - blocks UI
                    StringProvider.shared.initialize(...)
                }
        }
    }
}
```

### 2. Use @ObservedObject for Reactive UI

**✅ Good:**

```swift
struct ContentView: View {
    @ObservedObject var stringProvider = StringProvider.shared

    var body: some View {
        if stringProvider.isReady {
            // Automatically updates when state changes
        }
    }
}
```

**❌ Bad:**

```swift
struct ContentView: View {
    var body: some View {
        if StringProvider.shared.isReady {
            // Won't update automatically
        }
    }
}
```

### 3. Use SBText for Auto-Updating Strings

**✅ Good:**

```swift
SBText("welcome_message")
    .font(.title)
// Auto-updates on language change or network sync
```

**❌ Bad:**

```swift
@State var text = ""

Text(text)
    .onAppear {
        Task {
            text = await StringProvider.shared.get("welcome_message", lang: "en")
            // Won't update if language changes
        }
    }
```

### 4. Handle All Three States

**✅ Good:**

```swift
Group {
    if stringProvider.isReady {
        mainContent
    } else if let error = stringProvider.initializationError {
        errorView(error)
    } else {
        loadingView
    }
}
```

**❌ Bad:**

```swift
if stringProvider.isReady {
    mainContent
}
// No loading or error states
```

### 5. Disable UI During Language Change

**✅ Good:**

```swift
Picker("Language", selection: $selectedLanguage) {
    // ...
}
.disabled(stringProvider.isChangingLanguage)
.overlay {
    if stringProvider.isChangingLanguage {
        languageChangeOverlay
    }
}
```

**❌ Bad:**

```swift
Picker("Language", selection: $selectedLanguage) {
    // ...
}
// User can tap multiple times, causing issues
```

### 6. Persist Language Selection

**✅ Good:**

```swift
.onChange(of: stringProvider.currentLanguage) { _, newLang in
    if let newLang = newLang {
        selectedLanguage = newLang
        UserDefaults.standard.set(newLang, forKey: "com.stringboot.currentLanguage")
    }
}

// On app launch
init() {
    let savedLang = UserDefaults.standard.string(forKey: "com.stringboot.currentLanguage")
        ?? StringProvider.shared.deviceLocale()
    StringProvider.shared.setLocale(savedLang)
}
```

**❌ Bad:**

```swift
// No persistence - language resets on app restart
```

### 7. Use forceRefresh Sparingly

**✅ Good:**

```swift
// Only force refresh when user explicitly requests it
Button("Refresh from Network") {
    Task {
        await stringProvider.refreshFromNetwork(lang: lang, forceRefresh: true)
    }
}
```

**❌ Bad:**

```swift
// Force refresh on every view appear
.onAppear {
    Task {
        await stringProvider.refreshFromNetwork(lang: "en", forceRefresh: true)
    }
}
// Wastes bandwidth, battery, and data
```

### 8. Monitor Cache Health

**✅ Good:**

```swift
// Display cache stats for debugging
CacheStatsView()

// In production, log periodically
let stats = StringProvider.shared.getCacheStats()
if stats.hitRate < 0.8 {
    print("Cache hit rate low: \(stats.hitRate)")
}
```

---

## FAQ / Troubleshooting

### Q: Strings show "??key??" instead of translations

**A:** This means the string is not found in cache, Core Data, or network.

**Solutions:**

1. Check if SDK is initialized:
   ```swift
   if StringProvider.shared.isReady {
       // Safe to use
   }
   ```

2. Trigger manual sync:
   ```swift
   Task {
       await StringProvider.shared.refreshFromNetwork(lang: "en", forceRefresh: true)
   }
   ```

3. Verify the key exists in your Stringboot dashboard

4. Check network connectivity and API token validity

5. Check console logs:
   ```swift
   StringbootLogger.isLoggingEnabled = true
   StringbootLogger.logLevel = .debug
   ```

---

### Q: View doesn't update after language change

**A:** You must use `@ObservedObject` with StringProvider.

**Solution:**

```swift
// Correct
@ObservedObject var stringProvider = StringProvider.shared

// Incorrect
let stringProvider = StringProvider.shared
```

---

### Q: How do I test offline behavior?

**A:** Enable Airplane Mode after initial sync:

1. Launch app with network → SDK syncs data
2. Enable Airplane Mode
3. Force quit and relaunch app
4. All strings load from cache (no network needed)

---

### Q: Language picker triggers multiple times

**A:** Disable picker during language change:

```swift
Picker("Language", selection: $selectedLanguage) {
    // ...
}
.disabled(stringProvider.isChangingLanguage)
```

---

### Q: How to show loading overlay during language change?

**A:** Use the exact pattern from the demo app:

```swift
.overlay {
    if stringProvider.isChangingLanguage {
        ZStack {
            Color.black.opacity(0.4)
                .ignoresSafeArea()

            VStack(spacing: 16) {
                ProgressView()
                    .scaleEffect(1.5)
                    .tint(.white)

                Text("Changing language...")
                    .foregroundColor(.white)
                    .font(.headline)
            }
            .padding(32)
            .background(Color(.systemBackground))
            .cornerRadius(16)
            .shadow(radius: 20)
        }
    }
}
```

---

### Q: How to handle language change failures?

**A:** Use alert with error recovery:

```swift
.alert("Language Change Failed", isPresented: .constant(stringProvider.languageChangeError != nil)) {
    Button("OK") {
        selectedLanguage = stringProvider.currentLanguage ?? stringProvider.deviceLocale()
    }
} message: {
    Text(stringProvider.languageChangeError ?? "Unknown error")
}
```

---

### Q: Cache stats not updating

**A:** Use Timer to refresh periodically:

```swift
.onReceive(Timer.publish(every: 2.0, on: .main, in: .common).autoconnect()) { _ in
    stats = StringProvider.shared.getCacheStats()
}
```

---

### Q: How to log SDK activity?

**A:** Enable logging in app initialization:

```swift
init() {
    StringbootLogger.isLoggingEnabled = true
    StringbootLogger.logLevel = .debug

    StringProvider.shared.initialize(...)
}
```

Log levels:
- `.debug` - All SDK activity
- `.info` - Important operations
- `.warning` - Potential issues
- `.error` - Errors only

---

### Q: Does clearCache affect Core Data?

**A:** Depends on the parameter:

```swift
// Clear memory only (Core Data intact)
stringProvider.clearCache(clearDatabase: false)

// Clear everything (memory + Core Data)
stringProvider.clearCache(clearDatabase: true)
```

---

### Q: How to restore language on app restart?

**A:** Save to UserDefaults and restore in init:

```swift
// Save on change
.onChange(of: stringProvider.currentLanguage) { _, newLang in
    if let newLang = newLang {
        UserDefaults.standard.set(newLang, forKey: "com.stringboot.currentLanguage")
    }
}

// Restore on launch
init() {
    StringProvider.shared.initialize(...)

    let savedLang = UserDefaults.standard.string(forKey: "com.stringboot.currentLanguage")
        ?? StringProvider.shared.deviceLocale()
    StringProvider.shared.setLocale(savedLang)
}
```

---

## Changelog

### v2.0 (November 2025) - Current

**New Features:**
- Implemented String-Sync v2 protocol with delta sync
- ETag-based conditional requests for bandwidth optimization
- Async/await support throughout (Swift 5.9+)
- @Published properties for SwiftUI @ObservedObject integration
- `isReady`, `initializationError`, `isChangingLanguage`, `languageChangeError` states
- SwiftUI-first design with SBText component
- Language change with loading states and error handling
- Cache statistics with hit rate, evictions, and size tracking
- Retry failed initialization with `retryInitialization()`
- Device locale detection with `deviceLocale()`
- Persistent language selection support

**Performance:**
- <300ms string lookups on 4G networks
- <100ms delta sync for metadata checks
- <1ms in-memory cache hits
- ~5-20ms Core Data lookups
- Intelligent language prioritization in LRU cache

**Breaking Changes:**
- Changed initialization API (now uses `initialize()` method)
- Added required `@ObservedObject` for reactive UI updates

---

### v1.0 (October 2025)

**Initial Release:**
- Multi-layered caching system (memory + Core Data)
- String localization with fallback to app bundle
- Basic network sync
- Offline support

---

## Support

**Documentation:** https://docs.stringboot.com

**GitHub:** https://github.com/stringboot/ios-sdk

**Issues:** https://github.com/stringboot/ios-sdk/issues

**Email:** support@stringboot.com

---

**License:** Apache 2.0

**Copyright:** © 2025 Stringboot Inc.
