# FAQ Provider - iOS SDK

> Deliver dynamic, multilingual FAQ content to your users with offline-first caching

**Version:** 1.1.0+ | **Last Updated:** December 2024

---

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Core Concepts](#core-concepts)
- [Basic Usage](#basic-usage)
- [SwiftUI Integration](#swiftui-integration)
- [UIKit Integration](#uikit-integration)
- [Tag-Based Organization](#tag-based-organization)
- [Language Support](#language-support)
- [Advanced Usage](#advanced-usage)
- [Performance & Caching](#performance--caching)
- [Best Practices](#best-practices)
- [API Reference](#api-reference)
- [Examples](#examples)
- [Troubleshooting](#troubleshooting)

---

## Overview

The FAQ Provider enables you to deliver dynamic, context-aware FAQ content to your iOS app users using the same offline-first, cached architecture as string resources.

### Key Features

✅ **Tag-based Organization** - Categorize FAQs by primary tag and optional sub-tags
✅ **Multi-language Support** - Automatic language fallback to English
✅ **Offline-First** - Three-tier caching (memory → Core Data → network)
✅ **Delta Sync** - Only download changed FAQs
✅ **SwiftUI & UIKit** - Native integration for both frameworks
✅ **Search & Filter** - Filter by tags, sub-tags, and keywords
✅ **Rich Content** - Supports HTML/Markdown in answers

### Use Cases

- **In-app Help Centers** - Contextual help based on user location in app
- **Onboarding FAQs** - Guide new users with dynamic content
- **Feature-specific Help** - Show relevant FAQs for each feature
- **Multi-language Support Centers** - Automatic translation with fallback
- **Dynamic Knowledge Base** - Update help content without app updates

---

## Quick Start

### Step 1: Initialize

FAQProvider must be initialized separately from StringProvider:

**SwiftUI:**
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

        // Initialize FAQProvider separately
        FAQProvider.shared.initialize(
            cacheSize: 200,  // Default: 200 FAQs in memory
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
    // Initialize StringProvider
    StringProvider.shared.initialize(
        cacheSize: 1000,
        apiToken: "YOUR_API_TOKEN",
        baseURL: "https://api.stringboot.com"
    )

    // Initialize FAQProvider separately
    FAQProvider.shared.initialize(
        cacheSize: 200,
        apiToken: "YOUR_API_TOKEN",
        baseURL: "https://api.stringboot.com"
    )

    return true
}
```

### Step 2: Fetch FAQs

```swift
class FAQViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        Task {
            let faqs = await FAQProvider.shared.getFAQs(
                tag: "payments",
                subTags: ["refunds"],
                lang: "en",
                allowNetworkFetch: true
            )

            displayFAQs(faqs)
        }
    }
}
```

### Step 3: Display FAQs

```swift
func displayFAQs(_ faqs: [FAQ]) {
    faqs.forEach { faq in
        print("Q: \(faq.question)")
        print("A: \(faq.answer)")
    }
}
```

That's it! You're now serving dynamic FAQs to your users.

---

## Core Concepts

### FAQ Data Model

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

### Tag Hierarchy

FAQs are organized using a **two-level hierarchy**:

1. **Primary Tag** - Main category (required)
2. **Sub-Tags** - Specific topics within category (optional)

**Example:**

```
payments (Primary Tag)
├── refunds (Sub-tag)
├── disputes (Sub-tag)
├── methods (Sub-tag)
└── failed (Sub-tag)
```

### Caching Architecture

FAQProvider uses a **three-tier caching system**:

```
User Request
    ↓
[L1: Memory Cache] (NSCache, <5ms)
    ↓ (cache miss)
[L2: Core Data] (Persistent, <50ms)
    ↓ (cache miss)
[L3: Network API] (Delta sync)
    ↓
Return FAQs
```

**Benefits:**
- **Instant access** from memory cache
- **Offline support** via Core Data
- **Minimal bandwidth** with delta sync

---

## Basic Usage

### Fetch FAQs by Tag

```swift
// Get all FAQs for a tag
let faqs = await FAQProvider.shared.getFAQs(
    tag: "payments",
    lang: "en"
)

// Display in UI
displayFAQs(faqs)
```

### Fetch FAQs with Sub-tag Filter

```swift
// Get only refund-related FAQs
let refundFAQs = await FAQProvider.shared.getFAQs(
    tag: "payments",
    subTags: ["refunds"],
    lang: "en"
)

displayFAQs(refundFAQs)
```

### Fetch FAQs with Multiple Sub-tags (OR filter)

```swift
// Get FAQs matching ANY of the sub-tags
let paymentMethodFAQs = await FAQProvider.shared.getFAQs(
    tag: "payments",
    subTags: ["card", "bank_transfer", "wallet"],
    lang: "en"
)

displayFAQs(paymentMethodFAQs)
```

### Enable Network Fallback

```swift
let faqs = await FAQProvider.shared.getFAQs(
    tag: "identity_verification",
    subTags: ["document_upload"],
    lang: "en",
    allowNetworkFetch: true  // Fetch from network if not cached
)

displayFAQs(faqs)
```

---

## SwiftUI Integration

### Basic FAQ List

```swift
import SwiftUI
import StringbootSDK

struct FAQListView: View {
    @State private var faqs: [FAQ] = []
    @State private var isLoading = false

    let tag = "payments"

    var body: some View {
        List(faqs, id: \.id) { faq in
            FAQItemView(faq: faq)
        }
        .navigationTitle("FAQs")
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
            tag: tag,
            lang: "en",
            allowNetworkFetch: true
        )
        isLoading = false
    }

    func refreshFAQs() async {
        await FAQProvider.shared.refreshFromNetwork()
        await loadFAQs()
    }
}
```

### Expandable FAQ Items

```swift
struct FAQItemView: View {
    let faq: FAQ
    @State private var isExpanded = false

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            // Question (always visible)
            HStack {
                Text(faq.question)
                    .font(.headline)
                    .foregroundColor(.primary)

                Spacer()

                Image(systemName: isExpanded ? "chevron.up" : "chevron.down")
                    .foregroundColor(.secondary)
            }
            .contentShape(Rectangle())
            .onTapGesture {
                withAnimation {
                    isExpanded.toggle()
                }
            }

            // Answer (expandable)
            if isExpanded {
                Text(faq.answer)
                    .font(.body)
                    .foregroundColor(.secondary)
                    .padding(.top, 4)
                    .transition(.opacity)
            }
        }
        .padding(.vertical, 8)
    }
}
```

### FAQ Screen with Sub-tag Filters

```swift
struct FAQScreen: View {
    @State private var faqs: [FAQ] = []
    @State private var selectedSubTags: Set<String> = []
    @State private var isLoading = false

    let currentTag = "payments"
    let availableSubTags = ["refunds", "disputes", "methods"]

    var body: some View {
        VStack(spacing: 0) {
            // Sub-tag filter chips
            ScrollView(.horizontal, showsIndicators: false) {
                HStack(spacing: 12) {
                    // "All" chip
                    FilterChip(
                        title: "All FAQs",
                        isSelected: selectedSubTags.isEmpty,
                        action: {
                            selectedSubTags.removeAll()
                        }
                    )

                    // Sub-tag chips
                    ForEach(availableSubTags, id: \.self) { subTag in
                        FilterChip(
                            title: subTag,
                            isSelected: selectedSubTags.contains(subTag),
                            action: {
                                toggleSubTag(subTag)
                            }
                        )
                    }
                }
                .padding()
            }

            Divider()

            // FAQ List
            if isLoading {
                ProgressView("Loading FAQs...")
            } else if faqs.isEmpty {
                EmptyStateView(message: "No FAQs found")
            } else {
                List(faqs, id: \.id) { faq in
                    FAQItemView(faq: faq)
                }
            }
        }
        .navigationTitle("Help Center")
        .task {
            await loadFAQs()
        }
        .onChange(of: selectedSubTags) { _ in
            Task {
                await loadFAQs()
            }
        }
    }

    func loadFAQs() async {
        isLoading = true
        faqs = await FAQProvider.shared.getFAQs(
            tag: currentTag,
            subTags: Array(selectedSubTags),
            lang: "en",
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

### Searchable FAQs

```swift
struct SearchableFAQView: View {
    @State private var allFAQs: [FAQ] = []
    @State private var searchText = ""

    var filteredFAQs: [FAQ] {
        if searchText.isEmpty {
            return allFAQs
        }
        return allFAQs.filter { faq in
            faq.question.localizedCaseInsensitiveContains(searchText) ||
            faq.answer.localizedCaseInsensitiveContains(searchText)
        }
    }

    var body: some View {
        List(filteredFAQs, id: \.id) { faq in
            FAQItemView(faq: faq)
        }
        .searchable(text: $searchText, prompt: "Search FAQs")
        .navigationTitle("Help Center")
        .task {
            allFAQs = await FAQProvider.shared.getFAQs(
                tag: "payments",
                lang: "en",
                allowNetworkFetch: true
            )
        }
    }
}
```

---

## UIKit Integration

### Basic FAQ Table View

```swift
import UIKit
import StringbootSDK

class FAQViewController: UIViewController {

    var faqs: [FAQ] = []
    let tableView = UITableView()
    let activityIndicator = UIActivityIndicatorView(style: .large)

    let currentTag = "payments"

    override func viewDidLoad() {
        super.viewDidLoad()

        title = "FAQs"
        setupUI()
        loadFAQs()
    }

    func setupUI() {
        view.addSubview(tableView)
        view.addSubview(activityIndicator)

        tableView.frame = view.bounds
        tableView.delegate = self
        tableView.dataSource = self
        tableView.register(FAQCell.self, forCellReuseIdentifier: "FAQCell")

        activityIndicator.center = view.center

        // Add refresh control
        let refreshControl = UIRefreshControl()
        refreshControl.addTarget(self, action: #selector(refreshFAQs), for: .valueChanged)
        tableView.refreshControl = refreshControl
    }

    func loadFAQs() {
        activityIndicator.startAnimating()

        Task {
            faqs = await FAQProvider.shared.getFAQs(
                tag: currentTag,
                lang: "en",
                allowNetworkFetch: true
            )

            await MainActor.run {
                activityIndicator.stopAnimating()
                tableView.reloadData()
            }
        }
    }

    @objc func refreshFAQs() {
        Task {
            await FAQProvider.shared.refreshFromNetwork()

            faqs = await FAQProvider.shared.getFAQs(
                tag: currentTag,
                lang: "en",
                allowNetworkFetch: true
            )

            await MainActor.run {
                tableView.refreshControl?.endRefreshing()
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

        let faq = faqs[indexPath.row]
        showFAQDetail(faq)
    }

    func showFAQDetail(_ faq: FAQ) {
        let detailVC = FAQDetailViewController(faq: faq)
        navigationController?.pushViewController(detailVC, animated: true)
    }
}
```

### Expandable FAQ Cell

```swift
class FAQCell: UITableViewCell {

    let questionLabel = UILabel()
    let answerLabel = UILabel()
    let chevronImageView = UIImageView()

    var isExpanded = false

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)

        questionLabel.font = .systemFont(ofSize: 16, weight: .semibold)
        questionLabel.numberOfLines = 0

        answerLabel.font = .systemFont(ofSize: 14)
        answerLabel.numberOfLines = 0
        answerLabel.textColor = .secondaryLabel
        answerLabel.isHidden = true

        chevronImageView.image = UIImage(systemName: "chevron.down")
        chevronImageView.tintColor = .secondaryLabel

        // Layout setup...
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    func configure(with faq: FAQ) {
        questionLabel.text = faq.question
        answerLabel.text = faq.answer
    }

    func toggleExpanded() {
        isExpanded.toggle()
        answerLabel.isHidden = !isExpanded

        UIView.animate(withDuration: 0.3) {
            self.chevronImageView.transform = self.isExpanded ?
                CGAffineTransform(rotationAngle: .pi) : .identity
        }
    }
}
```

---

## Tag-Based Organization

### Common Tag Hierarchies

#### Payments & Billing
```swift
let tag = "payments"
let subTags = [
    "refunds",           // Refund process
    "disputes",          // Dispute resolution
    "methods",           // Payment methods
    "failed",            // Failed payments
    "invoices"           // Invoice questions
]
```

#### Account Management
```swift
let tag = "account"
let subTags = [
    "profile",           // Profile management
    "security",          // Security & privacy
    "deletion",          // Account deletion
    "privacy",           // Privacy settings
    "notifications"      // Notification preferences
]
```

#### Identity Verification
```swift
let tag = "identity_verification"
let subTags = [
    "document_upload",   // Document upload issues
    "verification_failed", // Verification failures
    "liveness",          // Liveness check
    "requirements"       // ID requirements
]
```

#### Technical Support
```swift
let tag = "support"
let subTags = [
    "contact",           // How to contact support
    "escalation",        // Escalation process
    "feedback",          // Submit feedback
    "bug_report"         // Report bugs
]
```

---

## Language Support

### Automatic Language Fallback

```swift
// User's device is set to French
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

### Check Language Returned

```swift
let faqs = await FAQProvider.shared.getFAQs(tag: "payments", lang: "fr")

if let firstFAQ = faqs.first {
    let actualLanguage = firstFAQ.languageCode ?? "unknown"

    if actualLanguage == "en" && "fr" != "en" {
        // Show notice that FAQs are in English
        showLanguageFallbackNotice()
    }
}
```

### Use Device Language

```swift
let deviceLang = StringProvider.shared.deviceLocale()

let faqs = await FAQProvider.shared.getFAQs(
    tag: "payments",
    lang: deviceLang,
    allowNetworkFetch: true
)
```

---

## Advanced Usage

### Refresh FAQs from Network

```swift
let success = await FAQProvider.shared.refreshFromNetwork()

if success {
    print("FAQs refreshed successfully")
} else {
    print("Failed to refresh FAQs")
}
```

### Categorized FAQ Sections

```swift
func loadCategorizedFAQs() async {
    async let paymentFAQs = FAQProvider.shared.getFAQs(tag: "payments")
    async let accountFAQs = FAQProvider.shared.getFAQs(tag: "account")
    async let supportFAQs = FAQProvider.shared.getFAQs(tag: "support")

    let (payments, account, support) = await (paymentFAQs, accountFAQs, supportFAQs)

    displaySection("Payments", faqs: payments)
    displaySection("Account", faqs: account)
    displaySection("Support", faqs: support)
}
```

### Custom Filtering

```swift
func getFilteredFAQs(
    tag: String,
    subTags: [String],
    searchQuery: String,
    lang: String
) async -> [FAQ] {
    let faqs = await FAQProvider.shared.getFAQs(
        tag: tag,
        subTags: subTags,
        lang: lang,
        allowNetworkFetch: true
    )

    guard !searchQuery.isEmpty else { return faqs }

    return faqs.filter { faq in
        faq.question.localizedCaseInsensitiveContains(searchQuery) ||
        faq.answer.localizedCaseInsensitiveContains(searchQuery) ||
        faq.subTags.contains { $0.localizedCaseInsensitiveContains(searchQuery) }
    }
}
```

---

## Performance & Caching

### Cache Strategy

**Cache Layers:**

1. **Memory Cache (L1)** - NSCache for instant access (<5ms)
2. **Core Data (L2)** - Persistent offline storage (<50ms)
3. **Network (L3)** - Delta sync for changed FAQs only

### Preloading FAQs

```swift
@main
struct MyApp: App {
    init() {
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "YOUR_API_TOKEN",
            baseURL: "https://api.stringboot.com"
        )

        // Preload common FAQs
        Task {
            _ = await FAQProvider.shared.getFAQs(tag: "payments", allowNetworkFetch: true)
            _ = await FAQProvider.shared.getFAQs(tag: "account", allowNetworkFetch: true)
        }
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

---

## Best Practices

### 1. Use Async/Await for FAQ Loading

```swift
// ✅ Good: Async/await
Task {
    let faqs = await FAQProvider.shared.getFAQs(tag: "payments")
    updateUI(faqs)
}

// ❌ Avoid: Blocking main thread
```

### 2. Enable Network Fetch for User-Facing Screens

```swift
// ✅ Good: Allow network fallback
let faqs = await FAQProvider.shared.getFAQs(
    tag: "payments",
    allowNetworkFetch: true
)
```

### 3. Handle Empty States

```swift
let faqs = await FAQProvider.shared.getFAQs(tag: "payments")

if faqs.isEmpty {
    showEmptyState()
} else {
    displayFAQs(faqs)
}
```

### 4. Refresh on App Launch

```swift
init() {
    StringProvider.shared.initialize(/*...*/)

    Task {
        await FAQProvider.shared.refreshFromNetwork()
    }
}
```

---

## API Reference

### FAQProvider Methods

#### `getFAQs()`

```swift
public func getFAQs(
    tag: String,
    subTags: [String] = [],
    lang: String? = nil,
    allowNetworkFetch: Bool = false
) async -> [FAQ]
```

#### `refreshFromNetwork()`

```swift
public func refreshFromNetwork() async -> Bool
```

#### `isInitialized`

```swift
public var isInitialized: Bool { get }
```

---

## Examples

### Complete Example

See [FAQDemoView.swift](../stringboot-ios-demoApp/Stringboot%20iOS%20Demo/FAQDemoView.swift) in the demo app for a full working implementation.

### Minimal Example

```swift
struct SimpleFAQView: View {
    @State private var faqs: [FAQ] = []

    var body: some View {
        List(faqs, id: \.id) { faq in
            VStack(alignment: .leading) {
                Text(faq.question).font(.headline)
                Text(faq.answer).font(.body)
            }
        }
        .task {
            faqs = await FAQProvider.shared.getFAQs(
                tag: "payments",
                lang: "en",
                allowNetworkFetch: true
            )
        }
    }
}
```

---

## Troubleshooting

### FAQs Not Appearing

```swift
// Check initialization
if !FAQProvider.shared.isInitialized {
    print("FAQProvider not initialized!")
}

// Enable logging
StringbootLogger.setLogLevel(.debug)

// Check network sync
let success = await FAQProvider.shared.refreshFromNetwork()
print("Network sync: \(success)")
```

### Language Fallback Not Working

```swift
let faqs = await FAQProvider.shared.getFAQs(tag: "payments", lang: "fr")
if let actualLang = faqs.first?.languageCode {
    print("Requested: fr, Got: \(actualLang)")
}
```

---

## Related Documentation

- [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md#-faq-provider-integration) - Complete integration guide
- [API_REFERENCE.md](API_REFERENCE.md) - Full API documentation
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Common issues & solutions
- [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md) - Advanced caching & performance

---

**Version:** 1.1.0+ | **Last Updated:** December 2024
