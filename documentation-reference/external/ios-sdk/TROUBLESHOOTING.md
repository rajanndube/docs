# Stringboot iOS SDK - Troubleshooting Guide

## ❌ No Strings Being Fetched from Backend

### Problem
Logs show "String not found" even though the SDK is initialized with an API token.

### ✅ **FIXED - API Endpoint Mismatch**
**Issue:** Earlier versions used incorrect API v2 endpoints.

**Solution:** The iOS SDK now uses the correct API v1 endpoints that match the Android SDK:
- ✅ Catalog: `/api/v1/client/strings/catalog?languageCode=en`
- ✅ Languages: `/api/v1/client/languages?onlyActive=true`
- ✅ Health: `/api/v1/client/health`

**Backend logs should now show:**
```
INFO  c.s.a.c.c.ClientStringController - Client API: Getting catalog for language: en
```

Instead of:
```
DEBUG o.s.w.s.r.ResourceHttpRequestHandler - Resource not found  // ❌ Wrong endpoint
```

### Common Causes & Solutions

#### 1. **iOS Simulator Network Configuration**

**Issue:** iOS Simulator uses different localhost addresses than Android Emulator.

- ❌ **Android Emulator:** Uses `http://10.0.2.2:8000` to access host machine
- ✅ **iOS Simulator:** Uses `http://localhost:8000` or `http://127.0.0.1:8000`

**Fix:**
```swift
// For iOS Simulator
StringProvider.shared.initialize(
    cacheSize: 1000,
    apiToken: "your-api-token",
    baseURL: "http://localhost:8000"  // ✅ Use localhost, not 10.0.2.2
)

// For physical iOS device on same network
StringProvider.shared.initialize(
    cacheSize: 1000,
    apiToken: "your-api-token",
    baseURL: "http://192.168.1.100:8000"  // Use your Mac's IP address
)
```

#### 2. **Auto-Sync Not Enabled**

**Issue:** The SDK doesn't automatically fetch strings on startup.

**Fix:**
```swift
StringProvider.shared.initialize(
    cacheSize: 1000,
    apiToken: "your-api-token",
    baseURL: "http://localhost:8000",
    autoSync: true  // ✅ Enable auto-sync
)
```

#### 3. **Network Fetch Not Allowed**

**Issue:** The `get()` method has `allowNetworkFetch` parameter set to `false` by default.

**Fix Option 1 - Use auto-sync (recommended):**
```swift
// Initialize with autoSync: true (done above)
// Then just use get() normally:
let text = await StringProvider.shared.get("key", lang: "en")
```

**Fix Option 2 - Manually enable network fetch:**
```swift
let text = await StringProvider.shared.get(
    "key",
    lang: "en",
    allowNetworkFetch: true  // Enable network fallback
)
```

**Fix Option 3 - Manual sync before using strings:**
```swift
// In your app startup
Task {
    await StringProvider.shared.refreshFromNetwork(lang: "en")
}
```

#### 4. **App Transport Security (ATS) Blocking HTTP**

**Issue:** iOS blocks non-HTTPS connections by default.

**Fix:** Add to your `Info.plist`:
```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsLocalNetworking</key>
    <true/>
    <!-- OR for specific domains -->
    <key>NSExceptionDomains</key>
    <dict>
        <key>localhost</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
        </dict>
    </dict>
</dict>
```

#### 5. **Backend Not Running or Unreachable**

**Check:**
```bash
# Test backend from terminal
curl -H "X-API-TOKEN: your-token" http://localhost:8000/api/v2/client/strings/en/catalog

# Check if backend is running
lsof -i :8000
```

## 📊 Debugging Steps

### 1. Enable Debug Logging

```swift
StringbootLogger.logLevel = .debug  // or .verbose for more details
```

### 2. Check Network Requests

Look for these logs:
```
[StringbootSDK] API Request: GET http://localhost:8000/api/v2/client/strings/en/catalog
[StringbootSDK] API request successful: /api/v2/client/strings/en/catalog
```

### 3. Verify Initialization

```swift
// Check if initialized
if StringProvider.shared.isInitialized {
    print("✅ Provider initialized")
}

// Check if network is configured
if StringProvider.shared.hasNetworkCapabilities() {
    print("✅ Network configured")
}
```

### 4. Test Manual Sync

```swift
Task {
    let success = await StringProvider.shared.refreshFromNetwork(lang: "en")
    print("Sync result: \(success)")

    // Check database
    let count = StringProvider.shared.getStringCount(lang: "en")
    print("Strings in DB: \(count)")
}
```

### 5. Check Cache Stats

```swift
let stats = StringProvider.shared.getCacheStats()
print("Cache size: \(stats.memorySize)")
print("Hit rate: \(stats.hitRate)")
print("Hits: \(stats.memoryHitCount)")
print("Misses: \(stats.memoryMissCount)")
```

## 🔧 Complete Working Example

```swift
import SwiftUI
import StringbootSDK

@main
struct MyApp: App {

    @StateObject private var syncMonitor = SyncMonitor()

    init() {
        // Configure for iOS Simulator
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "9bc7d015-c9e6-42ff-aec2-8c33b13284c0",
            baseURL: "http://localhost:8000",  // ✅ Correct for iOS Simulator
            autoSync: true
        )

        StringbootLogger.logLevel = .debug
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(syncMonitor)
        }
    }
}

// Monitor sync status
@MainActor
class SyncMonitor: ObservableObject {
    @Published var isSyncing = false
    @Published var syncComplete = false

    init() {
        // Monitor sync completion
        Task {
            isSyncing = true
            try? await Task.sleep(nanoseconds: 2_000_000_000) // Wait for sync
            isSyncing = false
            syncComplete = true
        }
    }
}

// In your view
struct ContentView: View {
    @EnvironmentObject var syncMonitor: SyncMonitor
    @State private var text = ""

    var body: some View {
        VStack {
            if syncMonitor.isSyncing {
                ProgressView("Syncing strings...")
            } else {
                Text(text)
                    .onAppear {
                        Task {
                            text = await StringProvider.shared.get("welcome_message", lang: "en")
                        }
                    }
            }
        }
    }
}
```

## 🌐 Network Configuration Cheat Sheet

| Environment | Base URL | Notes |
|-------------|----------|-------|
| **iOS Simulator** | `http://localhost:8000` | Direct localhost access |
| **Android Emulator** | `http://10.0.2.2:8000` | Special Android alias |
| **Physical Device (same network)** | `http://192.168.1.x:8000` | Use Mac's local IP |
| **Production** | `https://api.stringboot.com` | HTTPS in production |

## 📱 Finding Your Mac's IP Address

```bash
# macOS
ipconfig getifaddr en0  # WiFi
ipconfig getifaddr en1  # Ethernet

# Or use System Settings > Network
```

## ⚠️ Common Errors

### "Connection refused"
- Backend not running
- Wrong port number
- Firewall blocking connection

### "SSL/TLS error"
- Using HTTPS with self-signed cert
- Add ATS exception (see above)

### "Invalid response"
- API returning wrong format
- Check backend logs
- Verify API endpoints

### "No strings found - ??key??"
- Strings not synced yet (wait for autoSync)
- Wrong language code
- Key doesn't exist in backend

## 📝 Best Practices

1. **Always use auto-sync in production:**
   ```swift
   StringProvider.shared.initialize(autoSync: true)
   ```

2. **Handle sync delays in UI:**
   ```swift
   .onAppear {
       Task {
           try? await Task.sleep(nanoseconds: 500_000_000)
           await loadStrings()
       }
   }
   ```

3. **Test with different network conditions:**
   - Airplane mode (offline)
   - Slow network
   - Backend down

4. **Use proper error handling:**
   ```swift
   let text = await StringProvider.shared.get("key", lang: "en")
   if text.hasPrefix("??") && text.hasSuffix("??") {
       // String not found, show fallback
   }
   ```

## 🔍 Still Having Issues?

1. Check the logs with `.debug` level
2. Verify backend is accessible: `curl http://localhost:8000/api/v2/health`
3. Try manual sync: `await StringProvider.shared.refreshFromNetwork(lang: "en")`
4. Check database: `StringProvider.shared.getStringCount(lang: "en")`
5. Review this guide's network configuration section

## ❓ FAQ Provider Issues

### Problem: FAQProvider Not Initialized

**Error:**
```
Fatal error: FAQProvider not initialized. Call initialize() before using.
```

**Cause:** FAQProvider requires separate initialization from StringProvider.

**Solution:**
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

### Problem: FAQs Not Appearing

**Symptoms:**
- `getFAQs()` returns empty array
- No FAQs displayed in UI
- No error messages

**Solutions:**

#### 1. **Verify Backend Has FAQs**

```bash
# Test FAQ endpoint
curl -H "X-API-TOKEN: your-token" \
  "https://api.stringboot.com/api/v1/client/faqs?lang=en"
```

Expected response:
```json
{
  "faqs": [
    {
      "id": "faq-1",
      "question": "How do I get started?",
      "answer": "To get started...",
      "tags": ["getting-started"],
      "language": "en"
    }
  ]
}
```

#### 2. **Enable Auto-Sync for FAQ Provider**

```swift
// Initialize with auto-sync
FAQProvider.shared.initialize(
    cacheSize: 200,
    apiToken: "YOUR_API_TOKEN",
    baseURL: "https://api.stringboot.com",
    autoSync: true  // Enable auto-sync
)
```

#### 3. **Manual Sync Before Fetching**

```swift
// Sync FAQs first
Task {
    await FAQProvider.shared.syncFAQs(lang: "en")

    // Then fetch
    let faqs = await FAQProvider.shared.getFAQs(tag: "payments", lang: "en")
    print("Found \(faqs.count) FAQs")
}
```

#### 4. **Check Tag Spelling**

```swift
// ❌ Wrong - typo in tag
let faqs = await FAQProvider.shared.getFAQs(tag: "paymnt", lang: "en")

// ✅ Correct
let faqs = await FAQProvider.shared.getFAQs(tag: "payments", lang: "en")
```

#### 5. **Verify Language Code**

```swift
// ❌ Wrong - language might not exist
let faqs = await FAQProvider.shared.getFAQs(tag: "payments", lang: "xx")

// ✅ Correct - use valid language
let faqs = await FAQProvider.shared.getFAQs(tag: "payments", lang: "en")
```

### Problem: FAQ Publisher Not Updating

**Symptoms:**
- SwiftUI view not refreshing when FAQs change
- Combine publisher not emitting updates

**Solution - Use Combine Publisher:**

```swift
import SwiftUI
import Combine

struct FAQListView: View {
    @State private var faqs: [FAQ] = []
    @State private var cancellables = Set<AnyCancellable>()

    var body: some View {
        List(faqs) { faq in
            VStack(alignment: .leading) {
                Text(faq.question)
                    .font(.headline)
                Text(faq.answer)
                    .font(.body)
            }
        }
        .onAppear {
            // Subscribe to FAQ updates
            FAQProvider.shared.getFAQsPublisher(tag: "payments", lang: "en")
                .sink { updatedFAQs in
                    faqs = updatedFAQs
                }
                .store(in: &cancellables)
        }
    }
}
```

### Problem: Sub-Tags Not Filtering Correctly

**Issue:** Getting all FAQs instead of filtered by sub-tag.

**Solution:**

```swift
// ✅ Filter by primary tag only
let allPaymentFAQs = await FAQProvider.shared.getFAQs(
    tag: "payments",
    lang: "en"
)

// ✅ Filter by sub-tag in your code
let refundFAQs = allPaymentFAQs.filter { faq in
    faq.subTags?.contains("refunds") ?? false
}
```

### Problem: Performance Issues with Large FAQ Lists

**Symptoms:**
- Slow FAQ loading
- UI lag when displaying many FAQs
- High memory usage

**Solutions:**

#### 1. **Enable Caching**

```swift
// Increase cache size for FAQs
FAQProvider.shared.initialize(
    cacheSize: 500,  // Larger cache for more FAQs
    apiToken: "YOUR_API_TOKEN",
    baseURL: "https://api.stringboot.com"
)
```

#### 2. **Use Lazy Loading in SwiftUI**

```swift
struct FAQListView: View {
    @State private var faqs: [FAQ] = []

    var body: some View {
        ScrollView {
            LazyVStack(spacing: 16) {  // Lazy loading
                ForEach(faqs) { faq in
                    FAQItemView(faq: faq)
                }
            }
        }
    }
}
```

#### 3. **Implement Pagination**

```swift
class FAQViewModel: ObservableObject {
    @Published var faqs: [FAQ] = []
    private var allFAQs: [FAQ] = []
    private var currentPage = 0
    private let pageSize = 20

    func loadMore() async {
        if allFAQs.isEmpty {
            allFAQs = await FAQProvider.shared.getFAQs(tag: "payments", lang: "en")
        }

        let start = currentPage * pageSize
        let end = min(start + pageSize, allFAQs.count)

        if start < allFAQs.count {
            faqs.append(contentsOf: allFAQs[start..<end])
            currentPage += 1
        }
    }
}
```

### Problem: FAQ Cache Not Clearing

**Issue:** Old FAQs persist after updates.

**Solution:**

```swift
// Clear FAQ cache
FAQProvider.shared.clearCache()

// Re-sync from network
await FAQProvider.shared.syncFAQs(lang: "en")

// Or force refresh
let faqs = await FAQProvider.shared.getFAQs(
    tag: "payments",
    lang: "en",
    forceRefresh: true
)
```

### Debug FAQ Provider

```swift
// Enable debug logging
StringbootLogger.logLevel = .debug

// Check initialization status
if FAQProvider.shared.isInitialized {
    print("✅ FAQ Provider initialized")
}

// Check cache stats
let stats = FAQProvider.shared.getCacheStats()
print("FAQ Cache - Size: \(stats.size), Hits: \(stats.hits), Misses: \(stats.misses)")

// Test manual sync
Task {
    let success = await FAQProvider.shared.syncFAQs(lang: "en")
    print("FAQ sync result: \(success)")

    // Check database
    let count = FAQProvider.shared.getFAQCount(lang: "en")
    print("FAQs in database: \(count)")
}
```

## 🧪 A/B Testing Issues

### Problem: Experiment Not Assigned

**Symptoms:**
- `getVariant()` returns `nil`
- User not assigned to any experiment
- Control group always returned

**Solutions:**

#### 1. **Verify Experiment Exists in Backend**

```bash
# Check experiment endpoint
curl -H "X-API-TOKEN: your-token" \
  "https://api.stringboot.com/api/v1/client/experiments/button_color"
```

Expected response:
```json
{
  "experimentId": "button_color",
  "name": "Button Color Test",
  "variants": [
    {"id": "control", "weight": 50},
    {"id": "blue", "weight": 50}
  ],
  "status": "active"
}
```

#### 2. **Check Experiment Name Spelling**

```swift
// ❌ Wrong - typo
let variant = ExperimentManager.shared.getVariant(experimentId: "buton_color")

// ✅ Correct
let variant = ExperimentManager.shared.getVariant(experimentId: "button_color")
```

#### 3. **Verify User ID is Set**

```swift
// Must set user ID before getting variant
ExperimentManager.shared.setUserId("user-12345")

// Then get variant
let variant = ExperimentManager.shared.getVariant(experimentId: "button_color")
```

#### 4. **Check Experiment is Active**

```swift
// Verify experiment status
let experiment = await ExperimentManager.shared.getExperiment(id: "button_color")
if experiment?.status == .active {
    print("✅ Experiment is active")
} else {
    print("❌ Experiment is not active")
}
```

### Problem: Same Variant Every Time

**Symptoms:**
- User always gets same variant (e.g., always "control")
- Assignment doesn't seem random
- Weight distribution not working

**Cause:** Consistent hashing ensures same user always gets same variant (this is by design).

**To Test Different Variants:**

```swift
// Change user ID to see different variants
ExperimentManager.shared.setUserId("user-12345")  // Might get control
let variant1 = ExperimentManager.shared.getVariant(experimentId: "button_color")

ExperimentManager.shared.setUserId("user-67890")  // Might get blue
let variant2 = ExperimentManager.shared.getVariant(experimentId: "button_color")

// Or clear assignment cache
ExperimentManager.shared.clearAssignments()
```

### Problem: Variant Not Applying to UI

**Symptoms:**
- `getVariant()` returns correct variant
- UI not updating to show variant
- No visual change

**Solutions:**

#### 1. **SwiftUI - Use @State Properly**

```swift
struct ContentView: View {
    @State private var buttonColor: Color = .gray

    var body: some View {
        Button("Click Me") {
            // Action
        }
        .background(buttonColor)
        .onAppear {
            // Get variant on appear
            let variant = ExperimentManager.shared.getVariant(experimentId: "button_color")

            switch variant {
            case "blue":
                buttonColor = .blue
            case "green":
                buttonColor = .green
            default:
                buttonColor = .red  // control
            }
        }
    }
}
```

#### 2. **UIKit - Update UI on Main Thread**

```swift
class ViewController: UIViewController {
    @IBOutlet weak var actionButton: UIButton!

    override func viewDidLoad() {
        super.viewDidLoad()

        // Get variant
        let variant = ExperimentManager.shared.getVariant(experimentId: "button_color")

        // Update UI on main thread
        DispatchQueue.main.async {
            switch variant {
            case "blue":
                self.actionButton.backgroundColor = .systemBlue
            case "green":
                self.actionButton.backgroundColor = .systemGreen
            default:
                self.actionButton.backgroundColor = .systemRed
            }
        }
    }
}
```

### Problem: Analytics Not Firing

**Symptoms:**
- Experiment assignments not tracked
- No data in analytics dashboard
- Events not sent to backend

**Solutions:**

#### 1. **Enable Analytics Logging**

```swift
// Enable analytics
ExperimentManager.shared.enableAnalytics(true)

// Verify analytics events
StringbootLogger.logLevel = .debug
// Look for: [ExperimentManager] Analytics event: experiment_assigned
```

#### 2. **Manually Track Assignment**

```swift
let variant = ExperimentManager.shared.getVariant(experimentId: "button_color")

// Track assignment manually
ExperimentManager.shared.trackAssignment(
    experimentId: "button_color",
    variant: variant ?? "control",
    userId: "user-12345"
)
```

#### 3. **Integrate with Your Analytics Provider**

```swift
// Firebase Analytics integration
ExperimentManager.shared.onExperimentAssigned = { experimentId, variant in
    Analytics.logEvent("experiment_assigned", parameters: [
        "experiment_id": experimentId,
        "variant": variant
    ])
}

// Amplitude integration
ExperimentManager.shared.onExperimentAssigned = { experimentId, variant in
    Amplitude.instance().logEvent("Experiment Assigned", withEventProperties: [
        "experiment_id": experimentId,
        "variant": variant
    ])
}
```

### Problem: Experiments Not Syncing

**Symptoms:**
- Local experiments out of date
- New experiments not appearing
- Variant weights not updating

**Solution:**

```swift
// Force sync experiments
Task {
    await ExperimentManager.shared.syncExperiments()

    // Verify sync
    let experiments = ExperimentManager.shared.getAllExperiments()
    print("Synced \(experiments.count) experiments")
}

// Or enable auto-sync
ExperimentManager.shared.initialize(
    apiToken: "YOUR_API_TOKEN",
    baseURL: "https://api.stringboot.com",
    autoSync: true  // Auto-sync experiments
)
```

### Problem: Memory Leaks with Experiment Observers

**Symptoms:**
- Memory usage grows over time
- View controllers not deallocating
- Combine subscriptions leaking

**Solution:**

```swift
class ViewController: UIViewController {
    private var cancellables = Set<AnyCancellable>()  // Store cancellables

    override func viewDidLoad() {
        super.viewDidLoad()

        // Observe variant changes
        ExperimentManager.shared.variantPublisher(experimentId: "button_color")
            .sink { [weak self] variant in  // Use weak self
                self?.updateUI(for: variant)
            }
            .store(in: &cancellables)  // Store to cancel later
    }

    deinit {
        cancellables.removeAll()  // Clean up
    }
}
```

### Problem: Variant Override Not Working

**Symptoms:**
- Force-setting variant doesn't work
- Override not persisting
- Still getting random assignment

**Solution:**

```swift
// Force specific variant for testing
ExperimentManager.shared.forceVariant(
    experimentId: "button_color",
    variant: "blue"
)

// Verify override
let variant = ExperimentManager.shared.getVariant(experimentId: "button_color")
assert(variant == "blue", "Variant should be forced to blue")

// Clear override
ExperimentManager.shared.clearVariantOverride(experimentId: "button_color")
```

### Debug A/B Testing

```swift
// Enable debug logging
StringbootLogger.logLevel = .debug

// Check all experiments
let experiments = ExperimentManager.shared.getAllExperiments()
print("Available experiments: \(experiments.count)")
experiments.forEach { exp in
    print("  - \(exp.id): \(exp.variants.count) variants")
}

// Check user assignment
let userId = ExperimentManager.shared.getUserId()
print("User ID: \(userId ?? "not set")")

// Test variant assignment
let variant = ExperimentManager.shared.getVariant(experimentId: "button_color")
print("Assigned variant: \(variant ?? "none")")

// Check experiment status
Task {
    let experiment = await ExperimentManager.shared.getExperiment(id: "button_color")
    print("Status: \(experiment?.status.rawValue ?? "unknown")")
    print("Variants: \(experiment?.variants.map { $0.id }.joined(separator: ", ") ?? "none")")
}
```

### Complete A/B Testing Example

```swift
import SwiftUI

@main
struct MyApp: App {
    init() {
        // Initialize SDK
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "YOUR_API_TOKEN",
            baseURL: "https://api.stringboot.com"
        )

        // Initialize Experiment Manager
        ExperimentManager.shared.initialize(
            apiToken: "YOUR_API_TOKEN",
            baseURL: "https://api.stringboot.com",
            autoSync: true
        )

        // Set user ID
        ExperimentManager.shared.setUserId("user-12345")

        // Enable analytics
        ExperimentManager.shared.enableAnalytics(true)

        // Analytics callback
        ExperimentManager.shared.onExperimentAssigned = { experimentId, variant in
            print("📊 Assigned to \(experimentId): \(variant)")
        }
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

struct ContentView: View {
    @State private var buttonColor: Color = .gray
    @State private var buttonText: String = "Click Me"

    var body: some View {
        VStack(spacing: 20) {
            // Test button color variant
            Button(buttonText) {
                print("Button clicked")
            }
            .padding()
            .background(buttonColor)
            .foregroundColor(.white)
            .cornerRadius(8)

            // Debug info
            Text("Variant: \(ExperimentManager.shared.getVariant(experimentId: "button_color") ?? "none")")
                .font(.caption)
        }
        .onAppear {
            setupExperiment()
        }
    }

    func setupExperiment() {
        // Get variant
        let variant = ExperimentManager.shared.getVariant(experimentId: "button_color")

        // Apply variant
        switch variant {
        case "blue":
            buttonColor = .blue
            buttonText = "Try Blue"
        case "green":
            buttonColor = .green
            buttonText = "Go Green"
        default:
            buttonColor = .red
            buttonText = "Click Me"
        }
    }
}
```

## 📞 Support

- GitHub Issues: https://github.com/your-org/stringboot-ios-sdk/issues
- Documentation: [README.md](README.md)
- Setup Guide: [SETUP.md](SETUP.md)
- FAQ Provider Guide: [FAQ_PROVIDER.md](FAQ_PROVIDER.md)
- A/B Testing Guide: [AB_TESTING.md](AB_TESTING.md)
