# Stringboot iOS SDK - Integration Guide

> Comprehensive guide to integrating the Stringboot iOS SDK into your iOS/macOS application

**Version:** 2.0.0+ | **Last Updated:** December 2024

---

## Table of Contents

- [Quick Start](#-quick-start)
- [SwiftUI Integration](#-swiftui-integration)
- [UIKit Integration](#-uikit-integration)
- [Programmatic String Access](#-programmatic-string-access)
- [Reactive Publishers (Combine)](#-reactive-publishers-combine)
- [Language Switching](#-language-switching)
- [Performance Optimization](#-performance-optimization)
- [A/B Testing Integration](#-ab-testing-integration)
- [FAQ Provider Integration](#-faq-provider-integration)
- [Additional Resources](#-additional-resources)

---

## 🚀 Quick Start

### Step 1: Installation

Add Stringboot iOS SDK via Swift Package Manager:

```swift
dependencies: [
    .package(url: "https://github.com/your-org/stringboot-ios-sdk.git", from: "2.0.0")
]
```

Or in Xcode:
1. **File** > **Add Package Dependencies**
2. Enter repository URL
3. Select version and add to your target

### Step 2: Configuration

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

Add to your Xcode project:
1. Drag `stringboot-config.json` into your project navigator
2. Ensure **Copy items if needed** is checked
3. Add to target membership

### Step 3: Initialization

#### SwiftUI App

```swift
import SwiftUI
import StringbootSDK

@main
struct MyApp: App {
    init() {
        // Auto-initialize from stringboot-config.json
        StringbootManager.shared.autoInitialize()

        // OR manual initialization
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "YOUR_API_TOKEN",
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

#### UIKit AppDelegate

```swift
import UIKit
import StringbootSDK

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        // Auto-initialize from stringboot-config.json
        StringbootManager.shared.autoInitialize()

        return true
    }
}
```

### Step 4: Use Strings

#### SwiftUI

```swift
import SwiftUI
import StringbootSDK

struct WelcomeView: View {
    @StateObject private var languageManager = StringbootLanguageManager()

    var body: some View {
        VStack(spacing: 20) {
            // Auto-updating text
            SBText("welcome_message")
                .font(.title)
                .fontWeight(.bold)

            SBText("onboarding_subtitle")
                .font(.body)
                .foregroundColor(.secondary)

            // With custom language
            SBText("cta_button", lang: languageManager.currentLanguage)
                .font(.headline)
        }
    }
}
```

#### UIKit

```swift
import UIKit
import StringbootSDK

class WelcomeViewController: UIViewController {

    let titleLabel = SBLabel(key: "welcome_message")
    let subtitleLabel = SBLabel(key: "onboarding_subtitle")

    override func viewDidLoad() {
        super.viewDidLoad()

        // Auto-updating labels
        titleLabel.font = .systemFont(ofSize: 24, weight: .bold)
        view.addSubview(titleLabel)

        subtitleLabel.font = .systemFont(ofSize: 16)
        subtitleLabel.textColor = .secondaryLabel
        view.addSubview(subtitleLabel)
    }
}
```

---

## 🎨 SwiftUI Integration

### SBText - Auto-Updating Text View

`SBText` is a SwiftUI component that automatically updates when:
- Language changes
- String content is updated from the backend
- The specified string key is refreshed

#### Basic Usage

```swift
import SwiftUI
import StringbootSDK

struct ProductView: View {
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            SBText("product_name")
                .font(.largeTitle)
                .fontWeight(.bold)

            SBText("product_description")
                .font(.body)
                .foregroundColor(.secondary)
                .lineLimit(3)

            SBText("product_price")
                .font(.title2)
                .foregroundColor(.green)
        }
    }
}
```

#### With Language Parameter

```swift
struct LocalizedView: View {
    @State private var selectedLanguage = "en"

    var body: some View {
        VStack {
            SBText("greeting", lang: selectedLanguage)
                .font(.title)

            Picker("Language", selection: $selectedLanguage) {
                Text("English").tag("en")
                Text("Español").tag("es")
                Text("Français").tag("fr")
            }
            .pickerStyle(.segmented)
        }
    }
}
```

#### Custom Styling

```swift
SBText("error_message")
    .font(.headline)
    .foregroundColor(.red)
    .padding()
    .background(Color.red.opacity(0.1))
    .cornerRadius(8)
```

### StringbootLanguageManager

Use `StringbootLanguageManager` to manage language switching in SwiftUI:

```swift
struct ContentView: View {
    @StateObject private var languageManager = StringbootLanguageManager()

    var body: some View {
        NavigationView {
            VStack {
                SBText("title", lang: languageManager.currentLanguage)
                    .font(.largeTitle)

                // Language picker
                Picker("Language", selection: $languageManager.currentLanguage) {
                    ForEach(languageManager.availableLanguages, id: \.code) { lang in
                        Text(lang.name).tag(lang.code)
                    }
                }
                .pickerStyle(.menu)
                .onChange(of: languageManager.currentLanguage) { newLanguage in
                    Task {
                        await languageManager.switchLanguage(to: newLanguage)
                    }
                }
            }
            .navigationTitle("Settings")
        }
    }
}
```

### Property Wrappers

#### @StringResource

```swift
struct ProfileView: View {
    @StringResource("user_name") var userName
    @StringResource("user_bio") var userBio

    var body: some View {
        VStack(alignment: .leading) {
            Text(userName)
                .font(.title)

            Text(userBio)
                .font(.body)
        }
    }
}
```

### View Modifiers

```swift
extension View {
    func localizedNavigation() -> some View {
        navigationTitle(StringProvider.shared.get("screen_title", lang: "en"))
    }
}

// Usage
struct SettingsView: View {
    var body: some View {
        Form {
            // Settings content
        }
        .localizedNavigation()
    }
}
```

---

## 📱 UIKit Integration

### SBLabel - Auto-Updating UILabel

`SBLabel` is a UIKit component that automatically updates when strings change:

#### Basic Usage

```swift
import UIKit
import StringbootSDK

class ProductViewController: UIViewController {

    let titleLabel = SBLabel(key: "product_name")
    let descriptionLabel = SBLabel(key: "product_description")
    let priceLabel = SBLabel(key: "product_price")

    override func viewDidLoad() {
        super.viewDidLoad()

        // Styling
        titleLabel.font = .systemFont(ofSize: 28, weight: .bold)
        titleLabel.textColor = .label

        descriptionLabel.font = .systemFont(ofSize: 16)
        descriptionLabel.textColor = .secondaryLabel
        descriptionLabel.numberOfLines = 0

        priceLabel.font = .systemFont(ofSize: 22, weight: .semibold)
        priceLabel.textColor = .systemGreen

        // Layout
        setupLayout()
    }

    private func setupLayout() {
        let stackView = UIStackView(arrangedSubviews: [
            titleLabel,
            descriptionLabel,
            priceLabel
        ])
        stackView.axis = .vertical
        stackView.spacing = 12
        stackView.translatesAutoresizingMaskIntoConstraints = false

        view.addSubview(stackView)

        NSLayoutConstraint.activate([
            stackView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            stackView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            stackView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20)
        ])
    }
}
```

#### With Custom Language

```swift
let greetingLabel = SBLabel(key: "greeting", lang: "es")
greetingLabel.font = .systemFont(ofSize: 20)
view.addSubview(greetingLabel)
```

#### Dynamic Language Switching

```swift
class SettingsViewController: UIViewController {

    let welcomeLabel = SBLabel(key: "welcome_message")

    @objc func languageChanged(_ notification: Notification) {
        // SBLabel automatically updates when language changes
        // No manual refresh needed!
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        NotificationCenter.default.addObserver(
            self,
            selector: #selector(languageChanged),
            name: NSNotification.Name("StringbootLanguageChanged"),
            object: nil
        )
    }
}
```

### SBButton - Auto-Updating UIButton

```swift
let primaryButton = SBButton(key: "cta_button")
primaryButton.titleLabel?.font = .systemFont(ofSize: 18, weight: .semibold)
primaryButton.backgroundColor = .systemBlue
primaryButton.setTitleColor(.white, for: .normal)
primaryButton.layer.cornerRadius = 12
primaryButton.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)

view.addSubview(primaryButton)
```

### UITextField Placeholder

```swift
let emailTextField = UITextField()
Task {
    emailTextField.placeholder = await StringProvider.shared.get("email_placeholder")
}

// Or use placeholder helper
emailTextField.setLocalizedPlaceholder(key: "email_placeholder")
```

---

## 💻 Programmatic String Access

### Async/Await (Recommended)

```swift
import StringbootSDK

class DataManager {

    func loadUserProfile() async {
        let welcomeMessage = await StringProvider.shared.get("welcome_message")
        let subtitle = await StringProvider.shared.get("profile_subtitle", lang: "en")

        print("Welcome: \(welcomeMessage)")
        print("Subtitle: \(subtitle)")
    }

    func handleError() async {
        let errorTitle = await StringProvider.shared.get("error_title")
        let errorMessage = await StringProvider.shared.get("network_error_message")

        showAlert(title: errorTitle, message: errorMessage)
    }
}
```

### With Custom Language

```swift
func getLocalizedContent(for language: String) async -> String {
    return await StringProvider.shared.get("content_key", lang: language)
}

// Usage
Task {
    let frenchContent = await getLocalizedContent(for: "fr")
    let spanishContent = await getLocalizedContent(for: "es")
}
```

### Batch Retrieval

```swift
func loadMultipleStrings() async {
    async let title = StringProvider.shared.get("title")
    async let subtitle = StringProvider.shared.get("subtitle")
    async let cta = StringProvider.shared.get("cta_button")

    let (titleText, subtitleText, ctaText) = await (title, subtitle, cta)

    updateUI(title: titleText, subtitle: subtitleText, cta: ctaText)
}
```

### Synchronous Access (Not Recommended)

```swift
// ⚠️ Only use for non-critical UI or when async is not possible
let text = StringProvider.shared.getSync("key")
```

---

## 🔄 Reactive Publishers (Combine)

StringProvider offers Combine publishers for reactive string updates:

### Single String Publisher

```swift
import Combine
import StringbootSDK

class ViewModel: ObservableObject {
    @Published var welcomeText: String = ""
    private var cancellables = Set<AnyCancellable>()

    init() {
        StringProvider.shared.publisher(for: "welcome_message", lang: "en")
            .assign(to: &$welcomeText)
    }
}
```

### Multiple String Publishers

```swift
class ProductViewModel: ObservableObject {
    @Published var title: String = ""
    @Published var description: String = ""
    @Published var price: String = ""
    private var cancellables = Set<AnyCancellable>()

    init() {
        StringProvider.shared.publisher(for: "product_title")
            .assign(to: &$title)

        StringProvider.shared.publisher(for: "product_description")
            .assign(to: &$description)

        StringProvider.shared.publisher(for: "product_price")
            .assign(to: &$price)
    }
}
```

### Language Change Publisher

```swift
StringProvider.shared.$currentLanguage
    .dropFirst() // Ignore initial value
    .sink { newLanguage in
        print("Language changed to: \(newLanguage ?? "unknown")")
        // Reload data, refresh UI, etc.
    }
    .store(in: &cancellables)
```

### Combining Multiple Publishers

```swift
Publishers.CombineLatest3(
    StringProvider.shared.publisher(for: "first_name"),
    StringProvider.shared.publisher(for: "last_name"),
    StringProvider.shared.publisher(for: "greeting")
)
.map { firstName, lastName, greeting in
    "\(greeting) \(firstName) \(lastName)"
}
.assign(to: &$fullGreeting)
```

---

## 🌍 Language Switching

### Get Available Languages

```swift
Task {
    let languages = await StringProvider.shared.getAvailableLanguages()
    for lang in languages {
        print("\(lang.code): \(lang.name) (Default: \(lang.isDefault))")
    }
}
```

### Switch Language

```swift
func switchLanguage(to newLanguage: String) async {
    let success = await StringProvider.shared.switchLanguage(to: newLanguage)

    if success {
        print("Language switched to \(newLanguage)")
        // UI automatically updates via publishers and auto-updating components
    } else {
        print("Failed to switch language")
    }
}
```

### Detect Device Language

```swift
let deviceLanguage = StringProvider.shared.deviceLocale()
print("Device language: \(deviceLanguage)")
```

### SwiftUI Language Switching

```swift
struct LanguageSwitcher: View {
    @StateObject private var languageManager = StringbootLanguageManager()
    @State private var isLoading = false

    var body: some View {
        VStack {
            if isLoading {
                ProgressView()
            }

            Picker("Language", selection: $languageManager.currentLanguage) {
                ForEach(languageManager.availableLanguages, id: \.code) { lang in
                    HStack {
                        Text(lang.name)
                        if lang.isDefault {
                            Image(systemName: "star.fill")
                                .foregroundColor(.yellow)
                        }
                    }
                    .tag(lang.code)
                }
            }
            .pickerStyle(.wheel)
            .disabled(isLoading)
            .onChange(of: languageManager.currentLanguage) { newLang in
                Task {
                    isLoading = true
                    await languageManager.switchLanguage(to: newLang)
                    isLoading = false
                }
            }
        }
    }
}
```

### UIKit Language Switching

```swift
class LanguageSettingsViewController: UIViewController {

    let languagePicker = UIPickerView()
    var availableLanguages: [ActiveLanguage] = []

    override func viewDidLoad() {
        super.viewDidLoad()

        Task {
            availableLanguages = await StringProvider.shared.getAvailableLanguages()
            languagePicker.reloadAllComponents()
        }

        languagePicker.delegate = self
        languagePicker.dataSource = self
    }
}

extension LanguageSettingsViewController: UIPickerViewDelegate {
    func pickerView(_ pickerView: UIPickerView, didSelectRow row: Int, inComponent component: Int) {
        let selectedLanguage = availableLanguages[row]

        Task {
            let success = await StringProvider.shared.switchLanguage(to: selectedLanguage.code)
            if success {
                // UI components automatically update
                showSuccessMessage()
            }
        }
    }
}

extension LanguageSettingsViewController: UIPickerViewDataSource {
    func numberOfComponents(in pickerView: UIPickerView) -> Int { 1 }

    func pickerView(_ pickerView: UIPickerView, numberOfRowsInComponent component: Int) -> Int {
        availableLanguages.count
    }

    func pickerView(_ pickerView: UIPickerView, titleForRow row: Int, forComponent component: Int) -> String? {
        let lang = availableLanguages[row]
        return "\(lang.name) (\(lang.code))"
    }
}
```

---

## ⚡ Performance Optimization

### Preloading Languages

Preload frequently accessed languages to improve performance:

```swift
@main
struct MyApp: App {
    init() {
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "YOUR_API_TOKEN",
            baseURL: "https://api.stringboot.com"
        )

        // Preload common languages
        Task {
            await StringProvider.shared.preloadLanguage("en", maxStrings: 500)
            await StringProvider.shared.preloadLanguage("es", maxStrings: 500)
        }
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

### Cache Monitoring

```swift
func monitorCachePerformance() async {
    let stats = await StringProvider.shared.getCacheStatistics()

    print("Cache size: \(stats.currentSize)")
    print("Hit rate: \(stats.hitRate * 100)%")
    print("Miss rate: \(stats.missRate * 100)%")
    print("Total requests: \(stats.totalRequests)")
}
```

### Clear Cache

```swift
// Clear cache for specific language
await StringProvider.shared.clearCache(for: "en")

// Clear all cache
await StringProvider.shared.clearAllCache()
```

### Network Refresh

Manually refresh strings from the network:

```swift
func refreshStrings() async {
    let success = await StringProvider.shared.refreshFromNetwork()

    if success {
        print("Strings refreshed successfully")
    } else {
        print("Failed to refresh strings")
    }
}
```

### Background Refresh

```swift
import BackgroundTasks

func scheduleStringRefresh() {
    let request = BGAppRefreshTaskRequest(identifier: "com.myapp.stringRefresh")
    request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60) // 15 minutes

    do {
        try BGTaskScheduler.shared.submit(request)
    } catch {
        print("Could not schedule app refresh: \(error)")
    }
}

func handleAppRefresh(task: BGAppRefreshTask) {
    Task {
        await StringProvider.shared.refreshFromNetwork()
        task.setTaskCompleted(success: true)
    }

    task.expirationHandler = {
        task.setTaskCompleted(success: false)
    }
}
```

---

## 🧪 A/B Testing Integration

> **Available in:** SDK v2.0.0+

The Stringboot iOS SDK includes first-class A/B testing support, allowing you to test different string variations with your users.

### Overview

- **Automatic Assignment** - Users are automatically assigned to experiment variants
- **Device ID Management** - Consistent assignments across app sessions
- **Analytics Integration** - Firebase Analytics, Mixpanel, Amplitude support
- **Offline Support** - Assignments cached locally
- **No Code Changes** - Experiments managed from Stringboot dashboard

### Initialization with A/B Testing

```swift
import StringbootSDK
import FirebaseAnalytics

@main
struct MyApp: App {
    init() {
        // Initialize with analytics handler
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "YOUR_API_TOKEN",
            baseURL: "https://api.stringboot.com",
            analyticsHandler: FirebaseAnalyticsHandler()
        )
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

### Device ID Management

The SDK automatically generates and persists a device ID for consistent experiment assignments:

```swift
// Get current device ID
if let deviceId = await StringProvider.shared.getDeviceId() {
    print("Device ID: \(deviceId)")
}

// Provide your own device ID (e.g., user ID after login)
StringProvider.shared.initialize(
    cacheSize: 1000,
    apiToken: "YOUR_API_TOKEN",
    baseURL: "https://api.stringboot.com",
    providedDeviceId: "user_12345"
)
```

### Analytics Integration

#### Firebase Analytics

```swift
import FirebaseAnalytics

class FirebaseAnalyticsHandler: StringbootAnalyticsHandler {
    func onExperimentsAssigned(experiments: [ExperimentAssignment]) {
        for experiment in experiments {
            Analytics.logEvent("experiment_assigned", parameters: [
                "experiment_id": experiment.experimentId,
                "variant_name": experiment.variantName,
                "variant_id": experiment.variantId
            ])

            // Set user property for segmentation
            Analytics.setUserProperty(
                experiment.variantName,
                forName: "exp_\(experiment.experimentId)"
            )
        }
    }
}
```

#### Mixpanel

```swift
import Mixpanel

class MixpanelAnalyticsHandler: StringbootAnalyticsHandler {
    func onExperimentsAssigned(experiments: [ExperimentAssignment]) {
        for experiment in experiments {
            Mixpanel.mainInstance().track(event: "Experiment Assigned", properties: [
                "experiment_id": experiment.experimentId,
                "variant_name": experiment.variantName,
                "variant_id": experiment.variantId
            ])

            // Register super property
            Mixpanel.mainInstance().registerSuperProperties([
                "exp_\(experiment.experimentId)": experiment.variantName
            ])
        }
    }
}
```

#### Amplitude

```swift
import Amplitude

class AmplitudeAnalyticsHandler: StringbootAnalyticsHandler {
    func onExperimentsAssigned(experiments: [ExperimentAssignment]) {
        for experiment in experiments {
            Amplitude.instance().logEvent("Experiment Assigned", withEventProperties: [
                "experiment_id": experiment.experimentId,
                "variant_name": experiment.variantName,
                "variant_id": experiment.variantId
            ])

            // Set user property
            let identify = AMPIdentify()
                .set("exp_\(experiment.experimentId)", value: experiment.variantName as NSObject)
            Amplitude.instance().identify(identify)
        }
    }
}
```

### How It Works

1. **Backend Setup** - Create experiments in Stringboot dashboard
2. **String Variants** - Define different string variations for each experiment
3. **Automatic Assignment** - SDK receives assignments from API
4. **Analytics Callback** - Your analytics handler is notified
5. **String Delivery** - Correct variant is automatically shown to user

**Example:**

```swift
// On Stringboot backend:
// Experiment: "onboarding_cta_test"
// Variant A: "Get Started" (50% of users)
// Variant B: "Start Free Trial" (50% of users)

// In your app:
SBText("onboarding_cta_button")
    .font(.headline)
    .padding()
    .background(Color.blue)

// SDK automatically shows correct variant based on user's assignment
// Analytics handler receives notification for tracking
```

### Testing Experiments

```swift
// View active experiments (for debugging)
func debugExperiments() async {
    if let experiments = await StringProvider.shared.getActiveExperiments() {
        for exp in experiments {
            print("Experiment: \(exp.experimentId)")
            print("Variant: \(exp.variantName) (ID: \(exp.variantId))")
        }
    }
}
```

### Best Practices

1. **Initialize Early** - Set up analytics handler before any strings are fetched
2. **Use Device ID** - Provide consistent user ID for logged-in users
3. **Track Conversions** - Log conversion events with experiment context
4. **Test Thoroughly** - Verify analytics events before launching experiments

For complete details, see [AB_TESTING.md](AB_TESTING.md)

---

## 📋 FAQ Provider Integration

> **Available in:** SDK v1.1.0+

The FAQ Provider enables you to deliver dynamic, multilingual FAQ content to your users with the same offline-first, cached architecture as string resources.

### Overview

FAQProvider offers:
- **Tag-based organization** - Categorize FAQs by primary tag and optional sub-tags
- **Multi-language support** - Automatic language fallback to English
- **Offline-first** - Three-tier caching (memory → Core Data → network)
- **Delta sync** - Only download changed FAQs
- **Reactive updates** - Combine publishers for real-time UI updates

### Initialization

FAQProvider is automatically initialized when you initialize StringProvider:

```swift
import StringbootSDK

@main
struct MyApp: App {
    init() {
        // This initializes both StringProvider AND FAQProvider
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "YOUR_API_TOKEN",
            baseURL: "https://api.stringboot.com"
        )

        // Optionally initialize FAQProvider separately
        Task {
            FAQProvider.shared.initialize(
                cacheSize: 200,
                apiToken: "YOUR_API_TOKEN",
                baseURL: "https://api.stringboot.com"
            )
        }
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

### Basic Usage

#### Fetching FAQs

```swift
import StringbootSDK

// In your View or ViewController
func loadFAQs() async {
    let faqs = await FAQProvider.shared.getFAQs(
        tag: "Identity Verification",
        subTags: ["Document Upload"],
        lang: "en",
        allowNetworkFetch: true
    )

    // Display FAQs in UI
    self.faqs = faqs
}
```

#### FAQ Data Model

```swift
public struct FAQ {
    let id: Int                      // Unique FAQ ID
    let question: String             // FAQ question
    let answer: String               // FAQ answer (supports HTML/markdown)
    let tag: String                  // Primary tag (e.g., "payments")
    let subTags: [String]            // Sub-tags (e.g., ["refunds", "disputes"])
    let languageCode: String?        // Language code (e.g., "en", "es")
    let appId: String?               // App identifier
    let createdAt: String?           // ISO 8601 timestamp
    let updatedAt: String?           // ISO 8601 timestamp
}
```

### SwiftUI Integration

#### Basic FAQ List

```swift
import SwiftUI
import StringbootSDK

struct FAQListView: View {
    @State private var faqs: [FAQ] = []
    @State private var isLoading = false
    @State private var selectedTag = "Identity Verification"
    @State private var expandedFAQIds: Set<Int> = []

    var body: some View {
        List {
            ForEach(faqs, id: \.id) { faq in
                FAQItemView(
                    faq: faq,
                    isExpanded: expandedFAQIds.contains(faq.id),
                    onTap: { toggleExpanded(faq.id) }
                )
            }
        }
        .task {
            await loadFAQs()
        }
        .refreshable {
            await refreshFAQs()
        }
    }

    func loadFAQs() async {
        isLoading = true
        faqs = await FAQProvider.shared.getFAQs(
            tag: selectedTag,
            lang: StringProvider.shared.deviceLocale()
        )
        isLoading = false
    }

    func toggleExpanded(_ id: Int) {
        if expandedFAQIds.contains(id) {
            expandedFAQIds.remove(id)
        } else {
            expandedFAQIds.insert(id)
        }
    }

    func refreshFAQs() async {
        await FAQProvider.shared.refreshFromNetwork()
        await loadFAQs()
    }
}
```

#### Expandable FAQ Item

```swift
struct FAQItemView: View {
    let faq: FAQ
    let isExpanded: Bool
    let onTap: () -> Void

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            HStack {
                Text(faq.question)
                    .font(.headline)
                    .foregroundColor(.primary)

                Spacer()

                Image(systemName: isExpanded ? "chevron.up" : "chevron.down")
                    .foregroundColor(.secondary)
            }
            .contentShape(Rectangle())
            .onTapGesture(perform: onTap)

            if isExpanded {
                Text(faq.answer)
                    .font(.body)
                    .foregroundColor(.secondary)
                    .padding(.top, 4)
                    .transition(.opacity)
            }
        }
        .padding(.vertical, 8)
        .animation(.easeInOut(duration: 0.2), value: isExpanded)
    }
}
```

### Tag-Based Filtering

FAQs are organized by **primary tag** and optional **sub-tags** for precise categorization:

```swift
// Get all FAQs for "payments" tag
let allPaymentFAQs = await FAQProvider.shared.getFAQs(tag: "payments")

// Get only refund-related FAQs
let refundFAQs = await FAQProvider.shared.getFAQs(
    tag: "payments",
    subTags: ["refunds"]
)

// Get FAQs matching multiple sub-tags (OR filter)
let paymentMethodFAQs = await FAQProvider.shared.getFAQs(
    tag: "payments",
    subTags: ["card", "bank_transfer", "wallet"]
)
```

**Common Tag Hierarchies:**

| Primary Tag | Sub-Tags |
|------------|----------|
| `payments` | `refunds`, `disputes`, `methods`, `failed` |
| `account` | `profile`, `security`, `deletion`, `privacy` |
| `identity_verification` | `document_upload`, `verification_failed`, `liveness` |
| `support` | `contact`, `escalation`, `feedback` |

### Language Fallback

FAQProvider automatically falls back to English if FAQs aren't available in the requested language:

```swift
// User's device is set to "fr" (French)
let faqs = await FAQProvider.shared.getFAQs(
    tag: "payments",
    lang: "fr",  // Will try French first
    allowNetworkFetch: true
)

// Fallback order:
// 1. Memory cache (French)
// 2. Core Data (French)
// 3. Core Data (English fallback)
// 4. Network (French)
// 5. Empty array
```

**Check which language was returned:**

```swift
let faqs = await FAQProvider.shared.getFAQs(tag: "payments", lang: "fr")
if let firstFAQ = faqs.first {
    let actualLanguage = firstFAQ.languageCode ?? "unknown"
    if actualLanguage == "en" && "fr" != "en" {
        showLanguageFallbackNotice()
    }
}
```

### Network Refresh

Manually refresh FAQs from the network (uses delta sync for efficiency):

```swift
func refreshFAQs() async {
    let success = await FAQProvider.shared.refreshFromNetwork()

    if success {
        print("FAQs refreshed successfully")
    }
}
```

### Complete Example: FAQ Screen with Filters

```swift
import SwiftUI
import StringbootSDK

struct FAQScreen: View {
    @State private var faqs: [FAQ] = []
    @State private var isLoading = false
    @State private var errorMessage: String?
    @State private var expandedFAQIds: Set<Int> = []

    // Filter state
    @State private var currentTag = "Identity Verification"
    @State private var selectedSubTags: Set<String> = []
    @State private var currentLanguage = "en"

    let availableSubTags = ["Document Upload", "Verification Failed", "Liveness Check"]

    var body: some View {
        VStack(spacing: 0) {
            // Header
            VStack(alignment: .leading, spacing: 8) {
                SBText("faq-page-title")
                    .font(.largeTitle)
                    .fontWeight(.bold)

                Text("Tag: \(currentTag)")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            .frame(maxWidth: .infinity, alignment: .leading)
            .padding()

            Divider()

            // SubTag filters
            ScrollView(.horizontal, showsIndicators: false) {
                HStack(spacing: 12) {
                    // "All FAQs" chip
                    FilterChip(
                        title: "All FAQs",
                        isSelected: selectedSubTags.isEmpty,
                        action: {
                            selectedSubTags.removeAll()
                            Task { await loadFAQs() }
                        }
                    )

                    // SubTag chips
                    ForEach(availableSubTags, id: \.self) { subTag in
                        FilterChip(
                            title: subTag,
                            isSelected: selectedSubTags.contains(subTag),
                            action: { toggleSubTag(subTag) }
                        )
                    }
                }
                .padding(.horizontal)
                .padding(.vertical, 12)
            }

            Divider()

            // FAQ List
            if isLoading {
                Spacer()
                ProgressView("Loading FAQs...")
                Spacer()
            } else if let error = errorMessage {
                ErrorView(message: error) {
                    Task { await loadFAQs() }
                }
            } else if faqs.isEmpty {
                EmptyStateView(message: "No FAQs available for \(currentTag)")
            } else {
                List {
                    ForEach(faqs, id: \.id) { faq in
                        FAQItemView(
                            faq: faq,
                            isExpanded: expandedFAQIds.contains(faq.id),
                            onTap: { toggleExpanded(faq.id) }
                        )
                    }
                }
                .listStyle(.plain)
            }

            Divider()

            // Refresh button
            Button(action: { Task { await refreshFAQs() } }) {
                HStack {
                    Image(systemName: "arrow.clockwise")
                    Text("Refresh FAQs")
                }
                .padding()
            }
        }
        .task {
            await loadFAQs()
        }
    }

    func loadFAQs() async {
        isLoading = true
        errorMessage = nil

        faqs = await FAQProvider.shared.getFAQs(
            tag: currentTag,
            subTags: Array(selectedSubTags),
            lang: currentLanguage,
            allowNetworkFetch: true
        )

        isLoading = false
    }

    func toggleSubTag(_ subTag: String) {
        if selectedSubTags.contains(subTag) {
            selectedSubTags.remove(subTag)
        } else {
            selectedSubTags.insert(subTag)
        }
        Task { await loadFAQs() }
    }

    func toggleExpanded(_ id: Int) {
        if expandedFAQIds.contains(id) {
            expandedFAQIds.remove(id)
        } else {
            expandedFAQIds.insert(id)
        }
    }

    func refreshFAQs() async {
        await FAQProvider.shared.refreshFromNetwork()
        await loadFAQs()
    }
}

// Filter Chip Component
struct FilterChip: View {
    let title: String
    let isSelected: Bool
    let action: () -> Void

    var body: some View {
        Button(action: action) {
            Text(title)
                .font(.subheadline)
                .padding(.horizontal, 16)
                .padding(.vertical, 8)
                .background(isSelected ? Color.blue : Color.gray.opacity(0.2))
                .foregroundColor(isSelected ? .white : .primary)
                .cornerRadius(20)
        }
    }
}
```

### UIKit Integration

```swift
import UIKit
import StringbootSDK

class FAQViewController: UIViewController {

    var faqs: [FAQ] = []
    let tableView = UITableView()
    let activityIndicator = UIActivityIndicatorView(style: .large)

    var currentTag = "Identity Verification"
    var selectedSubTags: [String] = []

    override func viewDidLoad() {
        super.viewDidLoad()

        setupUI()
        loadFAQs()
    }

    func setupUI() {
        title = "FAQs"

        tableView.delegate = self
        tableView.dataSource = self
        tableView.register(FAQCell.self, forCellReuseIdentifier: "FAQCell")

        view.addSubview(tableView)
        view.addSubview(activityIndicator)

        tableView.frame = view.bounds
        activityIndicator.center = view.center
    }

    func loadFAQs() {
        activityIndicator.startAnimating()

        Task {
            faqs = await FAQProvider.shared.getFAQs(
                tag: currentTag,
                subTags: selectedSubTags,
                allowNetworkFetch: true
            )

            await MainActor.run {
                activityIndicator.stopAnimating()
                tableView.reloadData()
            }
        }
    }
}

extension FAQViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        faqs.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "FAQCell", for: indexPath) as! FAQCell
        cell.configure(with: faqs[indexPath.row])
        return cell
    }
}

extension FAQViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        // Expand/collapse FAQ
    }
}
```

### Demo App Reference

For a complete working implementation, see:
- [FAQDemoView.swift](stringboot-ios-demoApp/Stringboot%20iOS%20Demo/FAQDemoView.swift) - Full FAQ screen with filtering
- iOS Demo App → "FAQ Demo" section

### Performance & Caching

FAQProvider uses the same three-tier caching strategy as StringProvider:

1. **Memory Cache (L1)** - NSCache for instant access
2. **Core Data (L2)** - Persistent offline storage
3. **Network (L3)** - Delta sync for changed FAQs only

**Cache Performance:**
- Memory cache hit: <5ms
- Core Data hit: <50ms
- Network (delta sync): Downloads only changed FAQs since last sync
- Full catalog: Only on first launch or cache invalidation

### Best Practices

**1. Use async/await for FAQ Loading**

```swift
// ✅ Good: Async loading
Task {
    let faqs = await FAQProvider.shared.getFAQs(tag: "payments")
    updateUI(faqs)
}

// ❌ Avoid: Blocking main thread
let faqs = FAQProvider.shared.getFAQsSync(tag: "payments") // Not available
```

**2. Organize Tags Hierarchically**

```swift
// ✅ Good: Specific tag + sub-tags
await FAQProvider.shared.getFAQs(
    tag: "identity_verification",
    subTags: ["document_upload", "liveness_check"]
)

// ❌ Avoid: Generic tags that return too many FAQs
await FAQProvider.shared.getFAQs(tag: "general")
```

**3. Refresh on App Launch**

```swift
// In App.init() or viewDidLoad()
Task {
    await FAQProvider.shared.refreshFromNetwork()
}
```

**4. Handle Empty States**

```swift
let faqs = await FAQProvider.shared.getFAQs(tag: "payments")
if faqs.isEmpty {
    // Show "No FAQs available" message or contact support button
    showEmptyState()
}
```

### Troubleshooting

**FAQs not appearing:**
```swift
// 1. Check initialization
if !FAQProvider.shared.isInitialized {
    print("FAQProvider not initialized!")
}

// 2. Enable logging
StringbootLogger.setLogLevel(.debug)

// 3. Check network sync
let success = await FAQProvider.shared.refreshFromNetwork()
print("Network sync: \(success)")

// 4. Verify tag/subTag spelling
let faqs = await FAQProvider.shared.getFAQs(tag: "payments") // Case-sensitive!
```

**Language fallback not working:**
```swift
// Check actual language returned
let faqs = await FAQProvider.shared.getFAQs(tag: "payments", lang: "fr")
if let actualLang = faqs.first?.languageCode {
    print("Requested: fr, Got: \(actualLang)")
}
```

### API Reference

See [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md) for:
- FAQ cache management
- Custom cache sizes
- Network sync strategies
- Delta sync protocol details

---

## 📚 Additional Resources

- [QUICKSTART.md](QUICKSTART.md) - 10-minute integration guide
- [AB_TESTING.md](AB_TESTING.md) - Complete A/B testing guide
- [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md) - Advanced SDK features
- [API_REFERENCE.md](API_REFERENCE.md) - Complete API documentation
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Common issues & solutions
- [API Reference](../docs/API_REFERENCE.md) - Backend API endpoints
- [Delta Sync Protocol](../docs/DELTA_SYNC_PROTOCOL.md) - How sync works under the hood

---

## 🆘 Support

- 📧 Email: support@stringboot.com
- 🐛 Issues: [GitHub Issues](https://github.com/stringboot/ios-sdk/issues)
- 📚 Docs: [docs.stringboot.com](https://docs.stringboot.com)

---

**Made with ❤️ by the Stringboot Team**

SDK Version: 2.0.0 | Last Updated: December 2024 | A/B Testing & FAQ Support Added
