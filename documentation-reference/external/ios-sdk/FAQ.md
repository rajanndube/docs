# Stringboot iOS SDK - Frequently Asked Questions

> Common questions and answers about using the Stringboot iOS SDK

**Last Updated:** December 2024 | **SDK Version:** 2.0.0+

---

## Table of Contents

- [Getting Started](#getting-started)
- [Installation & Setup](#installation--setup)
- [String Management](#string-management)
- [SwiftUI Integration](#swiftui-integration)
- [UIKit Integration](#uikit-integration)
- [Language Switching](#language-switching)
- [Performance & Caching](#performance--caching)
- [A/B Testing](#ab-testing)
- [FAQ Provider](#faq-provider)
- [Offline Support](#offline-support)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

---

## Getting Started

### What is Stringboot?

Stringboot is a modern internationalization (i18n) SDK that allows you to manage and update app strings remotely without redeploying your app. It provides offline-first caching, automatic updates, language switching, and A/B testing capabilities.

### Why should I use Stringboot instead of NSLocalizedString?

**Stringboot offers several advantages:**

- **Remote Updates** - Update strings without releasing a new app version
- **Instant Fixes** - Fix typos or translations immediately
- **A/B Testing** - Test different copy variations with real users
- **Real-time Localization** - Add new languages on the fly
- **Offline-First** - Works perfectly without internet connection
- **Analytics Integration** - Track which string variations perform best
- **SwiftUI & UIKit Support** - Native integration for both frameworks

### Can I use Stringboot alongside NSLocalizedString?

Yes! Many apps use Stringboot for dynamic content (marketing copy, feature descriptions, CTAs) while keeping critical UI strings in Localizable.strings as fallbacks. This hybrid approach gives you flexibility while maintaining stability.

### What are the minimum requirements?

- **iOS:** 14.0+ / macOS 11.0+
- **Swift:** 5.9+
- **Xcode:** 15.0+
- **Dependencies:** Core Data, Combine, Foundation

---

## Installation & Setup

### How do I install the SDK?

**Swift Package Manager (recommended):**

1. In Xcode: **File** → **Add Package Dependencies**
2. Enter URL: `https://github.com/your-org/stringboot-ios-sdk.git`
3. Select version and add to target

Or add to `Package.swift`:

```swift
dependencies: [
    .package(url: "https://github.com/your-org/stringboot-ios-sdk.git", from: "2.0.0")
]
```

See [QUICKSTART.md](QUICKSTART.md) for complete installation instructions.

### Do I need to initialize the SDK in every View?

**No!** Initialize once in your App or AppDelegate:

**SwiftUI:**
```swift
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
```

**UIKit:**
```swift
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

### Where do I get my API token?

1. Sign up at [stringboot.com](https://stringboot.com)
2. Create a new project
3. Navigate to **Settings** → **API Keys**
4. Copy your iOS API token
5. Add it to your configuration

**Never commit your API token!** Use environment variables or Configuration files for production.

### Can I use Stringboot without network access?

Yes! Stringboot is **offline-first**. Once strings are cached locally (Core Data), the app works perfectly without internet. Network is only needed for initial sync and updates.

### How do I configure the SDK?

Create `stringboot-config.json` in your project:

```json
{
  "apiToken": "YOUR_API_TOKEN",
  "baseUrl": "https://api.stringboot.com",
  "appId": "your_app_id",
  "defaultLanguage": "en",
  "cacheSize": 1000,
  "autoSync": true
}
```

Or initialize programmatically:

```swift
StringProvider.shared.initialize(
    cacheSize: 1000,
    apiToken: "YOUR_API_TOKEN",
    baseURL: "https://api.stringboot.com",
    autoSync: true
)
```

---

## String Management

### How do I get a string?

**Async/await (recommended):**

```swift
let text = await StringProvider.shared.get("welcome_message", lang: "en")
print(text)
```

**SwiftUI with SBText:**

```swift
SBText("welcome_message")
    .font(.title)
```

**UIKit with SBLabel:**

```swift
let label = SBLabel(key: "welcome_message")
label.font = .systemFont(ofSize: 20)
view.addSubview(label)
```

### How do I make strings update automatically in my UI?

**SwiftUI - Use SBText:**

```swift
SBText("welcome_message", lang: selectedLanguage)
    .font(.headline)
// Automatically updates when language changes
```

**UIKit - Use SBLabel:**

```swift
let label = SBLabel(key: "welcome_message")
// Automatically updates when strings change
```

**Custom implementation with Combine:**

```swift
StringProvider.shared.publisher(for: "welcome_message", lang: "en")
    .sink { text in
        self.label.text = text
    }
    .store(in: &cancellables)
```

### What happens if a string key doesn't exist?

The SDK returns `"??key??"` (e.g., `"??welcome_message??"`). This makes missing strings obvious during development.

**Best practice:** Always test with all language variants before release.

### Can I pass parameters to strings?

Not directly through Stringboot. Use Swift string formatting:

```swift
let template = await StringProvider.shared.get("welcome_user") // "Welcome, %@!"
let message = String(format: template, userName)
```

Or use placeholders:

```swift
let template = await StringProvider.shared.get("items_count") // "You have {count} items"
let message = template.replacingOccurrences(of: "{count}", with: "\(count)")
```

### How do I bulk-fetch multiple strings?

Fetch strings in parallel using async let:

```swift
async let title = StringProvider.shared.get("title")
async let subtitle = StringProvider.shared.get("subtitle")
async let cta = StringProvider.shared.get("cta_button")

let (titleText, subtitleText, ctaText) = await (title, subtitle, cta)
```

---

## SwiftUI Integration

### What is SBText?

`SBText` is a SwiftUI component that automatically updates when:
- Language changes
- String content is updated from the backend
- The specified string key is refreshed

**Usage:**

```swift
SBText("welcome_message")
    .font(.title)
    .foregroundColor(.primary)
```

### Can I use SBText with custom language?

Yes!

```swift
@State private var selectedLanguage = "en"

SBText("greeting", lang: selectedLanguage)
    .font(.headline)
```

### How do I use StringProvider with @StateObject?

Use `StringbootLanguageManager` for reactive language management:

```swift
struct ContentView: View {
    @StateObject private var languageManager = StringbootLanguageManager()

    var body: some View {
        VStack {
            SBText("title", lang: languageManager.currentLanguage)

            Picker("Language", selection: $languageManager.currentLanguage) {
                ForEach(languageManager.availableLanguages, id: \.code) { lang in
                    Text(lang.name).tag(lang.code)
                }
            }
        }
    }
}
```

### Can I use Stringboot with @Published properties?

Yes, use Combine publishers:

```swift
class ViewModel: ObservableObject {
    @Published var welcomeText: String = ""
    private var cancellables = Set<AnyCancellable>()

    init() {
        StringProvider.shared.publisher(for: "welcome_message")
            .assign(to: &$welcomeText)
    }
}
```

---

## UIKit Integration

### What is SBLabel?

`SBLabel` is a UILabel subclass that automatically updates when strings change:

```swift
let titleLabel = SBLabel(key: "page_title")
titleLabel.font = .systemFont(ofSize: 24, weight: .bold)
view.addSubview(titleLabel)
```

No manual refresh needed!

### Can I use SBLabel with custom language?

Yes:

```swift
let greetingLabel = SBLabel(key: "greeting", lang: "es")
greetingLabel.font = .systemFont(ofSize: 20)
```

### How do I update UIButton titles with Stringboot?

**Option 1: SBButton**

```swift
let button = SBButton(key: "cta_button")
button.backgroundColor = .systemBlue
button.setTitleColor(.white, for: .normal)
button.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)
```

**Option 2: Manual async update**

```swift
let button = UIButton()
Task {
    let title = await StringProvider.shared.get("cta_button")
    button.setTitle(title, for: .normal)
}
```

### How do I set UITextField placeholders with Stringboot?

```swift
Task {
    emailTextField.placeholder = await StringProvider.shared.get("email_placeholder")
}
```

---

## Language Switching

### How do I switch languages?

```swift
Task {
    let success = await StringProvider.shared.switchLanguage(to: "es")
    if success {
        print("Language switched to Spanish")
        // UI with SBText/SBLabel automatically updates
    }
}
```

### How do I get the list of available languages?

```swift
Task {
    let languages = await StringProvider.shared.getAvailableLanguages()
    for lang in languages {
        print("\(lang.code): \(lang.name) (Default: \(lang.isDefault))")
    }
}
```

### Do I need to manually refresh the UI after language change?

**No!** If you're using:
- **SBText** (SwiftUI) - Updates automatically
- **SBLabel** (UIKit) - Updates automatically
- **Combine publishers** - Update automatically
- **Manual `get()` calls** - You need to re-fetch

### Can users switch languages independently of device settings?

Yes! Stringboot language is independent of iOS system language:

```swift
// Switch to Spanish regardless of device language
await StringProvider.shared.switchLanguage(to: "es")
```

### How do I detect the device language?

```swift
let deviceLang = StringProvider.shared.deviceLocale()
print("Device language: \(deviceLang)") // e.g., "en", "es", "fr"
```

### What happens if a string isn't available in the selected language?

The SDK automatically falls back to English (default language). Fallback order:
1. Requested language (e.g., "fr")
2. Default language ("en")
3. `"??key??"` if not found anywhere

---

## Performance & Caching

### How does caching work?

Stringboot uses a **three-tier caching system**:

1. **Memory Cache (L1)** - LRU cache for instant access (<5ms)
2. **Core Data (L2)** - Persistent local storage (<50ms)
3. **Network (L3)** - API calls with ETag-based caching

### How much data does Stringboot cache?

By default, **1,000 strings** in memory. Core Data stores unlimited strings. Typical usage:
- 1,000 strings ≈ 200-500 KB in memory
- 5,000 strings ≈ 1-2 MB in Core Data

### Can I change the cache size?

Yes, during initialization:

```swift
StringProvider.shared.initialize(
    cacheSize: 2000, // Cache 2000 strings in memory
    apiToken: "YOUR_API_TOKEN",
    baseURL: "https://api.stringboot.com"
)
```

### How often does the SDK sync with the network?

- **Initial sync:** On first app launch
- **Auto-sync:** When app becomes active (if 30+ minutes since last sync)
- **Manual sync:** Call `refreshFromNetwork()`

Network requests use ETag caching, so if nothing changed, only ~2 KB is downloaded.

### Does Stringboot impact app startup time?

**No.** Initialization is asynchronous and non-blocking:
- Cache initialization: <10ms
- Core Data setup: <50ms
- Network sync: Runs in background, doesn't block UI

### Can I preload languages for better performance?

Yes! Preload frequently used languages:

```swift
Task {
    await StringProvider.shared.preloadLanguage("en", maxStrings: 500)
    await StringProvider.shared.preloadLanguage("es", maxStrings: 500)
}
```

### How do I clear the cache?

```swift
// Clear cache for specific language
await StringProvider.shared.clearCache(for: "en")

// Clear all cache
await StringProvider.shared.clearAllCache()
```

**Note:** Users rarely need to clear cache. The SDK manages cache efficiently.

---

## A/B Testing

### What is A/B testing in Stringboot?

A/B testing lets you test different string variations (e.g., "Sign Up" vs "Get Started") with real users. The SDK automatically shows different variants to different users and integrates with your analytics.

See [AB_TESTING.md](AB_TESTING.md) for complete guide.

### How do I set up A/B testing?

1. **Create experiment** in Stringboot dashboard
2. **Define variants** (e.g., Variant A: "Buy Now", Variant B: "Purchase")
3. **Initialize SDK** with analytics handler:

```swift
StringProvider.shared.initialize(
    cacheSize: 1000,
    apiToken: "YOUR_API_TOKEN",
    baseURL: "https://api.stringboot.com",
    analyticsHandler: FirebaseAnalyticsHandler()
)
```

4. **Implement analytics handler:**

```swift
import FirebaseAnalytics

class FirebaseAnalyticsHandler: StringbootAnalyticsHandler {
    func onExperimentsAssigned(experiments: [ExperimentAssignment]) {
        for exp in experiments {
            Analytics.logEvent("experiment_assigned", parameters: [
                "experiment_id": exp.experimentId,
                "variant_name": exp.variantName
            ])
        }
    }
}
```

The SDK handles the rest automatically!

### Do I need to change my code for A/B tests?

**No!** Continue using strings normally:

```swift
let ctaText = await StringProvider.shared.get("cta_button")
// OR
SBText("cta_button")
```

The SDK automatically returns the correct variant based on the user's assignment.

### How does device ID work for A/B testing?

The SDK generates a persistent device ID (stored in UserDefaults) to ensure consistent experiment assignments across app sessions.

**Provide your own device ID:**

```swift
StringProvider.shared.initialize(
    cacheSize: 1000,
    apiToken: "YOUR_API_TOKEN",
    baseURL: "https://api.stringboot.com",
    providedDeviceId: "user_12345" // Your user ID
)
```

### Can I test experiments locally?

Yes! Check current assignments:

```swift
Task {
    if let experiments = await StringProvider.shared.getActiveExperiments() {
        for exp in experiments {
            print("Experiment: \(exp.experimentId), Variant: \(exp.variantName)")
        }
    }
}
```

---

## FAQ Provider

### What is the FAQ Provider?

FAQ Provider lets you deliver dynamic, multilingual FAQs to your app users using the same offline-first architecture as string resources. FAQs are organized by tags and sub-tags.

See [INTEGRATION_GUIDE.md#faq-provider](INTEGRATION_GUIDE.md#-faq-provider-integration) for complete guide.

### How do I fetch FAQs?

```swift
let faqs = await FAQProvider.shared.getFAQs(
    tag: "payments",
    subTags: ["refunds"],
    lang: "en",
    allowNetworkFetch: true
)

displayFAQs(faqs)
```

### How do I make FAQs update automatically in SwiftUI?

Use reactive state:

```swift
struct FAQView: View {
    @State private var faqs: [FAQ] = []

    var body: some View {
        List(faqs, id: \.id) { faq in
            VStack(alignment: .leading) {
                Text(faq.question).font(.headline)
                Text(faq.answer).font(.body)
            }
        }
        .task {
            faqs = await FAQProvider.shared.getFAQs(tag: "payments")
        }
        .refreshable {
            await FAQProvider.shared.refreshFromNetwork()
            faqs = await FAQProvider.shared.getFAQs(tag: "payments")
        }
    }
}
```

### What are tags and sub-tags?

**Tags** organize FAQs into categories. **Sub-tags** provide finer filtering:

- **Tag:** "payments"
- **Sub-tags:** ["refunds", "disputes", "methods"]

```swift
// Get all payment FAQs
let allPaymentFAQs = await FAQProvider.shared.getFAQs(tag: "payments")

// Get only refund FAQs
let refundFAQs = await FAQProvider.shared.getFAQs(
    tag: "payments",
    subTags: ["refunds"]
)
```

### Do FAQs support the same language fallback as strings?

Yes! If FAQs aren't available in the requested language, they automatically fall back to English.

### Can I cache FAQs?

Yes! FAQProvider uses the same three-tier caching system as StringProvider. FAQs are cached in memory and Core Data for offline access.

---

## Offline Support

### Does Stringboot work offline?

**Yes, 100% offline functionality!** Once strings are synced, the app works perfectly without internet:
- Strings are cached in Core Data
- Language switching works offline
- A/B test assignments are cached
- FAQs work offline

### What happens on first app install without internet?

The app won't have strings until it can sync with the network. **Best practice:** Include critical strings as fallbacks in Localizable.strings.

### How do I know if strings are cached?

Check the initialization status:

```swift
if StringProvider.shared.isReady {
    print("StringProvider is ready with cached strings")
}
```

### Can I force a network refresh?

Yes:

```swift
Task {
    let success = await StringProvider.shared.refreshFromNetwork()
    if success {
        print("Strings refreshed!")
    }
}
```

---

## Troubleshooting

### Strings showing as "??key??"

**Possible causes:**

1. **Key doesn't exist** - Check Stringboot dashboard for correct key name
2. **Language not available** - Verify language code is correct (e.g., "en", not "eng")
3. **Network sync failed** - Check internet connection and API token
4. **Not initialized** - Ensure `initialize()` was called

**Debug:**

```swift
// Enable debug logging
StringbootLogger.setLogLevel(.debug)

// Check initialization
if !StringProvider.shared.isReady {
    print("StringProvider not ready!")
}
```

### Strings not updating after changing language

**Possible causes:**

1. **Not using auto-updating components** - Use SBText/SBLabel or publishers
2. **Cache not refreshed** - Call `refreshFromNetwork()` after language change
3. **Language not synced** - Ensure new language is downloaded

**Solution:**

```swift
Task {
    await StringProvider.shared.switchLanguage(to: "es")
    await StringProvider.shared.refreshFromNetwork(lang: "es")
}
```

### Network sync failing

**Check these:**

1. **API token valid?** - Verify token in Stringboot dashboard
2. **Network available?** - Check device internet connection
3. **Base URL correct?** - Should be `https://api.stringboot.com`
4. **App Transport Security?** - Ensure HTTPS is allowed in Info.plist

**Debug network calls:**

```swift
// Check last sync
StringbootLogger.setLogLevel(.debug)

// Manually refresh
let success = await StringProvider.shared.refreshFromNetwork()
print("Refresh success: \(success)")
```

### App crashing on initialization

**Common causes:**

1. **Calling StringProvider before initialization**
2. **Invalid API token format**
3. **Missing dependencies** (Core Data, Combine)

**Solution:**

```swift
// Always initialize in App.init() or AppDelegate
init() {
    do {
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "YOUR_API_TOKEN",
            baseURL: "https://api.stringboot.com"
        )
    } catch {
        print("Stringboot initialization failed: \(error)")
    }
}
```

### Slow string retrieval

**Possible causes:**

1. **Blocking main thread** - Always use async/await
2. **Cache miss** - Preload frequently used languages
3. **Large cache size** - Reduce cache size if memory constrained

**Optimization:**

```swift
// ✅ Good: Async/await
Task {
    let text = await StringProvider.shared.get("key")
}

// ❌ Bad: Blocking (not supported)
// Don't use synchronous calls
```

### Memory warnings related to Stringboot

**Solutions:**

1. **Reduce cache size:**
```swift
StringProvider.shared.initialize(
    cacheSize: 500, // Smaller cache
    apiToken: "YOUR_API_TOKEN",
    baseURL: "https://api.stringboot.com"
)
```

2. **Clear cache on low memory:**
```swift
// In AppDelegate or SceneDelegate
func applicationDidReceiveMemoryWarning(_ application: UIApplication) {
    Task {
        await StringProvider.shared.clearMemoryCache()
    }
}
```

### FAQs not appearing

**Debug steps:**

1. **Check initialization:**
```swift
if !FAQProvider.shared.isInitialized {
    print("FAQProvider not initialized!")
}
```

2. **Verify tag spelling** (case-sensitive!):
```swift
let faqs = await FAQProvider.shared.getFAQs(tag: "payments") // Correct
let faqs = await FAQProvider.shared.getFAQs(tag: "Payments") // Wrong if tag is lowercase
```

3. **Enable network fetch:**
```swift
let faqs = await FAQProvider.shared.getFAQs(
    tag: "payments",
    allowNetworkFetch: true // Enable network fallback
)
```

---

## Best Practices

### When should I use Stringboot vs NSLocalizedString?

**Use Stringboot for:**
- Marketing copy that changes frequently
- A/B tested content
- Seasonal messaging
- User-facing content you want to update quickly
- Multi-region content variations

**Use NSLocalizedString for:**
- System-level strings (Settings, Permissions)
- Critical error messages
- Fallback strings for offline-first launch
- Strings unlikely to change

### How do I handle string updates in production?

**Recommended workflow:**

1. **Update strings** in Stringboot dashboard
2. **Test in TestFlight** environment first
3. **Publish to production**
4. **Monitor analytics** for issues
5. **Rollback if needed** (instant via dashboard)

No app update required!

### Should I cache strings indefinitely?

Yes! The SDK automatically invalidates cache when strings are updated on the backend (using ETags). Trust the SDK's cache management.

### How do I test different languages?

**Method 1: Device language**
```swift
let deviceLang = StringProvider.shared.deviceLocale()
await StringProvider.shared.switchLanguage(to: deviceLang)
```

**Method 2: Debug menu (dev builds only)**
```swift
#if DEBUG
showLanguageSwitcher()
#endif
```

**Method 3: Scheme environment variable**
Add `-AppleLanguages (es)` to launch arguments in Xcode scheme

### How do I handle pluralization?

Use separate keys for each plural form:

```swift
let count = 5
let key = switch count {
    case 0: "items_zero"
    case 1: "items_one"
    default: "items_many"
}
let text = await StringProvider.shared.get(key)
let formatted = text.replacingOccurrences(of: "{count}", with: "\(count)")
```

Or use `stringsdict` for NSLocalizedString and Stringboot together.

### Should I use SwiftUI or UIKit components?

**Use auto-updating components (SBText, SBLabel) when:**
- Content changes frequently
- User can switch languages in-app
- You want reactive UI updates

**Use manual `get()` calls when:**
- One-time string retrieval
- Building custom components
- Performance-critical scenarios (avoid overhead)

### How do I migrate from NSLocalizedString to Stringboot?

**Gradual migration approach:**

1. **Phase 1:** Keep existing strings, add Stringboot for new features
2. **Phase 2:** Migrate high-frequency strings (marketing, CTAs)
3. **Phase 3:** Move remaining strings, keep critical ones as fallbacks
4. **Phase 4:** Monitor for issues, adjust as needed

**Helper function:**

```swift
func getStringWithFallback(key: String, tableName: String? = nil) async -> String {
    let stringbootValue = await StringProvider.shared.get(key)
    if stringbootValue.hasPrefix("??") {
        // Stringboot key not found, use NSLocalizedString
        return NSLocalizedString(key, tableName: tableName, comment: "")
    }
    return stringbootValue
}
```

---

## Still Have Questions?

- 📧 **Email:** support@stringboot.com
- 📚 **Documentation:** [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)
- 🐛 **Report Issues:** [GitHub Issues](https://github.com/stringboot/ios-sdk/issues)
- 💬 **Community:** [Discord](https://discord.gg/stringboot)

---

**Last Updated:** December 2024 | **SDK Version:** 2.0.0+
