# Stringboot iOS SDK

> High-performance internationalization (i18n) SDK for iOS implementing String-Sync v2 protocol

The Stringboot iOS SDK provides a sophisticated multi-layered caching system with smart memory management, offline support, and network synchronization to minimize API calls and provide fast string lookups.

## Features

- ✅ **Smart Caching** - LRU cache with language-aware and frequency-based eviction
- ✅ **Offline Support** - Full functionality without network via Core Data
- ✅ **SwiftUI Integration** - Native property wrappers and view modifiers
- ✅ **UIKit Integration** - Extensions and custom components with automatic updates
- ✅ **Memory Management** - Automatic cache size adjustment and memory pressure handling
- ✅ **Network Optimization** - ETag-based conditional requests for bandwidth optimization
- ✅ **Integrity Verification** - SHA-256 verification of network responses
- ✅ **Reactive Updates** - Combine publishers for real-time UI updates

## Requirements

- iOS 14.0+ / macOS 11.0+
- Swift 5.9+
- Xcode 15.0+

## Installation

### Swift Package Manager

Add the following to your `Package.swift` file:

```swift
dependencies: [
    .package(url: "https://github.com/your-org/stringboot-ios-sdk.git", from: "2.0.0")
]
```

Or in Xcode:
1. File > Add Package Dependencies
2. Enter the repository URL
3. Select version and add to your target

## Quick Start

### 1. Initialize the SDK

In your `App` or `AppDelegate`:

```swift
import StringbootSDK

// In SwiftUI App
@main
struct MyApp: App {
    init() {
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "YOUR_API_TOKEN",
            baseURL: "https://api.stringboot.com"
        )
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

// In UIKit AppDelegate
func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    StringProvider.shared.initialize(
        cacheSize: 1000,
        apiToken: "YOUR_API_TOKEN",
        baseURL: "https://api.stringboot.com"
    )
    return true
}
```

### 2. SwiftUI Usage

```swift
import SwiftUI
import StringbootSDK

struct ContentView: View {
    @StateObject private var languageManager = StringbootLanguageManager()
    @State private var selectedLanguage = "en"

    var body: some View {
        VStack {
            // Using SBText (automatic localization)
            SBText("welcome_message", lang: selectedLanguage)
                .font(.title)

            SBText("description", lang: selectedLanguage)
                .font(.body)

            // Language picker
            Picker("Language", selection: $selectedLanguage) {
                ForEach(languageManager.availableLanguages, id: \.code) { lang in
                    Text(lang.name).tag(lang.code)
                }
            }
            .onChange(of: selectedLanguage) { newLang in
                languageManager.setLanguage(newLang)
            }
        }
    }
}
```

### 3. UIKit Usage

```swift
import UIKit
import StringbootSDK

class ViewController: UIViewController {

    // Using SBLabel (automatic updates on language change)
    let titleLabel = SBLabel(key: "welcome_message")
    let descriptionLabel = SBLabel(key: "description")

    // Manual integration
    let manualLabel = UILabel()

    override func viewDidLoad() {
        super.viewDidLoad()

        // Configure automatic labels
        titleLabel.font = .systemFont(ofSize: 24, weight: .bold)

        // Manual string loading
        manualLabel.setStringboot(key: "manual_text", lang: "en")

        setupUI()
    }

    @objc func changeLanguage() {
        StringbootLanguageNotifier.shared.changeLanguage(to: "es")
        // All SBLabel instances will update automatically
    }
}
```

### 4. Manual String Access

```swift
// Async/await
Task {
    let text = await StringProvider.shared.get("welcome_message", lang: "en")
    print(text)
}

// With reactive updates
let publisher = StringProvider.shared.publisher(for: "welcome_message", lang: "en")
publisher.sink { text in
    print(text)
}.store(in: &cancellables)
```

## Architecture

### Multi-Layered Repository

```
StringProvider → Smart Cache → Core Data → Network (String-Sync v2 API) → "??key??"
```

1. **Smart Cache** - In-memory LRU cache with language priority and access frequency tracking
2. **Core Data** - Local persistence for offline support
3. **Network** - String-Sync v2 API with ETag-based conditional requests
4. **Fallback** - Returns `??key??` for missing strings

### String-Sync v2 Protocol

The SDK implements the high-performance String-Sync v2 protocol:

1. **ManifestInterceptor** - Checks for changes using ETags (`/meta` endpoint)
2. **DeltaSyncWorker** - Downloads only changed strings (`/updates` endpoint)
3. **StringProvider** - Multi-layered repository with smart caching
4. **Core Data** - Local persistence with proper indexing

### Performance Targets

- ≤2 network requests per cold start (P95)
- <300ms strings ready in memory on 4G
- <2kB download when nothing changed
- 100% offline functionality

## API Reference

### StringProvider

Main entry point for the SDK:

```swift
// Initialization
StringProvider.shared.initialize(
    cacheSize: Int = 1000,
    apiToken: String? = nil,
    baseURL: String = "https://api.stringboot.com"
)

// String access
await StringProvider.shared.get(_ key: String, lang: String? = nil, allowNetworkFetch: Bool = false) -> String

// Language management
StringProvider.shared.setLocale(_ lang: String)
StringProvider.shared.deviceLocale() -> String
StringProvider.shared.getAvailableLanguages() -> [String]
await StringProvider.shared.getAvailableLanguagesFromServer() -> [ActiveLanguage]

// Cache management
StringProvider.shared.clearMemoryCache()
StringProvider.shared.clearLanguageCache(_ lang: String)
StringProvider.shared.getCacheStats() -> CacheStats
StringProvider.shared.getSmartCacheStats() -> SmartCacheStats?

// Network operations
await StringProvider.shared.refreshFromNetwork(lang: String? = nil) -> Bool

// Memory pressure handling
StringProvider.shared.handleMemoryPressure(level: MemoryPressureLevel)
```

### SwiftUI Components

```swift
// Property wrapper
@StringbootString("welcome_message")
var welcomeText: String

// Text view
SBText("key", lang: "en")

// Language manager
@StateObject var languageManager = StringbootLanguageManager()
languageManager.setLanguage("es")
languageManager.loadAvailableLanguages()
await languageManager.refresh()
```

### UIKit Components

```swift
// Custom labels and buttons
let label = SBLabel(key: "welcome_message", lang: "en")
let button = SBButton(key: "button_text", lang: "en")

// Extensions
label.setStringboot(key: "text", lang: "en")
button.setStringbootTitle(key: "button", for: .normal, lang: "en")
textField.setStringbootPlaceholder(key: "placeholder", lang: "en")

// Language notification
StringbootLanguageNotifier.shared.changeLanguage(to: "es")
```

## Configuration

### Info.plist Configuration (Optional)

You can configure the SDK via `Info.plist`:

```xml
<key>StringbootAPIURL</key>
<string>https://api.stringboot.com</string>
<key>StringbootAPIToken</key>
<string>YOUR_API_TOKEN</string>
<key>StringbootCacheSize</key>
<integer>1000</integer>
<key>StringbootDefaultLocale</key>
<string>en</string>
```

### Security

The SDK includes SSL certificate pinning for production security:

```swift
// Certificate pinning is enabled by default in Release builds
// It's automatically disabled in Debug builds for development convenience

// To customize (advanced use case):
let session = SecurityConfig.createSecureURLSession(enablePinning: true)

// Validate API URLs
let isValid = SecurityConfig.validateAPIURL("https://api.stringboot.com")
```

**Certificate Pinning:**
- Automatically enabled in **Release builds**
- Disabled in **Debug builds** for easier local development
- Pins are configured in `SecurityConfig.swift` (replace placeholder pins before production)
- Supports both RSA and EC certificates

**To update certificate pins:**
```bash
# Get current certificate pin
echo | openssl s_client -servername api.stringboot.com -connect api.stringboot.com:443 2>/dev/null | \
openssl x509 -pubkey -noout | openssl pkey -pubin -outform der | openssl dgst -sha256 -binary | base64

# Get backup certificate pin (intermediate CA)
echo | openssl s_client -servername api.stringboot.com -connect api.stringboot.com:443 -showcerts 2>/dev/null | \
awk '/BEGIN CERT/,/END CERT/ {if (++cert==2) print}' | \
openssl x509 -pubkey -noout | openssl pkey -pubin -outform der | openssl dgst -sha256 -binary | base64
```

### Logging

The SDK includes comprehensive logging that can be controlled by developers:

```swift
// Enable or disable all SDK logs (useful for production)
StringbootLogger.isLoggingEnabled = true  // Default: true in Debug, false in Release

// Set log level (when logging is enabled)
StringbootLogger.logLevel = .debug // .verbose, .debug, .info, .warning, .error, .none

// Example: Disable all logs in production
#if DEBUG
StringbootLogger.isLoggingEnabled = true
StringbootLogger.logLevel = .debug
#else
StringbootLogger.isLoggingEnabled = false  // No logs in production
#endif

// Manual logging (for SDK developers)
StringbootLogger.d("Debug message")
StringbootLogger.i("Info message")
StringbootLogger.w("Warning message")
StringbootLogger.e("Error message", error: someError)
```

**Default Behavior:**
- **Debug builds**: Logging enabled with `.debug` level
- **Release builds**: Logging completely disabled (no logs at all)

**Recommended Production Setup:**
```swift
// Disable all logs for production apps
StringbootLogger.isLoggingEnabled = false
```

## Advanced Usage

### Memory Pressure Handling

The SDK automatically responds to memory pressure. You can also manually trigger it:

```swift
// Listen to system memory warnings
NotificationCenter.default.addObserver(
    forName: UIApplication.didReceiveMemoryWarningNotification,
    object: nil,
    queue: .main
) { _ in
    StringProvider.shared.handleMemoryPressure(level: .critical)
}
```

### Preloading Languages

For better performance, preload languages on app start:

```swift
Task {
    await StringProvider.shared.preloadLanguage("en", maxStrings: 500)
    await StringProvider.shared.preloadLanguage("es", maxStrings: 500)
}
```

### Cache Statistics

Monitor cache performance:

```swift
let stats = StringProvider.shared.getCacheStats()
print("Hit rate: \(stats.hitRate)")
print("Size: \(stats.memorySize) / \(stats.memoryMaxSize)")

if let smartStats = StringProvider.shared.getSmartCacheStats() {
    print("Language distribution: \(smartStats.languageDistribution)")
    print("Average access frequency: \(smartStats.averageAccessFrequency)")
    print("Dominant language: \(smartStats.dominantLanguage ?? "none")")
}
```

### Force Refresh

Force refresh strings from network:

```swift
Task {
    let success = await StringProvider.shared.refreshFromNetwork(lang: "en")
    if success {
        print("Refresh successful")
    }
}
```

## Testing

### Unit Tests

```swift
import XCTest
@testable import StringbootSDK

class StringProviderTests: XCTestCase {

    override func setUp() {
        super.setUp()
        StringProvider.shared.initialize(cacheSize: 100)
    }

    func testStringRetrieval() async {
        let text = await StringProvider.shared.get("test_key", lang: "en")
        XCTAssertNotEqual(text, "??test_key??")
    }

    func testCacheHitRate() async {
        _ = await StringProvider.shared.get("key1", lang: "en")
        _ = await StringProvider.shared.get("key1", lang: "en")

        let stats = StringProvider.shared.getCacheStats()
        XCTAssertGreaterThan(stats.hitRate, 0)
    }
}
```

## Migration from Android SDK

The iOS SDK mirrors the Android SDK architecture for consistency:

| Android | iOS |
|---------|-----|
| `StringProvider.kt` | `StringProvider.swift` |
| `SmartStringCache` (LruCache) | `SmartStringCache` (custom LRU) |
| `Room Database` | `Core Data` |
| `DataStore` | `UserDefaults` |
| `Jetpack Compose` | `SwiftUI` |
| `View extensions` | `UIKit extensions` |

## Performance Tips

1. **Initialize early** - Call `initialize()` in `App.init()` or `AppDelegate`
2. **Preload languages** - Preload expected languages on app start
3. **Use smart cache** - Let the SDK manage cache based on usage patterns
4. **Monitor stats** - Use cache statistics to optimize cache size
5. **Handle memory pressure** - Respond to system memory warnings

## Troubleshooting

### Strings not loading

- Check API token configuration
- Verify network connectivity
- Check log output with `StringbootLogger.logLevel = .debug`

### Cache not working

- Ensure `initialize()` is called before first use
- Check cache stats to verify hits/misses
- Verify sufficient cache size for your use case

### Memory issues

- Reduce cache size in `initialize(cacheSize:)`
- Implement memory pressure handling
- Clear cache periodically if needed

## A/B Testing Integration

### Overview

The SDK supports A/B testing for strings, allowing you to test different copy variations and track performance.

### How It Works

1. **Device ID**: SDK generates a unique UUID per device installation
2. **X-Device-ID Header**: Automatically sent with all API requests
3. **Backend Assignment**: Backend assigns device to experiment variant
4. **Consistent Delivery**: Same device always gets same variant
5. **Analytics Integration**: Track experiments in your analytics platform

### Basic Setup

The SDK handles A/B testing automatically - no code changes required!

```swift
// User calls this normally
let text = await StringProvider.shared.get("welcome_message", lang: "en")

// Backend might return different variants:
// - "Welcome!" (control)
// - "Hey there!" (variant A)
// - "Hello friend!" (variant B)
```

### Analytics Integration (Optional)

Track experiment assignments in Firebase Analytics:

#### Step 1: Create Analytics Handler

```swift
import StringbootSDK
import FirebaseAnalytics

class FirebaseAnalyticsHandler: StringbootAnalyticsHandler {
    private let analytics: Analytics

    init() {
        self.analytics = Analytics.analytics()
    }

    func onExperimentsAssigned(experiments: [String: ExperimentAssignment]) {
        for (_, experiment) in experiments {
            // Set user property: stringboot_exp_{experimentKey}
            analytics.setUserProperty(
                experiment.variantName,
                forName: "stringboot_exp_\(experiment.experimentKey)"
            )

            print("Experiment: \(experiment.experimentKey) = \(experiment.variantName)")
        }
    }
}
```

#### Step 2: Initialize with Analytics Handler

```swift
import StringbootSDK

@main
struct MyApp: App {
    init() {
        let analyticsHandler = FirebaseAnalyticsHandler()

        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "your-api-token",
            analyticsHandler: analyticsHandler  // Add this
        )
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

### Advanced: Custom Device ID

Use your app's existing device identifier for consistency:

```swift
import StringbootSDK
import FirebaseAnalytics

StringProvider.shared.initialize(
    cacheSize: 1000,
    apiToken: "your-api-token",
    providedDeviceId: Analytics.appInstanceID(),  // Use Firebase ID
    analyticsHandler: analyticsHandler
)
```

### Property Naming Convention

```
stringboot_exp_{experimentKey}
```

**Example:**
- Experiment: `welcome_test`
- Property: `stringboot_exp_welcome_test`
- Value: `control`, `variant-a`, `variant-b`

### Debugging Experiments

```swift
// Get current experiments
let experiments = await StringProvider.shared.getExperiments()
for (stringKey, exp) in experiments {
    print("""
        String: \(stringKey)
        Experiment: \(exp.experimentKey)
        Variant: \(exp.variantName)
        Assigned: \(exp.assignedAt)
    """)
}
```

### Key Points

✅ **Automatic** - A/B testing works without code changes
✅ **Consistent** - Same device always gets same variant
✅ **Persistent** - Assignments survive app restarts
✅ **Optional** - Analytics integration is completely optional
✅ **Backward Compatible** - No breaking changes

For complete details, see [AB_TESTING_CLIENT_SDK_INTEGRATION.md](../AB_TESTING_CLIENT_SDK_INTEGRATION.md)

## FAQ Provider

### Overview

The FAQ Provider feature enables you to deliver dynamic, multilingual FAQ content to your iOS app with offline-first caching and automatic updates.

**Key Features:**
- 📦 **Offline-First**: FAQs cached locally with Core Data
- 🌍 **Multilingual**: Language fallback to English
- 🏷️ **Tag-Based**: Organize FAQs by tags and sub-tags
- 🔄 **Reactive**: Combine publishers for auto-updating UI
- ⚡ **High Performance**: Three-tier caching (memory → Core Data → network)

### Quick Start

#### 1. Initialize FAQ Provider

**IMPORTANT:** FAQProvider requires separate initialization from StringProvider.

```swift
@main
struct MyApp: App {
    init() {
        // Initialize StringProvider first
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "YOUR_API_TOKEN",
            baseURL: "https://api.stringboot.com"
        )

        // Initialize FAQProvider separately - REQUIRED!
        FAQProvider.shared.initialize(
            cacheSize: 200,
            apiToken: "YOUR_API_TOKEN",
            baseURL: "https://api.stringboot.com"
        )
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

#### 2. SwiftUI Usage

```swift
import SwiftUI
import StringbootSDK

struct FAQListView: View {
    @State private var faqs: [FAQ] = []
    @State private var loading = true

    var body: some View {
        Group {
            if loading {
                ProgressView("Loading FAQs...")
            } else {
                List(faqs) { faq in
                    VStack(alignment: .leading, spacing: 8) {
                        Text(faq.question)
                            .font(.headline)
                        Text(faq.answer)
                            .font(.body)
                            .foregroundColor(.secondary)
                    }
                    .padding(.vertical, 4)
                }
            }
        }
        .onAppear {
            loadFAQs()
        }
    }

    func loadFAQs() {
        Task {
            faqs = await FAQProvider.shared.getFAQs(tag: "payments", lang: "en")
            loading = false
        }
    }
}
```

#### 3. UIKit Usage

```swift
import UIKit
import StringbootSDK

class FAQViewController: UITableViewController {
    private var faqs: [FAQ] = []

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "FAQs"
        loadFAQs()
    }

    func loadFAQs() {
        Task {
            faqs = await FAQProvider.shared.getFAQs(tag: "payments", lang: "en")
            tableView.reloadData()
        }
    }

    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return faqs.count
    }

    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "FAQCell") ?? UITableViewCell(style: .subtitle, reuseIdentifier: "FAQCell")
        let faq = faqs[indexPath.row]
        cell.textLabel?.text = faq.question
        cell.detailTextLabel?.text = faq.answer
        cell.detailTextLabel?.numberOfLines = 2
        return cell
    }
}
```

### API Reference

#### Initialize

```swift
FAQProvider.shared.initialize(
    cacheSize: Int = 200,
    apiToken: String? = nil,
    baseURL: String = "https://api.stringboot.com"
)
```

#### Fetch FAQs

```swift
// Get FAQs by tag
let faqs = await FAQProvider.shared.getFAQs(
    tag: "payments",
    lang: "en"
)

// Get single FAQ by ID
let faq = await FAQProvider.shared.getFAQ(
    id: "faq-123",
    lang: "en"
)

// Get all tags
let tags = await FAQProvider.shared.getAllTags(lang: "en")
```

#### Reactive Updates with Combine

```swift
import Combine

class FAQViewModel: ObservableObject {
    @Published var faqs: [FAQ] = []
    private var cancellables = Set<AnyCancellable>()

    func subscribeTo(tag: String, lang: String) {
        FAQProvider.shared.getFAQsPublisher(tag: tag, lang: lang)
            .sink { [weak self] faqs in
                self?.faqs = faqs
            }
            .store(in: &cancellables)
    }
}
```

#### Manual Sync

```swift
// Sync FAQs from network
let success = await FAQProvider.shared.syncFAQs(lang: "en")

// Clear cache
FAQProvider.shared.clearCache()
```

### Filtering by Sub-Tags

```swift
// Get all payment FAQs
let allPaymentFAQs = await FAQProvider.shared.getFAQs(tag: "payments", lang: "en")

// Filter by sub-tag
let refundFAQs = allPaymentFAQs.filter { faq in
    faq.subTags?.contains("refunds") ?? false
}
```

### Complete Example: FAQ Screen

```swift
import SwiftUI
import Combine

struct FAQScreen: View {
    @StateObject private var viewModel = FAQViewModel()
    @State private var selectedTag = "payments"

    let tags = ["payments", "account", "shipping", "returns"]

    var body: some View {
        VStack {
            // Tag selector
            Picker("Category", selection: $selectedTag) {
                ForEach(tags, id: \.self) { tag in
                    Text(tag.capitalized).tag(tag)
                }
            }
            .pickerStyle(SegmentedPickerStyle())
            .padding()

            // FAQ list
            if viewModel.loading {
                ProgressView("Loading FAQs...")
            } else if viewModel.faqs.isEmpty {
                Text("No FAQs available")
                    .foregroundColor(.secondary)
            } else {
                List(viewModel.faqs) { faq in
                    DisclosureGroup {
                        Text(faq.answer)
                            .font(.body)
                            .padding(.top, 8)
                    } label: {
                        Text(faq.question)
                            .font(.headline)
                    }
                }
            }
        }
        .navigationTitle("Help & FAQs")
        .onAppear {
            viewModel.loadFAQs(tag: selectedTag)
        }
        .onChange(of: selectedTag) { newTag in
            viewModel.loadFAQs(tag: newTag)
        }
    }
}

class FAQViewModel: ObservableObject {
    @Published var faqs: [FAQ] = []
    @Published var loading = false

    func loadFAQs(tag: String, lang: String = "en") {
        loading = true
        Task {
            faqs = await FAQProvider.shared.getFAQs(tag: tag, lang: lang)
            loading = false
        }
    }
}
```

### Performance Tips

1. **Cache Size**: Adjust based on your FAQ volume
   ```swift
   FAQProvider.shared.initialize(cacheSize: 500)  // For large FAQ sets
   ```

2. **Preload FAQs**: Load FAQs on app start
   ```swift
   Task {
       await FAQProvider.shared.syncFAQs(lang: "en")
   }
   ```

3. **Use Tags Wisely**: Keep tags organized and specific
   ```swift
   let paymentFAQs = await FAQProvider.shared.getFAQs(tag: "payments")
   let shippingFAQs = await FAQProvider.shared.getFAQs(tag: "shipping")
   ```

### Troubleshooting

**FAQs not loading?**
- Ensure `FAQProvider.initialize()` was called (separate from StringProvider)
- Check network connectivity
- Verify API token is correct
- Enable debug logging: `StringbootLogger.logLevel = .debug`

**Performance issues?**
- Increase cache size for large FAQ sets
- Use lazy loading in SwiftUI (`LazyVStack`)
- Implement pagination for 100+ FAQs

For more details, see [FAQ_PROVIDER.md](FAQ_PROVIDER.md) and [TROUBLESHOOTING.md](TROUBLESHOOTING.md)

## License

MIT License - See LICENSE file for details

## Additional Resources

- 🚀 [Quick Start Guide](../docs/QUICKSTART.md) - Get started in 5 minutes
- 💻 [Implementation Examples](../docs/IMPLEMENTATION_EXAMPLES.md) - Android & iOS code samples
- 📖 [API Reference](../docs/API_REFERENCE.md) - Complete endpoint documentation
- 🔄 [Delta Sync Protocol](../docs/DELTA_SYNC_PROTOCOL.md) - How syncing works
- 🔧 [Setup Guide](SETUP.md) - Detailed installation instructions
- 🐛 [Troubleshooting](TROUBLESHOOTING.md) - Common issues and solutions

## Support

- Documentation: https://docs.stringboot.com
- Issues: https://github.com/your-org/stringboot-ios-sdk/issues
- Email: support@stringboot.com
