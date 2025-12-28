# FAQ Provider - Web SDK

> Deliver dynamic, multilingual FAQ content to your users with offline-first caching

**Version:** 1.1.0+ | **Last Updated:** December 2024

---

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Core Concepts](#core-concepts)
- [Basic Usage](#basic-usage)
- [React Integration](#react-integration)
- [Vue Integration](#vue-integration)
- [Tag-Based Organization](#tag-based-organization)
- [Language Support](#language-support)
- [UI Components](#ui-components)
- [Advanced Usage](#advanced-usage)
- [Performance & Caching](#performance--caching)
- [Best Practices](#best-practices)
- [API Reference](#api-reference)
- [Examples](#examples)
- [Troubleshooting](#troubleshooting)

---

## Overview

The FAQ Provider enables you to deliver dynamic, context-aware FAQ content to your web application users using the same offline-first, cached architecture as string resources.

### Key Features

✅ **Tag-based Organization** - Categorize FAQs by primary tag and optional sub-tags
✅ **Multi-language Support** - Automatic language fallback to English
✅ **Offline-First** - Three-tier caching (memory → IndexedDB → network)
✅ **Delta Sync** - Only download changed FAQs
✅ **Framework Support** - React, Vue, Next.js, or vanilla JS
✅ **Search & Filter** - Filter by tags, sub-tags, and keywords
✅ **Rich Content** - Supports HTML/Markdown in answers

### Use Cases

- **In-app Help Centers** - Contextual help based on user location in app
- **Onboarding FAQs** - Guide new users with dynamic content
- **Feature-specific Help** - Show relevant FAQs for each feature
- **Multi-language Support Centers** - Automatic translation with fallback
- **Dynamic Knowledge Base** - Update help content without redeployment

---

## Quick Start

### Step 1: Initialize

StringBoot SDK initialization also initializes the FAQ Provider:

```javascript
import StringBoot from '@stringboot/web-sdk';

// Initialize StringBoot (this also initializes FAQ Provider)
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en'
});

// FAQ Provider is now ready - no separate initialization needed
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'payments' });
```

**Note:** Unlike Android and iOS SDKs, the Web SDK automatically initializes FAQ Provider when you call `StringBoot.initialize()`. No separate initialization is required.

### Step 2: Fetch FAQs

**Vanilla JavaScript:**
```javascript
const faqs = await StringBoot.FAQ.getFAQs({
  tag: 'payments',
  subTags: ['refunds'],
  lang: 'en',
  allowNetworkFetch: true
});

displayFAQs(faqs);
```

**React:**
```tsx
import { useState, useEffect } from 'react';
import StringBoot from '@stringboot/web-sdk';

function FAQScreen() {
  const [faqs, setFaqs] = useState([]);

  useEffect(() => {
    StringBoot.FAQ.getFAQs({ tag: 'payments' }).then(setFaqs);
  }, []);

  return (
    <div>
      {faqs.map(faq => (
        <div key={faq.id}>
          <h3>{faq.question}</h3>
          <p>{faq.answer}</p>
        </div>
      ))}
    </div>
  );
}
```

### Step 3: Display FAQs

```javascript
function displayFAQs(faqs) {
  faqs.forEach(faq => {
    console.log(`Q: ${faq.question}`);
    console.log(`A: ${faq.answer}`);
  });
}
```

That's it! You're now serving dynamic FAQs to your users.

---

## Core Concepts

### FAQ Data Model

```typescript
interface FAQ {
  id: number;                      // Unique FAQ ID
  question: string;                // FAQ question
  answer: string;                  // FAQ answer (supports HTML/markdown)
  tag: string;                     // Primary tag (e.g., "payments")
  subTags: string[];               // Sub-tags (e.g., ["refunds", "disputes"])
  languageCode?: string;           // Language code (e.g., "en", "es")
  appId?: string;                  // App identifier
  createdAt?: string;              // ISO 8601 timestamp
  updatedAt?: string;              // ISO 8601 timestamp
}
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

FAQ Provider uses a **three-tier caching system**:

```
User Request
    ↓
[L1: Memory Cache] (LRU, <5ms)
    ↓ (cache miss)
[L2: IndexedDB] (Persistent, <50ms)
    ↓ (cache miss)
[L3: Network API] (Delta sync)
    ↓
Return FAQs
```

**Benefits:**
- **Instant access** from memory cache
- **Offline support** via IndexedDB
- **Minimal bandwidth** with delta sync

---

## Basic Usage

### Fetch FAQs by Tag

```javascript
// Get all FAQs for a tag
const faqs = await StringBoot.FAQ.getFAQs({
  tag: 'payments',
  lang: 'en'
});

displayFAQs(faqs);
```

### Fetch FAQs with Sub-tag Filter

```javascript
// Get only refund-related FAQs
const refundFAQs = await StringBoot.FAQ.getFAQs({
  tag: 'payments',
  subTags: ['refunds'],
  lang: 'en'
});

displayFAQs(refundFAQs);
```

### Fetch FAQs with Multiple Sub-tags (OR filter)

```javascript
// Get FAQs matching ANY of the sub-tags
const paymentMethodFAQs = await StringBoot.FAQ.getFAQs({
  tag: 'payments',
  subTags: ['card', 'bank_transfer', 'wallet'],
  lang: 'en'
});

displayFAQs(paymentMethodFAQs);
```

### Enable Network Fallback

```javascript
const faqs = await StringBoot.FAQ.getFAQs({
  tag: 'identity_verification',
  subTags: ['document_upload'],
  lang: 'en',
  allowNetworkFetch: true  // Fetch from network if not cached
});

displayFAQs(faqs);
```

---

## React Integration

### Basic FAQ Component

```tsx
import React, { useState, useEffect } from 'react';
import StringBoot from '@stringboot/web-sdk';

interface FAQ {
  id: number;
  question: string;
  answer: string;
  tag: string;
  subTags: string[];
}

function FAQList() {
  const [faqs, setFaqs] = useState<FAQ[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    loadFAQs();
  }, []);

  async function loadFAQs() {
    setIsLoading(true);
    const data = await StringBoot.FAQ.getFAQs({
      tag: 'payments',
      lang: 'en',
      allowNetworkFetch: true
    });
    setFaqs(data);
    setIsLoading(false);
  }

  if (isLoading) return <div>Loading FAQs...</div>;

  return (
    <div className="faq-list">
      {faqs.map(faq => (
        <FAQItem key={faq.id} faq={faq} />
      ))}
    </div>
  );
}
```

### Expandable FAQ Items

```tsx
function FAQItem({ faq }: { faq: FAQ }) {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <div className="faq-item">
      <div
        className="faq-question"
        onClick={() => setIsExpanded(!isExpanded)}
      >
        <h3>{faq.question}</h3>
        <span className={isExpanded ? 'chevron expanded' : 'chevron'}>
          ▼
        </span>
      </div>

      {isExpanded && (
        <div
          className="faq-answer"
          dangerouslySetInnerHTML={{ __html: faq.answer }}
        />
      )}
    </div>
  );
}
```

### FAQ Screen with Filters

```tsx
function FAQScreen() {
  const [faqs, setFaqs] = useState<FAQ[]>([]);
  const [selectedSubTags, setSelectedSubTags] = useState<string[]>([]);
  const [isLoading, setIsLoading] = useState(false);

  const currentTag = 'payments';
  const availableSubTags = ['refunds', 'disputes', 'methods'];

  useEffect(() => {
    loadFAQs();
  }, [selectedSubTags]);

  async function loadFAQs() {
    setIsLoading(true);
    const data = await StringBoot.FAQ.getFAQs({
      tag: currentTag,
      subTags: selectedSubTags,
      lang: 'en',
      allowNetworkFetch: true
    });
    setFaqs(data);
    setIsLoading(false);
  }

  function toggleSubTag(subTag: string) {
    setSelectedSubTags(prev =>
      prev.includes(subTag)
        ? prev.filter(t => t !== subTag)
        : [...prev, subTag]
    );
  }

  return (
    <div className="faq-screen">
      <h1>Frequently Asked Questions</h1>

      {/* Filter Chips */}
      <div className="filter-chips">
        <button
          className={selectedSubTags.length === 0 ? 'active' : ''}
          onClick={() => setSelectedSubTags([])}
        >
          All FAQs
        </button>
        {availableSubTags.map(subTag => (
          <button
            key={subTag}
            className={selectedSubTags.includes(subTag) ? 'active' : ''}
            onClick={() => toggleSubTag(subTag)}
          >
            {subTag}
          </button>
        ))}
      </div>

      {/* FAQ List */}
      {isLoading ? (
        <div>Loading...</div>
      ) : faqs.length === 0 ? (
        <div>No FAQs found</div>
      ) : (
        <div className="faq-list">
          {faqs.map(faq => (
            <FAQItem key={faq.id} faq={faq} />
          ))}
        </div>
      )}
    </div>
  );
}
```

### Searchable FAQs

```tsx
function SearchableFAQs() {
  const [allFAQs, setAllFAQs] = useState<FAQ[]>([]);
  const [searchQuery, setSearchQuery] = useState('');

  useEffect(() => {
    StringBoot.FAQ.getFAQs({ tag: 'payments' }).then(setAllFAQs);
  }, []);

  const filteredFAQs = searchQuery
    ? allFAQs.filter(faq =>
        faq.question.toLowerCase().includes(searchQuery.toLowerCase()) ||
        faq.answer.toLowerCase().includes(searchQuery.toLowerCase())
      )
    : allFAQs;

  return (
    <div>
      <input
        type="text"
        placeholder="Search FAQs..."
        value={searchQuery}
        onChange={(e) => setSearchQuery(e.target.value)}
      />

      {filteredFAQs.map(faq => (
        <FAQItem key={faq.id} faq={faq} />
      ))}
    </div>
  );
}
```

---

## Vue Integration

### Vue 3 Composition API

```vue
<template>
  <div class="faq-screen">
    <h1>Frequently Asked Questions</h1>

    <!-- Filter Chips -->
    <div class="filter-chips">
      <button
        :class="{ active: selectedSubTags.length === 0 }"
        @click="selectedSubTags = []"
      >
        All FAQs
      </button>
      <button
        v-for="subTag in availableSubTags"
        :key="subTag"
        :class="{ active: selectedSubTags.includes(subTag) }"
        @click="toggleSubTag(subTag)"
      >
        {{ subTag }}
      </button>
    </div>

    <!-- FAQ List -->
    <div v-if="isLoading">Loading FAQs...</div>
    <div v-else-if="faqs.length === 0">No FAQs found</div>
    <div v-else class="faq-list">
      <FAQItem
        v-for="faq in faqs"
        :key="faq.id"
        :faq="faq"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue';
import StringBoot from '@stringboot/web-sdk';

const faqs = ref([]);
const isLoading = ref(true);
const selectedSubTags = ref([]);

const currentTag = 'payments';
const availableSubTags = ['refunds', 'disputes', 'methods'];

onMounted(async () => {
  await loadFAQs();
});

watch(selectedSubTags, () => {
  loadFAQs();
}, { deep: true });

async function loadFAQs() {
  isLoading.value = true;
  faqs.value = await StringBoot.FAQ.getFAQs({
    tag: currentTag,
    subTags: selectedSubTags.value,
    lang: 'en',
    allowNetworkFetch: true
  });
  isLoading.value = false;
}

function toggleSubTag(subTag) {
  const index = selectedSubTags.value.indexOf(subTag);
  if (index > -1) {
    selectedSubTags.value.splice(index, 1);
  } else {
    selectedSubTags.value.push(subTag);
  }
}
</script>
```

### Expandable FAQ Item Component

```vue
<template>
  <div class="faq-item">
    <div class="faq-question" @click="isExpanded = !isExpanded">
      <h3>{{ faq.question }}</h3>
      <span :class="{ chevron: true, expanded: isExpanded }">▼</span>
    </div>

    <div v-if="isExpanded" class="faq-answer" v-html="faq.answer" />
  </div>
</template>

<script setup>
import { ref } from 'vue';

const props = defineProps({
  faq: {
    type: Object,
    required: true
  }
});

const isExpanded = ref(false);
</script>

<style scoped>
.faq-item {
  border: 1px solid #ddd;
  margin-bottom: 12px;
  border-radius: 8px;
  overflow: hidden;
}

.faq-question {
  padding: 16px;
  background: #f5f5f5;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.chevron {
  transition: transform 0.3s;
}

.chevron.expanded {
  transform: rotate(180deg);
}

.faq-answer {
  padding: 16px;
  border-top: 1px solid #ddd;
}
</style>
```

---

## Tag-Based Organization

### Common Tag Hierarchies

#### Payments & Billing
```javascript
const tag = 'payments';
const subTags = [
  'refunds',           // Refund process
  'disputes',          // Dispute resolution
  'methods',           // Payment methods
  'failed',            // Failed payments
  'invoices'           // Invoice questions
];
```

#### Account Management
```javascript
const tag = 'account';
const subTags = [
  'profile',           // Profile management
  'security',          // Security & privacy
  'deletion',          // Account deletion
  'privacy',           // Privacy settings
  'notifications'      // Notification preferences
];
```

#### Identity Verification
```javascript
const tag = 'identity_verification';
const subTags = [
  'document_upload',   // Document upload issues
  'verification_failed', // Verification failures
  'liveness',          // Liveness check
  'requirements'       // ID requirements
];
```

#### Technical Support
```javascript
const tag = 'support';
const subTags = [
  'contact',           // How to contact support
  'escalation',        // Escalation process
  'feedback',          // Submit feedback
  'bug_report'         // Report bugs
];
```

---

## Language Support

### Automatic Language Fallback

```javascript
// User's browser is set to French
const faqs = await StringBoot.FAQ.getFAQs({
  tag: 'payments',
  lang: 'fr',  // Will try French first
  allowNetworkFetch: true
});

// Fallback order:
// 1. Memory cache (French)
// 2. IndexedDB (French)
// 3. IndexedDB (English fallback)
// 4. Network (French)
// 5. Empty array
```

### Check Language Returned

```javascript
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'payments', lang: 'fr' });

if (faqs.length > 0) {
  const actualLanguage = faqs[0].languageCode;

  if (actualLanguage === 'en' && 'fr' !== 'en') {
    // Show notice that FAQs are in English
    showLanguageFallbackNotice();
  }
}
```

### Use Browser Language

```javascript
const browserLang = navigator.language.split('-')[0]; // "en" from "en-US"

const faqs = await StringBoot.FAQ.getFAQs({
  tag: 'payments',
  lang: browserLang,
  allowNetworkFetch: true
});
```

---

## UI Components

### Vanilla JavaScript FAQ List

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>FAQs</title>
  <style>
    .faq-item {
      border: 1px solid #ddd;
      margin-bottom: 12px;
      border-radius: 8px;
      overflow: hidden;
    }
    .faq-question {
      padding: 16px;
      background: #f5f5f5;
      cursor: pointer;
      display: flex;
      justify-content: space-between;
    }
    .faq-answer {
      padding: 16px;
      display: none;
      border-top: 1px solid #ddd;
    }
    .faq-item.expanded .faq-answer {
      display: block;
    }
  </style>
</head>
<body>
  <h1>Frequently Asked Questions</h1>
  <div id="faq-container"></div>

  <script type="module">
    import StringBoot from '@stringboot/web-sdk';

    await StringBoot.initialize({
      apiToken: 'YOUR_API_TOKEN',
      baseUrl: 'https://api.stringboot.com',
      defaultLanguage: 'en'
    });

    const faqs = await StringBoot.FAQ.getFAQs({
      tag: 'payments',
      lang: 'en',
      allowNetworkFetch: true
    });

    const container = document.getElementById('faq-container');

    faqs.forEach(faq => {
      const faqItem = document.createElement('div');
      faqItem.className = 'faq-item';
      faqItem.innerHTML = `
        <div class="faq-question">
          <span>${faq.question}</span>
          <span>▼</span>
        </div>
        <div class="faq-answer">${faq.answer}</div>
      `;

      faqItem.querySelector('.faq-question').addEventListener('click', () => {
        faqItem.classList.toggle('expanded');
      });

      container.appendChild(faqItem);
    });
  </script>
</body>
</html>
```

---

## Advanced Usage

### Refresh FAQs from Network

```javascript
const success = await StringBoot.FAQ.refreshFromNetwork();

if (success) {
  console.log('FAQs refreshed successfully');
} else {
  console.error('Failed to refresh FAQs');
}
```

### Categorized FAQ Sections

```javascript
async function loadCategorizedFAQs() {
  const [paymentFAQs, accountFAQs, supportFAQs] = await Promise.all([
    StringBoot.FAQ.getFAQs({ tag: 'payments' }),
    StringBoot.FAQ.getFAQs({ tag: 'account' }),
    StringBoot.FAQ.getFAQs({ tag: 'support' })
  ]);

  displaySection('Payments', paymentFAQs);
  displaySection('Account', accountFAQs);
  displaySection('Support', supportFAQs);
}
```

### Custom Filtering

```javascript
async function getFilteredFAQs(tag, subTags, searchQuery, lang) {
  const faqs = await StringBoot.FAQ.getFAQs({
    tag,
    subTags,
    lang,
    allowNetworkFetch: true
  });

  if (!searchQuery) return faqs;

  return faqs.filter(faq =>
    faq.question.toLowerCase().includes(searchQuery.toLowerCase()) ||
    faq.answer.toLowerCase().includes(searchQuery.toLowerCase()) ||
    faq.subTags.some(t => t.toLowerCase().includes(searchQuery.toLowerCase()))
  );
}
```

---

## Performance & Caching

### Cache Strategy

**Cache Layers:**

1. **Memory Cache (L1)** - LRU cache for instant access (<5ms)
2. **IndexedDB (L2)** - Persistent browser storage (<50ms)
3. **Network (L3)** - Delta sync for changed FAQs only

### Preloading FAQs

```javascript
// On app initialization
await StringBoot.initialize({/*...*/});

// Preload common FAQs
await StringBoot.FAQ.getFAQs({ tag: 'payments', allowNetworkFetch: true });
await StringBoot.FAQ.getFAQs({ tag: 'account', allowNetworkFetch: true });
```

---

## Best Practices

### 1. Enable Network Fetch for User-Facing Screens

```javascript
// ✅ Good: Allow network fallback
const faqs = await StringBoot.FAQ.getFAQs({
  tag: 'payments',
  allowNetworkFetch: true
});
```

### 2. Handle Empty States

```javascript
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'payments' });

if (faqs.length === 0) {
  showEmptyState();
} else {
  displayFAQs(faqs);
}
```

### 3. Refresh on Page Load

```javascript
await StringBoot.initialize({/*...*/});
await StringBoot.FAQ.refreshFromNetwork();
```

---

## API Reference

### getFAQs()

```typescript
async function getFAQs(options: {
  tag: string;
  subTags?: string[];
  lang?: string;
  allowNetworkFetch?: boolean;
}): Promise<FAQ[]>
```

### refreshFromNetwork()

```typescript
async function refreshFromNetwork(): Promise<boolean>
```

---

## Examples

### Minimal Example

```javascript
const faqs = await StringBoot.FAQ.getFAQs({
  tag: 'payments',
  lang: 'en',
  allowNetworkFetch: true
});

faqs.forEach(faq => {
  console.log(`Q: ${faq.question}`);
  console.log(`A: ${faq.answer}`);
});
```

---

## Troubleshooting

### FAQs Not Appearing

```javascript
// Enable logging
StringBoot.setLogLevel('debug');

// Check network sync
const success = await StringBoot.FAQ.refreshFromNetwork();
console.log('Network sync:', success);

// Verify tag spelling (case-sensitive!)
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'payments' }); // Correct
```

---

## Related Documentation

- [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md#-faq-provider-integration) - Complete integration guide
- [API_REFERENCE.md](API_REFERENCE.md) - Full API documentation
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Common issues & solutions
- [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md) - Advanced caching & performance

---

**Version:** 1.1.0+ | **Last Updated:** December 2024
