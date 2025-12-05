# FAQ Provider - Android SDK

> Deliver dynamic, multilingual FAQ content to your users with offline-first caching

**Version:** 1.1.0+ | **Last Updated:** December 2024

---

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Core Concepts](#core-concepts)
- [Basic Usage](#basic-usage)
- [Reactive Integration](#reactive-integration)
- [Tag-Based Organization](#tag-based-organization)
- [Language Support](#language-support)
- [UI Integration](#ui-integration)
- [Advanced Usage](#advanced-usage)
- [Performance & Caching](#performance--caching)
- [Best Practices](#best-practices)
- [API Reference](#api-reference)
- [Examples](#examples)
- [Troubleshooting](#troubleshooting)

---

## Overview

The FAQ Provider enables you to deliver dynamic, context-aware FAQ content to your Android app users using the same offline-first, cached architecture as string resources.

### Key Features

✅ **Tag-based Organization** - Categorize FAQs by primary tag and optional sub-tags
✅ **Multi-language Support** - Automatic language fallback to English
✅ **Offline-First** - Three-tier caching (memory → database → network)
✅ **Delta Sync** - Only download changed FAQs
✅ **Reactive Updates** - Auto-updating UI with Kotlin Flow
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

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        // Initialize StringProvider first
        val success = StringbootExtensions.autoInitialize(this)

        if (success) {
            // Get the API instance from StringProvider
            val api = StringProvider.getApi() // You'll need the API instance

            // Initialize FAQProvider separately
            FAQProvider.initialize(
                context = this,
                cacheSize = 200,  // Default: 200 FAQs in memory
                api = api         // Pass the same API instance
            )
        }
    }
}
```

**Alternative: Manual initialization with config**

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        // Read config
        val config = StringbootConfig.loadFromAssets(this)

        // Create API client
        val api = StringbootApi(
            apiToken = config.apiToken,
            baseUrl = config.baseUrl
        )

        // Initialize StringProvider
        StringProvider.initialize(
            context = this,
            cacheSize = config.cacheSize,
            api = api
        )

        // Initialize FAQProvider with the same API
        FAQProvider.initialize(
            context = this,
            cacheSize = 200,
            api = api
        )
    }
}
```

### Step 2: Fetch FAQs

```kotlin
class FAQActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        lifecycleScope.launch {
            val faqs = FAQProvider.getFAQs(
                tag = "payments",
                subTags = listOf("refunds"),
                lang = "en",
                allowNetworkFetch = true
            )

            displayFAQs(faqs)
        }
    }
}
```

### Step 3: Display FAQs

```kotlin
private fun displayFAQs(faqs: List<FAQ>) {
    faqs.forEach { faq ->
        println("Q: ${faq.question}")
        println("A: ${faq.answer}")
    }
}
```

That's it! You're now serving dynamic FAQs to your users.

---

## Core Concepts

### FAQ Data Model

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
[L1: Memory Cache] (LRU, <5ms)
    ↓ (cache miss)
[L2: Room Database] (Persistent, <50ms)
    ↓ (cache miss)
[L3: Network API] (Delta sync)
    ↓
Return FAQs
```

**Benefits:**
- **Instant access** from memory cache
- **Offline support** via Room database
- **Minimal bandwidth** with delta sync

---

## Basic Usage

### Fetch FAQs by Tag

```kotlin
// Get all FAQs for a tag
lifecycleScope.launch {
    val faqs = FAQProvider.getFAQs(
        tag = "payments",
        lang = "en"
    )

    // Display in UI
    faqAdapter.submitList(faqs)
}
```

### Fetch FAQs with Sub-tag Filter

```kotlin
// Get only refund-related FAQs
lifecycleScope.launch {
    val refundFAQs = FAQProvider.getFAQs(
        tag = "payments",
        subTags = listOf("refunds"),
        lang = "en"
    )

    displayFAQs(refundFAQs)
}
```

### Fetch FAQs with Multiple Sub-tags (OR filter)

```kotlin
// Get FAQs matching ANY of the sub-tags
lifecycleScope.launch {
    val paymentMethodFAQs = FAQProvider.getFAQs(
        tag = "payments",
        subTags = listOf("card", "bank_transfer", "wallet"),
        lang = "en"
    )

    displayFAQs(paymentMethodFAQs)
}
```

### Enable Network Fallback

```kotlin
lifecycleScope.launch {
    val faqs = FAQProvider.getFAQs(
        tag = "identity_verification",
        subTags = listOf("document_upload"),
        lang = "en",
        allowNetworkFetch = true  // Fetch from network if not cached
    )

    displayFAQs(faqs)
}
```

---

## Reactive Integration

### Auto-Updating FAQs with Flow

Use `getFAQsFlow()` for reactive UI that automatically updates when FAQs change:

```kotlin
class FAQActivity : AppCompatActivity() {

    private lateinit var faqAdapter: FAQAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setupRecyclerView()
        observeFAQs()
    }

    private fun observeFAQs() {
        FAQProvider.getFAQsFlow(
            tag = "payments",
            subTags = listOf("refunds"),
            lang = "en"
        ).onEach { faqs ->
            // UI automatically updates when FAQs change
            faqAdapter.submitList(faqs)

            if (faqs.isEmpty()) {
                showEmptyState()
            } else {
                hideEmptyState()
            }
        }.launchIn(lifecycleScope)
    }

    private fun setupRecyclerView() {
        faqAdapter = FAQAdapter()
        binding.recyclerView.apply {
            layoutManager = LinearLayoutManager(context)
            adapter = faqAdapter
        }
    }
}
```

### Benefits of Reactive Flow

- ✅ **Automatic Updates** - UI refreshes when FAQs change in database
- ✅ **Language Changes** - Auto-updates when user switches language
- ✅ **Network Sync** - Updates when new FAQs are synced from network
- ✅ **Lifecycle Aware** - Automatically cancels when Activity/Fragment is destroyed

---

## Tag-Based Organization

### Common Tag Hierarchies

Organize FAQs using these recommended tag structures:

#### Payments & Billing
```kotlin
tag = "payments"
subTags = listOf(
    "refunds",           // Refund process
    "disputes",          // Dispute resolution
    "methods",           // Payment methods
    "failed",            // Failed payments
    "invoices"           // Invoice questions
)
```

#### Account Management
```kotlin
tag = "account"
subTags = listOf(
    "profile",           // Profile management
    "security",          // Security & privacy
    "deletion",          // Account deletion
    "privacy",           // Privacy settings
    "notifications"      // Notification preferences
)
```

#### Identity Verification
```kotlin
tag = "identity_verification"
subTags = listOf(
    "document_upload",   // Document upload issues
    "verification_failed", // Verification failures
    "liveness",          // Liveness check
    "requirements"       // ID requirements
)
```

#### Technical Support
```kotlin
tag = "support"
subTags = listOf(
    "contact",           // How to contact support
    "escalation",        // Escalation process
    "feedback",          // Submit feedback
    "bug_report"         // Report bugs
)
```

### Dynamic Tag Filtering

Allow users to filter FAQs by selecting sub-tags:

```kotlin
class FAQActivity : AppCompatActivity() {

    private var selectedSubTags = mutableListOf<String>()
    private val currentTag = "payments"

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setupSubTagChips()
        loadFAQs()
    }

    private fun setupSubTagChips() {
        val subTags = listOf("refunds", "disputes", "methods")

        subTags.forEach { subTag ->
            val chip = Chip(this).apply {
                text = subTag
                isCheckable = true
                setOnCheckedChangeListener { _, isChecked ->
                    if (isChecked) {
                        selectedSubTags.add(subTag)
                    } else {
                        selectedSubTags.remove(subTag)
                    }
                    loadFAQs()
                }
            }
            binding.chipGroup.addView(chip)
        }
    }

    private fun loadFAQs() {
        FAQProvider.getFAQsFlow(
            tag = currentTag,
            subTags = selectedSubTags.toList(),
            lang = "en"
        ).onEach { faqs ->
            faqAdapter.submitList(faqs)
        }.launchIn(lifecycleScope)
    }
}
```

---

## Language Support

### Automatic Language Fallback

FAQProvider automatically falls back to English if FAQs aren't available in the requested language:

```kotlin
// User's device is set to French
val faqs = FAQProvider.getFAQs(
    tag = "payments",
    lang = "fr",  // Will try French first
    allowNetworkFetch = true
)

// Fallback order:
// 1. Memory cache (French)
// 2. Room database (French)
// 3. Room database (English fallback)
// 4. Network (French)
// 5. Empty list
```

### Check Language Returned

```kotlin
val faqs = FAQProvider.getFAQs(tag = "payments", lang = "fr")

if (faqs.isNotEmpty()) {
    val actualLanguage = faqs.first().languageCode

    if (actualLanguage == "en" && "fr" != "en") {
        // Show notice that FAQs are in English
        showLanguageFallbackNotice()
    }
}
```

### Use Device Language

```kotlin
val deviceLang = StringProvider.deviceLocale()

val faqs = FAQProvider.getFAQs(
    tag = "payments",
    lang = deviceLang,  // Use device language
    allowNetworkFetch = true
)
```

### Multi-language FAQ Screen

```kotlin
class FAQActivity : AppCompatActivity() {

    private var currentLanguage = "en"

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        currentLanguage = StringProvider.deviceLocale()

        setupLanguageSwitcher()
        loadFAQs()
    }

    private fun setupLanguageSwitcher() {
        val languages = listOf("en", "es", "fr", "de")

        languages.forEach { lang ->
            val button = Button(this).apply {
                text = lang.uppercase()
                setOnClickListener {
                    currentLanguage = lang
                    loadFAQs()
                }
            }
            binding.languageContainer.addView(button)
        }
    }

    private fun loadFAQs() {
        FAQProvider.getFAQsFlow(
            tag = "payments",
            lang = currentLanguage,
            allowNetworkFetch = true
        ).onEach { faqs ->
            faqAdapter.submitList(faqs)
        }.launchIn(lifecycleScope)
    }
}
```

---

## UI Integration

### Expandable FAQ List (Accordion)

```kotlin
class FAQAdapter : RecyclerView.Adapter<FAQViewHolder>() {

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

### Search FAQs

```kotlin
class FAQActivity : AppCompatActivity() {

    private var allFAQs = listOf<FAQ>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setupSearchView()
        loadFAQs()
    }

    private fun setupSearchView() {
        binding.searchView.setOnQueryTextListener(object : SearchView.OnQueryTextListener {
            override fun onQueryTextSubmit(query: String?): Boolean {
                return false
            }

            override fun onQueryTextChange(newText: String?): Boolean {
                filterFAQs(newText ?: "")
                return true
            }
        })
    }

    private fun loadFAQs() {
        FAQProvider.getFAQsFlow(
            tag = "payments",
            lang = "en"
        ).onEach { faqs ->
            allFAQs = faqs
            faqAdapter.submitList(faqs)
        }.launchIn(lifecycleScope)
    }

    private fun filterFAQs(query: String) {
        val filtered = if (query.isEmpty()) {
            allFAQs
        } else {
            allFAQs.filter { faq ->
                faq.question.contains(query, ignoreCase = true) ||
                faq.answer.contains(query, ignoreCase = true)
            }
        }
        faqAdapter.submitList(filtered)
    }
}
```

### Categorized FAQ Sections

```kotlin
class CategorizedFAQActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        loadCategorizedFAQs()
    }

    private fun loadCategorizedFAQs() {
        lifecycleScope.launch {
            // Load FAQs for multiple categories in parallel
            val paymentFAQs = async {
                FAQProvider.getFAQs(tag = "payments", lang = "en")
            }
            val accountFAQs = async {
                FAQProvider.getFAQs(tag = "account", lang = "en")
            }
            val supportFAQs = async {
                FAQProvider.getFAQs(tag = "support", lang = "en")
            }

            // Display in sections
            displaySection("Payments", paymentFAQs.await())
            displaySection("Account", accountFAQs.await())
            displaySection("Support", supportFAQs.await())
        }
    }

    private fun displaySection(title: String, faqs: List<FAQ>) {
        // Add section header and FAQs to UI
    }
}
```

---

## Advanced Usage

### Refresh FAQs from Network

Manually trigger a network sync to get latest FAQs:

```kotlin
lifecycleScope.launch {
    val success = FAQProvider.refreshFromNetwork()

    if (success) {
        Log.d("FAQ", "FAQs refreshed successfully")
    } else {
        Log.e("FAQ", "Failed to refresh FAQs")
    }
}
```

### Pull-to-Refresh FAQs

```kotlin
class FAQActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setupPullToRefresh()
        loadFAQs()
    }

    private fun setupPullToRefresh() {
        binding.swipeRefreshLayout.setOnRefreshListener {
            refreshFAQs()
        }
    }

    private fun refreshFAQs() {
        lifecycleScope.launch {
            val success = FAQProvider.refreshFromNetwork()

            binding.swipeRefreshLayout.isRefreshing = false

            if (success) {
                Toast.makeText(this@FAQActivity, "FAQs updated", Toast.LENGTH_SHORT).show()
            } else {
                Toast.makeText(this@FAQActivity, "Failed to update FAQs", Toast.LENGTH_SHORT).show()
            }
        }
    }

    private fun loadFAQs() {
        FAQProvider.getFAQsFlow(
            tag = "payments",
            lang = "en"
        ).onEach { faqs ->
            faqAdapter.submitList(faqs)
        }.launchIn(lifecycleScope)
    }
}
```

### Custom FAQ Filtering Logic

```kotlin
suspend fun getFilteredFAQs(
    tag: String,
    subTags: List<String>,
    searchQuery: String,
    lang: String
): List<FAQ> {
    val faqs = FAQProvider.getFAQs(
        tag = tag,
        subTags = subTags,
        lang = lang,
        allowNetworkFetch = true
    )

    return if (searchQuery.isEmpty()) {
        faqs
    } else {
        faqs.filter { faq ->
            faq.question.contains(searchQuery, ignoreCase = true) ||
            faq.answer.contains(searchQuery, ignoreCase = true) ||
            faq.subTags.any { it.contains(searchQuery, ignoreCase = true) }
        }
    }
}
```

---

## Performance & Caching

### Cache Strategy

FAQProvider uses the same efficient caching strategy as StringProvider:

**Cache Layers:**

1. **Memory Cache (L1)**
   - LRU cache for instant access
   - Default size: 200 FAQs
   - Access time: <5ms

2. **Room Database (L2)**
   - Persistent offline storage
   - Unlimited capacity
   - Access time: <50ms

3. **Network (L3)**
   - Delta sync for changed FAQs only
   - ETag-based conditional requests
   - Downloads only what changed

### Cache Performance

```
Operation                    | Latency
-----------------------------|----------
Memory cache hit             | <5ms
Database hit                 | <50ms
Network (delta sync)         | Downloads only changes
Network (full catalog)       | First launch only
```

### Cache Key Structure

Cache keys are generated based on language, tag, and sub-tags:

```
cache_key = "en:payments:refunds,disputes"
```

This ensures efficient cache lookups and proper isolation between different FAQ queries.

### Preloading FAQs

Preload frequently accessed FAQs on app start for instant access:

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        StringbootExtensions.autoInitialize(this)

        // Preload common FAQs
        lifecycleScope.launch {
            FAQProvider.getFAQs(tag = "payments", lang = "en", allowNetworkFetch = true)
            FAQProvider.getFAQs(tag = "account", lang = "en", allowNetworkFetch = true)
        }
    }
}
```

---

## Best Practices

### 1. Use Reactive Flow for FAQ Screens

```kotlin
// ✅ Good: Auto-updating UI
FAQProvider.getFAQsFlow(tag = "payments", lang = "en")
    .onEach { faqs ->
        updateUI(faqs)
    }
    .launchIn(lifecycleScope)

// ❌ Avoid: Manual refresh required
val faqs = FAQProvider.getFAQs(tag = "payments", lang = "en")
updateUI(faqs)
```

### 2. Organize Tags Hierarchically

```kotlin
// ✅ Good: Specific tag + sub-tags
FAQProvider.getFAQs(
    tag = "identity_verification",
    subTags = listOf("document_upload", "liveness_check")
)

// ❌ Avoid: Generic tags with too many FAQs
FAQProvider.getFAQs(tag = "general")
```

### 3. Enable Network Fetch for User-Facing Screens

```kotlin
// ✅ Good: Allow network fallback for fresh content
FAQProvider.getFAQs(
    tag = "payments",
    allowNetworkFetch = true
)

// ❌ Avoid: Relying only on cache (may be stale)
FAQProvider.getFAQs(
    tag = "payments",
    allowNetworkFetch = false
)
```

### 4. Handle Empty States Gracefully

```kotlin
FAQProvider.getFAQsFlow(tag = "payments", lang = "en")
    .onEach { faqs ->
        if (faqs.isEmpty()) {
            binding.emptyState.isVisible = true
            binding.recyclerView.isVisible = false
        } else {
            binding.emptyState.isVisible = false
            binding.recyclerView.isVisible = true
            faqAdapter.submitList(faqs)
        }
    }
    .launchIn(lifecycleScope)
```

### 5. Refresh FAQs on App Launch

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        lifecycleScope.launch {
            FAQProvider.refreshFromNetwork()
        }
    }
}
```

### 6. Use Sub-tags for Context-Specific Help

```kotlin
// In payment screen
FAQProvider.getFAQs(tag = "payments", subTags = listOf("methods"))

// In refund screen
FAQProvider.getFAQs(tag = "payments", subTags = listOf("refunds"))

// In dispute screen
FAQProvider.getFAQs(tag = "payments", subTags = listOf("disputes"))
```

---

## API Reference

### FAQProvider Methods

#### `getFAQs()`

Fetch FAQs filtered by tag and optional sub-tags.

```kotlin
suspend fun getFAQs(
    tag: String,
    subTags: List<String> = emptyList(),
    lang: String = deviceLocale(),
    allowNetworkFetch: Boolean = false
): List<FAQ>
```

**Parameters:**
- `tag` - Primary tag to filter by (required)
- `subTags` - List of sub-tags to filter by (optional, OR filter)
- `lang` - Language code (defaults to device locale)
- `allowNetworkFetch` - Whether to fetch from network if not cached

**Returns:** List of FAQs matching the filter criteria

---

#### `getFAQsFlow()`

Get a reactive Flow of FAQs that automatically updates when FAQs change.

```kotlin
fun getFAQsFlow(
    tag: String,
    subTags: List<String> = emptyList(),
    lang: String = deviceLocale()
): Flow<List<FAQ>>
```

**Parameters:**
- `tag` - Primary tag to filter by (required)
- `subTags` - List of sub-tags to filter by (optional, OR filter)
- `lang` - Language code (defaults to device locale)

**Returns:** Flow that emits FAQ list whenever it changes

---

#### `refreshFromNetwork()`

Manually refresh FAQs from the network using delta sync.

```kotlin
suspend fun refreshFromNetwork(): Boolean
```

**Returns:** `true` if refresh succeeded, `false` otherwise

---

#### `isInitialized()`

Check if FAQProvider is initialized.

```kotlin
fun isInitialized(): Boolean
```

**Returns:** `true` if initialized, `false` otherwise

---

### FAQ Data Model

```kotlin
data class FAQ(
    val id: Int,
    val question: String,
    val answer: String,
    val tag: String,
    val subTags: List<String>,
    val languageCode: String?,
    val appId: String?,
    val createdAt: String?,
    val updatedAt: String?
)
```

---

## Examples

### Complete FAQ Screen Example

See [FAQDemoActivity.kt](../stringboot-android-demoApp/app/src/main/java/com/stringboot/android/FAQDemoActivity.kt) in the demo app for a full working implementation including:

- Expandable accordion-style FAQ items
- Sub-tag filtering with chips
- Pull-to-refresh
- Empty state handling
- Language switching

### Minimal Example

```kotlin
class SimpleFAQActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        lifecycleScope.launch {
            val faqs = FAQProvider.getFAQs(
                tag = "payments",
                lang = "en",
                allowNetworkFetch = true
            )

            faqs.forEach { faq ->
                println("Q: ${faq.question}")
                println("A: ${faq.answer}\n")
            }
        }
    }
}
```

---

## Troubleshooting

### FAQs Not Appearing

**Check initialization:**
```kotlin
if (!FAQProvider.isInitialized()) {
    Log.e("FAQ", "FAQProvider not initialized!")
}
```

**Enable logging:**
```kotlin
StringbootLogger.setLogLevel(LogLevel.DEBUG)
```

**Check network sync:**
```kotlin
lifecycleScope.launch {
    val success = FAQProvider.refreshFromNetwork()
    Log.d("FAQ", "Network sync: $success")
}
```

**Verify tag spelling (case-sensitive!):**
```kotlin
// Correct
FAQProvider.getFAQs(tag = "payments")

// Wrong if tag is "payments" not "Payments"
FAQProvider.getFAQs(tag = "Payments")
```

### Language Fallback Not Working

**Check actual language returned:**
```kotlin
val faqs = FAQProvider.getFAQs(tag = "payments", lang = "fr")
if (faqs.isNotEmpty()) {
    val actualLang = faqs.first().languageCode
    Log.d("FAQ", "Requested: fr, Got: $actualLang")
}
```

### Flow Not Updating

**Ensure proper lifecycle scope:**
```kotlin
// ✅ Good: Use lifecycleScope
FAQProvider.getFAQsFlow(tag = "payments")
    .onEach { faqs -> updateUI(faqs) }
    .launchIn(lifecycleScope)

// ❌ Bad: Using GlobalScope (won't cancel)
FAQProvider.getFAQsFlow(tag = "payments")
    .onEach { faqs -> updateUI(faqs) }
    .launchIn(GlobalScope)
```

---

## Related Documentation

- [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md#-faq-provider-integration) - Complete integration guide
- [API_REFERENCE.md](API_REFERENCE.md) - Full API documentation
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Common issues & solutions
- [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md) - Advanced caching & performance

---

**Version:** 1.1.0+ | **Last Updated:** December 2024
