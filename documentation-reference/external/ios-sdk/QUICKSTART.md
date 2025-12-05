# Stringboot iOS SDK - Quick Start Guide

Get up and running with Stringboot in **under 10 minutes**. This guide will walk you through adding dynamic string management to your iOS app.

## Prerequisites

- Xcode 15.0 or later
- iOS 14.0+ / macOS 11.0+
- Swift 5.9+
- A Stringboot API token ([Get one here](https://stringboot.com/signup))

## Step 1: Add Dependency (2 minutes)

### Swift Package Manager (Recommended)

**Option A: Xcode UI**

1. Open your project in Xcode
2. Go to **File** → **Add Package Dependencies**
3. Enter the repository URL: `https://github.com/your-org/stringboot-ios-sdk.git`
4. Select version `2.0.0` or later
5. Click **Add Package**

**Option B: Package.swift**

Add to your `Package.swift`:

```swift
dependencies: [
    .package(url: "https://github.com/your-org/stringboot-ios-sdk.git", from: "2.0.0")
],
targets: [
    .target(
        name: "YourApp",
        dependencies: [
            .product(name: "StringbootSDK", package: "stringboot-ios-sdk")
        ]
    )
]
```

## Step 2: Initialize SDK (3 minutes)

### SwiftUI App

Initialize in your `App` struct:

```swift
import SwiftUI
import StringbootSDK

@main
struct MyApp: App {

    init() {
        // Enable logging for development
        StringbootLogger.isLoggingEnabled = true
        StringbootLogger.logLevel = .debug

        // Initialize Stringboot
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "YOUR_API_TOKEN_HERE",
            baseURL: "https://api.stringboot.com",
            autoSync: true  // Automatically sync in background
        )

        // Set initial language (optional - defaults to device locale)
        let initialLang = UserDefaults.standard.string(forKey: "app_language")
            ?? StringProvider.shared.deviceLocale()
        StringProvider.shared.setLocale(initialLang)
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

### UIKit App (AppDelegate)

Initialize in `AppDelegate.swift`:

```swift
import UIKit
import StringbootSDK

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {

        // Enable logging
        StringbootLogger.isLoggingEnabled = true
        StringbootLogger.logLevel = .debug

        // Initialize Stringboot
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "YOUR_API_TOKEN_HERE",
            baseURL: "https://api.stringboot.com",
            autoSync: true
        )

        // Set language
        let initialLang = UserDefaults.standard.string(forKey: "app_language")
            ?? StringProvider.shared.deviceLocale()
        StringProvider.shared.setLocale(initialLang)

        return true
    }
}
```

**⚠️ Security Note:** Store your API token in a secure location (Keychain or xcconfig file) rather than hardcoding it.

## Step 3: Use Strings in Your App (5 minutes)

### Method 1: SwiftUI with SBText (Easiest - Recommended)

Use the `SBText` component for automatic string management:

```swift
import SwiftUI
import StringbootSDK

struct ContentView: View {
    @ObservedObject var stringProvider = StringProvider.shared
    @State private var selectedLanguage = "en"

    var body: some View {
        VStack(spacing: 20) {
            // Automatic localized text - updates when language changes
            SBText("welcome_message")
                .font(.title)
                .fontWeight(.bold)

            SBText("app_description")
                .font(.body)
                .foregroundColor(.secondary)

            SBText("call_to_action")
                .font(.headline)

            // Language picker
            if stringProvider.isReady {
                Picker("Language", selection: $selectedLanguage) {
                    ForEach(stringProvider.availableLanguages, id: \.code) { language in
                        Text(language.name).tag(language.code)
                    }
                }
                .pickerStyle(.segmented)
                .onChange(of: selectedLanguage) { _, newLang in
                    Task {
                        await stringProvider.changeLanguage(to: newLang)
                    }
                }
            }
        }
        .padding()
    }
}
```

**That's it!** Your UI will automatically:
- ✅ Display strings from Stringboot
- ✅ Update when language changes
- ✅ Fall back to `Localizable.xcstrings` if offline
- ✅ Handle loading states

### Method 2: UIKit with SBLabel (Auto-Updating)

Use custom components that automatically update on language changes:

```swift
import UIKit
import StringbootSDK

class ViewController: UIViewController {

    // Auto-updating labels (update when language changes)
    let titleLabel = SBLabel(key: "welcome_message")
    let descriptionLabel = SBLabel(key: "app_description")

    // Auto-updating button
    let actionButton = SBButton(key: "button_get_started")

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground

        // Configure labels
        titleLabel.font = .systemFont(ofSize: 28, weight: .bold)
        titleLabel.textAlignment = .center

        descriptionLabel.font = .systemFont(ofSize: 16)
        descriptionLabel.numberOfLines = 0
        descriptionLabel.textAlignment = .center

        // Configure button
        actionButton.addTarget(self, action: #selector(actionTapped), for: .touchUpInside)

        setupLayout()
    }

    private func setupLayout() {
        let stackView = UIStackView(arrangedSubviews: [
            titleLabel,
            descriptionLabel,
            actionButton
        ])
        stackView.axis = .vertical
        stackView.spacing = 20
        stackView.translatesAutoresizingMaskIntoConstraints = false

        view.addSubview(stackView)

        NSLayoutConstraint.activate([
            stackView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stackView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            stackView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 40),
            stackView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -40)
        ])
    }

    @objc func actionTapped() {
        print("Action tapped!")
    }
}
```

### Method 3: Manual String Access (async/await)

For more control, fetch strings manually:

```swift
import SwiftUI
import StringbootSDK

struct ContentView: View {
    @State private var welcomeText = "Loading..."
    @State private var descriptionText = "Loading..."

    var body: some View {
        VStack(spacing: 20) {
            Text(welcomeText)
                .font(.title)

            Text(descriptionText)
                .font(.body)
        }
        .task {
            await loadStrings()
        }
    }

    func loadStrings() async {
        // Fetch strings asynchronously
        welcomeText = await StringProvider.shared.get("welcome_message")
        descriptionText = await StringProvider.shared.get("app_description")
    }
}
```

### Method 4: UIKit with Manual String Setting

For existing UIKit views:

```swift
override func viewDidLoad() {
    super.viewDidLoad()

    // Set strings on standard UIKit components
    titleLabel.setStringboot(key: "welcome_message")
    descriptionLabel.setStringboot(key: "app_description")
    actionButton.setStringbootTitle(key: "button_get_started")

    // Manual async loading
    Task {
        let text = await StringProvider.shared.get("additional_info")
        infoLabel.text = text
    }
}
```

## Step 4: Test It Out! (1 minute)

1. **Run your app** on simulator or device
2. **Check Console** for Stringboot logs:
   ```
   [Stringboot] ✅ Initialization successful
   [Stringboot] 📥 Syncing strings for language: en
   [Stringboot] ✅ Sync complete: 42 strings loaded
   ```
3. Your UI should display strings from Stringboot!

## Common Pitfalls & Troubleshooting

### ❌ Problem: Strings show as empty or "Loading..."

**Solution:**
- Check Console for initialization errors
- Verify your API token is correct
- Ensure you have network connectivity (SDK needs internet for first sync)
- Check that `StringProvider.shared.isReady` is `true`

### ❌ Problem: App crashes with "StringProvider not initialized"

**Solution:**
- Ensure `initialize()` is called in `App.init()` or `AppDelegate`
- Check that initialization happens **before** any views try to access strings
- Verify the SDK is properly added as a dependency

### ❌ Problem: Strings don't update when changing language

**Solution:**
- Use `SBText` or `SBLabel` components (they auto-update)
- For manual implementations, listen to `StringbootLanguageNotifier` notifications:
  ```swift
  NotificationCenter.default.addObserver(
      forName: .stringbootLanguageChanged,
      object: nil,
      queue: .main
  ) { _ in
      // Reload your strings here
  }
  ```

### ❌ Problem: Strings are outdated / not refreshing

**Solution:**
- Call `StringProvider.shared.forceRefresh()` to force a sync
- Check that `autoSync: true` is set during initialization
- Verify strings exist on Stringboot dashboard

### ❌ Problem: Memory warnings on older devices

**Solution:**
- Reduce cache size: `cacheSize: 500` during initialization
- The SDK automatically handles memory pressure
- Consider implementing `applicationDidReceiveMemoryWarning`

## Next Steps

### Switch Languages

```swift
// SwiftUI
Button("Switch to Spanish") {
    Task {
        await stringProvider.changeLanguage(to: "es")
    }
}

// UIKit
@objc func switchLanguage() {
    StringbootLanguageNotifier.shared.changeLanguage(to: "es")
    // All SBLabel/SBButton instances auto-update!
}
```

### Get Available Languages

```swift
Task {
    // Fetch from server
    await stringProvider.loadAvailableLanguages()

    // Access languages
    for language in stringProvider.availableLanguages {
        print("\(language.code): \(language.name)")
    }
}
```

### Use StringbootLanguageManager (SwiftUI)

For advanced language management:

```swift
struct ContentView: View {
    @StateObject private var languageManager = StringbootLanguageManager()

    var body: some View {
        VStack {
            // Current language display
            Text("Current: \(languageManager.currentLanguage)")

            // Language picker
            Picker("Language", selection: Binding(
                get: { languageManager.currentLanguage },
                set: { languageManager.setLanguage($0) }
            )) {
                ForEach(languageManager.availableLanguages, id: \.code) { lang in
                    Text(lang.name).tag(lang.code)
                }
            }

            if languageManager.isChangingLanguage {
                ProgressView("Switching language...")
            }

            if let error = languageManager.languageChangeError {
                Text("Error: \(error)")
                    .foregroundColor(.red)
            }
        }
    }
}
```

### Monitor Cache Performance

```swift
// Get cache statistics
let stats = StringProvider.shared.getCacheStatistics()
print("Hit rate: \(stats.hitRate * 100)%")
print("Memory usage: \(stats.memorySize) / \(stats.memoryMaxSize)")
print("Cache hits: \(stats.memoryHitCount)")
print("Cache misses: \(stats.memoryMissCount)")
```

### Debug Panel (Development Only)

Add cache monitoring to your debug builds:

```swift
#if DEBUG
struct CacheDebugView: View {
    @State private var stats: CacheStats?

    var body: some View {
        VStack(alignment: .leading) {
            Text("Cache Statistics").font(.headline)

            if let stats = stats {
                Text("Size: \(stats.memorySize) / \(stats.memoryMaxSize)")
                Text("Hit Rate: \(String(format: "%.1f%%", stats.hitRate * 100))")
                Text("Hits: \(stats.memoryHitCount)")
                Text("Misses: \(stats.memoryMissCount)")
            }
        }
        .onAppear {
            stats = StringProvider.shared.getCacheStatistics()
        }
    }
}
#endif
```

## Learn More

| Resource | Description |
|----------|-------------|
| **[INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)** | Comprehensive integration walkthrough |
| **[AB_TESTING.md](AB_TESTING.md)** | A/B testing and experiment setup |
| **[ADVANCED_FEATURES.md](ADVANCED_FEATURES.md)** | Smart caching, FAQ provider, memory management |
| **[API_REFERENCE.md](API_REFERENCE.md)** | Complete API documentation |
| **[FAQ.md](FAQ.md)** | Frequently asked questions |
| **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** | Common issues and solutions |
| **[Demo App](../stringboot-ios-demoApp/)** | Real-world SwiftUI & UIKit examples |

## Sample Code Repository

Check out our [complete demo app](../stringboot-ios-demoApp/) with:
- ✅ SwiftUI implementation with `SBText`
- ✅ UIKit implementation with `SBLabel` and `SBButton`
- ✅ Language switching UI
- ✅ FAQ integration example
- ✅ Cache statistics display
- ✅ Offline mode handling

## Need Help?

- **Email:** support@stringboot.com
- **Documentation:** https://docs.stringboot.com
- **Report Issues:** https://github.com/stringboot/ios-sdk/issues

---

**Congratulations!** 🎉 You've integrated Stringboot into your iOS app. Your strings are now managed remotely and can be updated without app releases!

