# Stringboot Android SDK

[![Version](https://img.shields.io/badge/version-1.1.0-blue.svg)](https://github.com/stringboot/android-sdk/releases)
[![Min SDK](https://img.shields.io/badge/minSdk-24-green.svg)](https://developer.android.com/about/versions/nougat)
[![Target SDK](https://img.shields.io/badge/targetSdk-34-green.svg)](https://developer.android.com/about/versions/14)
[![License](https://img.shields.io/badge/license-Proprietary-red.svg)](LICENSE)

A high-performance internationalization (i18n) SDK for Android implementing the String-Sync v2 protocol. Enables dynamic content updates without app releases, featuring smart caching, offline support, and memory-optimized string management.

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Core Capabilities](#core-capabilities)
- [Documentation](#documentation)
- [Version History](#version-history)
- [Support](#support)
- [License](#license)

## Features

### 🚀 **High Performance**
- **Sub-300ms string lookups** on 4G networks (P95)
- **≤2 network requests per cold start** via intelligent caching
- **ETag-based conditional requests** minimize bandwidth usage
- **Smart memory cache** with LRU eviction and language-aware prioritization

### 💾 **Offline-First Architecture**
- **Full offline functionality** via Room database persistence
- **Automatic fallback** to local resources when network unavailable
- **Three-tier retrieval**: Memory cache → Database → Network
- **Background sync** with delta updates for minimal data transfer

### 🧠 **Smart Caching System**
- **Multi-layered cache**: LRU memory cache + Room database
- **Language-aware eviction** prioritizes current language strings
- **Access frequency tracking** for intelligent cache warming
- **Memory pressure handling** with 5 automatic pressure levels
- **Dynamic cache sizing** based on device capabilities

### 🔄 **Reactive UI Updates**
- **Kotlin Flow support** for automatic UI synchronization
- **XML tag-based integration** with single-line setup
- **Custom sbTextView** component for declarative string binding
- **Data binding adapters** for seamless integration

### 🌍 **Advanced Language Management**
- **Multi-language support** with automatic device locale detection
- **Runtime language switching** with instant UI updates
- **Language fallback chains** (requested → device → English)
- **Deactivated language cleanup** to optimize storage

### 🧪 **A/B Testing & Experiments**
- **Device ID management** for consistent experiment assignment
- **Automatic variant delivery** based on server-side rules
- **Analytics handler integration** for tracking conversions
- **Optional experiment metadata** for debugging

### ❓ **FAQ Management (v1.1.0+)**
- **FAQProvider** for dynamic FAQ content
- **Tag-based filtering** with subtag support
- **Delta sync** for bandwidth-efficient updates
- **Reactive Flow** for real-time FAQ updates

### 🔒 **Security & Integrity**
- **SHA-256 checksum verification** for all network responses
- **Request deduplication** prevents redundant API calls
- **Response size limits** protect against DoS attacks
- **Secure token management** via Android Manifest metadata

## Requirements

| Requirement | Version/Details |
|------------|----------------|
| **Min SDK** | API 24 (Android 7.0 Nougat) |
| **Target SDK** | API 34 (Android 14) |
| **Kotlin** | 1.9.22+ |
| **Gradle** | 8.2.2+ |
| **Dependencies** | Room 2.6.1, Retrofit 2.9.0, Coroutines 1.7.3 |

## Installation

### Gradle Setup

Add the Stringboot SDK dependency to your app's `build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.stringboot:stringboot-android-sdk:1.1.0")
}
```

**Note:** Replace `1.1.0` with the [latest version](CHANGELOG.md).

### Sync Project

Click **Sync Now** in Android Studio to download the SDK.

## Quick Start

### 1. Add API Credentials to Manifest

Add your Stringboot credentials to `AndroidManifest.xml`:

```xml
<application
    android:name=".YourApplication"
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

    <!-- Your activities, services, etc. -->
</application>
```

### 2. Initialize in Application Class

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

        // Auto-initialize from manifest metadata
        val success = StringbootExtensions.autoInitialize(this)

        if (!success) {
            // Fallback to offline-only mode
            StringProvider.initialize(
                context = this,
                cacheSize = 1000,
                api = null
            )
        }

        // Trigger initial sync (critical for first app launch)
        if (success) {
            applicationScope.launch {
                try {
                    val locale = StringProvider.deviceLocale()
                    StringProvider.refreshFromNetwork(locale)
                    Log.i("Stringboot", "Initial sync completed")
                } catch (e: Exception) {
                    Log.e("Stringboot", "Initial sync failed", e)
                }
            }
        }
    }
}
```

### 3. Use Strings in Your App

#### Option A: XML Tags (Declarative)

Add `android:tag` attribute with your string key:

```xml
<TextView
    android:id="@+id/tvWelcome"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/default_welcome"
    android:tag="welcome_message" />
```

Then call `applyStringbootTags()` once:

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // Apply Stringboot to all tagged views
        binding.root.applyStringbootTags()
    }
}
```

#### Option B: Programmatic Access (Suspend Function)

```kotlin
lifecycleScope.launch {
    val welcomeText = StringProvider.get("welcome_message", "en")
    binding.tvWelcome.text = welcomeText
}
```

#### Option C: Reactive Flow (Auto-Updating)

```kotlin
lifecycleScope.launch {
    StringProvider.getFlow("welcome_message", "en")
        .collect { text ->
            binding.tvWelcome.text = text
        }
}
```

### 4. Switch Languages

```kotlin
lifecycleScope.launch {
    StringProvider.setLocale("es") // Switch to Spanish
    StringProvider.preloadLanguage("es", maxStrings = 500) // Warm cache
    StringProvider.refreshFromNetwork("es") // Fetch fresh data

    // Re-apply tags to update UI
    binding.root.applyStringbootTags()
}
```

**That's it!** You're now using Stringboot for dynamic string management.

For a detailed 5-minute walkthrough, see [QUICKSTART.md](QUICKSTART.md).

## Core Capabilities

### String Retrieval

| Method | Description | Return Type |
|--------|-------------|-------------|
| `get(key, lang)` | Get single string (suspend) | `String` |
| `getFlow(key, lang)` | Reactive Flow for UI updates | `Flow<String>` |
| `getStringCount(lang)` | Count cached strings | `Int` |
| `preloadLanguage(lang, max)` | Warm cache for language | `Unit` |

### Language Management

| Method | Description | Return Type |
|--------|-------------|-------------|
| `deviceLocale()` | Get device language code | `String` |
| `setLocale(lang)` | Set current language | `Unit` |
| `getAvailableLanguages()` | Get cached language codes | `List<String>` |
| `getAvailableLanguagesFromServer()` | Fetch active languages from API | `List<ActiveLanguage>` |

### Network & Sync

| Method | Description | Return Type |
|--------|-------------|-------------|
| `refreshFromNetwork(lang)` | Force sync from server | `Boolean` |
| `forceRefresh()` | Full catalog refresh | `Unit` |
| `smartWarmCache()` | Intelligent cache warming | `Unit` |

### Memory & Caching

| Method | Description | Return Type |
|--------|-------------|-------------|
| `handleMemoryPressure(level)` | Respond to memory pressure | `Unit` |
| `getCacheStatistics()` | Get cache hit/miss metrics | `CacheStats` |
| `clearCache(clearDatabase)` | Clear cache (optionally DB) | `Unit` |

### FAQ Management (v1.1.0+)

| Method | Description | Return Type |
|--------|-------------|-------------|
| `FAQProvider.getFAQs(tag, subTags, lang)` | Get filtered FAQs | `List<FAQ>` |
| `FAQProvider.getFAQsFlow(tag, subTags, lang)` | Reactive FAQ Flow | `Flow<List<FAQ>>` |
| `FAQProvider.refreshFromNetwork()` | Force FAQ sync | `Boolean` |

### A/B Testing

Configure device ID for consistent experiment assignment:

```kotlin
StringProvider.initialize(
    context = this,
    cacheSize = 1000,
    api = api,
    providedDeviceId = "your-custom-device-id", // Optional
    analyticsHandler = yourAnalyticsHandler // Optional
)
```

See [AB_TESTING.md](AB_TESTING.md) for complete integration guide.

## Documentation

### Customer-Facing Documentation

| Document | Description |
|----------|-------------|
| **[QUICKSTART.md](QUICKSTART.md)** | 5-minute integration guide with code examples |
| **[INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)** | Comprehensive integration walkthrough |
| **[AB_TESTING.md](AB_TESTING.md)** | A/B testing and analytics integration |
| **[FAQ.md](FAQ.md)** | Frequently asked questions and troubleshooting |
| **[ADVANCED_FEATURES.md](ADVANCED_FEATURES.md)** | Smart caching, memory management, FAQ provider |
| **[API_REFERENCE.md](API_REFERENCE.md)** | Complete StringProvider & FAQProvider API reference |
| **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** | Common issues and solutions |
| **[CHANGELOG.md](CHANGELOG.md)** | Version history and breaking changes |

### Demo Application

Explore the [Android Demo App](../stringboot-android-demoApp/) for real-world implementation examples:

- **Initialization patterns** in `StringbootApplication.kt`
- **XML tag integration** in `MainActivity.kt`
- **FAQ implementation** in `FAQDemoActivity.kt`
- **Dynamic UI creation** in `DynamicUIExample.kt`

### Internal Documentation

For SDK developers and contributors:

| Document | Description |
|----------|-------------|
| **[Architecture Overview](../docs/ARCHITECTURE.md)** | System design and protocol details |
| **[String-Sync v2 Protocol](../docs/DELTA_SYNC_PROTOCOL.md)** | Delta sync implementation |
| **[Backend API Spec](../docs/BACKEND_API_SPECIFICATION.md)** | Server endpoint specifications |
| **[Development Guide](../docs/DEVELOPMENT_GUIDE.md)** | Contributing and building the SDK |
| **[PUBLISHING_GUIDE.md](../docs/PUBLISHING_GUIDES/ANDROID_PUBLISHING.md)** | Release and publishing process |

## Version History

### Current Version: 1.1.0 (2025-11-05)

**Added:**
- FAQProvider with delta sync support
- Tag-based FAQ filtering with subtags
- Reactive Flow for FAQ updates
- Separate ETag management for FAQ sync
- `StringProvider.getApi()` for sharing API instance

**Fixed:**
- SmartStringCache size calculation bug
- FAQ network fetch initialization issue
- NullPointerException in FAQ responses
- ETag collision between string and FAQ sync

See [CHANGELOG.md](CHANGELOG.md) for complete version history.

### Migration Notes

- **1.0.8 → 1.1.0:** No breaking changes. FAQ features are additive.
- **v1 → v2:** See [Migration Guide](../docs/MIGRATION_GUIDES/V1_TO_V2.md) for upgrade path.

## Architecture Highlights

### String-Sync v2 Protocol

The SDK implements a bandwidth-efficient sync protocol:

1. **Meta Check** (`/api/v1/client/strings/meta`) - Lightweight change detection via ETag
2. **Delta Sync** (`/api/v1/client/strings/updates`) - Download only changed strings
3. **Catalog Fallback** (`/api/v1/client/strings/catalog`) - Full refresh when needed

**Performance Targets (P95):**
- ≤2 network requests per cold start
- <300ms strings ready in memory on 4G
- <2kB download when nothing changed

### Multi-Layered Caching

```
UI Layer
   ↓
StringProvider (API Facade)
   ↓
Smart Memory Cache (LRU, language-aware)
   ↓
Room Database (Offline persistence)
   ↓
Network Layer (String-Sync v2 API)
```

### Memory Management

The SDK automatically handles memory pressure with 5 levels:

| Level | Action |
|-------|--------|
| **Low** | Normal operation |
| **Moderate** | Clear 20% of cache |
| **Running Low** | Clear 40% of cache |
| **Critical** | Clear 60% of cache |
| **Complete** | Clear all cache |

Register in your `Application` class:

```kotlin
override fun onTrimMemory(level: Int) {
    super.onTrimMemory(level)
    StringProvider.handleMemoryPressure(level)
}
```

## Support

### Getting Help

- **Documentation:** Start with [QUICKSTART.md](QUICKSTART.md)
- **FAQs:** Check [FAQ.md](FAQ.md) for common questions
- **Troubleshooting:** See [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- **Demo App:** Explore [stringboot-android-demoApp](../stringboot-android-demoApp/)
- **API Reference:** See [API_REFERENCE.md](API_REFERENCE.md)

### Reporting Issues

Found a bug? [Report an issue](https://github.com/stringboot/android-sdk/issues) with:

- SDK version (check [CHANGELOG.md](CHANGELOG.md))
- Android version and device model
- Minimal code to reproduce
- Logcat output

### Contact

- **Email:** support@stringboot.com
- **Website:** https://stringboot.com
- **Documentation:** https://docs.stringboot.com

## License

Copyright © 2024-2025 Stringboot. All rights reserved.

This SDK is proprietary software. See [LICENSE](LICENSE) for terms of use.

---

**Made with ❤️ by the Stringboot Team**

