# 🚀 Stringboot Android SDK - Integration Guide

Complete integration guide for the Stringboot Android SDK v1.0.8+

> **📌 Updated:** Added critical information about initial network sync requirement for first app launch.

## What is Stringboot?

Stringboot is an internationalization (i18n) SDK that fetches translations from the cloud, allowing you to update app text without releasing new versions. The SDK features smart caching, offline support, and reactive UI updates.

---

## ⚡ Quick Start (5 minutes)

### Step 1: Add Dependency

Add the SDK to your app's `build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.stringboot:stringboot-android-sdk:1.0.7")
}
```

### Step 2: Initialize in Application Class

Create an Application class if you don't have one:

```kotlin
package com.example.myapp

import android.app.Application
import android.util.Log
import com.stringboot.sdk.StringProvider
import com.stringboot.sdk.StringbootExtensions
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.SupervisorJob
import kotlinx.coroutines.launch

class MyApplication : Application() {

    // Application-level coroutine scope for background work
    private val applicationScope = CoroutineScope(SupervisorJob() + Dispatchers.IO)

    override fun onCreate() {
        super.onCreate()

        // Step 1: Initialize Stringboot SDK
        val success = StringbootExtensions.autoInitialize(this)

        if (!success) {
            // Fallback to offline mode
            StringProvider.initialize(
                context = this,
                cacheSize = 1000,
                api = null // Offline mode
            )
        }

        // Step 2: CRITICAL - Trigger initial sync to populate database
        // This ensures strings are available when UI loads
        if (success) {
            applicationScope.launch {
                try {
                    val locale = StringProvider.deviceLocale()
                    val refreshSuccess = StringProvider.refreshFromNetwork(locale)
                    if (refreshSuccess) {
                        val count = StringProvider.getStringCount(locale)
                        Log.i("Stringboot", "✅ Initial sync completed: $count strings for $locale")
                    } else {
                        Log.w("Stringboot", "⚠️ Initial sync failed for $locale - using cached data")
                    }
                } catch (e: Exception) {
                    Log.e("Stringboot", "❌ Initial sync error: ${e.message}", e)
                }
            }
        }
    }
}
```

Register it in `AndroidManifest.xml`:

```xml
<application
    android:name=".MyApplication"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name">

    <!-- Stringboot configuration (for auto-initialization) -->
    <meta-data
        android:name="com.stringboot.api_url"
        android:value="https://api.stringboot.com" />
    <meta-data
        android:name="com.stringboot.api_token"
        android:value="YOUR_API_TOKEN_HERE" />
    <meta-data
        android:name="com.stringboot.cache_size"
        android:value="1000" />

    <activity android:name=".MainActivity">
        <!-- ... -->
    </activity>
</application>
```

### Step 3: Use Stringboot in Your Activity

```kotlin
import androidx.lifecycle.lifecycleScope
import com.stringboot.sdk.StringProvider
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        lifecycleScope.launch {
            // Get a translated string
            val text = StringProvider.get("welcome_message", "en")
            findViewById<TextView>(R.id.textView).text = text
        }
    }
}
```

**That's it!** 🎉

> **⚠️ IMPORTANT:** The initial `refreshFromNetwork()` call in Step 2 is **critical** for first-time app launches. Without it, the database will be empty and `applyStringbootTags()` won't find any strings. The SDK uses an offline-first architecture (Cache → Database → Network), so the database must be populated before UI elements can display Stringboot strings.

---

## 🏷️ XML Tag-Based Integration (No Code!)

The easiest way to use Stringboot is with XML tags. Just add `android:tag` to your TextViews and call one function.

### XML Layout

```xml
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- Add android:tag with your string key -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:tag="welcome_message"
        android:text="Welcome!" />

    <!-- Supports stringboot: prefix too -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:tag="stringboot:login_button"
        android:text="Login" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:tag="save_button"
        android:text="Save" />
</LinearLayout>
```

### Activity Code

```kotlin
import com.stringboot.sdk.views.applyStringbootTags

class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // Auto-apply Stringboot to all tagged TextViews
        binding.root.applyStringbootTags()
    }
}
```

### How it Works

1. Scans all views in the hierarchy
2. Finds TextViews with `android:tag` set
3. Extracts the string key from the tag
4. Fetches the translation using `StringProvider`
5. Updates the TextView text
6. Sets up reactive updates (auto-updates when language changes)

### Tag Format Options

Both formats work identically:

```xml
<!-- Direct key (recommended) -->
<TextView android:tag="welcome_message" />

<!-- With prefix (more explicit) -->
<TextView android:tag="stringboot:welcome_message" />
```

---

## 📱 Programmatic Integration

### Extension Functions on TextView

Apply Stringboot to individual TextViews programmatically:

```kotlin
import com.stringboot.sdk.views.stringboot
import com.stringboot.sdk.views.stringbootOnce

// Reactive version (auto-updates on language change)
textView.stringboot(
    key = "welcome_message",
    language = "en",          // Optional: defaults to device locale
    defaultText = "Welcome",  // Optional: fallback text
    reactive = true           // Default: true
)

// Non-reactive version (loads once)
textView.stringbootOnce(
    key = "login_button",
    language = null,          // Uses device locale
    defaultText = "Login"
)
```

### Direct StringProvider API

Use `StringProvider` directly for full control:

```kotlin
import androidx.lifecycle.lifecycleScope
import com.stringboot.sdk.StringProvider
import kotlinx.coroutines.launch

lifecycleScope.launch {
    // Get a string
    val text = StringProvider.get(
        key = "welcome_message",
        lang = "en",
        allowNetworkFetch = false  // Optional: allow network if not cached
    )
    textView.text = text
}
```

---

## 🔄 Reactive Updates with Flow

Want UI to auto-update when translations change? Use Flows:

```kotlin
import com.stringboot.sdk.StringProvider
import kotlinx.coroutines.flow.collectLatest

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        lifecycleScope.launch {
            // Flow automatically emits new values when translation changes
            StringProvider.getFlow("welcome_message", "en")
                .collectLatest { text ->
                    textView.text = text
                }
        }
    }
}
```

---

## 🌍 Language Switching

### Switch Language Dynamically

```kotlin
private fun switchLanguage(newLang: String) {
    lifecycleScope.launch {
        // 1. Set the new locale
        StringProvider.setLocale(newLang)

        // 2. Refresh strings from network
        val success = StringProvider.refreshFromNetwork(newLang)

        if (success) {
            // 3. Re-apply tags to update UI
            binding.root.applyStringbootTags()

            Toast.makeText(this@MainActivity,
                "Language switched to $newLang",
                Toast.LENGTH_SHORT).show()
        }
    }
}
```

### Get Available Languages

```kotlin
lifecycleScope.launch {
    // Get languages from server (with cache fallback)
    val languages = StringProvider.getAvailableLanguagesFromServer()

    languages.forEach { lang ->
        println("${lang.name} (${lang.code})")
        // Example: English (en), Spanish (es), French (fr)
    }

    // Or get locally cached language codes
    val cachedLangCodes = StringProvider.getAvailableLanguages()
}
```

### Complete Language Picker Example

```kotlin
import com.stringboot.sdk.models.ActiveLanguage

private fun showLanguagePicker() {
    lifecycleScope.launch {
        val languages = StringProvider.getAvailableLanguagesFromServer()

        val languageNames = languages.map { "${it.name} - ${it.code.uppercase()}" }.toTypedArray()

        AlertDialog.Builder(this@MainActivity)
            .setTitle("Select Language")
            .setItems(languageNames) { _, which ->
                switchLanguage(languages[which].code)
            }
            .show()
    }
}
```

---

## 🚀 Performance Optimization

### Preload Language on Startup

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        lifecycleScope.launch {
            // Preload 500 most common strings into memory
            StringProvider.preloadLanguage("en", maxStrings = 500)

            // Now UI operations are super fast!
        }
    }
}
```

### Smart Cache Warming

```kotlin
lifecycleScope.launch {
    // Intelligently warm cache with frequently accessed strings
    StringProvider.smartWarmCache(lang = "en", limit = 200)
}
```

### Get Cache Statistics

```kotlin
val stats = StringProvider.getCacheStatistics()
println("Cache size: ${stats["size"]}")
println("Hit rate: ${stats["hitRate"]}")
println("Current language: ${stats["currentLanguage"]}")
```

---

## 🧠 How StringProvider Works

### Three-Layer Architecture

```
┌─────────────────────────────────────────┐
│  Your Code                               │
│  textView.text = StringProvider.get()   │
└────────────────┬────────────────────────┘
                 ↓
┌────────────────────────────────────────┐
│  Layer 1: Memory Cache (RAM)           │
│  - Instant access (< 1ms)              │
│  - Smart LRU eviction                  │
│  - Language-aware prioritization       │
└────────────────┬───────────────────────┘
                 ↓ (Cache miss)
┌────────────────────────────────────────┐
│  Layer 2: Local Database (Room)        │
│  - Persistent offline storage          │
│  - Fast indexed lookups (< 50ms)       │
│  - Survives app restarts               │
└────────────────┬───────────────────────┘
                 ↓ (Not in DB)
┌────────────────────────────────────────┐
│  Layer 3: Network (Stringboot API)     │
│  - ETag-based delta sync               │
│  - Only downloads changes              │
│  - SHA-256 integrity verification      │
└────────────────────────────────────────┘
```

### ⚠️ Critical: Database Must Be Populated First

**The database (Layer 2) is empty on first app launch!** This is why the initial `refreshFromNetwork()` call in your Application class is essential:

```kotlin
// In Application.onCreate()
applicationScope.launch {
    val locale = StringProvider.deviceLocale()
    StringProvider.refreshFromNetwork(locale) // Populates Layer 2 (Database)
}
```

**Without this:**
- `StringProvider.get()` checks Cache (empty) → Database (empty) → Returns `"??key??"`
- `StringProvider.getFlow()` observes Database (empty) → Emits `"??key??"` and waits
- Network is never triggered because `getFlow()` doesn't have `allowNetworkFetch` parameter

**With initial refresh:**
- Database is populated with all strings from backend
- Cache and Flow queries find strings immediately
- Subsequent app launches use cached data (fast!)

### Memory Management

The SDK automatically handles memory pressure:

```kotlin
// SDK responds to Android's low memory callbacks:
// - TRIM_MEMORY_MODERATE → Evict 25% of cache
// - TRIM_MEMORY_LOW → Evict 50% of cache
// - TRIM_MEMORY_CRITICAL → Keep only current language

// You can manually trigger if needed:
StringProvider.handleMemoryPressure(ComponentCallbacks2.TRIM_MEMORY_MODERATE)
```

---

## 📖 API Reference

### Initialization

```kotlin
// Manual initialization (recommended in Application.onCreate())
StringProvider.initialize(
    context: Context,           // Application context
    cacheSize: Int = 1000,      // Number of strings in memory
    api: StringbootApi? = null  // API client (null = offline mode)
)

// Auto-initialization from manifest
StringbootExtensions.autoInitialize(context: Context): Boolean

// Check if initialized
StringProvider.isInitialized(): Boolean
```

### String Access

```kotlin
// Get a string (suspend function)
suspend fun get(
    key: String,                    // String key
    lang: String = deviceLocale(),  // Language code
    allowNetworkFetch: Boolean = false  // Allow network if not cached
): String

// Get a reactive Flow
fun getFlow(
    key: String,
    lang: String = deviceLocale()
): Flow<String>

// Get string count for a language
suspend fun getStringCount(lang: String = deviceLocale()): Int
```

### Language Management

```kotlin
// Set current locale
fun setLocale(lang: String)

// Get device locale
fun deviceLocale(): String

// Get cached language codes
suspend fun getAvailableLanguages(): List<String>

// Get languages from server (with ActiveLanguage details)
suspend fun getAvailableLanguagesFromServer(): List<ActiveLanguage>
```

### Network Operations

```kotlin
// Refresh strings from network
suspend fun refreshFromNetwork(lang: String = deviceLocale()): Boolean

// Force refresh all languages
suspend fun forceRefresh(): Boolean
```

### Cache Operations

```kotlin
// Preload language into memory
suspend fun preloadLanguage(lang: String, maxStrings: Int = 500)

// Smart cache warming (frequently accessed strings)
suspend fun smartWarmCache(lang: String = deviceLocale(), limit: Int = 200)

// Clear memory cache only
fun clearMemoryCache()

// Clear specific language cache
fun clearLanguageCache(lang: String)

// Clear all caches
suspend fun clearCache(clearDatabase: Boolean = false)

// Get cache statistics
fun getCacheStatistics(): Map<String, Any>

// Get cache entries for debugging
suspend fun getCacheEntries(limit: Int = 100): List<CacheEntry>
```

### Extension Functions

```kotlin
// Apply Stringboot to TextView (reactive)
TextView.stringboot(
    key: String,
    language: String? = null,
    defaultText: String? = null,
    reactive: Boolean = true
)

// Non-reactive version
TextView.stringbootOnce(
    key: String,
    language: String? = null,
    defaultText: String? = null
)

// Auto-apply to all tagged TextViews
View.applyStringbootTags()
```

---

## 🔒 Security Best Practices

### API Token Security

**Don't hardcode tokens:**

```kotlin
// ❌ DON'T
val token = "sk_live_123456789"

// ✅ DO - Use manifest metadata
<meta-data
    android:name="com.stringboot.api_token"
    android:value="${STRINGBOOT_API_TOKEN}" />

// ✅ DO - Use BuildConfig
val token = BuildConfig.STRINGBOOT_API_TOKEN
```

**In `build.gradle.kts`:**

```kotlin
android {
    defaultConfig {
        buildConfigField("String", "STRINGBOOT_API_TOKEN",
            "\"${project.findProperty("stringboot.api.token")}\"")
    }
}
```

**In `local.properties` (gitignored):**

```properties
stringboot.api.token=your-secret-token-here
```

### ProGuard/R8

The SDK includes consumer ProGuard rules that are automatically applied. When you enable minification, the SDK's public API is preserved:

```kotlin
android {
    buildTypes {
        release {
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
}
```

**No additional ProGuard configuration needed!**

---

## 🐛 Debugging & Troubleshooting

### Enable Logging

Logging is automatically enabled in debug builds and disabled in release builds via `StringbootLogger`.

### Common Issues

**Strings show as "??key??" (Most Common Issue)**

This happens when the database is empty on first app launch. **Solution:**

```kotlin
// In your Application class onCreate()
if (success) {
    applicationScope.launch {
        val locale = StringProvider.deviceLocale()
        StringProvider.refreshFromNetwork(locale) // Populates database
    }
}
```

**Other causes:**
- String key doesn't exist in your Stringboot dashboard
- API token is incorrect or missing
- Network request failed (check logs for API errors)

**Language not switching**
- Call both `setLocale()` and `refreshFromNetwork()`
- Re-apply tags: `binding.root.applyStringbootTags()`
- Check if language exists on server

**App feels slow**
- Use `preloadLanguage()` during startup
- Don't call `refreshFromNetwork()` on every screen
- Use `allowNetworkFetch = false` when calling `get()`

**Out of memory**
- Reduce cache size: `initialize(cacheSize = 200)`
- SDK handles memory pressure automatically
- Don't preload too many strings at once

### Debug Methods

```kotlin
// Check initialization
if (!StringProvider.isInitialized()) {
    Log.e("MyApp", "StringProvider not initialized!")
}

// View cache stats
val stats = StringProvider.getCacheStatistics()
Log.d("MyApp", "Cache: $stats")

// Inspect cache entries
lifecycleScope.launch {
    val entries = StringProvider.getCacheEntries(limit = 50)
    entries.forEach { entry ->
        Log.d("MyApp", "Key: ${entry.key}, Value: ${entry.value}")
    }
}

// Force refresh
lifecycleScope.launch {
    val success = StringProvider.forceRefresh()
    if (success) {
        Toast.makeText(this@MainActivity, "Refreshed!", Toast.LENGTH_SHORT).show()
    }
}

// Clear caches
lifecycleScope.launch {
    StringProvider.clearCache(clearDatabase = true)
}
```

---

## 📋 Complete Working Example

Here's a complete `MainActivity` showing all features:

```kotlin
package com.example.myapp

import android.os.Bundle
import android.widget.Button
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AlertDialog
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import com.stringboot.sdk.StringProvider
import com.stringboot.sdk.views.applyStringbootTags
import com.stringboot.sdk.views.stringboot
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private var currentLanguage = "en"

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 1. Apply Stringboot tags to XML TextViews
        findViewById<android.view.View>(android.R.id.content).applyStringbootTags()

        // 2. Programmatically apply to specific TextView
        findViewById<TextView>(R.id.tvDynamic).stringboot(
            key = "welcome_message",
            defaultText = "Welcome!"
        )

        // 3. Set up language switcher
        findViewById<Button>(R.id.btnChangeLanguage).setOnClickListener {
            showLanguagePicker()
        }

        // 4. Preload strings for performance
        lifecycleScope.launch {
            StringProvider.preloadLanguage(currentLanguage, maxStrings = 500)
        }
    }

    private fun showLanguagePicker() {
        lifecycleScope.launch {
            val languages = StringProvider.getAvailableLanguagesFromServer()

            val names = languages.map { "${it.name} - ${it.code.uppercase()}" }.toTypedArray()

            AlertDialog.Builder(this@MainActivity)
                .setTitle("Select Language")
                .setItems(names) { _, which ->
                    switchLanguage(languages[which].code)
                }
                .show()
        }
    }

    private fun switchLanguage(newLang: String) {
        if (newLang == currentLanguage) return

        lifecycleScope.launch {
            currentLanguage = newLang
            StringProvider.setLocale(newLang)

            val success = StringProvider.refreshFromNetwork(newLang)

            if (success) {
                // Re-apply tags to update UI
                findViewById<android.view.View>(android.R.id.content).applyStringbootTags()

                Toast.makeText(
                    this@MainActivity,
                    "Language switched to $newLang",
                    Toast.LENGTH_SHORT
                ).show()
            }
        }
    }
}
```

**XML Layout:**

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <!-- Tag-based TextViews -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:tag="app_title"
        android:text="My App"
        android:textSize="24sp" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:tag="welcome_message"
        android:text="Welcome!" />

    <!-- Programmatic TextView -->
    <TextView
        android:id="@+id/tvDynamic"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Loading..." />

    <!-- Language switcher -->
    <Button
        android:id="@+id/btnChangeLanguage"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:tag="change_language_button"
        android:text="Change Language" />
</LinearLayout>
```

---

## 🎯 Best Practices

1. **⚠️ CRITICAL: Always Call `refreshFromNetwork()` in Application** - The database is empty on first launch. Call `refreshFromNetwork()` in `Application.onCreate()` to populate the database before any UI loads.
2. **Initialize Once** - Call `StringProvider.initialize()` only once in `Application.onCreate()`
3. **Use Coroutines** - Always call suspend functions inside `lifecycleScope.launch` or application scope
4. **Preload Early** - Call `preloadLanguage()` during app startup for better performance
5. **Cache Wisely** - Use `cacheSize` between 1000-2000 for most apps
6. **Handle Offline** - SDK works offline automatically, but test it!
7. **Descriptive Keys** - Use clear string keys like `welcome_message` not `msg1`
8. **Use Tags** - XML tag-based integration is the easiest and most maintainable
9. **Reactive UI** - Use `getFlow()` or `stringboot()` for auto-updating UI
10. **Secure Tokens** - Never hardcode API tokens, use BuildConfig or manifest metadata
11. **Test Minification** - Always test release builds with ProGuard/R8 enabled
12. **Test Fresh Install** - Clear app data and test first-time launch to verify initial sync works

---

## 🧪 A/B Testing Integration

### Overview

The SDK supports A/B testing for strings, allowing you to test different copy variations and track which ones perform better.

### How It Works

1. **Device ID**: SDK generates a unique UUID per device installation
2. **X-Device-ID Header**: Automatically sent with all API requests
3. **Backend Assignment**: Backend assigns device to experiment variant
4. **Consistent Delivery**: Same device always gets same variant
5. **Analytics Integration**: Track experiments in your analytics platform

### Basic Setup

The SDK handles A/B testing automatically - no code changes required! When experiments are active:

```kotlin
// User calls this normally
val text = StringProvider.get("welcome_message", "en")

// Backend might return:
// - "Welcome!" (control variant)
// - "Hey there!" (variant A)
// - "Hello friend!" (variant B)

// The SDK transparently delivers the assigned variant
```

### Analytics Integration (Optional)

Track experiment assignments in Firebase Analytics, Mixpanel, or Amplitude:

#### Step 1: Create Analytics Handler

```kotlin
import com.stringboot.sdk.analytics.StringbootAnalyticsHandler
import com.stringboot.sdk.models.ExperimentAssignment
import com.google.firebase.analytics.FirebaseAnalytics

class FirebaseAnalyticsHandler(
    private val firebaseAnalytics: FirebaseAnalytics
) : StringbootAnalyticsHandler {
    override fun onExperimentsAssigned(experiments: Map<String, ExperimentAssignment>) {
        experiments.forEach { (stringKey, experiment) ->
            // Set user property: stringboot_exp_{experimentKey}
            firebaseAnalytics.setUserProperty(
                "stringboot_exp_${experiment.experimentKey}",
                experiment.variantName
            )

            Log.d("Analytics", "Experiment: ${experiment.experimentKey} = ${experiment.variantName}")
        }
    }
}
```

#### Step 2: Initialize with Analytics Handler

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        val api = StringbootExtensions.createApiInstance(apiUrl, apiToken)
        val analyticsHandler = FirebaseAnalyticsHandler(
            FirebaseAnalytics.getInstance(this)
        )

        StringProvider.initialize(
            context = this,
            api = api,
            analyticsHandler = analyticsHandler  // Add this
        )
    }
}
```

### Advanced: Custom Device ID

If your app already has a device identifier (e.g., from another SDK), use it for consistency:

```kotlin
import com.google.firebase.installations.FirebaseInstallations

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        // Get device ID from Firebase or your analytics SDK
        FirebaseInstallations.getInstance().id.addOnSuccessListener { deviceId ->
            val api = StringbootExtensions.createApiInstance(apiUrl, apiToken)

            StringProvider.initialize(
                context = this,
                api = api,
                providedDeviceId = deviceId,  // Use consistent device ID
                analyticsHandler = analyticsHandler
            )
        }
    }
}
```

### Property Naming Convention

All experiments are tracked with this naming pattern:
```
stringboot_exp_{experimentKey}
```

**Example:**
- Experiment: `welcome_test`
- Firebase Property: `stringboot_exp_welcome_test`
- Value: `control`, `variant-a`, `variant-b`, etc.

### Debugging Experiments

Check active experiment assignments:

```kotlin
// Get current experiments
val experiments = runBlocking { StringProvider.getExperiments() }
experiments.forEach { (stringKey, exp) ->
    Log.d("Stringboot", """
        String: $stringKey
        Experiment: ${exp.experimentKey}
        Variant: ${exp.variantName}
        Assigned: ${exp.assignedAt}
    """.trimIndent())
}

// Get current device ID
val deviceId = StringProvider.getDeviceId()
Log.d("Stringboot", "Device ID: $deviceId")
```

### Analytics Platform Examples

#### Mixpanel
```kotlin
class MixpanelAnalyticsHandler(
    private val mixpanel: MixpanelAPI
) : StringbootAnalyticsHandler {
    override fun onExperimentsAssigned(experiments: Map<String, ExperimentAssignment>) {
        experiments.forEach { (_, exp) ->
            mixpanel.people.set("stringboot_exp_${exp.experimentKey}", exp.variantName)
        }
    }
}
```

#### Amplitude
```kotlin
class AmplitudeAnalyticsHandler(
    private val amplitude: AmplitudeClient
) : StringbootAnalyticsHandler {
    override fun onExperimentsAssigned(experiments: Map<String, ExperimentAssignment>) {
        val identify = Identify()
        experiments.forEach { (_, exp) ->
            identify.set("stringboot_exp_${exp.experimentKey}", exp.variantName)
        }
        amplitude.identify(identify)
    }
}
```

### Key Points

✅ **Automatic** - A/B testing works without code changes
✅ **Consistent** - Same device always gets same variant
✅ **Persistent** - Assignments survive app restarts
✅ **Optional** - Analytics integration is completely optional
✅ **Backward Compatible** - No breaking changes to existing apps

For complete details, see [AB_TESTING.md](AB_TESTING.md)

---

## 📋 FAQ Provider Integration

> **Available in:** SDK v1.1.0+

The FAQ Provider enables you to deliver dynamic, multilingual FAQ content to your users with the same offline-first, cached architecture as string resources.

### Overview

FAQProvider offers:
- **Tag-based organization** - Categorize FAQs by primary tag and optional sub-tags
- **Multi-language support** - Automatic language fallback to English
- **Offline-first** - Three-tier caching (memory → database → network)
- **Delta sync** - Only download changed FAQs
- **Reactive updates** - Auto-updating UI with Kotlin Flow

### Initialization

FAQProvider is automatically initialized when you initialize StringProvider:

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        // This initializes both StringProvider AND FAQProvider
        val success = StringbootExtensions.autoInitialize(this)

        if (success) {
            applicationScope.launch {
                // Refresh both strings and FAQs from network
                val locale = StringProvider.deviceLocale()
                StringProvider.refreshFromNetwork(locale)
                FAQProvider.refreshFromNetwork(locale)
            }
        }
    }
}
```

### Basic Usage

#### Fetching FAQs

```kotlin
import com.stringboot.sdk.FAQProvider
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext

// In your Activity or Fragment
lifecycleScope.launch {
    val faqs = withContext(Dispatchers.IO) {
        FAQProvider.getFAQs(
            tag = "Identity Verification",
            subTags = listOf("Document Upload"),
            lang = "en",
            allowNetworkFetch = true
        )
    }

    // Display FAQs in RecyclerView or UI
    displayFAQs(faqs)
}
```

#### FAQ Data Model

```kotlin
data class FAQ(
    val id: Int,                      // Unique FAQ ID
    val question: String,             // FAQ question
    val answer: String,               // FAQ answer (supports HTML/markdown)
    val tag: String,                  // Primary tag (e.g., "payments")
    val subTags: List<String>,        // Sub-tags (e.g., ["refunds", "disputes"])
    val languageCode: String?,        // Language code (e.g., "en", "es")
    val appId: String?,               // App identifier
    val createdAt: String?,           // ISO 8601 timestamp
    val updatedAt: String?            // ISO 8601 timestamp
)
```

### Reactive Flow for Auto-Updating UI

For UI that automatically updates when FAQs change (similar to StringProvider's `getFlow`):

```kotlin
import androidx.lifecycle.lifecycleScope
import kotlinx.coroutines.flow.launchIn
import kotlinx.coroutines.flow.onEach

class FAQActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Collect FAQ updates reactively
        FAQProvider.getFAQsFlow(
            tag = "Identity Verification",
            subTags = listOf("Document Upload"),
            lang = "en"
        ).onEach { faqs ->
            // UI automatically updates when FAQs change in database
            updateRecyclerView(faqs)
        }.launchIn(lifecycleScope)
    }

    private fun updateRecyclerView(faqs: List<FAQ>) {
        faqAdapter.submitList(faqs)
        if (faqs.isEmpty()) {
            showEmptyState()
        }
    }
}
```

### Tag-Based Filtering

FAQs are organized by **primary tag** and optional **sub-tags** for precise categorization:

```kotlin
// Get all FAQs for "payments" tag
val allPaymentFAQs = FAQProvider.getFAQs(
    tag = "payments",
    lang = "en"
)

// Get only refund-related FAQs
val refundFAQs = FAQProvider.getFAQs(
    tag = "payments",
    subTags = listOf("refunds"),
    lang = "en"
)

// Get FAQs matching multiple sub-tags (OR filter)
val paymentMethodFAQs = FAQProvider.getFAQs(
    tag = "payments",
    subTags = listOf("card", "bank_transfer", "wallet"),
    lang = "en"
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

```kotlin
// User's device is set to "fr" (French)
val faqs = FAQProvider.getFAQs(
    tag = "payments",
    lang = "fr",  // Will try French first
    allowNetworkFetch = true
)

// Fallback order:
// 1. Memory cache (French)
// 2. Database (French)
// 3. Database (English fallback)
// 4. Network (French)
// 5. Empty list
```

**Check which language was returned:**

```kotlin
val faqs = FAQProvider.getFAQs(tag = "payments", lang = "fr")
if (faqs.isNotEmpty()) {
    val actualLanguage = faqs.first().languageCode
    if (actualLanguage == "en" && "fr" != "en") {
        showLanguageFallbackNotice()
    }
}
```

### Network Refresh

Manually refresh FAQs from the network (uses delta sync for efficiency):

```kotlin
import kotlinx.coroutines.launch

// In Application.onCreate() or onResume()
applicationScope.launch {
    val locale = StringProvider.deviceLocale()

    // Refresh FAQs (only downloads changed FAQs since last sync)
    val success = FAQProvider.refreshFromNetwork(locale)

    if (success) {
        Log.d("FAQs", "Successfully synced latest FAQs")
    }
}
```

### Complete Example: FAQ Screen

Here's a complete implementation showing tag filtering, reactive updates, and RecyclerView integration:

```kotlin
class FAQActivity : AppCompatActivity() {

    private lateinit var binding: ActivityFaqBinding
    private lateinit var faqAdapter: FAQAdapter

    private var currentTag = "Identity Verification"
    private var selectedSubTags = emptyList<String>()
    private var currentLanguage = "en"

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityFaqBinding.inflate(layoutInflater)
        setContentView(binding.root)

        currentLanguage = StringProvider.deviceLocale()

        setupRecyclerView()
        setupSubTagFilters()
        loadFAQs()
    }

    private fun setupRecyclerView() {
        faqAdapter = FAQAdapter { faq ->
            // Handle FAQ click (expand/collapse)
            Toast.makeText(this, faq.question, Toast.LENGTH_SHORT).show()
        }
        binding.recyclerView.apply {
            layoutManager = LinearLayoutManager(context)
            adapter = faqAdapter
        }
    }

    private fun setupSubTagFilters() {
        val subTags = listOf("Document Upload", "Verification Failed", "Liveness Check")

        subTags.forEach { subTag ->
            val chip = Chip(this).apply {
                text = subTag
                isCheckable = true
                setOnClickListener {
                    selectedSubTags = if (selectedSubTags.contains(subTag)) {
                        selectedSubTags - subTag
                    } else {
                        selectedSubTags + subTag
                    }
                    loadFAQs()
                }
            }
            binding.chipGroup.addView(chip)
        }
    }

    private fun loadFAQs() {
        // Option 1: Reactive Flow (recommended)
        FAQProvider.getFAQsFlow(
            tag = currentTag,
            subTags = selectedSubTags,
            lang = currentLanguage
        ).onEach { faqs ->
            faqAdapter.submitList(faqs)
            binding.emptyState.isVisible = faqs.isEmpty()
        }.launchIn(lifecycleScope)

        // Option 2: One-time fetch
        // lifecycleScope.launch {
        //     val faqs = withContext(Dispatchers.IO) {
        //         FAQProvider.getFAQs(
        //             tag = currentTag,
        //             subTags = selectedSubTags,
        //             lang = currentLanguage,
        //             allowNetworkFetch = true
        //         )
        //     }
        //     faqAdapter.submitList(faqs)
        // }
    }
}

// RecyclerView Adapter with expandable items
class FAQAdapter(
    private val onFAQClick: (FAQ) -> Unit
) : RecyclerView.Adapter<FAQViewHolder>() {

    private var faqs = listOf<FAQ>()
    private val expandedPositions = mutableSetOf<Int>()

    fun submitList(newFAQs: List<FAQ>) {
        faqs = newFAQs
        notifyDataSetChanged()
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): FAQViewHolder {
        val binding = ItemFaqBinding.inflate(
            LayoutInflater.from(parent.context), parent, false
        )
        return FAQViewHolder(binding)
    }

    override fun onBindViewHolder(holder: FAQViewHolder, position: Int) {
        val faq = faqs[position]
        val isExpanded = expandedPositions.contains(position)

        holder.bind(faq, isExpanded) {
            if (isExpanded) {
                expandedPositions.remove(position)
            } else {
                expandedPositions.add(position)
            }
            notifyItemChanged(position)
            onFAQClick(faq)
        }
    }

    override fun getItemCount() = faqs.size
}

class FAQViewHolder(private val binding: ItemFaqBinding) :
    RecyclerView.ViewHolder(binding.root) {

    fun bind(faq: FAQ, isExpanded: Boolean, onClick: () -> Unit) {
        binding.questionText.text = faq.question
        binding.answerText.text = faq.answer
        binding.answerText.isVisible = isExpanded
        binding.expandIcon.rotation = if (isExpanded) 180f else 0f

        binding.root.setOnClickListener { onClick() }
    }
}
```

### Demo App Reference

For a complete working implementation, see:
- [FAQDemoActivity.kt](stringboot-android-demoApp/app/src/main/java/com/stringboot/android/FAQDemoActivity.kt) - Full FAQ screen with filtering
- Android Demo App → "FAQ Demo" section

### Performance & Caching

FAQProvider uses the same three-tier caching strategy as StringProvider:

1. **Memory Cache (L1)** - LRU cache for instant access
2. **Room Database (L2)** - Persistent offline storage
3. **Network (L3)** - Delta sync for changed FAQs only

**Cache Performance:**
- Memory cache hit: <5ms
- Database hit: <50ms
- Network (delta sync): Downloads only changed FAQs since last sync
- Full catalog: Only on first launch or cache invalidation

### Best Practices

**1. Use Reactive Flow for FAQ Screens**

```kotlin
// ✅ Good: Auto-updates when FAQs change
FAQProvider.getFAQsFlow(tag, subTags, lang)
    .onEach { faqs -> updateUI(faqs) }
    .launchIn(lifecycleScope)

// ❌ Avoid: Requires manual refresh
val faqs = FAQProvider.getFAQs(tag, subTags, lang)
```

**2. Organize Tags Hierarchically**

```kotlin
// ✅ Good: Specific tag + sub-tags
FAQProvider.getFAQs(
    tag = "identity_verification",
    subTags = listOf("document_upload", "liveness_check")
)

// ❌ Avoid: Generic tags that return too many FAQs
FAQProvider.getFAQs(tag = "general")
```

**3. Refresh on App Launch**

```kotlin
// In Application.onCreate()
applicationScope.launch {
    FAQProvider.refreshFromNetwork(StringProvider.deviceLocale())
}
```

**4. Handle Empty States**

```kotlin
val faqs = FAQProvider.getFAQs(tag = "payments")
if (faqs.isEmpty()) {
    // Show "No FAQs available" message or contact support button
    binding.emptyState.isVisible = true
}
```

### Troubleshooting

**FAQs not appearing:**
```kotlin
// 1. Check initialization
if (!FAQProvider.isInitialized()) {
    Log.e("FAQ", "FAQProvider not initialized!")
}

// 2. Enable logging
StringbootLogger.setLogLevel(LogLevel.DEBUG)

// 3. Check network sync
lifecycleScope.launch {
    val success = FAQProvider.refreshFromNetwork("en")
    Log.d("FAQ", "Network sync: $success")
}

// 4. Verify tag/subTag spelling
val faqs = FAQProvider.getFAQs(tag = "payments") // Case-sensitive!
```

**Language fallback not working:**
```kotlin
// Check actual language returned
val faqs = FAQProvider.getFAQs(tag = "payments", lang = "fr")
val actualLang = faqs.firstOrNull()?.languageCode
Log.d("FAQ", "Requested: fr, Got: $actualLang")
```

### API Reference

See [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md) for:
- FAQ cache management
- Custom cache sizes
- Network sync strategies
- Delta sync protocol details

---

## 📚 Additional Resources

- [API Reference](../docs/API_REFERENCE.md) - Complete API endpoint documentation
- [Implementation Examples](../docs/IMPLEMENTATION_EXAMPLES.md) - Real code from demo app
- [Delta Sync Protocol](../docs/DELTA_SYNC_PROTOCOL.md) - How sync works under the hood
- [Quick Start](../docs/QUICKSTART.md) - 5-minute setup guide
- [A/B Testing Integration](../AB_TESTING_CLIENT_SDK_INTEGRATION.md) - Complete A/B testing guide

---

## 🆘 Support

- 📧 Email: support@stringboot.com
- 🐛 Issues: [GitHub Issues](https://github.com/stringboot/android-sdk/issues)
- 📚 Docs: [docs.stringboot.com](https://docs.stringboot.com)

---

**Made with ❤️ by the Stringboot Team**

SDK Version: 1.2.0 | Last Updated: 2025-11 | A/B Testing Support Added
