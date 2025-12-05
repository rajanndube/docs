# Stringboot Web SDK - Frequently Asked Questions

> Common questions and answers about using the Stringboot Web SDK

**Last Updated:** December 2024 | **SDK Version:** 1.0.0+

---

## Table of Contents

- [Getting Started](#getting-started)
- [Installation & Setup](#installation--setup)
- [String Management](#string-management)
- [React Integration](#react-integration)
- [Vue Integration](#vue-integration)
- [Next.js Integration](#nextjs-integration)
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

Stringboot is a modern internationalization (i18n) SDK for web applications that allows you to manage and update strings remotely without redeploying your app. It provides offline-first caching, automatic updates, language switching, and A/B testing capabilities.

### Why should I use Stringboot instead of i18next or react-intl?

**Stringboot offers several advantages:**

- **Remote Updates** - Update strings without redeploying your app
- **Instant Fixes** - Fix typos or translations immediately
- **A/B Testing** - Test different copy variations with real users
- **Zero Configuration** - Works out of the box with minimal setup
- **Offline-First** - Works perfectly without internet connection (IndexedDB)
- **Analytics Integration** - Track which string variations perform best
- **Framework Agnostic** - Works with React, Vue, Next.js, or vanilla JS

### Can I use Stringboot alongside existing i18n solutions?

Yes! Many apps use Stringboot for dynamic content (marketing copy, feature descriptions, CTAs) while keeping critical UI strings in traditional i18n files as fallbacks.

### What are the minimum requirements?

- **Browser Support:** Modern browsers with IndexedDB support (Chrome 24+, Firefox 16+, Safari 10+, Edge 12+)
- **JavaScript:** ES2020+
- **TypeScript:** 4.5+ (optional)
- **React:** 16.8+ (for React hooks)
- **Vue:** 3.0+ (for Vue integration)
- **Next.js:** 12+ (for Next.js integration)

---

## Installation & Setup

### How do I install the SDK?

**NPM:**
```bash
npm install @stringboot/web-sdk
```

**Yarn:**
```bash
yarn add @stringboot/web-sdk
```

**CDN:**
```html
<script src="https://unpkg.com/@stringboot/web-sdk@latest/dist/stringboot.min.js"></script>
```

See [QUICKSTART.md](QUICKSTART.md) for complete installation instructions.

### Do I need to initialize the SDK on every page?

**No!** Initialize once at app startup:

**Vanilla JS:**
```javascript
import StringBoot from '@stringboot/web-sdk';

await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en'
});
```

**React:**
```tsx
import { useStringBoot } from '@stringboot/web-sdk/react';

function App() {
  const { initialized } = useStringBoot({
    apiToken: 'YOUR_API_TOKEN',
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en'
  });

  if (!initialized) return <div>Loading...</div>;
  return <MyApp />;
}
```

### Where do I get my API token?

1. Sign up at [stringboot.com](https://stringboot.com)
2. Create a new project
3. Navigate to **Settings** → **API Keys**
4. Copy your Web API token
5. Add it to your environment variables

**Never commit your API token!** Use environment variables:
- React: `REACT_APP_STRINGBOOT_TOKEN`
- Next.js: `NEXT_PUBLIC_STRINGBOOT_TOKEN`
- Vue: `VITE_STRINGBOOT_TOKEN`

### Can I use Stringboot without network access?

Yes! Stringboot is **offline-first**. Once strings are cached locally (IndexedDB), the app works perfectly without internet. Network is only needed for initial sync and updates.

### How do I configure the SDK?

Create `stringboot-config.json` in your public folder:

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

```javascript
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  cacheSize: 1000,
  autoSync: true,
  indexedDBEnabled: true
});
```

---

## String Management

### How do I get a string?

**Vanilla JavaScript:**
```javascript
const text = await StringBoot.get('welcome_message');
console.log(text); // "Welcome to our app!"
```

**React Hook:**
```tsx
import { useString } from '@stringboot/web-sdk/react';

function MyComponent() {
  const welcomeText = useString('welcome_message');
  return <h1>{welcomeText}</h1>;
}
```

**Vue Composition API:**
```vue
<script setup>
import { ref, onMounted } from 'vue';
import StringBoot from '@stringboot/web-sdk';

const welcomeText = ref('');

onMounted(async () => {
  welcomeText.value = await StringBoot.get('welcome_message');
});
</script>

<template>
  <h1>{{ welcomeText }}</h1>
</template>
```

### How do I make strings update automatically in my UI?

**React - Use hooks:**
```tsx
const text = useString('welcome_message');
// Automatically re-renders when string changes
```

**Vanilla JS - Use event listeners:**
```javascript
StringBoot.on('stringsUpdated', async () => {
  const newText = await StringBoot.get('welcome_message');
  document.getElementById('title').textContent = newText;
});
```

**Vue - Use reactive refs:**
```vue
<script setup>
import { ref } from 'vue';

const text = ref('');

// Update on language change
StringBoot.on('languageChange', async () => {
  text.value = await StringBoot.get('welcome_message');
});
</script>
```

### What happens if a string key doesn't exist?

The SDK returns `"??key??"` (e.g., `"??welcome_message??"`). This makes missing strings obvious during development.

**Best practice:** Always test with all language variants before deployment.

### Can I pass parameters to strings?

Not directly through Stringboot. Use JavaScript template strings:

```javascript
const template = await StringBoot.get('welcome_user'); // "Welcome, {name}!"
const message = template.replace('{name}', userName);
```

Or use template literals:

```javascript
const greeting = await StringBoot.get('greeting'); // "Hello"
const message = `${greeting}, ${userName}!`;
```

### How do I bulk-fetch multiple strings?

```javascript
const strings = await StringBoot.getBatch([
  'title',
  'subtitle',
  'cta_button'
]);

const [title, subtitle, cta] = strings;
```

**React:**
```tsx
import { useBatchStrings } from '@stringboot/web-sdk/react';

function MyComponent() {
  const [title, subtitle, cta] = useBatchStrings([
    'title',
    'subtitle',
    'cta_button'
  ]);

  return (
    <div>
      <h1>{title}</h1>
      <p>{subtitle}</p>
      <button>{cta}</button>
    </div>
  );
}
```

---

## React Integration

### What React hooks are available?

- **`useStringBoot(config)`** - Initialize SDK
- **`useString(key, options)`** - Get single string (auto-updating)
- **`useBatchStrings(keys)`** - Get multiple strings
- **`useLanguage()`** - Manage language switching
- **`useAvailableLanguages()`** - Get list of languages

### How do I initialize StringBoot in React?

**Option 1: Root component with hook**
```tsx
import { useStringBoot } from '@stringboot/web-sdk/react';

function App() {
  const { initialized, error, loading } = useStringBoot({
    apiToken: process.env.REACT_APP_STRINGBOOT_TOKEN!,
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en'
  });

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!initialized) return null;

  return <MyApp />;
}
```

**Option 2: Context provider**
```tsx
import { StringProvider } from '@stringboot/web-sdk/react';

function App() {
  return (
    <StringProvider
      apiToken={process.env.REACT_APP_STRINGBOOT_TOKEN!}
      baseUrl="https://api.stringboot.com"
      defaultLanguage="en"
    >
      <MyApp />
    </StringProvider>
  );
}
```

### How do I use useString hook?

```tsx
import { useString } from '@stringboot/web-sdk/react';

function WelcomeScreen() {
  const title = useString('welcome_title');
  const subtitle = useString('welcome_subtitle');

  return (
    <div>
      <h1>{title}</h1>
      <p>{subtitle}</p>
    </div>
  );
}
```

**With custom language:**
```tsx
const title = useString('title', { lang: 'es' });
```

### How do I manage language switching in React?

```tsx
import { useLanguage } from '@stringboot/web-sdk/react';

function LanguageSwitcher() {
  const [currentLang, setLanguage, availableLanguages] = useLanguage();

  return (
    <select value={currentLang} onChange={(e) => setLanguage(e.target.value)}>
      {availableLanguages.map(lang => (
        <option key={lang.code} value={lang.code}>
          {lang.name}
        </option>
      ))}
    </select>
  );
}
```

---

## Vue Integration

### How do I use StringBoot with Vue 3?

**Composition API:**
```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p>{{ subtitle }}</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import StringBoot from '@stringboot/web-sdk';

const title = ref('');
const subtitle = ref('');

onMounted(async () => {
  await StringBoot.initialize({
    apiToken: import.meta.env.VITE_STRINGBOOT_TOKEN,
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en'
  });

  title.value = await StringBoot.get('title');
  subtitle.value = await StringBoot.get('subtitle');
});
</script>
```

**Options API:**
```vue
<script>
import StringBoot from '@stringboot/web-sdk';

export default {
  data() {
    return {
      title: '',
      subtitle: ''
    };
  },
  async mounted() {
    await StringBoot.initialize({/*...*/});
    this.title = await StringBoot.get('title');
    this.subtitle = await StringBoot.get('subtitle');
  }
};
</script>
```

### Can I create a Vue plugin for StringBoot?

Yes! See [INTEGRATION_GUIDE.md#vue-3-plugin](INTEGRATION_GUIDE.md#vue-3-plugin) for a complete example.

---

## Next.js Integration

### How do I use StringBoot with Next.js App Router?

```tsx
// app/layout.tsx
import { StringBootProvider } from '@stringboot/web-sdk/nextjs';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <StringBootProvider
          apiToken={process.env.NEXT_PUBLIC_STRINGBOOT_TOKEN!}
          baseUrl="https://api.stringboot.com"
          defaultLanguage="en"
        >
          {children}
        </StringBootProvider>
      </body>
    </html>
  );
}
```

### How do I use StringBoot with Next.js Pages Router?

```tsx
// pages/_app.tsx
import { useStringBoot } from '@stringboot/web-sdk/react';

function MyApp({ Component, pageProps }) {
  const { initialized, error } = useStringBoot({
    apiToken: process.env.NEXT_PUBLIC_STRINGBOOT_TOKEN!,
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en'
  });

  if (!initialized) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return <Component {...pageProps} />;
}

export default MyApp;
```

### Can I use StringBoot with Server-Side Rendering (SSR)?

Yes! Fetch strings in `getServerSideProps`:

```tsx
export const getServerSideProps = async (context) => {
  await StringBoot.initialize({
    apiToken: process.env.STRINGBOOT_TOKEN!,
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: context.locale || 'en'
  });

  const title = await StringBoot.get('page_title');

  return { props: { title } };
};
```

See [NEXTJS_INTEGRATION.md](NEXTJS_INTEGRATION.md) for complete Next.js guide.

---

## Language Switching

### How do I switch languages?

**Vanilla JS:**
```javascript
await StringBoot.changeLanguage('es');
```

**React:**
```tsx
const [currentLang, setLanguage] = useLanguage();
await setLanguage('es');
```

**Vue:**
```vue
<script setup>
const changeLang = async (newLang) => {
  await StringBoot.changeLanguage(newLang);
  // Reload strings
};
</script>
```

### How do I get the list of available languages?

```javascript
const languages = await StringBoot.getAvailableLanguages();
// Returns: [{ code: 'en', name: 'English', isDefault: true }, ...]

languages.forEach(lang => {
  console.log(`${lang.code}: ${lang.name}`);
});
```

### Do I need to manually refresh the UI after language change?

**React hooks - No!** Hooks automatically re-render:
```tsx
const text = useString('welcome'); // Auto-updates
```

**Vanilla JS - Yes:** Re-fetch strings or use event listeners:
```javascript
StringBoot.on('languageChange', async (newLang) => {
  const text = await StringBoot.get('welcome');
  updateUI(text);
});
```

### Can users switch languages independently of browser settings?

Yes! Stringboot language is independent of browser language:

```javascript
// Switch to Spanish regardless of browser language
await StringBoot.changeLanguage('es');
```

### How do I detect the browser language?

```javascript
const browserLang = navigator.language.split('-')[0]; // "en" from "en-US"
await StringBoot.changeLanguage(browserLang);
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
2. **IndexedDB (L2)** - Persistent browser storage (<50ms)
3. **Network (L3)** - API calls with ETag-based caching

### How much data does Stringboot cache?

By default, **1,000 strings** in memory. IndexedDB stores unlimited strings. Typical usage:
- 1,000 strings ≈ 200-500 KB in memory
- 5,000 strings ≈ 1-2 MB in IndexedDB

### Can I change the cache size?

Yes, during initialization:

```javascript
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  cacheSize: 2000, // Cache 2000 strings in memory
  indexedDBEnabled: true
});
```

### How often does the SDK sync with the network?

- **Initial sync:** On first page load
- **Auto-sync:** When page regains focus (if 30+ minutes since last sync)
- **Manual sync:** Call `StringBoot.refreshFromNetwork()`

Network requests use ETag caching, so if nothing changed, only ~2 KB is downloaded.

### Does Stringboot impact page load time?

**No.** Initialization is asynchronous and non-blocking:
- Cache initialization: <10ms
- IndexedDB setup: <50ms
- Network sync: Runs in background, doesn't block rendering

### Can I preload languages for better performance?

Yes! Preload frequently used languages:

```javascript
await StringBoot.preloadLanguage('en');
await StringBoot.preloadLanguage('es');
```

### How do I clear the cache?

```javascript
// Clear cache for specific language
await StringBoot.clearCache('en');

// Clear all cache (memory + IndexedDB)
await StringBoot.clearAllCache();

// Clear only memory cache (keep IndexedDB)
await StringBoot.clearMemoryCache();
```

### Does StringBoot work with Service Workers?

Yes! StringBoot is compatible with Service Workers and works great for offline-first PWAs.

---

## A/B Testing

### What is A/B testing in Stringboot?

A/B testing lets you test different string variations (e.g., "Sign Up" vs "Get Started") with real users. The SDK automatically shows different variants to different users and integrates with your analytics.

See [AB_TESTING.md](AB_TESTING.md) for complete guide.

### How do I set up A/B testing?

1. **Create experiment** in Stringboot dashboard
2. **Define variants** (e.g., Variant A: "Buy Now", Variant B: "Purchase")
3. **Initialize SDK** with analytics handler:

```javascript
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  analyticsHandler: {
    onExperimentsAssigned: (experiments) => {
      experiments.forEach(exp => {
        gtag('event', 'experiment_assigned', {
          experiment_id: exp.experimentId,
          variant_name: exp.variantName
        });
      });
    }
  }
});
```

The SDK handles the rest automatically!

### Do I need to change my code for A/B tests?

**No!** Continue using strings normally:

```javascript
const ctaText = await StringBoot.get('cta_button');
```

The SDK automatically returns the correct variant based on the user's assignment.

### How does device ID work for A/B testing?

The SDK generates a persistent device ID (stored in LocalStorage) to ensure consistent experiment assignments across browser sessions.

**Provide your own device ID:**

```javascript
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  providedDeviceId: 'user_12345' // Your user ID
});
```

### Can I test experiments locally?

Yes! Check current assignments:

```javascript
const experiments = await StringBoot.getActiveExperiments();
experiments.forEach(exp => {
  console.log(`Experiment: ${exp.experimentId}, Variant: ${exp.variantName}`);
});
```

---

## FAQ Provider

### What is the FAQ Provider?

FAQ Provider lets you deliver dynamic, multilingual FAQs to your users using the same offline-first architecture as string resources. FAQs are organized by tags and sub-tags.

See [INTEGRATION_GUIDE.md#faq-provider](INTEGRATION_GUIDE.md#-faq-provider-integration) for complete guide.

### How do I fetch FAQs?

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
const [faqs, setFaqs] = useState([]);

useEffect(() => {
  StringBoot.FAQ.getFAQs({ tag: 'payments' }).then(setFaqs);
}, []);
```

### What are tags and sub-tags?

**Tags** organize FAQs into categories. **Sub-tags** provide finer filtering:

- **Tag:** "payments"
- **Sub-tags:** ["refunds", "disputes", "methods"]

```javascript
// Get all payment FAQs
const allPaymentFAQs = await StringBoot.FAQ.getFAQs({ tag: 'payments' });

// Get only refund FAQs
const refundFAQs = await StringBoot.FAQ.getFAQs({
  tag: 'payments',
  subTags: ['refunds']
});
```

### Do FAQs support the same language fallback as strings?

Yes! If FAQs aren't available in the requested language, they automatically fall back to English.

### Can I cache FAQs?

Yes! FAQProvider uses the same three-tier caching system as StringBoot. FAQs are cached in memory and IndexedDB for offline access.

---

## Offline Support

### Does Stringboot work offline?

**Yes, 100% offline functionality!** Once strings are synced, the app works perfectly without internet:
- Strings are cached in IndexedDB
- Language switching works offline
- A/B test assignments are cached
- FAQs work offline

### What happens on first visit without internet?

The app won't have strings until it can sync with the network. **Best practice:** Include critical strings as fallbacks in your app.

### How do I know if strings are cached?

```javascript
const stats = await StringBoot.getCacheStats();
console.log(`Cached strings: ${stats.currentSize}`);
```

### Can I force a network refresh?

Yes:

```javascript
const success = await StringBoot.refreshFromNetwork();
if (success) {
  console.log('Strings refreshed!');
}
```

---

## Troubleshooting

### Strings showing as "??key??"

**Possible causes:**

1. **Key doesn't exist** - Check Stringboot dashboard for correct key name
2. **Language not available** - Verify language code is correct (e.g., "en", not "eng")
3. **Network sync failed** - Check internet connection and API token
4. **Not initialized** - Ensure `initialize()` was called

**Debug:**

```javascript
// Enable debug logging
StringBoot.setLogLevel('debug');

// Check initialization
if (!StringBoot.isInitialized()) {
  console.error('StringBoot not initialized!');
}
```

### Strings not updating after changing language

**Possible causes:**

1. **Not using reactive approach** - Use hooks or event listeners
2. **Cache not refreshed** - Call `refreshFromNetwork()` after language change
3. **Language not synced** - Ensure new language is downloaded

**Solution:**

```javascript
await StringBoot.changeLanguage('es');
await StringBoot.refreshFromNetwork('es');
// UI with hooks automatically updates
```

### Network sync failing

**Check these:**

1. **API token valid?** - Verify token in Stringboot dashboard
2. **CORS enabled?** - Ensure your domain is whitelisted
3. **Network available?** - Check browser internet connection
4. **Base URL correct?** - Should be `https://api.stringboot.com`

**Debug network calls:**

```javascript
// Check last sync
StringBoot.setLogLevel('debug');

// Manually refresh
const success = await StringBoot.refreshFromNetwork();
console.log('Refresh success:', success);
```

### React hooks not updating

**Common causes:**

1. **StringBoot not initialized before using hooks**
2. **Not wrapping app in provider**
3. **Using hooks outside React component**

**Solution:**

```tsx
// ✅ Good: Initialize before using hooks
function App() {
  const { initialized } = useStringBoot({/*...*/});

  if (!initialized) return <div>Loading...</div>;

  return <ComponentsUsingHooks />;
}

// ❌ Bad: Using hooks before initialization
function App() {
  const text = useString('key'); // May fail!
  useStringBoot({/*...*/});
  return <div>{text}</div>;
}
```

### IndexedDB quota exceeded

**Solution:**

1. **Reduce cache size:**
```javascript
await StringBoot.initialize({
  cacheSize: 500, // Smaller cache
  indexedDBEnabled: true
});
```

2. **Clear old data:**
```javascript
await StringBoot.clearAllCache();
```

### TypeScript type errors

**Ensure you have type declarations:**

```bash
npm install --save-dev @types/stringboot-web-sdk
```

**Or use type imports:**

```typescript
import StringBoot, { StringBootConfig, FAQ } from '@stringboot/web-sdk';

const config: StringBootConfig = {
  apiToken: 'YOUR_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en'
};
```

---

## Best Practices

### When should I use Stringboot vs traditional i18n files?

**Use Stringboot for:**
- Marketing copy that changes frequently
- A/B tested content
- Seasonal messaging
- User-facing content you want to update quickly
- Multi-region content variations

**Use traditional i18n files for:**
- System-level strings (error messages, validation)
- Critical UI labels
- Fallback strings for offline-first launch
- Strings unlikely to change

### How do I handle string updates in production?

**Recommended workflow:**

1. **Update strings** in Stringboot dashboard
2. **Test in staging** environment first
3. **Publish to production**
4. **Monitor analytics** for issues
5. **Rollback if needed** (instant via dashboard)

No deployment required!

### Should I cache strings indefinitely?

Yes! The SDK automatically invalidates cache when strings are updated on the backend (using ETags). Trust the SDK's cache management.

### How do I test different languages?

**Method 1: Browser language**
```javascript
const browserLang = navigator.language.split('-')[0];
await StringBoot.changeLanguage(browserLang);
```

**Method 2: Query parameter**
```javascript
const urlParams = new URLSearchParams(window.location.search);
const lang = urlParams.get('lang');
if (lang) await StringBoot.changeLanguage(lang);
```

**Method 3: Debug menu (dev only)**
```javascript
if (process.env.NODE_ENV === 'development') {
  window.StringBootDebug = StringBoot;
}
```

### How do I handle pluralization?

Use separate keys for each plural form:

```javascript
const count = 5;
const key = count === 1 ? 'item_one' : 'items_many';
const text = await StringBoot.get(key);
const formatted = text.replace('{count}', count);
```

### Should I use SSR or CSR for strings?

**SSR (Server-Side Rendering):**
- Better SEO
- Faster initial paint
- Good for marketing pages

**CSR (Client-Side Rendering):**
- Simpler implementation
- Better for dynamic content
- Good for web apps

**Recommendation:** Use SSR for public pages, CSR for authenticated areas.

### How do I migrate from i18next to Stringboot?

**Gradual migration approach:**

1. **Phase 1:** Keep existing i18next, add Stringboot for new features
2. **Phase 2:** Migrate high-frequency strings (marketing, CTAs)
3. **Phase 3:** Move remaining strings, keep critical ones as fallbacks
4. **Phase 4:** Monitor for issues, adjust as needed

**Helper function:**

```javascript
async function getStringWithFallback(key) {
  const stringbootValue = await StringBoot.get(key);
  if (stringbootValue.startsWith('??')) {
    // Stringboot key not found, use i18next
    return i18next.t(key);
  }
  return stringbootValue;
}
```

### How do I handle multi-tab synchronization?

StringBoot automatically syncs across tabs using BroadcastChannel API:

```javascript
// Tab 1: Change language
await StringBoot.changeLanguage('es');

// Tab 2: Automatically receives language change event
StringBoot.on('languageChange', (newLang) => {
  console.log(`Language changed to ${newLang} in another tab`);
});
```

---

## Still Have Questions?

- 📧 **Email:** support@stringboot.com
- 📚 **Documentation:** [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)
- 🐛 **Report Issues:** [GitHub Issues](https://github.com/stringboot/web-sdk/issues)
- 💬 **Community:** [Discord](https://discord.gg/stringboot)

---

**Last Updated:** December 2024 | **SDK Version:** 1.0.0+
