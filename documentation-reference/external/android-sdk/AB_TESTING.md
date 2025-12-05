# A/B Testing with Stringboot Android SDK

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
- ✅ Test different string variations without code changes
- ✅ Run experiments on headlines, CTAs, error messages, onboarding copy
- ✅ Automatically assign users to consistent experiment variants
- ✅ Track conversions and analytics for each variant
- ✅ Make data-driven decisions on string effectiveness

### Key Benefits

- **No App Release Required** - Create and deploy experiments instantly
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
│  Android SDK    │ ← Requests strings with device ID
│  (Your App)     │ ← Receives assigned variant
│                 │ ← Displays string to user
└─────────────────┘
```

### The Flow

1. **Backend receives request** with `X-Device-ID` header
2. **Checks for active experiments** for the requested string key
3. **Assigns variant** based on device ID hash (if first time)
4. **Stores assignment** in database for consistency
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

```kotlin
// Option A: Auto-generated device ID (recommended)
StringProvider.initialize(
    context = context,
    cacheSize = 1000,
    api = stringbootApi
)
// Device ID automatically generated and persisted

// Option B: Custom device ID
StringProvider.initialize(
    context = context,
    cacheSize = 1000,
    api = stringbootApi,
    providedDeviceId = "your-custom-device-id" // e.g., from Firebase
)
```

**That's it!** Your app now supports A/B testing. The SDK automatically:
- ✅ Generates and persists a unique device ID
- ✅ Sends device ID with all string requests
- ✅ Receives and caches experiment assignments
- ✅ Returns assigned variants for experiment strings

### Step 2: Use Strings Normally

No code changes needed! Use strings exactly as before:

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // This string might be part of an A/B test
        // SDK automatically returns the assigned variant
        binding.root.applyStringbootTags()

        // Or programmatically:
        lifecycleScope.launch {
            val welcomeText = StringProvider.get("welcome_message", "en")
            // Returns "Welcome!" or "Hey there!" depending on assignment
            binding.tvWelcome.text = welcomeText
        }
    }
}
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

By default, the SDK generates a UUID and persists it in SharedPreferences:

```kotlin
// Automatic - SDK handles this internally
StringProvider.initialize(context, cacheSize = 1000, api = api)

// Device ID is generated on first launch and stored at:
// SharedPreferences: "stringboot" → "device_id"
```

### Custom Device ID

Provide your own device ID for cross-platform consistency or integration with other systems:

```kotlin
// Use Firebase Installation ID
val firebaseDeviceId = Firebase.installations.getId().await()

StringProvider.initialize(
    context = context,
    cacheSize = 1000,
    api = api,
    providedDeviceId = firebaseDeviceId
)
```

#### Common Custom ID Sources

| Source | Use Case | Example |
|--------|----------|---------|
| **Firebase IID** | Cross-platform consistency | `Firebase.installations.getId()` |
| **Advertising ID** | Marketing attribution (with user consent) | `AdvertisingIdClient.getAdvertisingIdInfo()` |
| **User ID** | Logged-in users only | `currentUser.id` |
| **Custom UUID** | Full control | `UUID.randomUUID().toString()` |

⚠️ **Privacy Note:** Never use personally identifiable information (email, phone, IMEI) as device ID.

### Retrieve Current Device ID

```kotlin
// The SDK doesn't expose device ID publicly for privacy reasons
// But you can access it from SharedPreferences if needed:

val prefs = context.getSharedPreferences("stringboot", Context.MODE_PRIVATE)
val deviceId = prefs.getString("device_id", null)
Log.d("Stringboot", "Current device ID: $deviceId")
```

## Analytics Integration

### Overview

Track experiment assignments and conversions by implementing a custom analytics handler.

### Implement Analytics Handler

Create a class implementing `StringbootAnalyticsHandler`:

```kotlin
package com.example.myapp.analytics

import com.stringboot.sdk.analytics.StringbootAnalyticsHandler
import com.stringboot.sdk.analytics.ExperimentAssignment

class MyAnalyticsHandler : StringbootAnalyticsHandler {

    override fun onExperimentAssigned(assignment: ExperimentAssignment) {
        // Called when device is assigned to an experiment variant

        // Send to Firebase Analytics
        Firebase.analytics.logEvent("experiment_assigned") {
            param("experiment_id", assignment.experimentId)
            param("experiment_key", assignment.experimentKey)
            param("string_key", assignment.stringKey)
            param("variant_name", assignment.variantName)
            param("language", assignment.languageCode)
        }

        // Or send to Mixpanel
        mixpanel.track("Experiment Assigned", JSONObject().apply {
            put("experiment_id", assignment.experimentId)
            put("experiment_key", assignment.experimentKey)
            put("variant_name", assignment.variantName)
        })

        // Or send to Amplitude
        Amplitude.getInstance().logEvent("Experiment Assigned", JSONObject().apply {
            put("experiment_key", assignment.experimentKey)
            put("variant", assignment.variantName)
        })
    }

    override fun onExperimentViewed(experimentKey: String, variantName: String) {
        // Called when string is actually displayed to user
        Firebase.analytics.logEvent("experiment_viewed") {
            param("experiment_key", experimentKey)
            param("variant_name", variantName)
        }
    }
}
```

### Register Analytics Handler

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        val analyticsHandler = MyAnalyticsHandler()

        StringProvider.initialize(
            context = this,
            cacheSize = 1000,
            api = stringbootApi,
            analyticsHandler = analyticsHandler
        )
    }
}
```

### Track Conversions

Track user actions to measure experiment effectiveness:

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        binding.root.applyStringbootTags()

        // Track conversion when user clicks CTA button
        binding.btnGetStarted.setOnClickListener {
            trackConversion("button_clicked", "welcome_experiment")
        }
    }

    private fun trackConversion(action: String, experimentKey: String) {
        // Get current device ID
        val prefs = getSharedPreferences("stringboot", Context.MODE_PRIVATE)
        val deviceId = prefs.getString("device_id", null)

        // Log conversion event
        Firebase.analytics.logEvent("experiment_conversion") {
            param("experiment_key", experimentKey)
            param("action", action)
            param("device_id", deviceId ?: "unknown")
        }

        Log.i("Analytics", "Conversion tracked: $action for $experimentKey")
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

```kotlin
// User 1 (device ID: "abc123")
val text1 = StringProvider.get("welcome_message", "en")
// → Backend hashes "abc123" → assigns to Variant A
// → Returns "Hey there!"
// → Assignment cached in database

// User 2 (device ID: "xyz789")
val text2 = StringProvider.get("welcome_message", "en")
// → Backend hashes "xyz789" → assigns to Control
// → Returns "Welcome!"

// User 1 again (same device, different session)
val text3 = StringProvider.get("welcome_message", "en")
// → Backend retrieves cached assignment
// → Returns "Hey there!" (consistent!)
```

### Experiment Metadata

The backend may include experiment metadata in responses:

```kotlin
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

```kotlin
// Test with multiple device IDs to see different variants
StringProvider.initialize(
    context = context,
    cacheSize = 1000,
    api = api,
    providedDeviceId = "test-device-001" // Variant A
)

StringProvider.initialize(
    context = context,
    cacheSize = 1000,
    api = api,
    providedDeviceId = "test-device-002" // Variant B
)
```

#### Method 2: Clear Cache and Reinstall

```bash
# Uninstall app (clears device ID)
adb uninstall com.example.myapp

# Reinstall app
./gradlew installDebug

# New device ID generated → different variant assignment
```

#### Method 3: Backend Override (QA/Testing Only)

If your backend supports it, use override headers:

```kotlin
// Add to StringSyncV2Client for QA builds
val client = OkHttpClient.Builder()
    .addInterceptor { chain ->
        val request = chain.request().newBuilder()
            .header("X-Experiment-Override", "welcome_test=variant-a")
            .build()
        chain.proceed(request)
    }
    .build()
```

### Verify Analytics Events

Check that analytics events are firing:

```kotlin
// Enable logging
StringbootLogger.isLoggingEnabled = true
StringbootLogger.logLevel = StringbootLogger.LogLevel.DEBUG

// Watch logcat for:
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
- Use Firebase IID for cross-platform consistency
- Document your device ID strategy

❌ **DON'T:**
- Use personal data (email, phone, IMEI)
- Change device ID generation mid-experiment
- Share device IDs with third parties without consent

### 3. Analytics Integration

✅ **DO:**
- Track assignment events for all experiments
- Include device ID in conversion events
- Set up proper event taxonomy (consistent naming)
- Test analytics in staging environment

❌ **DON'T:**
- Send excessive events (batch if possible)
- Block UI on analytics calls (always async)
- Forget to handle analytics failures gracefully

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
- Document QA test plan

❌ **DON'T:**
- Skip testing analytics integration
- Assume experiments work without verification
- Test only on one variant

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
- Verify SharedPreferences are not being cleared
- Ensure you're not generating a new device ID on each init

### Problem: Analytics events not firing

**Solution:**
- Verify `analyticsHandler` is passed to `initialize()`
- Check that handler methods are being called (add logs)
- Ensure analytics SDK (Firebase/Mixpanel) is initialized
- Check network requests in analytics dashboard

### Problem: Can't test specific variant

**Solution:**
- Use custom device ID for testing
- Ask backend team to manually assign device ID to variant
- Use experiment override header (if supported)
- Check experiment traffic allocation (might be 0% for some variants)

## Examples

### Complete Integration Example

```kotlin
// MyApplication.kt
class MyApplication : Application() {

    override fun onCreate() {
        super.onCreate()

        // Create analytics handler
        val analyticsHandler = object : StringbootAnalyticsHandler {
            override fun onExperimentAssigned(assignment: ExperimentAssignment) {
                Firebase.analytics.logEvent("sb_experiment_assigned") {
                    param("experiment_key", assignment.experimentKey)
                    param("variant", assignment.variantName)
                    param("string_key", assignment.stringKey)
                }
            }

            override fun onExperimentViewed(experimentKey: String, variantName: String) {
                Firebase.analytics.logEvent("sb_experiment_viewed") {
                    param("experiment_key", experimentKey)
                    param("variant", variantName)
                }
            }
        }

        // Initialize Stringboot with analytics
        val api = StringSyncV2Client(
            baseUrl = "https://api.stringboot.com",
            apiToken = BuildConfig.STRINGBOOT_API_TOKEN
        )

        StringProvider.initialize(
            context = this,
            cacheSize = 1000,
            api = api,
            analyticsHandler = analyticsHandler
        )
    }
}

// MainActivity.kt
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // Display experiment strings
        binding.root.applyStringbootTags()

        // Track conversions
        binding.btnGetStarted.setOnClickListener {
            // Track button click as conversion
            Firebase.analytics.logEvent("sb_experiment_conversion") {
                param("experiment_key", "welcome_experiment")
                param("action", "cta_clicked")
                param("variant", getCurrentVariant("welcome_message"))
            }

            // Navigate to next screen
            startActivity(Intent(this, OnboardingActivity::class.java))
        }
    }

    private fun getCurrentVariant(stringKey: String): String {
        // You can track which variant was shown by logging when string loads
        // Or retrieve from analytics handler if you cache assignments
        return "unknown" // Implement based on your needs
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

