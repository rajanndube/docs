# StringBoot Web SDK

[![npm version](https://badge.fury.io/js/%40stringboot%2Fweb-sdk.svg)](https://www.npmjs.com/package/@stringboot/web-sdk)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**Modern internationalization (i18n) SDK for web applications** with automatic updates, offline support, and zero configuration.

## Why StringBoot?

- 🚀 **5-Minute Setup** - One API call to get started
- 🔄 **Auto-Sync** - Strings update automatically without redeploying
- 💾 **Offline-First** - Works without network connection
- ⚛️ **React Hooks** - Built-in React integration
- 🎯 **TypeScript** - Full type safety included
- 📦 **Lightweight** - Only 6.6 kB gzipped

## Installation

```bash
npm install @stringboot/web-sdk
```

## Quick Start

> **Using Next.js?** See the complete [Next.js Integration Guide](https://github.com/rajanndube/Stringboot-Client/blob/main/stringboot-web-sdk/NEXTJS_INTEGRATION.md)

### JavaScript / TypeScript

```javascript
import StringBoot from '@stringboot/web-sdk';

// 1. Initialize once at app startup
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en'
});

// 2. Get strings anywhere in your app
const welcomeText = await StringBoot.get('welcome_message');
console.log(welcomeText); // "Welcome to our app!"

// 3. Change language instantly
await StringBoot.changeLanguage('es');
```

### React

```tsx
import { useStringBoot, useString, useLanguage } from '@stringboot/web-sdk/react';

function App() {
  // Initialize SDK
  const { initialized, error } = useStringBoot({
    apiToken: 'YOUR_API_TOKEN',
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en',
  });

  if (!initialized) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return <MyApp />;
}

function MyComponent() {
  // Auto-updating strings
  const title = useString('page_title');
  const [lang, setLang] = useLanguage();

  return (
    <div>
      <h1>{title}</h1>
      <select value={lang} onChange={(e) => setLang(e.target.value)}>
        <option value="en">English</option>
        <option value="es">Español</option>
        <option value="fr">Français</option>
      </select>
    </div>
  );
}
```

### Vue 3

```vue
<template>
  <h1>{{ welcomeText }}</h1>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import StringBoot from '@stringboot/web-sdk';

const welcomeText = ref('');

onMounted(async () => {
  await StringBoot.initialize({
    apiToken: 'YOUR_API_TOKEN',
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en',
  });

  StringBoot.watch('welcome_message', (value) => {
    welcomeText.value = value;
  });
});
</script>
```

## Core API

### Initialize SDK

```javascript
await StringBoot.initialize({
  apiToken: string,                      // Required: Your StringBoot API token
  baseUrl?: string,                      // Optional: API URL (default: https://api.stringboot.com)
  defaultLanguage?: string,              // Optional: Default language code (default: browser language)
  debug?: boolean,                       // Optional: Enable debug logs (default: false)
  analyticsHandler?: AnalyticsHandler   // Optional: A/B testing analytics integration
});
```

### Get Strings

```javascript
// Get a single string
const text = await StringBoot.get('key', 'en');

// Watch for updates (reactive)
const unsubscribe = StringBoot.watch('key', (value) => {
  console.log('Updated:', value);
});
```

### Change Language

```javascript
await StringBoot.changeLanguage('es');
// All watched strings automatically update!
```

### Get Available Languages

```javascript
const languages = await StringBoot.getActiveLanguages();
// Returns: [{ code: 'en', name: 'English', isDefault: true }, ...]
```

### Manual Sync

```javascript
// Force sync with server
await StringBoot.syncNow();

// Clear cache and refresh all strings
await StringBoot.forceRefresh();
```

## React Hooks

### `useStringBoot(config)`
Initialize the SDK in your app root.

```tsx
const { initialized, error } = useStringBoot({
  apiToken: 'YOUR_TOKEN',
  baseUrl: 'https://api.stringboot.com'
});
```

### `useString(key, lang?)`
Get a single string with automatic updates.

```tsx
const welcomeText = useString('welcome_message');
return <h1>{welcomeText}</h1>;
```

### `useStrings(keys, lang?)`
Get multiple strings at once.

```tsx
const { title, subtitle, cta } = useStrings(['title', 'subtitle', 'cta']);
```

### `useLanguage()`
Manage current language.

```tsx
const [lang, setLang] = useLanguage();
return (
  <select value={lang} onChange={(e) => setLang(e.target.value)}>
    <option value="en">English</option>
    <option value="es">Spanish</option>
  </select>
);
```

### `useActiveLanguages()`
Fetch available languages.

```tsx
const { languages, loading, error } = useActiveLanguages();
```

### `useSync()`
Manual sync control.

```tsx
const { sync, syncing } = useSync();
return (
  <button onClick={sync} disabled={syncing}>
    {syncing ? 'Syncing...' : 'Sync Now'}
  </button>
);
```

## Advanced Features

### A/B Testing & Experiments

StringBoot Web SDK includes built-in A/B testing support with automatic experiment tracking and analytics integration.

#### Basic Setup with Analytics

```typescript
import StringBoot from '@stringboot/web-sdk';
import type { StringBootAnalyticsHandler, ExperimentAssignment } from '@stringboot/web-sdk';

// Implement analytics handler for your platform
class GoogleAnalyticsHandler implements StringBootAnalyticsHandler {
  onExperimentsAssigned(experiments: Record<string, ExperimentAssignment>) {
    Object.entries(experiments).forEach(([stringKey, experiment]) => {
      // Track experiment assignments in Google Analytics
      gtag('set', {
        [`stringboot_exp_${experiment.experimentKey}`]: experiment.variantName
      });

      console.log(`User assigned to experiment: ${experiment.experimentKey} = ${experiment.variantName}`);
    });
  }
}

// Initialize SDK with analytics handler
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  analyticsHandler: new GoogleAnalyticsHandler()
});
```

#### Firebase Analytics Example

```typescript
import { getAnalytics, setUserProperties } from 'firebase/analytics';
import type { StringBootAnalyticsHandler } from '@stringboot/web-sdk';

class FirebaseAnalyticsHandler implements StringBootAnalyticsHandler {
  private analytics = getAnalytics();

  onExperimentsAssigned(experiments: Record<string, ExperimentAssignment>) {
    const properties: Record<string, string> = {};

    Object.entries(experiments).forEach(([stringKey, experiment]) => {
      properties[`stringboot_exp_${experiment.experimentKey}`] = experiment.variantName;
    });

    setUserProperties(this.analytics, properties);
  }
}
```

#### Mixpanel Example

```typescript
import mixpanel from 'mixpanel-browser';
import type { StringBootAnalyticsHandler } from '@stringboot/web-sdk';

class MixpanelAnalyticsHandler implements StringBootAnalyticsHandler {
  onExperimentsAssigned(experiments: Record<string, ExperimentAssignment>) {
    Object.entries(experiments).forEach(([stringKey, experiment]) => {
      mixpanel.register({
        [`stringboot_exp_${experiment.experimentKey}`]: experiment.variantName
      });
    });
  }
}
```

#### Get Current Experiments

```typescript
// Get all active experiment assignments
const experiments = StringBoot.getExperiments();
console.log(experiments);
/* Returns:
{
  "welcome_message": {
    "experimentId": "exp_123",
    "experimentKey": "welcome_copy_test",
    "variantName": "variant_b",
    "assignedAt": "2025-01-15T10:30:00Z"
  }
}
*/

// Get specific experiment by string key
const experiment = StringBoot.getExperiment('welcome_message');
if (experiment) {
  console.log(`Variant: ${experiment.variantName}`);
}
```

#### React Integration with A/B Testing

```tsx
import { useStringBoot, useString } from '@stringboot/web-sdk/react';
import type { StringBootAnalyticsHandler } from '@stringboot/web-sdk';

class MyAnalyticsHandler implements StringBootAnalyticsHandler {
  onExperimentsAssigned(experiments) {
    // Send to your analytics platform
    console.log('Experiments assigned:', experiments);
  }
}

function App() {
  const { initialized } = useStringBoot({
    apiToken: 'YOUR_API_TOKEN',
    analyticsHandler: new MyAnalyticsHandler()
  });

  return initialized ? <MyApp /> : <div>Loading...</div>;
}

function MyComponent() {
  // String automatically uses A/B test variant if assigned
  const ctaText = useString('cta_button');

  return <button>{ctaText}</button>;
}
```

#### How A/B Testing Works

1. **Automatic Assignment**: When your app initializes, the server assigns experiment variants based on your configured experiments
2. **Cached Assignments**: Experiment assignments are cached in localStorage for consistency across sessions
3. **Analytics Integration**: The SDK automatically notifies your analytics handler when assignments are made
4. **No Code Changes**: Your existing `StringBoot.get()` or `useString()` calls automatically return the variant value

### FAQ Provider

The FAQ Provider feature enables you to deliver dynamic, multilingual FAQ content to your web application with offline-first caching and automatic updates.

**Key Features:**
- 📦 **Offline-First**: FAQs cached locally in IndexedDB
- 🌍 **Multilingual**: Automatic language fallback to English
- 🏷️ **Tag-Based**: Organize FAQs by tags and sub-tags
- 🔄 **Reactive**: Watch for FAQ updates with callbacks
- ⚡ **High Performance**: Three-tier caching (memory → IndexedDB → network)
- ✨ **Auto-Initialize**: FAQ Provider auto-initializes when you initialize StringBoot

#### Quick Start

**Note:** Unlike Android/iOS SDKs, Web SDK automatically initializes FAQ Provider when you call `StringBoot.initialize()`.

```javascript
import StringBoot from '@stringboot/web-sdk';

// Initialize StringBoot (FAQ Provider auto-initializes)
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en'
});

// FAQ Provider is ready to use immediately!
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'payments' });
console.log(faqs);
```

#### JavaScript/TypeScript Usage

```javascript
// Get FAQs by tag
const paymentFAQs = await StringBoot.FAQ.getFAQs({
  tag: 'payments',
  lang: 'en'
});

// Get single FAQ by ID
const faq = await StringBoot.FAQ.getFAQ({
  id: 'faq-123',
  lang: 'en'
});

// Get all tags
const tags = await StringBoot.FAQ.getAllTags('en');

// Filter by sub-tag
const allPaymentFAQs = await StringBoot.FAQ.getFAQs({ tag: 'payments' });
const refundFAQs = allPaymentFAQs.filter(faq =>
  faq.subTags?.includes('refunds')
);
```

#### React Usage

```tsx
import { useState, useEffect } from 'react';
import StringBoot from '@stringboot/web-sdk';

function FAQList() {
  const [faqs, setFaqs] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function loadFAQs() {
      try {
        const faqData = await StringBoot.FAQ.getFAQs({
          tag: 'payments',
          lang: 'en'
        });
        setFaqs(faqData);
      } catch (error) {
        console.error('Failed to load FAQs:', error);
      } finally {
        setLoading(false);
      }
    }

    loadFAQs();
  }, []);

  if (loading) return <div>Loading FAQs...</div>;

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

#### Vue 3 Usage

```vue
<template>
  <div>
    <div v-if="loading">Loading FAQs...</div>
    <div v-else>
      <div v-for="faq in faqs" :key="faq.id" class="faq-item">
        <h3>{{ faq.question }}</h3>
        <p>{{ faq.answer }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import StringBoot from '@stringboot/web-sdk';

const faqs = ref([]);
const loading = ref(true);

onMounted(async () => {
  try {
    faqs.value = await StringBoot.FAQ.getFAQs({
      tag: 'payments',
      lang: 'en'
    });
  } catch (error) {
    console.error('Failed to load FAQs:', error);
  } finally {
    loading.value = false;
  }
});
</script>
```

#### Complete Example: FAQ Page with Categories

```tsx
import { useState, useEffect } from 'react';
import StringBoot from '@stringboot/web-sdk';

function FAQPage() {
  const [faqs, setFaqs] = useState([]);
  const [selectedTag, setSelectedTag] = useState('payments');
  const [tags, setTags] = useState([]);
  const [loading, setLoading] = useState(true);

  // Load available tags
  useEffect(() => {
    async function loadTags() {
      const availableTags = await StringBoot.FAQ.getAllTags('en');
      setTags(availableTags);
    }
    loadTags();
  }, []);

  // Load FAQs when tag changes
  useEffect(() => {
    async function loadFAQs() {
      setLoading(true);
      try {
        const faqData = await StringBoot.FAQ.getFAQs({
          tag: selectedTag,
          lang: 'en'
        });
        setFaqs(faqData);
      } catch (error) {
        console.error('Failed to load FAQs:', error);
      } finally {
        setLoading(false);
      }
    }

    loadFAQs();
  }, [selectedTag]);

  return (
    <div className="faq-page">
      <h1>Frequently Asked Questions</h1>

      {/* Category selector */}
      <div className="category-tabs">
        {tags.map(tag => (
          <button
            key={tag}
            className={selectedTag === tag ? 'active' : ''}
            onClick={() => setSelectedTag(tag)}
          >
            {tag.charAt(0).toUpperCase() + tag.slice(1)}
          </button>
        ))}
      </div>

      {/* FAQ list */}
      {loading ? (
        <div className="loading">Loading FAQs...</div>
      ) : (
        <div className="faq-list">
          {faqs.length === 0 ? (
            <p>No FAQs available for this category.</p>
          ) : (
            faqs.map(faq => (
              <details key={faq.id} className="faq-item">
                <summary>{faq.question}</summary>
                <p>{faq.answer}</p>
                {faq.subTags && (
                  <div className="tags">
                    {faq.subTags.map(tag => (
                      <span key={tag} className="tag">{tag}</span>
                    ))}
                  </div>
                )}
              </details>
            ))
          )}
        </div>
      )}
    </div>
  );
}
```

#### API Reference

```typescript
// FAQ Provider methods
StringBoot.FAQ.getFAQs(options: {
  tag: string;
  lang?: string;
}): Promise<FAQ[]>

StringBoot.FAQ.getFAQ(options: {
  id: string;
  lang?: string;
}): Promise<FAQ | null>

StringBoot.FAQ.getAllTags(lang?: string): Promise<string[]>

StringBoot.FAQ.syncFAQs(lang?: string): Promise<boolean>

StringBoot.FAQ.clearCache(): void

// FAQ type definition
interface FAQ {
  id: string;
  question: string;
  answer: string;
  tags: string[];
  subTags?: string[];
  language: string;
  createdAt: string;
  updatedAt: string;
}
```

#### Performance Optimization

**1. Pagination for Large FAQ Lists**

```javascript
function usePaginatedFAQs(tag, pageSize = 20) {
  const [faqs, setFaqs] = useState([]);
  const [page, setPage] = useState(0);
  const [allFAQs, setAllFAQs] = useState([]);

  useEffect(() => {
    async function loadAll() {
      const all = await StringBoot.FAQ.getFAQs({ tag });
      setAllFAQs(all);
      setFaqs(all.slice(0, pageSize));
    }
    loadAll();
  }, [tag]);

  const loadMore = () => {
    const nextPage = page + 1;
    const start = 0;
    const end = (nextPage + 1) * pageSize;
    setFaqs(allFAQs.slice(start, end));
    setPage(nextPage);
  };

  return { faqs, loadMore, hasMore: (page + 1) * pageSize < allFAQs.length };
}
```

**2. Lazy Loading with Intersection Observer**

```tsx
function LazyFAQList() {
  const [displayCount, setDisplayCount] = useState(20);
  const [faqs, setFaqs] = useState([]);
  const observerRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          setDisplayCount(prev => prev + 20);
        }
      },
      { threshold: 1.0 }
    );

    if (observerRef.current) {
      observer.observe(observerRef.current);
    }

    return () => observer.disconnect();
  }, []);

  return (
    <div>
      {faqs.slice(0, displayCount).map(faq => (
        <FAQItem key={faq.id} faq={faq} />
      ))}
      <div ref={observerRef} style={{ height: 20 }} />
    </div>
  );
}
```

**3. Virtual Scrolling (for 500+ FAQs)**

```tsx
import { FixedSizeList } from 'react-window';

function VirtualizedFAQList({ faqs }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      <div className="faq-item">
        <h3>{faqs[index].question}</h3>
        <p>{faqs[index].answer}</p>
      </div>
    </div>
  );

  return (
    <FixedSizeList
      height={600}
      itemCount={faqs.length}
      itemSize={120}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

#### Troubleshooting

**FAQs not loading?**
- Verify `StringBoot.initialize()` was called successfully
- Check network connectivity
- Verify API token is correct
- Check browser console for errors

**Performance issues with large FAQ lists?**
- Implement pagination (see examples above)
- Use virtual scrolling for 500+ FAQs
- Increase cache size if needed

**FAQs not updating?**
- Call `await StringBoot.FAQ.syncFAQs()` to force refresh
- Clear cache: `StringBoot.FAQ.clearCache()`

For complete documentation, see [FAQ_PROVIDER.md](FAQ_PROVIDER.md) and [TROUBLESHOOTING.md](TROUBLESHOOTING.md).

### String Interpolation

```javascript
const template = await StringBoot.get('greeting_with_name'); // "Hello, {name}!"
const greeting = template.replace('{name}', userName);
```

### Fallback Values

```javascript
// If key doesn't exist, returns ??key??
const text = await StringBoot.get('missing_key');
// Returns: "??missing_key??"
```

### Offline Support

The SDK automatically caches strings in IndexedDB. Your app works offline with previously loaded strings.

```javascript
// No network needed after first load!
const text = await StringBoot.get('welcome_message');
```

### Multi-Tab Sync

Open multiple tabs - when you change language in one tab, all tabs update automatically via BroadcastChannel.

## Configuration

### Local Development

```javascript
await StringBoot.initialize({
  apiToken: 'YOUR_TOKEN',
  baseUrl: 'http://localhost:8000',  // Your local backend
  defaultLanguage: 'en',
  debug: true                         // See debug logs
});
```

### Production

```javascript
await StringBoot.initialize({
  apiToken: process.env.STRINGBOOT_TOKEN,
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en'
});
```

## TypeScript Support

Full TypeScript support included:

```typescript
import StringBoot from '@stringboot/web-sdk';
import type {
  StringBootConfig,
  ActiveLanguage,
  ExperimentAssignment,
  StringBootAnalyticsHandler
} from '@stringboot/web-sdk';

const config: StringBootConfig = {
  apiToken: 'YOUR_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  analyticsHandler: myAnalyticsHandler // Optional: for A/B testing
};

await StringBoot.initialize(config);
```

## Browser Support

- Chrome 87+
- Firefox 78+
- Safari 14+
- Edge 88+

**Required APIs:** IndexedDB, localStorage, Fetch API, ES2020

## Bundle Size

| Format | Size (gzipped) |
|--------|----------------|
| ESM    | 6.6 kB         |
| CJS    | 4.6 kB         |

## Migration Guide

### From other i18n libraries

**From react-i18next:**
```tsx
// Before
const { t } = useTranslation();
<h1>{t('welcome')}</h1>

// After
const welcome = useString('welcome');
<h1>{welcome}</h1>
```

**From i18next:**
```javascript
// Before
i18next.t('welcome');

// After
StringBoot.get('welcome');
```

## Troubleshooting

### Strings not loading?

1. Check your API token is correct
2. Verify `baseUrl` matches your backend
3. Check browser console for errors (enable `debug: true`)
4. Ensure your backend allows CORS from your domain

### Language not changing?

```javascript
// Make sure you await the language change
await StringBoot.changeLanguage('es');

// If using React hooks, useLanguage handles this automatically
const [lang, setLang] = useLanguage();
setLang('es'); // Automatically awaits
```

### Getting `??key??` instead of strings?

- Key doesn't exist in your StringBoot dashboard
- String hasn't synced yet (check network tab)
- Wrong language code

## Getting Help

- 📖 [Documentation](https://docs.stringboot.com)
- 🐛 [Report Issues](https://github.com/rajanndube/Stringboot-Client/issues)
- 💬 [Discord Community](https://discord.gg/stringboot)
- 📧 [Email Support](mailto:support@stringboot.com)

## License

MIT © StringBoot Engineering Team

---

**Made with ❤️ by the StringBoot Team**
