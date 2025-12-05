# A/B Testing with Stringboot iOS SDK

Stringboot enables **server-side A/B testing for strings** without app releases. Run experiments on copy, messaging, and UI text to optimize user engagement and conversions.

## Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [Quick Start](#quick-start)
- [Device ID Management](#device-id-management)
- [Analytics Integration](#analytics-integration)
- [Experiment Assignment](#experiment-assignment)
- [Testing Your Integration](#testing-your-integration)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Examples](#examples)

## Overview

### What is A/B Testing with Stringboot?

Stringboot's A/B testing allows you to:
- ✅ Test different string variations without code changes or app submissions
- ✅ Run experiments on headlines, CTAs, error messages, onboarding copy
- ✅ Automatically assign users to consistent experiment variants
- ✅ Track conversions and analytics for each variant
- ✅ Make data-driven decisions on string effectiveness

### Key Benefits

- **No App Review Required** - Create and deploy experiments instantly
- **Consistent Assignment** - Each device always sees the same variant
- **Automatic Tracking** - Optional analytics integration for conversion tracking
- **Privacy-Friendly** - Uses anonymous device IDs (not personal data)
- **Backend-Controlled** - Experiments managed from Stringboot dashboard

## How It Works

### Architecture

```
┌─────────────────┐
│  Stringboot     │
│  Dashboard      │ ← Create experiment: "welcome_message"
│                 │   - Control: "Welcome!"
│                 │   - Variant A: "Hey there!"
│                 │   - Variant B: "Welcome back!"
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│   Backend       │ ← Assigns devices to variants
│   (Assignment   │   based on device ID hash
│    Engine)      │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│   iOS SDK       │ ← Requests strings with device ID
│  (Your App)     │ ← Receives assigned variant
│                 │ ← Displays string to user
└─────────────────┘
```

### The Flow

1. **Backend receives request** with `X-Device-ID` header
2. **Checks for active experiments** for the requested string key
3. **Assigns variant** based on device ID hash (if first time)
4. **Stores assignment** in Core Data for consistency
5. **Returns variant string** to SDK
6. **SDK caches assignment** for offline use

### Consistency Guarantee

Once a device is assigned to a variant, it **always** receives the same variant, even:
- ✅ Across app restarts
- ✅ After cache clears
- ✅ On different devices with same device ID
- ✅ Until experiment ends

## Quick Start

### Step 1: Enable A/B Testing (30 seconds)

The SDK automatically supports A/B testing. You just need to provide a device ID:

```swift
import StringbootSDK

// Option A: Auto-generated device ID (recommended)
StringProvider.shared.initialize(
    cacheSize: 1000,
    apiToken: "YOUR_API_TOKEN",
    baseURL: "https://api.stringboot.com",
    autoSync: true
)
// Device ID automatically generated and persisted

// Option B: Custom device ID
StringProvider.shared.initialize(
    cacheSize: 1000,
    apiToken: "YOUR_API_TOKEN",
    baseURL: "https://api.stringboot.com",
    autoSync: true,
    providedDeviceId: "your-custom-device-id" // e.g., from Firebase
)
```

**That's it!** Your app now supports A/B testing. The SDK automatically:
- ✅ Generates and persists a unique device ID (UUID)
- ✅ Sends device ID with all string requests
- ✅ Receives and caches experiment assignments
- ✅ Returns assigned variants for experiment strings

### Step 2: Use Strings Normally

No code changes needed! Use strings exactly as before:

```swift
// SwiftUI
struct ContentView: View {
    var body: some View {
        VStack {
            // This string might be part of an A/B test
            // SDK automatically returns the assigned variant
            SBText("welcome_message")
                .font(.title)

            SBText("cta_button")
                .font(.headline)
        }
    }
}

// UIKit
class ViewController: UIViewController {
    let titleLabel = SBLabel(key: "welcome_message")

    override func viewDidLoad() {
        super.viewDidLoad()
        // Returns "Welcome!" or "Hey there!" depending on assignment
    }
}

// Manual (async/await)
let welcomeText = await StringProvider.shared.get("welcome_message")
// Returns assigned variant automatically
```

### Step 3: Create Experiment on Dashboard

1. Go to Stringboot Dashboard → Experiments
2. Click **Create Experiment**
3. Select string key (e.g., `welcome_message`)
4. Add variants:
   - Control: "Welcome!"
   - Variant A: "Hey there!"
   - Variant B: "Welcome back!"
5. Set traffic allocation (e.g., 33% / 33% / 34%)
6. Click **Start Experiment**

**Users automatically assigned** to variants based on their device ID!

## Device ID Management

### Auto-Generated Device ID

By default, the SDK generates a UUID and persists it in UserDefaults:

```swift
// Automatic - SDK handles this internally
StringProvider.shared.initialize(
    cacheSize: 1000,
    apiToken: "YOUR_API_TOKEN",
    baseURL: "https://api.stringboot.com"
)

// Device ID is generated on first launch and stored in:
// UserDefaults: "com.stringboot.deviceId"
```

### Custom Device ID

Provide your own device ID for cross-platform consistency or integration with other systems:

```swift
import FirebaseInstallations

// Use Firebase Installation ID
Installations.installations().installationID { (id, error) in
    guard let firebaseID = id else { return }

    StringProvider.shared.initialize(
        cacheSize: 1000,
        apiToken: "YOUR_API_TOKEN",
        baseURL: "https://api.stringboot.com",
        providedDeviceId: firebaseID
    )
}

// Or using async/await (iOS 15+)
Task {
    let firebaseID = try await Installations.installations().installationID()

    StringProvider.shared.initialize(
        cacheSize: 1000,
        apiToken: "YOUR_API_TOKEN",
        baseURL: "https://api.stringboot.com",
        providedDeviceId: firebaseID
    )
}
```

#### Common Custom ID Sources

| Source | Use Case | Example |
|--------|----------|---------|
| **Firebase IID** | Cross-platform consistency | `Installations.installations().installationID()` |
| **IDFV** | Vendor-specific tracking | `UIDevice.current.identifierForVendor?.uuidString` |
| **User ID** | Logged-in users only | `Auth.auth().currentUser?.uid` |
| **Custom UUID** | Full control | `UUID().uuidString` |

⚠️ **Privacy Note:** Never use personally identifiable information (email, phone, IDFA without consent) as device ID. Always comply with App Store privacy requirements.

### Retrieve Current Device ID

```swift
// The SDK doesn't expose device ID publicly for privacy reasons
// But you can access it from UserDefaults if needed:

let deviceId = UserDefaults.standard.string(forKey: "com.stringboot.deviceId")
print("Current device ID: \(deviceId ?? "not set")")
```

## Analytics Integration

### Overview

Track experiment assignments and conversions by implementing a custom analytics handler.

### Implement Analytics Handler

Create a class conforming to `StringbootAnalyticsHandler`:

```swift
import StringbootSDK
import FirebaseAnalytics

class MyAnalyticsHandler: StringbootAnalyticsHandler {

    func onExperimentAssigned(_ assignment: ExperimentAssignment) {
        // Called when device is assigned to an experiment variant

        // Send to Firebase Analytics
        Analytics.logEvent("experiment_assigned", parameters: [
            "experiment_id": assignment.experimentId,
            "experiment_key": assignment.experimentKey,
            "string_key": assignment.stringKey,
            "variant_name": assignment.variantName,
            "language": assignment.languageCode
        ])

        // Or send to Mixpanel
        Mixpanel.mainInstance().track(event: "Experiment Assigned", properties: [
            "experiment_id": assignment.experimentId,
            "experiment_key": assignment.experimentKey,
            "variant_name": assignment.variantName
        ])

        // Or send to Amplitude
        Amplitude.instance().logEvent("Experiment Assigned", withEventProperties: [
            "experiment_key": assignment.experimentKey,
            "variant": assignment.variantName
        ])
    }

    func onExperimentViewed(_ experimentKey: String, variantName: String) {
        // Called when string is actually displayed to user
        Analytics.logEvent("experiment_viewed", parameters: [
            "experiment_key": experimentKey,
            "variant_name": variantName
        ])
    }
}
```

### Register Analytics Handler

```swift
// In App.swift (SwiftUI)
@main
struct MyApp: App {
    init() {
        let analyticsHandler = MyAnalyticsHandler()

        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: "YOUR_API_TOKEN",
            baseURL: "https://api.stringboot.com",
            analyticsHandler: analyticsHandler
        )
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

// In AppDelegate.swift (UIKit)
func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

    let analyticsHandler = MyAnalyticsHandler()

    StringProvider.shared.initialize(
        cacheSize: 1000,
        apiToken: "YOUR_API_TOKEN",
        baseURL: "https://api.stringboot.com",
        analyticsHandler: analyticsHandler
    )

    return true
}
```

### Track Conversions

Track user actions to measure experiment effectiveness:

```swift
// SwiftUI
struct ContentView: View {
    var body: some View {
        VStack {
            SBText("welcome_message")
                .font(.title)

            Button(action: {
                trackConversion(action: "button_clicked", experimentKey: "welcome_experiment")
            }) {
                SBText("cta_button")
            }
        }
    }

    func trackConversion(action: String, experimentKey: String) {
        // Get current device ID
        let deviceId = UserDefaults.standard.string(forKey: "com.stringboot.deviceId")

        // Log conversion event
        Analytics.logEvent("experiment_conversion", parameters: [
            "experiment_key": experimentKey,
            "action": action,
            "device_id": deviceId ?? "unknown"
        ])

        print("Conversion tracked: \(action) for \(experimentKey)")
    }
}

// UIKit
class ViewController: UIViewController {

    let ctaButton = SBButton(key: "cta_button")

    override func viewDidLoad() {
        super.viewDidLoad()

        ctaButton.addTarget(self, action: #selector(ctaTapped), for: .touchUpInside)
    }

    @objc func ctaTapped() {
        trackConversion(action: "button_clicked", experimentKey: "welcome_experiment")
    }

    func trackConversion(action: String, experimentKey: String) {
        let deviceId = UserDefaults.standard.string(forKey: "com.stringboot.deviceId")

        Analytics.logEvent("experiment_conversion", parameters: [
            "experiment_key": experimentKey,
            "action": action,
            "device_id": deviceId ?? "unknown"
        ])
    }
}
```

## Experiment Assignment

### How Assignment Works

1. **Hash-Based Distribution**: Device ID is hashed to determine variant
2. **Deterministic**: Same device ID always gets same variant
3. **Weighted Random**: Traffic split (e.g., 50/50, 33/33/34) honored
4. **Sticky**: Assignment persisted across app sessions

### Example Assignment Flow

```swift
// User 1 (device ID: "abc123")
let text1 = await StringProvider.shared.get("welcome_message", lang: "en")
// → Backend hashes "abc123" → assigns to Variant A
// → Returns "Hey there!"
// → Assignment cached in Core Data

// User 2 (device ID: "xyz789")
let text2 = await StringProvider.shared.get("welcome_message", lang: "en")
// → Backend hashes "xyz789" → assigns to Control
// → Returns "Welcome!"

// User 1 again (same device, different session)
let text3 = await StringProvider.shared.get("welcome_message", lang: "en")
// → Backend retrieves cached assignment
// → Returns "Hey there!" (consistent!)
```

### Experiment Metadata

The backend may include experiment metadata in responses:

```swift
// SDK handles this internally, but for reference:
// Response includes:
{
  "strings": {
    "en": {
      "welcome_message": "Hey there!"  // ← Assigned variant
    }
  },
  "experiments": {  // ← Optional metadata
    "welcome_message": {
      "experimentId": "exp-123",
      "experimentKey": "welcome_test",
      "variantName": "variant-a",
      "assignedAt": "2025-01-15T10:30:00Z"
    }
  }
}
```

The SDK automatically caches this metadata and uses it for analytics callbacks.

## Testing Your Integration

### Test Different Variants

#### Method 1: Use Different Device IDs

```swift
// Test with multiple device IDs to see different variants
StringProvider.shared.initialize(
    cacheSize: 1000,
    apiToken: "YOUR_API_TOKEN",
    baseURL: "https://api.stringboot.com",
    providedDeviceId: "test-device-001" // Variant A
)

StringProvider.shared.initialize(
    cacheSize: 1000,
    apiToken: "YOUR_API_TOKEN",
    baseURL: "https://api.stringboot.com",
    providedDeviceId: "test-device-002" // Variant B
)
```

#### Method 2: Clear App Data and Reinstall

```bash
# Delete app from simulator/device (clears device ID)
# Reinstall app via Xcode

# New device ID generated → different variant assignment
```

#### Method 3: Reset UserDefaults (Development Only)

```swift
#if DEBUG
func resetDeviceID() {
    UserDefaults.standard.removeObject(forKey: "com.stringboot.deviceId")
    print("Device ID reset - restart app for new assignment")
}
#endif
```

### Verify Analytics Events

Check that analytics events are firing:

```swift
// Enable logging
StringbootLogger.isLoggingEnabled = true
StringbootLogger.logLevel = .debug

// Watch console for:
// [Stringboot] 🎯 Experiment assigned: welcome_test → variant-a
// [Analytics] Experiment Assigned: {...}
```

## Best Practices

### 1. Experiment Design

✅ **DO:**
- Test one variable at a time (e.g., headline copy, not headline + CTA)
- Run experiments for sufficient sample size (typically 1-2 weeks minimum)
- Define success metrics before starting
- Use descriptive experiment keys (`onboarding_headline_test`)

❌ **DON'T:**
- Test too many variants simultaneously (3-4 max recommended)
- End experiments too early (statistical significance requires time)
- Change experiment mid-flight (invalidates results)

### 2. Device ID Strategy

✅ **DO:**
- Use auto-generated UUID for simplicity
- Use Firebase Installation ID for cross-platform consistency
- Document your device ID strategy
- Follow App Store privacy guidelines

❌ **DON'T:**
- Use personal data (email, phone)
- Use IDFA without user consent and proper ATT framework
- Change device ID generation mid-experiment
- Share device IDs with third parties without consent

### 3. Analytics Integration

✅ **DO:**
- Track assignment events for all experiments
- Include device ID in conversion events
- Set up proper event taxonomy (consistent naming)
- Test analytics in TestFlight before production

❌ **DON'T:**
- Send excessive events (batch if possible)
- Block UI on analytics calls (always async)
- Forget to handle analytics failures gracefully
- Track personal data without user consent

### 4. String Keys

✅ **DO:**
- Use descriptive keys: `home_cta_button`, `onboarding_welcome_headline`
- Group related strings: `checkout_*`, `onboarding_*`
- Document experiment strings in your dashboard

❌ **DON'T:**
- Use generic keys: `string1`, `text_a`
- Reuse experiment keys for different purposes

### 5. Testing

✅ **DO:**
- Test with multiple device IDs before launch
- Verify analytics events fire correctly
- Test offline behavior (cached assignments)
- Test in TestFlight with beta testers
- Document QA test plan

❌ **DON'T:**
- Skip testing analytics integration
- Assume experiments work without verification
- Test only on one variant
- Skip TestFlight testing

## Troubleshooting

### Problem: All users see the same variant

**Solution:**
- Verify different device IDs are being generated
- Check backend experiment configuration (traffic split)
- Ensure experiment is "running" (not paused/ended)
- Clear app data and test with new device ID

### Problem: Variant changes between app sessions

**Solution:**
- Check that device ID is being persisted correctly
- Verify UserDefaults are not being cleared
- Ensure you're not generating a new device ID on each init
- Check iCloud sync isn't causing UserDefaults conflicts

### Problem: Analytics events not firing

**Solution:**
- Verify `analyticsHandler` is passed to `initialize()`
- Check that handler methods are being called (add print statements)
- Ensure analytics SDK (Firebase/Mixpanel) is initialized before Stringboot
- Check network requests in analytics dashboard
- Verify analytics SDK is properly configured in Info.plist

### Problem: Can't test specific variant

**Solution:**
- Use custom device ID for testing
- Ask backend team to manually assign device ID to variant
- Use experiment override header (if supported)
- Check experiment traffic allocation (might be 0% for some variants)

## Examples

### Complete Integration Example

```swift
// MyApp.swift (SwiftUI)
import SwiftUI
import StringbootSDK
import FirebaseCore
import FirebaseAnalytics

@main
struct MyApp: App {

    init() {
        // Initialize Firebase first
        FirebaseApp.configure()

        // Create analytics handler
        let analyticsHandler = StringbootAnalyticsHandler()

        // Initialize Stringboot with analytics
        StringProvider.shared.initialize(
            cacheSize: 1000,
            apiToken: Bundle.main.object(forInfoDictionaryKey: "STRINGBOOT_API_TOKEN") as! String,
            baseURL: "https://api.stringboot.com",
            autoSync: true,
            analyticsHandler: analyticsHandler
        )
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

// StringbootAnalyticsHandler.swift
class StringbootAnalyticsHandler: StringbootAnalyticsHandler {

    func onExperimentAssigned(_ assignment: ExperimentAssignment) {
        Analytics.logEvent("sb_experiment_assigned", parameters: [
            "experiment_key": assignment.experimentKey,
            "variant": assignment.variantName,
            "string_key": assignment.stringKey
        ])
    }

    func onExperimentViewed(_ experimentKey: String, variantName: String) {
        Analytics.logEvent("sb_experiment_viewed", parameters: [
            "experiment_key": experimentKey,
            "variant": variantName
        ])
    }
}

// ContentView.swift
struct ContentView: View {
    @ObservedObject var stringProvider = StringProvider.shared

    var body: some View {
        VStack(spacing: 20) {
            // Display experiment strings
            SBText("welcome_message")
                .font(.title)

            SBText("app_description")
                .font(.body)

            Button(action: handleCTAClick) {
                SBText("cta_button")
                    .font(.headline)
            }
        }
        .padding()
    }

    func handleCTAClick() {
        // Track conversion
        let deviceId = UserDefaults.standard.string(forKey: "com.stringboot.deviceId")

        Analytics.logEvent("sb_experiment_conversion", parameters: [
            "experiment_key": "welcome_experiment",
            "action": "cta_clicked",
            "device_id": deviceId ?? "unknown"
        ])

        // Navigate or perform action
    }
}
```

## Related Documentation

- [QUICKSTART.md](QUICKSTART.md) - Basic SDK integration
- [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md) - Comprehensive integration guide
- [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md) - Advanced SDK capabilities
- [API_REFERENCE.md](API_REFERENCE.md) - Complete API documentation

## Need Help?

- **Email:** support@stringboot.com
- **Documentation:** https://docs.stringboot.com
- **Dashboard:** https://app.stringboot.com

---

**Start optimizing your app copy with data-driven A/B tests!** 📊

