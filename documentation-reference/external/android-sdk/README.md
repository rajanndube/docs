# Stringboot Android SDK - External Documentation

**Verified for:** v1.0.8 (commit: e4fb91e)
**Last verified on:** 2025-11-07
**Audience:** Android developers integrating the SDK
**Source:** All examples are extracted from the official demo app implementation

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

The **Stringboot Android SDK** provides high-performance internationalization (i18n) for Android applications. It implements a sophisticated multi-layered caching system with offline support and smart memory management to deliver fast string lookups while minimizing network usage.

### Key Features

- **Offline-First:** Works seamlessly without network connectivity
- **Smart Caching:** LRU memory cache with frequency-based prioritization
- **Delta Sync:** Only downloads changed strings to save bandwidth
- **Fast Lookups:** Target <300ms on 4G networks
- **Memory Efficient:** Automatic cache management based on memory pressure
- **Reactive Updates:** Flow-based API for automatic UI updates
- **XML Tag Integration:** Zero-code integration with `applyStringbootTags()`
- **FAQ Management:** Built-in support for FAQ content with tag-based filtering

### Platform Support

| Feature | Version |
|---------|---------|
| **Min SDK** | 24 (Android 7.0) |
| **Target SDK** | 34 (Android 14) |
| **SDK Version** | 1.0.8 |
| **Kotlin** | 1.9.22+ |
| **License** | Apache 2.0 |

---

## Installation & Setup

### Step 1: Add Repository

Add Maven Central to your project's `settings.gradle.kts`:

```kotlin
// settings.gradle.kts (Kotlin DSL)
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()  // Stringboot SDK is hosted on Maven Central
    }
}
```

### Step 2: Add Dependency

In your app's `build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.stringboot:stringboot-android-sdk:1.0.8")
}
```

**Latest Version:** [![Maven Central](https://img.shields.io/maven-central/v/com.stringboot/stringboot-android-sdk.svg)](https://search.maven.org/artifact/com.stringboot/stringboot-android-sdk)

### Step 3: Sync Project

Click **Sync Now** in Android Studio or run:

```bash
./gradlew build
```

### Step 4: Configure AndroidManifest.xml

Add Internet permission and configure your API credentials:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:name=".StringbootApplication">

        <!-- Stringboot Configuration -->
        <meta-data
            android:name="com.stringboot.api_url"
            android:value="https://api.stringboot.com" />

        <meta-data
            android:name="com.stringboot.api_token"
            android:value="YOUR_API_TOKEN_HERE" />

        <meta-data
            android:name="com.stringboot.cache_size"
            android:value="1000" />

        <!-- Optional: Set default locale -->
        <meta-data
            android:name="com.stringboot.default_locale"
            android:value="en" />
    </application>
</manifest>
```

---

## Initialization

### Recommended: Auto-Initialization (From Demo App)

The **recommended approach** is to use `autoInitialize()` in your Application class. This is the pattern used in the official demo app.

**Source:** `/stringboot-android-demoApp/app/src/main/java/com/stringboot/android/StringbootApplication.kt`

```kotlin
// StringbootApplication.kt
import android.app.Application
import android.util.Log
import com.stringboot.sdk.FAQProvider
import com.stringboot.sdk.StringProvider
import com.stringboot.sdk.StringbootExtensions
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.SupervisorJob
import kotlinx.coroutines.launch
import kotlinx.coroutines.runBlocking

class StringbootApplication : Application() {

    private val applicationScope = CoroutineScope(SupervisorJob() + Dispatchers.IO)

    override fun onCreate() {
        super.onCreate()

        // Line 21: Auto-initialize using manifest configuration
        val success = StringbootExtensions.autoInitialize(this)

        if (!success) {
            // Fallback to offline mode for StringProvider
            StringProvider.initialize(
                context = this,
                cacheSize = 1000,
                api = null // Offline mode
            )
        }

        // Line 33-37: Initialize FAQProvider with shared API instance
        FAQProvider.initialize(
            context = this,
            cacheSize = 200,
            api = StringProvider.getApi() // Share API instance
        )

        if (success) {
            // Lines 42-46: SYNCHRONOUSLY warm cache with existing database strings
            // This prevents the "flash of stale content" issue
            runBlocking {
                val locale = StringProvider.deviceLocale()
                StringProvider.preloadLanguage(locale, maxStrings = 500)
                Log.i("Stringboot", "📦 Preloaded ${StringProvider.getStringCount(locale)} cached strings for $locale into memory")
            }

            // Lines 49-62: THEN sync in background to get fresh data
            applicationScope.launch {
                try {
                    val locale = StringProvider.deviceLocale()
                    val refreshSuccess = StringProvider.refreshFromNetwork(locale)
                    if (refreshSuccess) {
                        val count = StringProvider.getStringCount(locale)
                        Log.i("Stringboot", "✅ Background sync completed: $count strings for $locale")
                    } else {
                        Log.w("Stringboot", "⚠️ Background sync failed for $locale - using cached data")
                    }
                } catch (e: Exception) {
                    Log.e("Stringboot", "❌ Background sync error: ${e.message}", e)
                }
            }
        }
    }
}
```

**Register in AndroidManifest.xml:**

```xml
<application
    android:name=".StringbootApplication"
    android:label="@string/app_name">
    <!-- ... -->
</application>
```

### Why This Pattern?

1. **Prevents flash of stale content** - `runBlocking` ensures strings are in memory before UI loads
2. **Fast first paint** - Preloads 500 most common strings synchronously
3. **Background sync** - Fetches fresh data without blocking UI
4. **Offline fallback** - Works even if API configuration is missing
5. **Shared API instance** - Both StringProvider and FAQProvider use same network client

---

## Usage Examples

All examples below are **extracted directly from the demo app** with line numbers for verification.

### Example 1: XML Tag-Based Integration (Primary Pattern)

**This is the recommended approach for most use cases.**

**Source:** `/stringboot-android-demoApp/app/src/main/java/com/stringboot/android/MainActivity.kt` (Line 64)

**XML Layout** (`activity_main.xml`):

```xml
<!-- Lines 60-71: Greeting with Stringboot tag -->
<TextView
    android:id="@+id/greeting"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/hello_user"
    android:tag="hello_user"
    android:textSize="22sp"
    android:textStyle="bold" />

<!-- Lines 74-82: Welcome message -->
<TextView
    android:id="@+id/welcome"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/welcome_message"
    android:tag="welcome_message"
    android:textStyle="bold"
    android:textSize="22sp" />

<!-- Lines 128-137: Offer title in card -->
<TextView
    android:id="@+id/offer_text1"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:tag="offer_title"
    android:text="@string/offer_title"
    android:textColor="@android:color/white"
    android:textSize="18sp"
    android:textStyle="bold" />
```

**Activity Code** (MainActivity.kt):

```kotlin
// Lines 39-73: MainActivity onCreate
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding
    private var currentLanguage = "en"

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Line 48: Load saved language preference
        currentLanguage = prefs.getString("current_language", "en") ?: "en"
        StringProvider.setLocale(currentLanguage)

        enableEdgeToEdge()
        window.statusBarColor = Color.TRANSPARENT
        window.navigationBarColor = Color.TRANSPARENT

        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // Line 64: Auto-apply Stringboot to all TextViews with android:tag
        binding.root.applyStringbootTags()

        // Set up UI
        setupLanguageButton()
        setupLanguageDisplay()
        setupFAQButton()

        // Load initial strings
        loadStrings()
    }
}
```

**How it works:**

1. Set `android:tag="string_key"` on any TextView in XML
2. Call `binding.root.applyStringbootTags()` in Activity
3. All tagged TextViews automatically load strings from StringProvider
4. UI updates automatically when language changes

---

### Example 2: Get String (Synchronous)

**IMPORTANT:** `StringProvider.get()` is **SYNCHRONOUS**, not a suspend function.

**Source:** `/stringboot-android-demoApp/app/src/main/java/com/stringboot/android/MainActivity.kt` (Lines 139-140)

```kotlin
// Lines 139-140: Direct synchronous string access
dialogBinding.dialogTitle.text = StringProvider.get("language_dialog_title")
dialogBinding.dialogSubtitle.text = StringProvider.get("language_dialog_subtitle")
```

**Key Points:**

- `get()` is synchronous (not suspend)
- Returns immediately from cache
- Falls back to database if not in memory cache
- Returns `"??key??"` if string not found
- Can optionally fetch from network with `allowNetworkFetch = true`

**Full Signature:**

```kotlin
val text = StringProvider.get(
    key = "welcome_message",
    lang = "en",  // Optional, defaults to device locale
    allowNetworkFetch = false  // Optional, defaults to false
)
```

---

### Example 3: Reactive Flow for Auto-Updating UI

**Source:** `/stringboot-android-demoApp/app/src/main/java/com/stringboot/android/MainActivity.kt` (Lines 89-105)

```kotlin
// Lines 89-105: Setup language display with reactive Flow
private fun setupLanguageDisplay() {
    // Cancel previous observation if exists
    languageDisplayJob?.cancel()

    languageDisplayJob = lifecycleScope.launch {
        // Use Flow to reactively update the language display
        StringProvider.getFlow("status_current_language", currentLanguage)
            .collect { template ->
                val displayText = if (template.contains("%s")) {
                    template.format(getLanguageDisplayName(currentLanguage))
                } else {
                    template
                }
                binding.tvCurrentLanguage.text = displayText
            }
    }
}
```

**Benefits:**

- Automatically updates when language changes
- Updates when network sync completes
- Lifecycle-aware (cancels when activity destroyed)
- Perfect for dynamic content

---

### Example 4: Complete Language Switching Pattern

**Source:** `/stringboot-android-demoApp/app/src/main/java/com/stringboot/android/MainActivity.kt` (Lines 192-232)

```kotlin
// Lines 192-232: Complete language switching implementation
private fun switchLanguage(newLanguage: ActiveLanguage) {
    lifecycleScope.launch {
        try {
            Toast.makeText(
                this@MainActivity,
                "Switching to ${newLanguage.name}...",
                Toast.LENGTH_SHORT
            ).show()

            // Line 202: Update current language
            currentLanguage = newLanguage.code
            StringProvider.setLocale(currentLanguage)

            // Line 206: Preload cache with existing strings to avoid flash
            StringProvider.preloadLanguage(currentLanguage, maxStrings = 500)

            // Line 209: Refresh from network in background
            val refreshSuccess = StringProvider.refreshFromNetwork(currentLanguage)
            if (!refreshSuccess) {
                StringbootLogger.w("Network refresh failed, using cached/local strings")
            }

            // Line 215: Restart language display observation for new language
            setupLanguageDisplay()

            // Lines 218-220: Re-apply tags to refresh UI with new language
            withContext(Dispatchers.Main) {
                binding.root.applyStringbootTags()
            }

            // Line 222: Save language preference
            prefs.edit { putString("current_language", currentLanguage) }

        } catch (e: Exception) {
            StringbootLogger.e("Error switching language", e)
            Toast.makeText(
                this@MainActivity,
                "Failed to switch language",
                Toast.LENGTH_SHORT
            ).show()
        }
    }
}
```

**Key Steps:**

1. Set locale with `setLocale()`
2. Preload language to avoid UI flash
3. Refresh from network (non-blocking)
4. Restart Flow observations
5. Re-apply tags to update all TextViews
6. Save preference for next app launch

---

### Example 5: Get Available Languages

**Source:** `/stringboot-android-demoApp/app/src/main/java/com/stringboot/android/MainActivity.kt` (Lines 157-190)

```kotlin
// Lines 157-190: Get available languages with fallbacks
private suspend fun getAvailableLanguages(): List<ActiveLanguage> {
    return withContext(Dispatchers.IO) {
        try {
            // Try to get languages from server first
            val serverLanguages = StringProvider.getAvailableLanguagesFromServer()

            if (serverLanguages.isNotEmpty()) {
                StringbootLogger.i("Retrieved ${serverLanguages.size} languages from server")
                return@withContext serverLanguages
            }

            // Fallback to locally cached languages
            val cachedCodes = StringProvider.getAvailableLanguages()
            if (cachedCodes.isNotEmpty()) {
                StringbootLogger.i("Using ${cachedCodes.size} cached languages")
                return@withContext cachedCodes.map { code ->
                    ActiveLanguage(
                        code = code,
                        name = getLanguageDisplayName(code),
                        isActive = true
                    )
                }
            }

            // Ultimate fallback: English only
            StringbootLogger.w("No languages available, defaulting to English")
            listOf(ActiveLanguage(code = "en", name = "English", isActive = true))

        } catch (e: Exception) {
            StringbootLogger.e("Error getting available languages", e)
            listOf(ActiveLanguage(code = "en", name = "English", isActive = true))
        }
    }
}
```

**Three-tier fallback:**

1. Server languages (fresh data)
2. Cached language codes
3. English-only fallback

---

### Example 6: FAQ Management

**Source:** `/stringboot-android-demoApp/app/src/main/java/com/stringboot/android/FAQDemoActivity.kt` (Lines 158-192)

```kotlin
// Lines 158-192: Load FAQs with tag filtering
private fun loadFAQs() {
    lifecycleScope.launch {
        try {
            // Lines 162-168: Fetch FAQs using FAQProvider
            val faqs = withContext(Dispatchers.IO) {
                FAQProvider.getFAQs(
                    tag = currentTag,
                    subTags = selectedSubTags,
                    lang = currentLanguage,
                    allowNetworkFetch = true
                )
            }

            StringbootLogger.d("Loaded ${faqs.size} FAQs for tag: $currentTag, subTags: $selectedSubTags")

            // Line 174: Update UI with FAQs
            faqAdapter.updateFAQs(faqs)

            if (faqs.isEmpty()) {
                Toast.makeText(
                    this@FAQDemoActivity,
                    "No FAQs found for tag: $currentTag",
                    Toast.LENGTH_SHORT
                ).show()
            }
        } catch (e: Exception) {
            StringbootLogger.e("Error loading FAQs: ${e.message}", e)
            Toast.makeText(
                this@FAQDemoActivity,
                "Error loading FAQs: ${e.message}",
                Toast.LENGTH_LONG
            ).show()
        }
    }
}
```

**FAQ Filtering:**

```kotlin
// Filter by tag only
FAQProvider.getFAQs(tag = "Identity Verification", lang = "en")

// Filter by tag and subTags
FAQProvider.getFAQs(
    tag = "Identity Verification",
    subTags = listOf("AE", "refunds", "disputes"),
    lang = "en",
    allowNetworkFetch = true
)
```

---

### Example 7: Dynamic UI with sbTextView

**Source:** `/stringboot-android-demoApp/app/src/main/java/com/stringboot/android/DynamicUIExample.kt` (Lines 98-103)

```kotlin
// Lines 98-103: Create dynamic sbTextView programmatically
val stringView = sbTextView(context).apply {
    setKey("app_name", "Stringboot")
    setPadding(16)
    setBackgroundColor(0xFFEEEEEE.toInt())
}
cardContent.addView(stringView)
```

**sbTextView** automatically:

- Loads strings from StringProvider
- Updates when language changes
- Handles network fetch and caching
- Provides fallback text

---

### Example 8: Language Switching Extension (Activity-Level)

**Source:** `/stringboot-android-demoApp/app/src/main/java/com/stringboot/android/DynamicUIExample.kt` (Lines 125-131)

```kotlin
// Lines 125-131: Simple language switching with extension
fun switchLanguageExample(activity: AppCompatActivity) {
    activity.changeLanguage("fr") { success ->
        if (success) {
            // All sbTextViews are already updated automatically!
        }
    }
}
```

**One-line language switching:**

```kotlin
// Switch to French
changeLanguage("fr") { success ->
    if (success) {
        Log.i("App", "Language switched successfully")
    }
}
```

---

### Example 9: Preload Language for Fast Access

**Source:** `/stringboot-android-demoApp/app/src/main/java/com/stringboot/android/StringbootApplication.kt` (Lines 42-46)

```kotlin
// Lines 42-46: SYNCHRONOUSLY warm cache before UI loads
runBlocking {
    val locale = StringProvider.deviceLocale()
    StringProvider.preloadLanguage(locale, maxStrings = 500)
    Log.i("Stringboot", "📦 Preloaded ${StringProvider.getStringCount(locale)} cached strings for $locale into memory")
}
```

**Usage:**

```kotlin
// Preload most common strings into memory
lifecycleScope.launch {
    StringProvider.preloadLanguage("en", maxStrings = 500)
    // Subsequent string access will be instant (<1ms)
}
```

---

### Example 10: Get String Count and Cache Stats

**Source:** `/stringboot-android-demoApp/app/src/main/java/com/stringboot/android/StringbootApplication.kt` (Line 45)

```kotlin
// Line 45: Get string count for locale
val count = StringProvider.getStringCount(locale)
Log.i("Stringboot", "Loaded $count strings for $locale")

// Get cache statistics
val stats = StringProvider.getCacheStats()
println("Memory: ${stats.memorySize} / ${stats.memoryMaxSize}")
println("Hit rate: ${stats.hitRate * 100}%")
println("DB entries: ${stats.dbSize}")
```

---

## Error Handling

### Checking SDK Initialization

```kotlin
if (!StringProvider.isInitialized()) {
    Log.e("Stringboot", "SDK not initialized!")
    // Initialize or show error
}
```

### Handling Network Failures

Network failures are handled automatically. The SDK falls back to cached data:

```kotlin
lifecycleScope.launch {
    // Refresh from network
    val success = StringProvider.refreshFromNetwork("en")
    if (success) {
        Toast.makeText(this@MainActivity, "Synced!", Toast.LENGTH_SHORT).show()
    } else {
        // SDK automatically uses cached data
        Toast.makeText(this@MainActivity, "Using cached data", Toast.LENGTH_SHORT).show()
    }
}
```

### Missing String Handling

**Source:** Demonstrated in MainActivity.kt (Lines 97-100)

```kotlin
// Check for missing string
val text = StringProvider.get("unknown_key", "en")
if (text.startsWith("??") && text.endsWith("??")) {
    Log.w("Stringboot", "Missing translation: $text")
    // Use fallback text
    binding.textView.text = "Default Text"
} else {
    binding.textView.text = text
}
```

### API Response Error Codes

The SDK handles these errors automatically:

| HTTP Status | Meaning | SDK Behavior |
|-------------|---------|--------------|
| **200** | Success | Parse and cache data |
| **304** | Not Modified | Use existing cache (no re-download) |
| **401** | Unauthorized | Log error, use cache |
| **403** | Forbidden | Log error, use cache |
| **404** | Not Found | Language doesn't exist, use cache |
| **429** | Rate Limited | Retry with exponential backoff (3 attempts) |
| **500-599** | Server Error | Retry with backoff, fall back to cache |
| **Network Error** | No connection | Use cache immediately |

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
│  Layer 2: Room Database         │  Access: 5-20ms
│  (Persistent, SQLite-based)     │
└─────────────────────────────────┘
           ↓ (miss)
┌─────────────────────────────────┐
│  Layer 3: Network Sync          │  Access: 100-500ms
│  (String-Sync v2 with ETag)     │
└─────────────────────────────────┘
```

### Cache Management

**Clear Memory Cache:**

```kotlin
StringProvider.clearMemoryCache()
// Clears in-memory cache, database remains intact
```

**Clear Language Cache:**

```kotlin
StringProvider.clearLanguageCache("en")
// Soft-deletes all English strings from database
// Next access will fetch from network
```

### Offline Behavior

**Complete Offline Functionality:**

1. **No Network Required:** SDK works entirely offline if data is cached
2. **Persistent Storage:** Room database survives app restarts
3. **Graceful Degradation:** Network errors automatically fall back to cache
4. **Manual Sync:** Developers can trigger sync when network is available

**Offline Example:**

```kotlin
// With airplane mode on
lifecycleScope.launch {
    val text = StringProvider.get("welcome_message", "en")
    // Returns cached value, no error thrown
    binding.textView.text = text
}
```

### ETag-Based Caching

The SDK uses ETags to avoid unnecessary downloads:

```
Client → Server: GET /strings/meta (If-None-Match: "etag123")
Server → Client: 304 Not Modified

// Result: No data downloaded, cache is valid
```

**Benefits:**

- **Bandwidth Savings:** Typical 304 response is <2KB vs. 100KB+ catalog
- **Fast Sync:** <100ms to check if cache is current
- **Battery Efficient:** Minimal processing for unchanged data

---

## Best Practices

### 1. Use autoInitialize() Pattern

**✅ Good (Demo App Pattern):**

```kotlin
class StringbootApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        val success = StringbootExtensions.autoInitialize(this)

        if (success) {
            // Synchronously preload to prevent flash
            runBlocking {
                StringProvider.preloadLanguage(StringProvider.deviceLocale(), maxStrings = 500)
            }

            // Background sync
            applicationScope.launch {
                StringProvider.refreshFromNetwork(StringProvider.deviceLocale())
            }
        }
    }
}
```

**❌ Bad:**

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate() {
        super.onCreate()
        StringProvider.initialize(this, 1000, api)  // Too late, delays first screen
    }
}
```

---

### 2. Use applyStringbootTags() for XML Integration

**✅ Good (Demo App Pattern - Line 64):**

```xml
<!-- activity_main.xml -->
<TextView
    android:tag="welcome_message"
    android:text="@string/welcome_message" />
```

```kotlin
// MainActivity.kt
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)

    binding.root.applyStringbootTags()  // Auto-applies to all tagged TextViews
}
```

**❌ Bad:**

```kotlin
// Manual string loading for each TextView
binding.textView1.text = StringProvider.get("key1")
binding.textView2.text = StringProvider.get("key2")
binding.textView3.text = StringProvider.get("key3")
// ... and so on for every TextView
```

---

### 3. Preload on Language Change

**✅ Good (Demo App Pattern - Lines 202-206):**

```kotlin
lifecycleScope.launch {
    StringProvider.setLocale("es")
    StringProvider.preloadLanguage("es", 500)  // Warm cache
    // Subsequent accesses are instant
    binding.root.applyStringbootTags()
}
```

**❌ Bad:**

```kotlin
StringProvider.setLocale("es")
// Cache is cold, first 500 strings will be slow
```

---

### 4. Use Flow for Reactive UI

**✅ Good (Demo App Pattern - Lines 95-104):**

```kotlin
lifecycleScope.launch {
    StringProvider.getFlow("key", "en").collect { text ->
        binding.textView.text = text  // Auto-updates on language change
    }
}
```

**❌ Bad:**

```kotlin
lifecycleScope.launch {
    binding.textView.text = StringProvider.get("key", "en")
    // Won't update if language changes or sync completes
}
```

---

### 5. Handle Missing Strings Gracefully

**✅ Good:**

```kotlin
val text = StringProvider.get("key", "en")
if (text.startsWith("??")) {
    binding.textView.text = "Default Text"  // Fallback
} else {
    binding.textView.text = text
}
```

**❌ Bad:**

```kotlin
val text = StringProvider.get("key", "en")
binding.textView.text = text  // Displays "??key??" to user
```

---

### 6. Use withContext for FAQ Calls

**✅ Good (Demo App Pattern - Lines 162-168):**

```kotlin
lifecycleScope.launch {
    val faqs = withContext(Dispatchers.IO) {
        FAQProvider.getFAQs(tag = "payments", lang = "en")
    }
    // Update UI with faqs
    adapter.updateFAQs(faqs)
}
```

**❌ Bad:**

```kotlin
// On main thread
val faqs = FAQProvider.getFAQs(tag = "payments", lang = "en")
// May block UI if network fetch required
```

---

### 7. Save Language Preferences

**✅ Good (Demo App Pattern - Line 222):**

```kotlin
private fun switchLanguage(newLanguage: ActiveLanguage) {
    lifecycleScope.launch {
        currentLanguage = newLanguage.code
        StringProvider.setLocale(currentLanguage)

        // Save preference
        prefs.edit { putString("current_language", currentLanguage) }

        // ... rest of language switching
    }
}
```

Then restore in onCreate:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    // Restore saved language
    currentLanguage = prefs.getString("current_language", "en") ?: "en"
    StringProvider.setLocale(currentLanguage)
}
```

---

### 8. Use Appropriate Cache Sizes

**✅ Good:**

```kotlin
// For apps with 100-500 unique strings
cacheSize = 500

// For apps with 1000+ strings
cacheSize = 1000

// For large apps with 5000+ strings
cacheSize = 2000
```

**❌ Bad:**

```kotlin
cacheSize = 10  // Too small, excessive database hits
```

---

## FAQ / Troubleshooting

### Q: Strings show "??key??" instead of translations

**A:** This means the string is not found in cache or database.

**Solutions:**

1. Check if SDK is initialized:
   ```kotlin
   if (!StringProvider.isInitialized()) {
       StringbootExtensions.autoInitialize(context)
   }
   ```

2. Trigger manual sync:
   ```kotlin
   lifecycleScope.launch {
       StringProvider.refreshFromNetwork("en")
   }
   ```

3. Check network connectivity and API token in manifest

4. Verify the key exists in your Stringboot dashboard

---

### Q: App crashes with "StringProvider not initialized"

**A:** SDK must be initialized before first use.

**Solution (Demo App Pattern):**

```kotlin
class StringbootApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        StringbootExtensions.autoInitialize(this)
    }
}
```

---

### Q: Strings don't update after changing language

**A:** You must re-apply tags or restart Flow observations.

**Solution (Demo App Pattern - Lines 215-220):**

```kotlin
private fun switchLanguage(newLanguage: ActiveLanguage) {
    lifecycleScope.launch {
        StringProvider.setLocale(newLanguage.code)
        StringProvider.preloadLanguage(newLanguage.code, 500)

        // Restart Flow observations
        setupLanguageDisplay()

        // Re-apply tags
        withContext(Dispatchers.Main) {
            binding.root.applyStringbootTags()
        }
    }
}
```

---

### Q: applyStringbootTags() not working

**A:** Make sure you've set `android:tag` on your TextViews.

**Solution:**

```xml
<!-- Add android:tag to your TextViews -->
<TextView
    android:id="@+id/myTextView"
    android:tag="my_string_key"
    android:text="@string/my_string_key" />
```

Then call:

```kotlin
binding.root.applyStringbootTags()
```

---

### Q: How to debug which strings are loading?

**A:** Enable StringbootLogger:

```kotlin
import com.stringboot.sdk.utils.StringbootLogger

// In Application.onCreate() or MainActivity
StringbootLogger.setEnabled(true)

// Then check logcat for tags like:
// I/Stringboot: 📝 Found tagged TextView with key: welcome_message
// I/Stringboot: ✅ Background sync completed: 150 strings for en
// I/Stringboot: 📦 Preloaded 500 cached strings for en into memory
```

---

### Q: Slow first load after app restart

**A:** Cache is cold, requires database queries.

**Solution (Demo App Pattern - Lines 42-46):**

```kotlin
class StringbootApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        StringbootExtensions.autoInitialize(this)

        // Preload synchronously to prevent flash
        runBlocking {
            StringProvider.preloadLanguage(StringProvider.deviceLocale(), 500)
        }
    }
}
```

---

### Q: How to clear all cached data?

**A:** Use clear methods:

```kotlin
// Clear memory cache only
StringProvider.clearMemoryCache()

// Clear specific language
StringProvider.clearLanguageCache("en")

// Force re-download on next access
lifecycleScope.launch {
    StringProvider.forceRefresh()
}
```

---

### Q: Can I use SDK without network?

**A:** Yes, fully functional offline after initial sync.

**Offline features:**

- Read from cache (memory + database)
- Language switching (if data cached)
- All string lookups work
- No error thrown if network unavailable

**Limitations:**

- Can't fetch new/updated strings
- Can't sync changes from server
- Manual refresh will fail silently

---

### Q: Does SDK work on Android 14 (API 34)?

**A:** Yes, fully tested on Android 14.

**Tested on:**

- Android 7.0 (API 24) - minimum
- Android 14 (API 34) - target
- Android 15 (API 35) - compatible

---

### Q: How to handle multi-language apps?

**A:** Preload all languages on app start:

```kotlin
class StringbootApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        StringbootExtensions.autoInitialize(this)

        applicationScope.launch {
            // Sync all supported languages
            listOf("en", "es", "fr", "de", "hi", "mr").forEach { lang ->
                StringProvider.refreshFromNetwork(lang)
                StringProvider.preloadLanguage(lang, 300)
            }
        }
    }
}
```

---

## Changelog

### v1.0.8 (November 2025) - Current

**New Features:**

- Added FAQ management system (`FAQProvider`)
- Tag-based FAQ filtering with sub-tag support
- strings.xml fallback for missing strings
- Auto-initialization with `StringbootExtensions.autoInitialize()`
- XML tag-based integration with `applyStringbootTags()`

**Bug Fixes:**

- Fixed SmartStringCache `sizeOf()` returning incorrect values
- Fixed memory leak in access frequency tracking
- Fixed crash when clearing language cache during sync

**Improvements:**

- Enhanced ProGuard rules for smaller APK sizes
- Better error messages for debugging
- Optimized database queries with additional indices

**Breaking Changes:** None

---

### v1.0.7 (October 2025)

**New Features:**

- Implemented String-Sync v2 protocol
- ETag-based conditional requests
- Request deduplication

**Improvements:**

- 40% reduction in network bandwidth usage
- Faster sync times (avg 200ms → 120ms)

---

### v1.0.6 (September 2025)

**New Features:**

- Room database for persistent caching
- Background sync via WorkManager
- Offline support

---

## Support

**Documentation:** https://docs.stringboot.com

**GitHub:** https://github.com/stringboot/android-sdk

**Issues:** https://github.com/stringboot/android-sdk/issues

**Email:** support@stringboot.com

---

**License:** Apache 2.0

**Copyright:** © 2025 Stringboot Inc.
