# Stringboot Android SDK - Quick Start Guide

Get up and running with Stringboot in **under 10 minutes**. This guide will walk you through adding dynamic string management to your Android app.

## Prerequisites

- Android Studio Arctic Fox (2020.3.1) or later
- Minimum SDK: API 24 (Android 7.0)
- Kotlin 1.9.22+
- Gradle 8.2.2+
- A Stringboot API token ([Get one here](https://stringboot.com/signup))

## Step 1: Add Dependency (1 minute)

Add the Stringboot SDK to your app's `build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.stringboot:stringboot-android-sdk:1.1.0")
}
```

Click **Sync Now** in Android Studio.

## Step 2: Configure API Credentials (2 minutes)

### Option A: AndroidManifest.xml (Recommended)

Add your API credentials to `AndroidManifest.xml`:

```xml
<application
    android:name=".MyApplication"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name">

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

</application>
```

**⚠️ Security Note:** For production apps, store your API token in `local.properties` or use Android's Build Config to keep it out of version control.

### Option B: Programmatic Configuration

```kotlin
StringProvider.initialize(
    context = this,
    cacheSize = 1000,
    api = StringSyncV2Client(
        baseUrl = "https://api.stringboot.com",
        apiToken = BuildConfig.STRINGBOOT_API_TOKEN
    )
)
```

## Step 3: Initialize SDK (3 minutes)

Create or update your `Application` class:

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

    private val applicationScope = CoroutineScope(SupervisorJob() + Dispatchers.IO)

    override fun onCreate() {
        super.onCreate()

        // Initialize from manifest metadata
        val success = StringbootExtensions.autoInitialize(this)

        if (!success) {
            Log.w("Stringboot", "Auto-initialization failed, using offline mode")
            StringProvider.initialize(
                context = this,
                cacheSize = 1000,
                api = null // Offline-only mode
            )
        }

        // Critical: Trigger initial sync for first app launch
        if (success) {
            applicationScope.launch {
                try {
                    val locale = StringProvider.deviceLocale()
                    val refreshSuccess = StringProvider.refreshFromNetwork(locale)

                    if (refreshSuccess) {
                        val count = StringProvider.getStringCount(locale)
                        Log.i("Stringboot", "✅ Initial sync: $count strings for $locale")
                    } else {
                        Log.w("Stringboot", "⚠️ Initial sync failed - using cached data")
                    }
                } catch (e: Exception) {
                    Log.e("Stringboot", "❌ Initial sync error: ${e.message}", e)
                }
            }
        }
    }

    override fun onTrimMemory(level: Int) {
        super.onTrimMemory(level)
        // Handle memory pressure
        StringProvider.handleMemoryPressure(level)
    }
}
```

### Register Application Class

Update `AndroidManifest.xml` to use your Application class:

```xml
<application
    android:name=".MyApplication"
    ...>
```

## Step 4: Use Strings in Your UI (4 minutes)

### Method 1: XML Tags (Easiest - Recommended)

**1. Add `android:tag` to your layout XML:**

```xml
<TextView
    android:id="@+id/tvWelcome"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/default_welcome"
    android:tag="welcome_message" />

<TextView
    android:id="@+id/tvDescription"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/default_description"
    android:tag="app_description" />

<Button
    android:id="@+id/btnAction"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/default_action"
    android:tag="button_get_started" />
```

**2. Apply tags in your Activity/Fragment:**

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // Single line - applies Stringboot to all tagged views!
        binding.root.applyStringbootTags()
    }
}
```

**That's it!** All views with `android:tag` attributes will automatically display Stringboot strings.

### Method 2: Programmatic Access (More Control)

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // Get current language
        val currentLang = StringProvider.deviceLocale()

        // Fetch and display strings
        lifecycleScope.launch {
            val welcomeText = StringProvider.get("welcome_message", currentLang)
            binding.tvWelcome.text = welcomeText

            val descriptionText = StringProvider.get("app_description", currentLang)
            binding.tvDescription.text = descriptionText
        }
    }
}
```

### Method 3: Reactive Flow (Auto-Updating UI)

For strings that might change (e.g., during language switching or A/B tests):

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        val currentLang = StringProvider.deviceLocale()

        // Reactive Flow - automatically updates when string changes
        lifecycleScope.launch {
            StringProvider.getFlow("welcome_message", currentLang)
                .collect { text ->
                    binding.tvWelcome.text = text
                }
        }

        lifecycleScope.launch {
            StringProvider.getFlow("app_description", currentLang)
                .collect { text ->
                    binding.tvDescription.text = text
                }
        }
    }
}
```

## Step 5: Test It Out! (1 minute)

1. **Run your app** on an emulator or device
2. **Check Logcat** for Stringboot initialization messages:
   ```
   I/Stringboot: ✅ Initial sync: 42 strings for en
   ```
3. Your UI should display strings from Stringboot!

## Common Pitfalls & Troubleshooting

### ❌ Problem: Strings don't appear / showing fallback text

**Solution:**
- Check that initial sync completed successfully (look for log: `✅ Initial sync`)
- Verify your API token is correct in `AndroidManifest.xml`
- Ensure you called `applyStringbootTags()` after setting content view
- Check network connectivity (SDK requires internet for first sync)

### ❌ Problem: App crashes with `StringProvider not initialized`

**Solution:**
- Ensure `MyApplication.onCreate()` is called before using `StringProvider`
- Verify `android:name=".MyApplication"` is set in manifest
- Check that `autoInitialize()` or `initialize()` completed

### ❌ Problem: Strings are outdated / not refreshing

**Solution:**
- Call `StringProvider.refreshFromNetwork(locale)` to force sync
- Check that your device has network connectivity
- Verify the strings exist on Stringboot dashboard for the current language

### ❌ Problem: Memory issues on low-end devices

**Solution:**
- Reduce cache size in manifest: `android:value="500"`
- Implement `onTrimMemory()` in your Application class (see Step 3)

## Next Steps

### Switch Languages

```kotlin
lifecycleScope.launch {
    // Set new language
    StringProvider.setLocale("es")

    // Preload cache (optional but recommended)
    StringProvider.preloadLanguage("es", maxStrings = 500)

    // Fetch fresh data
    StringProvider.refreshFromNetwork("es")

    // Re-apply to update UI
    binding.root.applyStringbootTags()
}
```

### Get Available Languages

```kotlin
lifecycleScope.launch {
    val languages = StringProvider.getAvailableLanguagesFromServer()

    languages.forEach { lang ->
        Log.i("Stringboot", "Available: ${lang.code} - ${lang.name}")
    }
}
```

### Use Custom sbTextView Component

For more advanced use cases, use the custom `sbTextView`:

```xml
<com.stringboot.sdk.ui.sbTextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:stringKey="welcome_message"
    app:stringLang="en"
    app:reactiveMode="true"
    app:fallbackText="Welcome!" />
```

## Learn More

| Resource | Description |
|----------|-------------|
| **[INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)** | Comprehensive integration walkthrough |
| **[AB_TESTING.md](AB_TESTING.md)** | A/B testing and experiment setup |
| **[ADVANCED_FEATURES.md](ADVANCED_FEATURES.md)** | Smart caching, FAQ provider, memory management |
| **[API_REFERENCE.md](API_REFERENCE.md)** | Complete API documentation |
| **[FAQ.md](FAQ.md)** | Frequently asked questions |
| **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** | Common issues and solutions |
| **[Demo App](../stringboot-android-demoApp/)** | Real-world implementation examples |

## Sample Code Repository

Check out our [complete sample app](../stringboot-android-demoApp/) with:
- ✅ Best practices for initialization
- ✅ Language switching UI
- ✅ FAQ integration example
- ✅ Material Design 3 implementation
- ✅ Offline mode handling

## Need Help?

- **Email:** support@stringboot.com
- **Documentation:** https://docs.stringboot.com
- **Report Issues:** https://github.com/stringboot/android-sdk/issues

---

**Congratulations!** 🎉 You've integrated Stringboot into your Android app. Your strings are now managed remotely and can be updated without app releases!

