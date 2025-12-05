# Stringboot Android SDK - Frequently Asked Questions

> Common questions and answers about using the Stringboot Android SDK

**Last Updated:** December 2024 | **SDK Version:** 1.2.0+

---

## Table of Contents

- [Getting Started](#getting-started)
- [Installation & Setup](#installation--setup)
- [String Management](#string-management)
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

### Why should I use Stringboot instead of Android string resources?

**Stringboot offers several advantages:**

- **Remote Updates** - Update strings without releasing a new app version
- **Instant Fixes** - Fix typos or translations immediately
- **A/B Testing** - Test different copy variations with real users
- **Real-time Localization** - Add new languages on the fly
- **Offline-First** - Works perfectly without internet connection
- **Analytics Integration** - Track which string variations perform best
- **Centralized Management** - Manage all app strings from one dashboard

### Can I use Stringboot alongside Android string resources?

Yes! Many apps use Stringboot for dynamic content (marketing copy, feature descriptions, CTAs) while keeping critical UI strings in Android resources as fallbacks. This hybrid approach gives you flexibility while maintaining stability.

### What are the minimum requirements?

- **Min SDK:** 24 (Android 7.0)
- **Target SDK:** 34+
- **Kotlin:** 1.9.0+
- **Gradle:** 8.0+
- **Dependencies:** Room, Retrofit, Coroutines, Compose (optional)

---

## Installation & Setup

### How do I install the SDK?

Add the dependency to your `build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.stringboot:stringboot-android-sdk:1.2.0")
}
```

See [QUICKSTART.md](QUICKSTART.md) for complete installation instructions.

### Do I need to initialize the SDK in every Activity?

**No!** Initialize once in your `Application` class:

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        // Initialize once
        StringbootExtensions.autoInitialize(this)
    }
}
```

The SDK remains initialized throughout your app's lifecycle.

### Where do I get my API token?

1. Sign up at [stringboot.com](https://stringboot.com)
2. Create a new project
3. Navigate to **Settings** → **API Keys**
4. Copy your Android API token
5. Add it to `stringboot-config.json` in your `assets` folder

**Never commit your API token to version control!** Use BuildConfig or environment variables for production.

### Can I use Stringboot without network access?

Yes! Stringboot is **offline-first**. Once strings are cached locally (Room database), the app works perfectly without internet. Network is only needed for initial sync and updates.

### How do I configure the SDK?

Create `assets/stringboot-config.json`:

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

```kotlin
StringProvider.initialize(
    context = this,
    cacheSize = 1000,
    api = StringbootApi(apiToken = "YOUR_TOKEN")
)
```

---

## String Management

### How do I get a string?

**Simple async retrieval:**

```kotlin
lifecycleScope.launch {
    val text = StringProvider.get("welcome_message", "en")
    textView.text = text
}
```

**XML tags (auto-updating):**

```xml
<TextView
    android:id="@+id/welcomeText"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:stringboot_key="welcome_message"
    app:stringboot_lang="en" />
```

### How do I make strings update automatically in my UI?

**Use Kotlin Flow:**

```kotlin
StringProvider.getFlow("welcome_message", "en")
    .onEach { text ->
        binding.textView.text = text
    }
    .launchIn(lifecycleScope)
```

**Or use XML tags with `applyStringbootTags()`:**

```kotlin
binding.root.applyStringbootTags()
```

Both methods automatically update the UI when strings change.

### What happens if a string key doesn't exist?

The SDK returns `"??key??"` (e.g., `"??welcome_message??"`). This makes missing strings obvious during development.

**Best practice:** Always test with all language variants before release.

### Can I pass parameters to strings?

Not directly through Stringboot. Use standard Kotlin string formatting:

```kotlin
val template = StringProvider.get("welcome_user", "en") // "Welcome, %s!"
val message = template.format(userName)
```

Or use placeholders:

```kotlin
val template = StringProvider.get("items_count", "en") // "You have {count} items"
val message = template.replace("{count}", count.toString())
```

### How do I bulk-fetch multiple strings?

Currently, fetch strings individually but use coroutines for parallel execution:

```kotlin
val (title, subtitle, cta) = withContext(Dispatchers.IO) {
    awaitAll(
        async { StringProvider.get("title", "en") },
        async { StringProvider.get("subtitle", "en") },
        async { StringProvider.get("cta", "en") }
    )
}
```

---

## Language Switching

### How do I switch languages?

```kotlin
lifecycleScope.launch {
    StringProvider.switchLanguage(context, "es")
    // UI with Flow/XML tags automatically updates
}
```

### How do I get the list of available languages?

```kotlin
lifecycleScope.launch {
    val languages = StringProvider.getAvailableLanguages()
    languages.forEach { lang ->
        println("${lang.code}: ${lang.name} (Default: ${lang.isDefault})")
    }
}
```

### Do I need to manually refresh the UI after language change?

**No!** If you're using:
- **Kotlin Flow** (`getFlow()`) - UI updates automatically
- **XML tags** with `applyStringbootTags()` - UI updates automatically
- **Manual calls** to `StringProvider.get()` - You need to re-fetch

### Can users switch languages independently of device settings?

Yes! Stringboot language is independent of Android system language. You control it programmatically:

```kotlin
// Switch to Spanish regardless of device language
StringProvider.switchLanguage(context, "es")
```

### How do I detect the device language?

```kotlin
val deviceLang = StringProvider.deviceLocale()
println("Device language: $deviceLang") // e.g., "en", "es", "fr"
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
2. **Room Database (L2)** - Persistent local storage (<50ms)
3. **Network (L3)** - API calls with ETag-based caching

### How much data does Stringboot cache?

By default, **1,000 strings** in memory. Database stores unlimited strings. Typical usage:
- 1,000 strings ≈ 200-500 KB in memory
- 5,000 strings ≈ 1-2 MB in database

### Can I change the cache size?

Yes, during initialization:

```kotlin
StringProvider.initialize(
    context = this,
    cacheSize = 2000, // Cache 2000 strings in memory
    api = stringbootApi
)
```

### How often does the SDK sync with the network?

- **Initial sync:** On first app launch
- **Auto-sync:** When app comes to foreground (if 30+ minutes since last sync)
- **Manual sync:** Call `StringProvider.refreshFromNetwork()`

Network requests use ETag caching, so if nothing changed, only ~2 KB is downloaded.

### Does Stringboot impact app startup time?

**No.** Initialization is asynchronous and non-blocking:
- Cache initialization: <10ms
- Database setup: <50ms
- Network sync: Runs in background, doesn't block UI

### Can I preload languages for better performance?

Yes! Preload frequently used languages:

```kotlin
lifecycleScope.launch {
    StringProvider.preloadLanguage("en", maxStrings = 500)
    StringProvider.preloadLanguage("es", maxStrings = 500)
}
```

### How do I clear the cache?

```kotlin
// Clear cache for specific language
StringProvider.clearCache("en")

// Clear all cache
StringProvider.clearAllCache()
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

```kotlin
StringProvider.initialize(
    context = this,
    cacheSize = 1000,
    api = stringbootApi,
    analyticsHandler = MyAnalyticsHandler()
)
```

4. **Track events** in your analytics handler:

```kotlin
class MyAnalyticsHandler : StringbootAnalyticsHandler {
    override fun onExperimentAssigned(assignment: ExperimentAssignment) {
        Firebase.analytics.logEvent("experiment_assigned") {
            param("experiment_id", assignment.experimentId)
            param("variant_name", assignment.variantName)
        }
    }
}
```

The SDK handles the rest automatically!

### Do I need to change my code for A/B tests?

**No!** Continue using strings normally:

```kotlin
val ctaText = StringProvider.get("cta_button", "en")
```

The SDK automatically returns the correct variant based on the user's assignment.

### How does device ID work for A/B testing?

The SDK generates a persistent device ID (stored in DataStore) to ensure consistent experiment assignments across app sessions.

**Provide your own device ID:**

```kotlin
StringProvider.initialize(
    context = this,
    cacheSize = 1000,
    api = stringbootApi,
    providedDeviceId = "user_12345" // Your user ID
)
```

### Can I test experiments locally?

Yes! Check current assignments:

```kotlin
lifecycleScope.launch {
    val experiments = StringProvider.getActiveExperiments()
    experiments.forEach { exp ->
        println("Experiment: ${exp.experimentId}, Variant: ${exp.variantName}")
    }
}
```

---

## FAQ Provider

### What is the FAQ Provider?

FAQ Provider lets you deliver dynamic, multilingual FAQs to your app users using the same offline-first architecture as string resources. FAQs are organized by tags and sub-tags.

See [INTEGRATION_GUIDE.md#faq-provider](INTEGRATION_GUIDE.md#-faq-provider-integration) for complete guide.

### How do I fetch FAQs?

```kotlin
lifecycleScope.launch {
    val faqs = FAQProvider.getFAQs(
        tag = "payments",
        subTags = listOf("refunds"),
        lang = "en",
        allowNetworkFetch = true
    )

    displayFAQs(faqs)
}
```

### How do I make FAQs update automatically?

Use reactive Flow:

```kotlin
FAQProvider.getFAQsFlow(
    tag = "payments",
    subTags = listOf("refunds"),
    lang = "en"
).onEach { faqs ->
    adapter.updateFAQs(faqs)
}.launchIn(lifecycleScope)
```

### What are tags and sub-tags?

**Tags** organize FAQs into categories. **Sub-tags** provide finer filtering:

- **Tag:** "payments"
- **Sub-tags:** ["refunds", "disputes", "methods"]

```kotlin
// Get all payment FAQs
val allPaymentFAQs = FAQProvider.getFAQs(tag = "payments")

// Get only refund FAQs
val refundFAQs = FAQProvider.getFAQs(
    tag = "payments",
    subTags = listOf("refunds")
)
```

### Do FAQs support the same language fallback as strings?

Yes! If FAQs aren't available in the requested language, they automatically fall back to English.

### Can I cache FAQs?

Yes! FAQProvider uses the same three-tier caching system as StringProvider. FAQs are cached in memory and Room database for offline access.

---

## Offline Support

### Does Stringboot work offline?

**Yes, 100% offline functionality!** Once strings are synced, the app works perfectly without internet:
- Strings are cached in Room database
- Language switching works offline
- A/B test assignments are cached
- FAQs work offline

### What happens on first app install without internet?

The app won't have strings until it can sync with the network. **Best practice:** Include critical strings as fallbacks in Android resources.

### How do I know if strings are cached?

```kotlin
lifecycleScope.launch {
    val count = StringProvider.getCachedStringCount("en")
    println("Cached strings: $count")
}
```

### Can I force a network refresh?

Yes:

```kotlin
lifecycleScope.launch {
    val success = StringProvider.refreshFromNetwork("en")
    if (success) {
        println("Strings refreshed!")
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
4. **Not initialized** - Ensure `StringProvider.initialize()` was called

**Debug:**

```kotlin
// Enable debug logging
StringbootLogger.setLogLevel(LogLevel.DEBUG)

// Check if initialized
if (!StringProvider.isInitialized()) {
    Log.e("Stringboot", "Not initialized!")
}
```

### Strings not updating after changing language

**Possible causes:**

1. **Not using reactive approach** - Use `getFlow()` or XML tags for auto-updates
2. **Cache not refreshed** - Call `refreshFromNetwork()` after language change
3. **Language not synced** - Ensure new language is downloaded

**Solution:**

```kotlin
lifecycleScope.launch {
    StringProvider.switchLanguage(context, "es")
    StringProvider.refreshFromNetwork("es")
}
```

### Network sync failing

**Check these:**

1. **API token valid?** - Verify token in Stringboot dashboard
2. **Internet permission?** - Ensure `INTERNET` permission in manifest
3. **Network available?** - Check device internet connection
4. **Base URL correct?** - Should be `https://api.stringboot.com`

**Debug network calls:**

```kotlin
// Check last sync time
val lastSync = dataStore.getLastSyncTimestamp("en")
println("Last sync: $lastSync")

// Manually refresh
val success = StringProvider.refreshFromNetwork("en")
println("Refresh success: $success")
```

### App crashing on initialization

**Common causes:**

1. **Calling StringProvider before initialization**
2. **Invalid API token format**
3. **Missing dependencies** (Room, Retrofit, Coroutines)

**Solution:**

```kotlin
// Always initialize in Application.onCreate()
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        try {
            StringbootExtensions.autoInitialize(this)
        } catch (e: Exception) {
            Log.e("Stringboot", "Initialization failed", e)
        }
    }
}
```

### Slow string retrieval

**Possible causes:**

1. **Network fetch on main thread** - Always use coroutines
2. **Cache miss** - Preload frequently used languages
3. **Large cache size** - Reduce cache size if memory constrained

**Optimization:**

```kotlin
// ✅ Good: Background fetch
lifecycleScope.launch {
    val text = withContext(Dispatchers.IO) {
        StringProvider.get("key", "en")
    }
}

// ❌ Bad: Main thread (blocks UI)
runBlocking {
    val text = StringProvider.get("key", "en")
}
```

### Memory warnings related to Stringboot

**Solutions:**

1. **Reduce cache size:**
```kotlin
StringProvider.initialize(
    context = this,
    cacheSize = 500, // Smaller cache
    api = stringbootApi
)
```

2. **Clear cache periodically:**
```kotlin
// On low memory
override fun onLowMemory() {
    super.onLowMemory()
    StringProvider.clearMemoryCache()
}
```

### FAQs not appearing

**Debug steps:**

1. **Check initialization:**
```kotlin
if (!FAQProvider.isInitialized()) {
    Log.e("FAQ", "FAQProvider not initialized!")
}
```

2. **Verify tag spelling** (case-sensitive!):
```kotlin
val faqs = FAQProvider.getFAQs(tag = "payments") // Correct
val faqs = FAQProvider.getFAQs(tag = "Payments") // Wrong if tag is lowercase
```

3. **Enable network fetch:**
```kotlin
val faqs = FAQProvider.getFAQs(
    tag = "payments",
    allowNetworkFetch = true // Enable network fallback
)
```

---

## Best Practices

### When should I use Stringboot vs Android resources?

**Use Stringboot for:**
- Marketing copy that changes frequently
- A/B tested content
- Seasonal messaging
- User-facing content you want to update quickly
- Multi-region content variations

**Use Android resources for:**
- System-level strings (Settings, Permissions)
- Critical error messages
- Fallback strings for offline-first launch
- Strings unlikely to change

### How do I handle string updates in production?

**Recommended workflow:**

1. **Update strings** in Stringboot dashboard
2. **Test in staging** environment first
3. **Publish to production**
4. **Monitor analytics** for issues
5. **Rollback if needed** (instant via dashboard)

No app update required!

### Should I cache strings indefinitely?

Yes! The SDK automatically invalidates cache when strings are updated on the backend (using ETags). Trust the SDK's cache management.

### How do I test different languages?

**Method 1: Device language**
```kotlin
val deviceLang = StringProvider.deviceLocale()
StringProvider.switchLanguage(context, deviceLang)
```

**Method 2: Debug menu**
```kotlin
// Add debug language switcher in dev builds
if (BuildConfig.DEBUG) {
    showLanguageSwitcher()
}
```

**Method 3: ADB**
```bash
# Change device language via ADB
adb shell am start -a android.settings.LOCALE_SETTINGS
```

### How do I handle pluralization?

Use separate keys for each plural form:

```kotlin
val count = 5
val key = when {
    count == 0 -> "items_zero"
    count == 1 -> "items_one"
    else -> "items_many"
}
val text = StringProvider.get(key, "en").replace("{count}", count.toString())
```

Or manage plurals on the backend with dynamic placeholders.

### Should I add Proguard rules?

Yes, add these to `proguard-rules.pro`:

```proguard
# Stringboot SDK
-keep class com.stringboot.sdk.** { *; }
-keepclassmembers class com.stringboot.sdk.** { *; }

# Room
-keep class * extends androidx.room.RoomDatabase
-keep @androidx.room.Entity class *
```

### How do I migrate from Android resources to Stringboot?

**Gradual migration approach:**

1. **Phase 1:** Keep existing strings, add Stringboot for new features
2. **Phase 2:** Migrate high-frequency strings (marketing, CTAs)
3. **Phase 3:** Move remaining strings, keep critical ones as fallbacks
4. **Phase 4:** Monitor for issues, adjust as needed

**Helper function:**

```kotlin
suspend fun getStringWithFallback(key: String, @StringRes fallbackRes: Int): String {
    val stringbootValue = StringProvider.get(key, "en")
    return if (stringbootValue.startsWith("??")) {
        // Stringboot key not found, use Android resource
        getString(fallbackRes)
    } else {
        stringbootValue
    }
}
```

---

## Still Have Questions?

- 📧 **Email:** support@stringboot.com
- 📚 **Documentation:** [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)
- 🐛 **Report Issues:** [GitHub Issues](https://github.com/stringboot/android-sdk/issues)
- 💬 **Community:** [Discord](https://discord.gg/stringboot)

---

**Last Updated:** December 2024 | **SDK Version:** 1.2.0+
