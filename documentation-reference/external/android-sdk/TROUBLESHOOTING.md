# Troubleshooting - Android SDK

> Common issues and solutions for the Stringboot Android SDK

**Last Updated:** December 2024 | **SDK Version:** 1.2.0+

---

## Table of Contents

- [Installation Issues](#installation-issues)
- [Initialization Problems](#initialization-problems)
- [String Display Issues](#string-display-issues)
- [Language Switching](#language-switching)
- [Network & Sync Issues](#network--sync-issues)
- [A/B Testing Issues](#ab-testing-issues)
- [FAQ Provider Issues](#faq-provider-issues)
- [Performance Issues](#performance-issues)
- [Build & Compilation Errors](#build--compilation-errors)
- [Runtime Crashes](#runtime-crashes)

---

## Installation Issues

### Dependency Resolution Failed

**Problem:** Gradle can't resolve Stringboot SDK dependency

```
Could not resolve com.stringboot:stringboot-android-sdk:1.2.0
```

**Solutions:**

1. **Check repository configuration:**
```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        // Add if using private repository
        maven { url = uri("https://maven.stringboot.com/releases") }
    }
}
```

2. **Verify version exists:**
   - Check [releases page](https://github.com/stringboot/android-sdk/releases)
   - Try latest stable version

3. **Clear Gradle cache:**
```bash
./gradlew clean
./gradlew --refresh-dependencies
```

### Min SDK Version Conflict

**Problem:** App minSdk is lower than SDK requirement

```
Manifest merger failed : uses-sdk:minSdkVersion 21 cannot be smaller than version 24
```

**Solution:**

Update your app's `build.gradle.kts`:
```kotlin
android {
    defaultConfig {
        minSdk = 24  // Stringboot requires API 24+
    }
}
```

### Room Schema Export Error

**Problem:** Room schema export directory not found

**Solution:**

Add schema export directory:
```kotlin
android {
    defaultConfig {
        javaCompileOptions {
            annotationProcessorOptions {
                arguments += mapOf(
                    "room.schemaLocation" to "$projectDir/schemas"
                )
            }
        }
    }
}
```

---

## Initialization Problems

### SDK Not Initialized

**Problem:** Calling StringProvider before initialization

```kotlin
val text = StringProvider.get("key", "en") // Crash!
```

**Error:**
```
java.lang.IllegalStateException: StringProvider not initialized
```

**Solution:**

Initialize in `Application.onCreate()`:
```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        // Initialize before any StringProvider calls
        StringbootExtensions.autoInitialize(this)
    }
}
```

Don't forget to register in `AndroidManifest.xml`:
```xml
<application
    android:name=".MyApplication"
    ...>
```

### Config File Not Found

**Problem:** `stringboot-config.json` not found in assets

**Error:**
```
FileNotFoundException: stringboot-config.json
```

**Solution:**

1. **Create file in correct location:**
   ```
   app/src/main/assets/stringboot-config.json
   ```

2. **Verify file is included in build:**
   ```kotlin
   // build.gradle.kts
   android {
       sourceSets {
           getByName("main") {
               assets.srcDirs("src/main/assets")
           }
       }
   }
   ```

3. **Check file contents:**
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

### Invalid API Token

**Problem:** API token is invalid or expired

**Error:**
```
HTTP 401: Unauthorized
```

**Solution:**

1. Verify token in Stringboot dashboard
2. Ensure token is for correct environment (dev/prod)
3. Check token hasn't expired
4. Don't commit tokens to version control - use BuildConfig:

```kotlin
// build.gradle.kts
android {
    defaultConfig {
        buildConfigField("String", "STRINGBOOT_TOKEN", "\"${project.findProperty("stringboot.token")}\"")
    }
}

// Usage
StringProvider.initialize(
    context = this,
    cacheSize = 1000,
    api = StringbootApi(apiToken = BuildConfig.STRINGBOOT_TOKEN)
)
```

---

## String Display Issues

### Strings Showing as "??key??"

**Problem:** Strings display as `??welcome_message??` instead of actual text

**Possible Causes:**

1. **Key doesn't exist in dashboard**
   - Verify key name in Stringboot dashboard
   - Check for typos

2. **Language not available**
   - Ensure language is published
   - Check language code is correct ("en", not "eng")

3. **Not synced from network**
   - Call `refreshFromNetwork()` manually
   - Check internet connection

4. **Database empty (first launch)**
   - Wait for initial sync to complete

**Solution:**

```kotlin
// Enable debug logging to see what's happening
StringbootLogger.setLogLevel(LogLevel.DEBUG)

// Check if key exists
lifecycleScope.launch {
    val text = StringProvider.get("welcome_message", "en")
    if (text.startsWith("??")) {
        Log.e("Stringboot", "Key not found: welcome_message")

        // Force network refresh
        val success = StringProvider.refreshFromNetwork("en")
        if (!success) {
            Log.e("Stringboot", "Network refresh failed")
        }
    }
}
```

### XML Tag-based Strings Not Updating

**Problem:** Strings set via XML tags don't appear

```xml
<TextView
    app:stringboot_key="welcome_message"
    app:stringboot_lang="en" />
```

**Possible Causes:**

1. **Forgot to call `applyStringbootTags()`**
2. **Called before initialization**
3. **Wrong binding object**

**Solution:**

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // IMPORTANT: Call this AFTER StringProvider is initialized
        lifecycleScope.launch {
            // Wait for initialization if needed
            delay(100)
            binding.root.applyStringbootTags()
        }
    }
}
```

### Strings Not Auto-Updating

**Problem:** Strings don't update when language changes

**Solution:**

Use reactive Flow instead of one-time get:

```kotlin
// ❌ Bad: One-time fetch, won't update
lifecycleScope.launch {
    val text = StringProvider.get("key", "en")
    binding.textView.text = text
}

// ✅ Good: Auto-updating with Flow
StringProvider.getFlow("key", "en")
    .onEach { text ->
        binding.textView.text = text
    }
    .launchIn(lifecycleScope)
```

---

## Language Switching

### Language Change Doesn't Update UI

**Problem:** Called `switchLanguage()` but UI still shows old language

**Solutions:**

1. **Using Flow (automatic):**
```kotlin
// Use getFlow() - updates automatically
StringProvider.getFlow("welcome", currentLang)
    .onEach { text ->
        binding.textView.text = text
    }
    .launchIn(lifecycleScope)

// Switch language - UI updates automatically
lifecycleScope.launch {
    StringProvider.switchLanguage(this@MainActivity, "es")
}
```

2. **Using XML tags (automatic):**
```kotlin
// Apply tags once
binding.root.applyStringbootTags()

// Switch language - tags update automatically
lifecycleScope.launch {
    StringProvider.switchLanguage(this@MainActivity, "es")
}
```

3. **Manual refresh (if needed):**
```kotlin
lifecycleScope.launch {
    StringProvider.switchLanguage(this@MainActivity, "es")

    // Manually re-fetch if not using Flow
    updateAllStrings()
}
```

### Available Languages List Empty

**Problem:** `getAvailableLanguages()` returns empty list

**Solution:**

```kotlin
// Ensure network sync completed first
lifecycleScope.launch {
    // Refresh from network
    StringProvider.refreshFromNetwork()

    // Wait a moment for sync
    delay(500)

    // Now get languages
    val languages = StringProvider.getAvailableLanguages()
    if (languages.isEmpty()) {
        Log.e("Stringboot", "No languages available - check API token")
    }
}
```

---

## Network & Sync Issues

### Network Requests Failing

**Problem:** Network sync always fails

**Error:**
```
java.net.UnknownHostException: Unable to resolve host "api.stringboot.com"
```

**Solutions:**

1. **Add INTERNET permission:**
```xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

2. **Check network availability:**
```kotlin
fun isNetworkAvailable(context: Context): Boolean {
    val cm = context.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
    val network = cm.activeNetwork ?: return false
    val capabilities = cm.getNetworkCapabilities(network) ?: return false
    return capabilities.hasCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
}

// Use before sync
if (isNetworkAvailable(this)) {
    StringProvider.refreshFromNetwork()
}
```

3. **Verify base URL:**
```kotlin
// Ensure correct URL (no trailing slash)
StringbootApi(
    apiToken = "YOUR_TOKEN",
    baseUrl = "https://api.stringboot.com"  // ✅ Correct
    // baseUrl = "https://api.stringboot.com/" // ❌ Wrong
)
```

### ETag Cache Issues

**Problem:** Strings not updating despite changes on backend

**Solution:**

Force refresh bypassing ETag cache:

```kotlin
lifecycleScope.launch {
    // Clear local cache
    StringProvider.clearCache("en")

    // Force network refresh
    StringProvider.refreshFromNetwork("en")
}
```

### SSL/TLS Certificate Errors

**Problem:** SSL handshake failed

**Error:**
```
javax.net.ssl.SSLHandshakeException
```

**Solutions:**

1. **Update device/emulator:**
   - Ensure Android version is up to date
   - Old devices may have outdated certificates

2. **Check cleartext traffic (dev only):**
```xml
<!-- AndroidManifest.xml - DEV ONLY -->
<application
    android:usesCleartextTraffic="true"  <!-- Only for localhost testing -->
    ...>
```

**⚠️ Never enable cleartext traffic in production!**

---

## A/B Testing Issues

### Experiments Not Assigned

**Problem:** `getActiveExperiments()` returns empty list

**Solutions:**

1. **Ensure analytics handler is set:**
```kotlin
StringProvider.initialize(
    context = this,
    cacheSize = 1000,
    api = api,
    analyticsHandler = MyAnalyticsHandler()  // Don't forget this!
)
```

2. **Check experiment is active in dashboard**
3. **Verify device ID is generated:**
```kotlin
lifecycleScope.launch {
    val deviceId = StringProvider.getDeviceId()
    Log.d("A/B Testing", "Device ID: $deviceId")
}
```

### Analytics Events Not Firing

**Problem:** `onExperimentAssigned()` never called

**Solution:**

```kotlin
class MyAnalyticsHandler : StringbootAnalyticsHandler {
    override fun onExperimentAssigned(assignment: ExperimentAssignment) {
        Log.d("Analytics", "Experiment assigned: ${assignment.experimentId}")

        // Your analytics code here
        Firebase.analytics.logEvent("experiment_assigned") {
            param("experiment_id", assignment.experimentId)
            param("variant_name", assignment.variantName)
        }
    }
}

// Ensure handler is passed during initialization
StringProvider.initialize(
    context = this,
    cacheSize = 1000,
    api = api,
    analyticsHandler = MyAnalyticsHandler()  // ← Must be here
)
```

### Variant Not Changing

**Problem:** User always sees same variant even after clearing data

**Explanation:** This is intentional! Device ID persists in DataStore to ensure consistent assignments.

**To test different variants:**

```kotlin
// For testing only - provide different device IDs
StringProvider.initialize(
    context = this,
    cacheSize = 1000,
    api = api,
    providedDeviceId = "test_user_1"  // Try "test_user_2", "test_user_3", etc.
)
```

---

## FAQ Provider Issues

### FAQProvider Not Initialized

**Problem:** Calling FAQProvider before initialization

**Error:**
```
java.lang.IllegalStateException: FAQProvider not initialized
```

**Solution:**

FAQProvider requires **separate initialization**:

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        // Initialize StringProvider first
        val success = StringbootExtensions.autoInitialize(this)

        if (success) {
            // Get API instance
            val api = /* get from StringProvider or create new */

            // Initialize FAQProvider separately
            FAQProvider.initialize(
                context = this,
                cacheSize = 200,
                api = api
            )
        }
    }
}
```

### FAQs Not Appearing

**Problem:** `getFAQs()` returns empty list

**Solutions:**

1. **Check tag spelling (case-sensitive!):**
```kotlin
// ✅ Correct
val faqs = FAQProvider.getFAQs(tag = "payments")

// ❌ Wrong if backend tag is lowercase
val faqs = FAQProvider.getFAQs(tag = "Payments")
```

2. **Enable network fetch:**
```kotlin
val faqs = FAQProvider.getFAQs(
    tag = "payments",
    allowNetworkFetch = true  // ← Enable this
)
```

3. **Check initialization:**
```kotlin
if (!FAQProvider.isInitialized()) {
    Log.e("FAQ", "FAQProvider not initialized!")
}
```

4. **Enable debug logging:**
```kotlin
StringbootLogger.setLogLevel(LogLevel.DEBUG)
```

### FAQ Flow Not Updating

**Problem:** `getFAQsFlow()` doesn't emit new values

**Solution:**

Ensure you're using the correct lifecycle scope:

```kotlin
// ✅ Good: Use lifecycleScope
FAQProvider.getFAQsFlow(tag = "payments")
    .onEach { faqs ->
        updateUI(faqs)
    }
    .launchIn(lifecycleScope)  // ← Correct scope

// ❌ Bad: Using GlobalScope (won't cancel)
FAQProvider.getFAQsFlow(tag = "payments")
    .onEach { faqs -> updateUI(faqs) }
    .launchIn(GlobalScope)  // ← Wrong - leaks
```

---

## Performance Issues

### Slow String Retrieval

**Problem:** String lookups taking >100ms

**Solutions:**

1. **Use IO dispatcher:**
```kotlin
// ✅ Good: Background thread
lifecycleScope.launch {
    val text = withContext(Dispatchers.IO) {
        StringProvider.get("key", "en")
    }
    binding.textView.text = text
}

// ❌ Bad: Main thread
runBlocking {
    val text = StringProvider.get("key", "en")  // Blocks UI!
}
```

2. **Preload frequently used languages:**
```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        StringbootExtensions.autoInitialize(this)

        // Preload common languages
        lifecycleScope.launch {
            StringProvider.preloadLanguage("en", maxStrings = 500)
            StringProvider.preloadLanguage("es", maxStrings = 500)
        }
    }
}
```

3. **Increase cache size:**
```kotlin
StringProvider.initialize(
    context = this,
    cacheSize = 2000,  // Increase from default 1000
    api = api
)
```

### Memory Warnings

**Problem:** App receiving low memory warnings

**Solutions:**

1. **Reduce cache size:**
```kotlin
StringProvider.initialize(
    context = this,
    cacheSize = 500,  // Reduce if memory constrained
    api = api
)
```

2. **Clear cache on low memory:**
```kotlin
class MyApplication : Application() {
    override fun onLowMemory() {
        super.onLowMemory()
        StringProvider.clearMemoryCache()
        FAQProvider.clearMemoryCache()
    }
}
```

3. **Use onTrimMemory:**
```kotlin
override fun onTrimMemory(level: Int) {
    super.onTrimMemory(level)

    when (level) {
        ComponentCallbacks2.TRIM_MEMORY_RUNNING_CRITICAL,
        ComponentCallbacks2.TRIM_MEMORY_COMPLETE -> {
            StringProvider.clearMemoryCache()
        }
    }
}
```

---

## Build & Compilation Errors

### Kapt Errors with Room

**Problem:** Annotation processing failing

**Solution:**

Add kapt plugin and dependencies:

```kotlin
plugins {
    id("kotlin-kapt")
}

dependencies {
    implementation("com.stringboot:stringboot-android-sdk:1.2.0")

    // If Room version conflicts
    kapt("androidx.room:room-compiler:2.6.1")
}
```

### Proguard/R8 Obfuscation Issues

**Problem:** App crashes in release build only

**Solution:**

Add Proguard rules:

```proguard
# proguard-rules.pro

# Stringboot SDK
-keep class com.stringboot.sdk.** { *; }
-keepclassmembers class com.stringboot.sdk.** { *; }

# Room
-keep class * extends androidx.room.RoomDatabase
-keep @androidx.room.Entity class *
-dontwarn androidx.room.paging.**

# Retrofit
-keepattributes Signature
-keepattributes Exceptions
-keep class retrofit2.** { *; }

# Gson
-keepattributes *Annotation*
-keep class com.stringboot.sdk.models.** { *; }
```

### Duplicate Class Errors

**Problem:** Duplicate class errors during build

```
Duplicate class androidx.lifecycle.ViewModelLazy found in modules
```

**Solution:**

Add dependency resolution strategy:

```kotlin
configurations.all {
    resolutionStrategy {
        force("androidx.lifecycle:lifecycle-viewmodel-ktx:2.6.2")
    }
}
```

---

## Runtime Crashes

### NullPointerException on Context

**Problem:** Crash when accessing StringProvider

```
java.lang.NullPointerException: Context is null
```

**Solution:**

Ensure initialization completed before use:

```kotlin
class MyApplication : Application() {
    companion object {
        var isStringbootReady = false
    }

    override fun onCreate() {
        super.onCreate()

        val success = StringbootExtensions.autoInitialize(this)
        isStringbootReady = success
    }
}

// In Activity
if (MyApplication.isStringbootReady) {
    loadStrings()
}
```

### ConcurrentModificationException

**Problem:** Crash when modifying list while iterating

**Solution:**

Use immutable lists or proper synchronization:

```kotlin
// ✅ Good: Create new list
val faqs = FAQProvider.getFAQs(tag = "payments")
val mutableFaqs = faqs.toMutableList()
mutableFaqs.removeIf { it.question.isEmpty() }

// ❌ Bad: Modifying original list
val faqs = FAQProvider.getFAQs(tag = "payments")
faqs.forEach { faq ->
    if (faq.question.isEmpty()) {
        faqs.remove(faq)  // ConcurrentModificationException!
    }
}
```

### IllegalStateException: Lifecycle

**Problem:** Calling lifecycleScope after Activity destroyed

**Solution:**

Check lifecycle state:

```kotlin
if (lifecycle.currentState.isAtLeast(Lifecycle.State.STARTED)) {
    lifecycleScope.launch {
        val text = StringProvider.get("key", "en")
        binding.textView.text = text
    }
}
```

---

## Still Having Issues?

### Enable Debug Logging

```kotlin
// In Application.onCreate()
StringbootLogger.setLogLevel(LogLevel.DEBUG)
```

### Check SDK Status

```kotlin
lifecycleScope.launch {
    Log.d("Stringboot", "Initialized: ${StringProvider.isInitialized()}")
    Log.d("Stringboot", "FAQ Initialized: ${FAQProvider.isInitialized()}")

    val stats = StringProvider.getCacheStatistics()
    Log.d("Stringboot", "Cache size: ${stats.currentSize}")
    Log.d("Stringboot", "Hit rate: ${stats.hitRate}")

    val deviceId = StringProvider.getDeviceId()
    Log.d("Stringboot", "Device ID: $deviceId")
}
```

### Get Help

- 📧 **Email:** support@stringboot.com
- 🐛 **GitHub Issues:** [Report a bug](https://github.com/stringboot/android-sdk/issues)
- 📚 **Documentation:** [Full docs](https://docs.stringboot.com)
- 💬 **Discord:** [Join community](https://discord.gg/stringboot)

---

**Last Updated:** December 2024 | **SDK Version:** 1.2.0+
