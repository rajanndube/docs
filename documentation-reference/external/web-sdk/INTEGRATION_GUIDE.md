# Stringboot Web SDK - Integration Guide

> Comprehensive guide to integrating the Stringboot Web SDK into your web application

**Version:** 1.0.0+ | **Last Updated:** December 2024

---

## Table of Contents

- [Quick Start](#-quick-start)
- [Vanilla JavaScript Integration](#-vanilla-javascript-integration)
- [React Integration](#-react-integration)
- [Vue 3 Integration](#-vue-3-integration)
- [Next.js Integration](#-nextjs-integration)
- [Language Switching](#-language-switching)
- [Performance Optimization](#-performance-optimization)
- [A/B Testing Integration](#-ab-testing-integration)
- [FAQ Provider Integration](#-faq-provider-integration)
- [Additional Resources](#-additional-resources)

---

## 🚀 Quick Start

### Step 1: Installation

```bash
npm install @stringboot/web-sdk
```

Or with Yarn:

```bash
yarn add @stringboot/web-sdk
```

### Step 2: Initialization

#### Vanilla JavaScript / TypeScript

```javascript
import StringBoot from '@stringboot/web-sdk';

// Initialize the SDK
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',  // Optional, defaults to api.stringboot.com
  defaultLanguage: 'en',                   // Optional, defaults to browser language
  debug: false,                             // Optional, enables debug logging
  enableIntegrityCheck: false               // Optional, enables HMAC verification
});

// Get strings
const welcomeMessage = await StringBoot.get('welcome_message');
console.log(welcomeMessage); // "Welcome to our app!"
```

**Configuration Options:**

| Option | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| `apiToken` | `string` | Yes | - | Your Stringboot API token |
| `baseUrl` | `string` | No | `https://api.stringboot.com` | API base URL |
| `defaultLanguage` | `string` | No | Browser language | Initial language code |
| `debug` | `boolean` | No | `false` | Enable debug logging |
| `enableIntegrityCheck` | `boolean` | No | `false` | Enable HMAC signature verification |
| `analyticsHandler` | `object` | No | - | A/B testing analytics callback |

#### React

```tsx
import { useStringBoot } from '@stringboot/web-sdk/react';

function App() {
  const { initialized, error } = useStringBoot({
    apiToken: 'YOUR_API_TOKEN',
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en'
  });

  if (!initialized) return <div>Loading StringBoot...</div>;
  if (error) return <div>Error: {error}</div>;

  return <MyApp />;
}
```

### Step 3: Use Strings

```javascript
// Get a single string
const text = await StringBoot.get('welcome_message');

// Get string with specific language
const spanishText = await StringBoot.get('welcome_message', 'es');

// Watch for updates (reactive)
const unsubscribe = StringBoot.watch('welcome_message', (newValue) => {
  console.log('String updated:', newValue);
});

// React Hook (auto-updates on language change)
import { useString } from '@stringboot/web-sdk/react';
const text = useString('welcome_message');
```

---

## 📦 Vanilla JavaScript Integration

### Basic Usage

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>StringBoot Example</title>
</head>
<body>
  <h1 id="title"></h1>
  <p id="description"></p>
  <button id="languageToggle">Switch to Spanish</button>

  <script type="module">
    import StringBoot from '@stringboot/web-sdk';

    // Initialize
    await StringBoot.initialize({
      apiToken: 'YOUR_API_TOKEN',
      baseUrl: 'https://api.stringboot.com',
      defaultLanguage: 'en'
    });

    // Update UI with strings
    async function updateUI() {
      const title = await StringBoot.get('page_title');
      const description = await StringBoot.get('page_description');

      document.getElementById('title').textContent = title;
      document.getElementById('description').textContent = description;
    }

    // Initial render
    await updateUI();

    // Language switcher
    document.getElementById('languageToggle').addEventListener('click', async () => {
      const currentLang = StringBoot.getCurrentLanguage();
      const newLang = currentLang === 'en' ? 'es' : 'en';

      await StringBoot.changeLanguage(newLang);
      await updateUI();

      document.getElementById('languageToggle').textContent =
        newLang === 'en' ? 'Switch to Spanish' : 'Cambiar a Inglés';
    });
  </script>
</body>
</html>
```

### TypeScript Integration

```typescript
import StringBoot from '@stringboot/web-sdk';

interface Config {
  apiToken: string;
  baseUrl: string;
  defaultLanguage: string;
  cacheSize?: number;
}

async function initializeApp(config: Config): Promise<void> {
  await StringBoot.initialize(config);

  // Type-safe string retrieval
  const welcomeMessage: string = await StringBoot.get('welcome_message');
  console.log(welcomeMessage);
}

// Usage
await initializeApp({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  cacheSize: 1000
});
```

### Event Listeners

Listen for string updates and language changes:

```javascript
// Language change event
const unsubscribe = StringBoot.onLanguageChange((newLanguage) => {
  console.log(`Language changed to: ${newLanguage}`);
  updateUIForLanguage(newLanguage);
});

// String update event (watch specific string)
const unwatch = StringBoot.watch('welcome_message', (newValue) => {
  console.log('String updated:', newValue);
  document.getElementById('title').textContent = newValue;
});

// Cleanup when done
unsubscribe();
unwatch();
```

### Dynamic String Loading

```javascript
// Load multiple strings sequentially
async function loadProductDetails(productId) {
  const name = await StringBoot.get(`product_${productId}_name`);
  const description = await StringBoot.get(`product_${productId}_description`);
  const price = await StringBoot.get(`product_${productId}_price`);

  return { name, description, price };
}

// Or load in parallel
async function loadProductDetailsParallel(productId) {
  const [name, description, price] = await Promise.all([
    StringBoot.get(`product_${productId}_name`),
    StringBoot.get(`product_${productId}_description`),
    StringBoot.get(`product_${productId}_price`)
  ]);

  return { name, description, price };
}

// Usage
const product = await loadProductDetails(123);
```

---

## ⚛️ React Integration

### useStringBoot Hook

Initialize StringBoot in your root App component:

```tsx
import React from 'react';
import { useStringBoot } from '@stringboot/web-sdk/react';

function App() {
  const { initialized, error } = useStringBoot({
    apiToken: process.env.REACT_APP_STRINGBOOT_TOKEN!,
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en',
    debug: true  // Enable debug logging
  });

  if (error) {
    return (
      <div className="error-container">
        <h1>Failed to initialize StringBoot</h1>
        <p>{error}</p>
        <button onClick={() => window.location.reload()}>
          Retry
        </button>
      </div>
    );
  }

  if (!initialized) {
    return (
      <div className="loading-container">
        <div className="spinner" />
        <p>Initializing StringBoot...</p>
      </div>
    );
  }

  return <MyApp />;
}

export default App;
```

**Hook Signature:**
```typescript
useStringBoot(config: StringBootConfig): {
  initialized: boolean;
  error: string | null;
}
```

### useString Hook

Auto-updating strings in components:

```tsx
import { useString } from '@stringboot/web-sdk/react';

function WelcomeScreen() {
  const title = useString('welcome_title');
  const subtitle = useString('welcome_subtitle');
  const ctaButton = useString('cta_button');

  return (
    <div className="welcome-screen">
      <h1>{title}</h1>
      <p>{subtitle}</p>
      <button>{ctaButton}</button>
    </div>
  );
}
```

**Hook Signature:**
```typescript
useString(key: string, lang?: string): string
```

With custom language:

```tsx
function LocalizedContent({ language }: { language: string }) {
  const title = useString('title', language);
  const description = useString('description', language);

  return (
    <div>
      <h2>{title}</h2>
      <p>{description}</p>
    </div>
  );
}
```

### useStrings Hook

Load multiple strings with auto-updates:

```tsx
import { useStrings } from '@stringboot/web-sdk/react';

function ProductCard({ productId }: { productId: number }) {
  const strings = useStrings([
    `product_${productId}_name`,
    `product_${productId}_description`,
    `product_${productId}_price`
  ]);

  return (
    <div className="product-card">
      <h3>{strings[`product_${productId}_name`]}</h3>
      <p>{strings[`product_${productId}_description`]}</p>
      <span className="price">{strings[`product_${productId}_price`]}</span>
    </div>
  );
}
```

**Hook Signature:**
```typescript
useStrings(keys: string[], lang?: string): Record<string, string>
```

### useLanguage Hook

Manage language switching:

```tsx
import { useLanguage, useString } from '@stringboot/web-sdk/react';

function LanguageSwitcher() {
  const [currentLang, setLanguage] = useLanguage();
  const switchingLabel = useString('select_language');

  return (
    <div className="language-switcher">
      <label>{switchingLabel}</label>
      <select
        value={currentLang}
        onChange={(e) => setLanguage(e.target.value)}
      >
        <option value="en">English</option>
        <option value="es">Spanish</option>
        <option value="fr">French</option>
      </select>
    </div>
  );
}
```

**Hook Signature:**
```typescript
useLanguage(): [string, (lang: string) => Promise<void>]
```

### useActiveLanguages Hook

Get available languages from backend:

```tsx
import { useActiveLanguages } from '@stringboot/web-sdk/react';

function LanguageSelector() {
  const { languages, loading, error } = useActiveLanguages();

  if (loading) return <div>Loading languages...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <select>
      {languages.map(lang => (
        <option key={lang.code} value={lang.code}>
          {lang.name} {lang.isDefault && '(Default)'}
        </option>
      ))}
    </select>
  );
}
```

**Hook Signature:**
```typescript
useActiveLanguages(): {
  languages: ActiveLanguage[];
  loading: boolean;
  error: string | null;
}
```

### useSync Hook

Manually trigger synchronization:

```tsx
import { useSync } from '@stringboot/web-sdk/react';

function SyncButton() {
  const { sync, syncing } = useSync();

  return (
    <button onClick={() => sync()} disabled={syncing}>
      {syncing ? 'Syncing...' : 'Sync Now'}
    </button>
  );
}
```

**Hook Signature:**
```typescript
useSync(): {
  sync: (lang?: string) => Promise<void>;
  syncing: boolean;
}
```

### Complete React Example

```tsx
import React from 'react';
import {
  useStringBoot,
  useString,
  useLanguage,
  useStrings,
  useActiveLanguages
} from '@stringboot/web-sdk/react';

function App() {
  const { initialized, error } = useStringBoot({
    apiToken: process.env.REACT_APP_STRINGBOOT_TOKEN!,
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en'
  });

  if (error) return <ErrorScreen error={error} />;
  if (!initialized) return <LoadingScreen />;

  return (
    <div className="app">
      <Header />
      <MainContent />
      <Footer />
    </div>
  );
}

function Header() {
  const [currentLang, setLanguage] = useLanguage();
  const { languages, loading } = useActiveLanguages();
  const appTitle = useString('app_title');

  return (
    <header>
      <h1>{appTitle}</h1>
      {!loading && (
        <select value={currentLang} onChange={(e) => setLanguage(e.target.value)}>
          {languages.map(lang => (
            <option key={lang.code} value={lang.code}>{lang.name}</option>
          ))}
        </select>
      )}
    </header>
  );
}

function MainContent() {
  const strings = useStrings([
    'welcome_message',
    'welcome_subtitle',
    'cta_button'
  ]);

  return (
    <main>
      <h2>{strings.welcome_message}</h2>
      <p>{strings.welcome_subtitle}</p>
      <button>{strings.cta_button}</button>
    </main>
  );
}

function Footer() {
  const copyright = useString('footer_copyright');
  const contact = useString('footer_contact');

  return (
    <footer>
      <p>{copyright}</p>
      <p>{contact}</p>
    </footer>
  );
}

export default App;
```

---

## 🎨 Vue 3 Integration

### Composition API

```vue
<template>
  <div class="app">
    <header>
      <h1>{{ appTitle }}</h1>
      <select v-model="currentLanguage" @change="changeLanguage">
        <option v-for="lang in languages" :key="lang.code" :value="lang.code">
          {{ lang.name }}
        </option>
      </select>
    </header>

    <main>
      <h2>{{ welcomeMessage }}</h2>
      <p>{{ description }}</p>
      <button>{{ ctaButton }}</button>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';
import StringBoot from '@stringboot/web-sdk';

const appTitle = ref('');
const welcomeMessage = ref('');
const description = ref('');
const ctaButton = ref('');
const currentLanguage = ref('en');
const languages = ref([]);

onMounted(async () => {
  // Initialize StringBoot
  await StringBoot.initialize({
    apiToken: import.meta.env.VITE_STRINGBOOT_TOKEN,
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en'
  });

  // Load available languages
  languages.value = await StringBoot.getAvailableLanguages();

  // Load initial strings
  await loadStrings();
});

async function loadStrings() {
  appTitle.value = await StringBoot.get('app_title');
  welcomeMessage.value = await StringBoot.get('welcome_message');
  description.value = await StringBoot.get('description');
  ctaButton.value = await StringBoot.get('cta_button');
}

async function changeLanguage() {
  await StringBoot.changeLanguage(currentLanguage.value);
  await loadStrings();
}

// Watch for language changes
watch(currentLanguage, async (newLang) => {
  await StringBoot.changeLanguage(newLang);
  await loadStrings();
});
</script>
```

### Options API

```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p>{{ subtitle }}</p>
  </div>
</template>

<script>
import StringBoot from '@stringboot/web-sdk';

export default {
  name: 'WelcomeComponent',
  data() {
    return {
      title: '',
      subtitle: '',
      currentLanguage: 'en'
    };
  },
  async mounted() {
    await StringBoot.initialize({
      apiToken: process.env.VUE_APP_STRINGBOOT_TOKEN,
      baseUrl: 'https://api.stringboot.com',
      defaultLanguage: 'en'
    });

    await this.loadStrings();
  },
  methods: {
    async loadStrings() {
      this.title = await StringBoot.get('page_title');
      this.subtitle = await StringBoot.get('page_subtitle');
    },
    async changeLanguage(newLang) {
      await StringBoot.changeLanguage(newLang);
      await this.loadStrings();
    }
  }
};
</script>
```

### Vue 3 Plugin

Create a global Vue plugin:

```typescript
// plugins/stringboot.ts
import { App } from 'vue';
import StringBoot from '@stringboot/web-sdk';

export default {
  install(app: App, options: any) {
    // Initialize StringBoot
    StringBoot.initialize(options);

    // Add global properties
    app.config.globalProperties.$stringboot = StringBoot;

    // Add global mixin for easy access
    app.mixin({
      methods: {
        async $getString(key: string) {
          return await StringBoot.get(key);
        }
      }
    });
  }
};
```

Usage:

```typescript
// main.ts
import { createApp } from 'vue';
import App from './App.vue';
import stringbootPlugin from './plugins/stringboot';

const app = createApp(App);

app.use(stringbootPlugin, {
  apiToken: import.meta.env.VITE_STRINGBOOT_TOKEN,
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en'
});

app.mount('#app');
```

```vue
<!-- In any component -->
<script setup>
import { ref, onMounted, getCurrentInstance } from 'vue';

const title = ref('');
const instance = getCurrentInstance();

onMounted(async () => {
  title.value = await instance?.proxy?.$getString('page_title');
});
</script>
```

---

## 📄 Next.js Integration

### App Router (Next.js 13+)

#### Server Component

```tsx
// app/layout.tsx
import { StringBootProvider } from '@stringboot/web-sdk/nextjs';

export default function RootLayout({
  children,
}: {
  children: React.Node;
}) {
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

#### Client Component

```tsx
// app/components/WelcomeSection.tsx
'use client';

import { useString, useLanguage } from '@stringboot/web-sdk/react';

export default function WelcomeSection() {
  const title = useString('welcome_title');
  const subtitle = useString('welcome_subtitle');
  const [lang, setLang] = useLanguage();

  return (
    <section>
      <h1>{title}</h1>
      <p>{subtitle}</p>
      <button onClick={() => setLang(lang === 'en' ? 'es' : 'en')}>
        Switch Language
      </button>
    </section>
  );
}
```

### Pages Router (Next.js 12 and below)

```tsx
// pages/_app.tsx
import type { AppProps } from 'next/app';
import { useStringBoot } from '@stringboot/web-sdk/react';

function MyApp({ Component, pageProps }: AppProps) {
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

### Server-Side Rendering (SSR)

```tsx
// pages/index.tsx
import { GetServerSideProps } from 'next';
import StringBoot from '@stringboot/web-sdk';

interface Props {
  welcomeMessage: string;
  title: string;
}

export default function HomePage({ welcomeMessage, title }: Props) {
  return (
    <div>
      <h1>{title}</h1>
      <p>{welcomeMessage}</p>
    </div>
  );
}

export const getServerSideProps: GetServerSideProps<Props> = async (context) => {
  // Initialize StringBoot on server
  await StringBoot.initialize({
    apiToken: process.env.STRINGBOOT_TOKEN!,
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: context.locale || 'en'
  });

  // Fetch strings server-side
  const [welcomeMessage, title] = await StringBoot.getBatch([
    'welcome_message',
    'page_title'
  ]);

  return {
    props: {
      welcomeMessage,
      title
    }
  };
};
```

### Static Generation (SSG)

```tsx
// pages/about.tsx
import { GetStaticProps } from 'next';
import StringBoot from '@stringboot/web-sdk';

interface Props {
  pageTitle: string;
  content: string;
}

export default function AboutPage({ pageTitle, content }: Props) {
  return (
    <div>
      <h1>{pageTitle}</h1>
      <div dangerouslySetInnerHTML={{ __html: content }} />
    </div>
  );
}

export const getStaticProps: GetStaticProps<Props> = async () => {
  await StringBoot.initialize({
    apiToken: process.env.STRINGBOOT_TOKEN!,
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en'
  });

  const [pageTitle, content] = await StringBoot.getBatch([
    'about_page_title',
    'about_page_content'
  ]);

  return {
    props: {
      pageTitle,
      content
    },
    revalidate: 3600 // Revalidate every hour
  };
};
```

For complete Next.js integration details, see [NEXTJS_INTEGRATION.md](NEXTJS_INTEGRATION.md)

---

## 🌍 Language Switching

### Get Available Languages

```javascript
const languages = await StringBoot.getAvailableLanguages();
// Returns: [{ code: 'en', name: 'English', isDefault: true }, ...]

languages.forEach(lang => {
  console.log(`${lang.code}: ${lang.name}`);
});
```

### Change Language

```javascript
// Simple language change
await StringBoot.changeLanguage('es');

// With callback
await StringBoot.changeLanguage('fr', () => {
  console.log('Language changed to French');
  updateUI();
});
```

### Get Current Language

```javascript
const currentLang = StringBoot.getCurrentLanguage();
console.log(`Current language: ${currentLang}`); // "en"
```

### React Language Switcher

```tsx
import { useLanguage, useString } from '@stringboot/web-sdk/react';

function LanguageSelector() {
  const [currentLang, setLanguage, availableLanguages] = useLanguage();
  const selectLabel = useString('select_language');

  return (
    <div className="language-selector">
      <label>{selectLabel}</label>
      <div className="language-options">
        {availableLanguages.map(lang => (
          <button
            key={lang.code}
            className={currentLang === lang.code ? 'active' : ''}
            onClick={() => setLanguage(lang.code)}
          >
            {lang.name}
            {lang.isDefault && <span className="badge">Default</span>}
          </button>
        ))}
      </div>
    </div>
  );
}
```

### Vanilla JS Language Switcher

```javascript
function createLanguageSwitcher() {
  const container = document.createElement('div');
  container.className = 'language-switcher';

  StringBoot.getAvailableLanguages().then(languages => {
    languages.forEach(lang => {
      const button = document.createElement('button');
      button.textContent = lang.name;
      button.onclick = async () => {
        await StringBoot.changeLanguage(lang.code);
        updateAllStrings();
        highlightActiveButton(button);
      };

      if (lang.code === StringBoot.getCurrentLanguage()) {
        button.classList.add('active');
      }

      container.appendChild(button);
    });
  });

  return container;
}

// Usage
document.body.appendChild(createLanguageSwitcher());
```

---

## ⚡ Performance Optimization

### Preloading Languages

Preload frequently used languages to improve performance:

```javascript
// Preload languages on app initialization
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  preloadLanguages: ['en', 'es', 'fr'] // Preload these languages
});

// Or manually preload after initialization
await StringBoot.preloadLanguage('de');
```

### Batch String Retrieval

Fetch multiple strings in a single network request:

```javascript
// ✅ Good: Batch retrieval (1 request)
const [title, subtitle, cta] = await StringBoot.getBatch([
  'page_title',
  'page_subtitle',
  'cta_button'
]);

// ❌ Avoid: Individual requests (3 requests)
const title = await StringBoot.get('page_title');
const subtitle = await StringBoot.get('page_subtitle');
const cta = await StringBoot.get('cta_button');
```

### Cache Configuration

Configure cache size for optimal performance:

```javascript
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  cacheSize: 2000, // Store 2000 strings in memory
  indexedDBEnabled: true, // Enable IndexedDB for offline support
  indexedDBName: 'stringboot-cache' // Custom database name
});
```

### Cache Statistics

Monitor cache performance:

```javascript
const stats = await StringBoot.getCacheStats();

console.log(`Cache size: ${stats.currentSize}`);
console.log(`Hit rate: ${stats.hitRate * 100}%`);
console.log(`Miss rate: ${stats.missRate * 100}%`);
console.log(`Total requests: ${stats.totalRequests}`);
```

### Clear Cache

```javascript
// Clear cache for specific language
await StringBoot.clearCache('en');

// Clear all cache
await StringBoot.clearAllCache();

// Clear only memory cache (keep IndexedDB)
await StringBoot.clearMemoryCache();
```

### Network Refresh

Manually refresh strings from the network:

```javascript
// Refresh strings for current language
await StringBoot.refreshFromNetwork();

// Refresh for specific language
await StringBoot.refreshFromNetwork('es');

// Force refresh (bypass ETag check)
await StringBoot.refreshFromNetwork('en', { force: true });
```

### Background Sync

Enable automatic background sync:

```javascript
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  backgroundSync: true, // Enable background sync
  syncInterval: 15 * 60 * 1000 // Sync every 15 minutes
});

// Or manually control sync
StringBoot.startBackgroundSync(30 * 60 * 1000); // Every 30 minutes
StringBoot.stopBackgroundSync();
```

---

## 🧪 A/B Testing Integration

> **Available in:** SDK v1.0.0+

The Stringboot Web SDK includes first-class A/B testing support, allowing you to test different string variations with your users.

### Overview

- **Automatic Assignment** - Users are automatically assigned to experiment variants
- **Device ID Management** - Consistent assignments across browser sessions (LocalStorage)
- **Analytics Integration** - Google Analytics 4, Mixpanel, Amplitude, Segment support
- **Offline Support** - Assignments cached in IndexedDB
- **No Code Changes** - Experiments managed from Stringboot dashboard

### Initialization with A/B Testing

```javascript
import StringBoot from '@stringboot/web-sdk';

await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  analyticsHandler: {
    onExperimentsAssigned: (experiments) => {
      experiments.forEach(exp => {
        console.log(`Assigned to ${exp.experimentId}: ${exp.variantName}`);

        // Track in your analytics
        analytics.track('Experiment Assigned', {
          experimentId: exp.experimentId,
          variantName: exp.variantName,
          variantId: exp.variantId
        });
      });
    }
  }
});
```

### Device ID Management

The SDK automatically generates and persists a device ID in LocalStorage for consistent experiment assignments:

```javascript
// Get current device ID
const deviceId = StringBoot.getDeviceId();
console.log(`Device ID: ${deviceId}`);

// Provide your own device ID (e.g., user ID after login)
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  providedDeviceId: 'user_12345' // Use your own ID
});
```

### Analytics Integration

#### Google Analytics 4

```javascript
import StringBoot from '@stringboot/web-sdk';

await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  analyticsHandler: {
    onExperimentsAssigned: (experiments) => {
      experiments.forEach(exp => {
        // Track event
        gtag('event', 'experiment_assigned', {
          experiment_id: exp.experimentId,
          variant_name: exp.variantName,
          variant_id: exp.variantId
        });

        // Set user property for segmentation
        gtag('set', 'user_properties', {
          [`exp_${exp.experimentId}`]: exp.variantName
        });
      });
    }
  }
});
```

#### Mixpanel

```javascript
import mixpanel from 'mixpanel-browser';

await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  analyticsHandler: {
    onExperimentsAssigned: (experiments) => {
      experiments.forEach(exp => {
        // Track event
        mixpanel.track('Experiment Assigned', {
          experiment_id: exp.experimentId,
          variant_name: exp.variantName,
          variant_id: exp.variantId
        });

        // Register super property
        mixpanel.register({
          [`exp_${exp.experimentId}`]: exp.variantName
        });
      });
    }
  }
});
```

#### Amplitude

```javascript
import amplitude from 'amplitude-js';

await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  analyticsHandler: {
    onExperimentsAssigned: (experiments) => {
      experiments.forEach(exp => {
        // Track event
        amplitude.getInstance().logEvent('Experiment Assigned', {
          experiment_id: exp.experimentId,
          variant_name: exp.variantName,
          variant_id: exp.variantId
        });

        // Set user property
        const identify = new amplitude.Identify()
          .set(`exp_${exp.experimentId}`, exp.variantName);
        amplitude.getInstance().identify(identify);
      });
    }
  }
});
```

#### Segment

```javascript
import analytics from '@segment/analytics-next';

await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  analyticsHandler: {
    onExperimentsAssigned: (experiments) => {
      experiments.forEach(exp => {
        // Track event
        analytics.track('Experiment Assigned', {
          experimentId: exp.experimentId,
          variantName: exp.variantName,
          variantId: exp.variantId
        });

        // Identify user with experiment property
        analytics.identify({
          [`exp_${exp.experimentId}`]: exp.variantName
        });
      });
    }
  }
});
```

### React Integration

```tsx
import { useStringBoot, useString } from '@stringboot/web-sdk/react';

function App() {
  const { initialized } = useStringBoot({
    apiToken: process.env.REACT_APP_STRINGBOOT_TOKEN!,
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en',
    analyticsHandler: {
      onExperimentsAssigned: (experiments) => {
        experiments.forEach(exp => {
          // Your analytics tracking
          console.log(`Experiment: ${exp.experimentId}, Variant: ${exp.variantName}`);
        });
      }
    }
  });

  if (!initialized) return <div>Loading...</div>;

  return <MyApp />;
}

function CTAButton() {
  // This button text is part of an A/B test
  // SDK automatically shows correct variant based on user's assignment
  const buttonText = useString('onboarding_cta_button');

  return (
    <button onClick={handleCTA}>
      {buttonText}
    </button>
  );
}
```

### How It Works

1. **Backend Setup** - Create experiments in Stringboot dashboard
2. **String Variants** - Define different string variations for each experiment
3. **Automatic Assignment** - SDK receives assignments from API
4. **Analytics Callback** - Your analytics handler is notified
5. **String Delivery** - Correct variant is automatically shown to user

**Example:**

```javascript
// On Stringboot backend:
// Experiment: "onboarding_cta_test"
// Variant A: "Get Started" (50% of users)
// Variant B: "Start Free Trial" (50% of users)

// In your app:
const buttonText = await StringBoot.get('onboarding_cta_button');
// Returns either "Get Started" or "Start Free Trial" based on user's assignment

// Analytics handler automatically receives notification for tracking
```

### Testing Experiments

```javascript
// View active experiments (for debugging)
const experiments = await StringBoot.getActiveExperiments();
experiments.forEach(exp => {
  console.log(`Experiment: ${exp.experimentId}`);
  console.log(`Variant: ${exp.variantName} (ID: ${exp.variantId})`);
});

// Get device ID
const deviceId = StringBoot.getDeviceId();
console.log(`Device ID: ${deviceId}`);
```

### Best Practices

1. **Initialize Early** - Set up analytics handler before any strings are fetched
2. **Use Device ID** - Provide consistent user ID for logged-in users
3. **Track Conversions** - Log conversion events with experiment context:

```javascript
// Track conversion with experiment context
const experiments = await StringBoot.getActiveExperiments();

gtag('event', 'purchase', {
  value: 99.99,
  // Include experiment assignments as event parameters
  ...experiments.reduce((acc, exp) => ({
    ...acc,
    [`exp_${exp.experimentId}`]: exp.variantName
  }), {})
});
```

4. **Test Thoroughly** - Verify analytics events before launching experiments

For complete details, see [AB_TESTING.md](AB_TESTING.md)

---

## 📋 FAQ Provider Integration

> **Available in:** SDK v1.1.0+

The FAQ Provider enables you to deliver dynamic, multilingual FAQ content to your users with the same offline-first, cached architecture as string resources.

### Overview

FAQProvider offers:
- **Tag-based organization** - Categorize FAQs by primary tag and optional sub-tags
- **Multi-language support** - Automatic language fallback to English
- **Offline-first** - Three-tier caching (memory → IndexedDB → network)
- **Delta sync** - Only download changed FAQs
- **Reactive updates** - Real-time UI updates when FAQs change

### Initialization

**IMPORTANT:** FAQ Provider requires **separate initialization** after StringBoot:

```javascript
import StringBoot from '@stringboot/web-sdk';

// Step 1: Initialize StringBoot
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en'
});

// Step 2: Initialize FAQ Provider separately
await StringBoot.FAQ.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  cacheSize: 200  // Optional: LRU cache size (default: 200)
});

// Now FAQ Provider is ready to use
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'payments', lang: 'en', allowNetworkFetch: true });
```

**FAQ Configuration Options:**

| Option | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| `apiToken` | `string` | Yes | - | Your Stringboot API token |
| `baseUrl` | `string` | No | `https://api.stringboot.com` | API base URL |
| `cacheSize` | `number` | No | `200` | Memory LRU cache size |

### Tag-Based Filtering (2-Level System)

**Tags are MANDATORY, sub-tags are OPTIONAL:**

```javascript
// Fetch ALL FAQs (use "all" tag)
const allFAQs = await StringBoot.FAQ.getFAQs({
  tag: 'all',
  lang: 'en',
  allowNetworkFetch: true
});

// Fetch FAQs by specific tag
const identityFAQs = await StringBoot.FAQ.getFAQs({
  tag: 'Identity Verification',
  lang: 'en',
  allowNetworkFetch: true
});

// Fetch with sub-tag filtering (2-level filtering)
const documentFAQs = await StringBoot.FAQ.getFAQs({
  tag: 'Identity Verification',
  subTags: ['Document Upload', 'Liveness Check'],
  lang: 'en',
  allowNetworkFetch: true
});

// Display FAQs
identityFAQs.forEach(faq => {
  console.log(`Q: ${faq.question}`);
  console.log(`A: ${faq.answer}`);
  console.log(`Tag: ${faq.tag}, Sub-tags: ${faq.subTags.join(', ')}`);
});
```

**Key Points:**
- Use `tag: 'all'` to fetch the entire FAQ catalog
- Tag parameter is mandatory (cannot be empty)
- Sub-tags are optional and filter within the tag
- `allowNetworkFetch: true` enables network fallback if cache is empty

#### FAQ Data Model

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

### Vanilla JavaScript FAQ List

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>FAQs - Identity Verification</title>
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
      font-weight: 600;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .faq-answer {
      padding: 16px;
      display: none;
      border-top: 1px solid #ddd;
    }
    .faq-item.expanded .faq-answer {
      display: block;
    }
    .faq-item.expanded .chevron {
      transform: rotate(180deg);
    }
  </style>
</head>
<body>
  <h1>Frequently Asked Questions</h1>

  <!-- Filter chips -->
  <div id="filter-chips"></div>

  <!-- FAQ List -->
  <div id="faq-container"></div>

  <script type="module">
    import StringBoot from '@stringboot/web-sdk';

    let currentTag = 'Identity Verification';
    let selectedSubTags = [];

    // Initialize StringBoot first
    await StringBoot.initialize({
      apiToken: 'YOUR_API_TOKEN',
      baseUrl: 'https://api.stringboot.com',
      defaultLanguage: 'en'
    });

    // Then initialize FAQ Provider
    await StringBoot.FAQ.initialize({
      apiToken: 'YOUR_API_TOKEN',
      baseUrl: 'https://api.stringboot.com',
      cacheSize: 200
    });

    // Load and render FAQs
    async function loadFAQs() {
      const faqs = await StringBoot.FAQ.getFAQs({
        tag: currentTag,
        subTags: selectedSubTags.length > 0 ? selectedSubTags : undefined,
        lang: 'en',
        allowNetworkFetch: true
      });

      renderFAQs(faqs);
    }

    function renderFAQs(faqs) {
      const container = document.getElementById('faq-container');
      container.innerHTML = '';

      faqs.forEach(faq => {
        const faqItem = document.createElement('div');
        faqItem.className = 'faq-item';
        faqItem.innerHTML = `
          <div class="faq-question">
            <span>${faq.question}</span>
            <span class="chevron">▼</span>
          </div>
          <div class="faq-answer">${faq.answer}</div>
        `;

        // Toggle expand/collapse
        faqItem.querySelector('.faq-question').addEventListener('click', () => {
          faqItem.classList.toggle('expanded');
        });

        container.appendChild(faqItem);
      });
    }

    // Initial load
    await loadFAQs();
  </script>
</body>
</html>
```

### React Integration

```tsx
import React, { useState, useEffect } from 'react';
import { useLanguage } from '@stringboot/web-sdk/react';
import StringBoot from '@stringboot/web-sdk';

interface FAQ {
  id: number;
  question: string;
  answer: string;
  tag: string;
  subTags: string[];
}

function FAQSection() {
  const [currentLang] = useLanguage();
  const [faqs, setFaqs] = useState<FAQ[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [faqInitialized, setFaqInitialized] = useState(false);
  const [expandedIds, setExpandedIds] = useState<Set<number>>(new Set());
  const [selectedTag, setSelectedTag] = useState<string>('all');
  const [selectedSubTags, setSelectedSubTags] = useState<string[]>([]);

  const availableTags = ['all', 'Identity Verification', 'Transfers', 'Referrals'];
  const availableSubTags = ['Document Upload', 'Verification Failed', 'Liveness Check'];

  // Initialize FAQ Provider once
  useEffect(() => {
    async function initializeFAQ() {
      await StringBoot.FAQ.initialize({
        apiToken: 'YOUR_API_TOKEN',
        baseUrl: 'https://api.stringboot.com',
        cacheSize: 200
      });
      setFaqInitialized(true);
    }
    initializeFAQ();
  }, []);

  // Load FAQs when tag, sub-tags, or language changes
  useEffect(() => {
    if (faqInitialized) {
      loadFAQs();
    }
  }, [selectedTag, selectedSubTags, currentLang, faqInitialized]);

  async function loadFAQs() {
    setIsLoading(true);
    try {
      const fetchedFAQs = await StringBoot.FAQ.getFAQs({
        tag: selectedTag,
        subTags: selectedSubTags.length > 0 ? selectedSubTags : undefined,
        lang: currentLang,
        allowNetworkFetch: true
      });
      setFaqs(fetchedFAQs);
    } catch (error) {
      console.error('Failed to load FAQs:', error);
      setFaqs([]);
    } finally {
      setIsLoading(false);
    }
  }

  function toggleExpanded(id: number) {
    const newExpanded = new Set(expandedIds);
    if (newExpanded.has(id)) {
      newExpanded.delete(id);
    } else {
      newExpanded.add(id);
    }
    setExpandedIds(newExpanded);
  }

  function toggleSubTag(subTag: string) {
    setSelectedSubTags(prev =>
      prev.includes(subTag)
        ? prev.filter(t => t !== subTag)
        : [...prev, subTag]
    );
  }

  if (isLoading) return <div>Loading FAQs...</div>;

  return (
    <div className="faq-screen">
      <h1>Frequently Asked Questions</h1>

      {/* Tag Selector */}
      <div className="tag-selector">
        <label>Category:</label>
        {availableTags.map(tag => (
          <button
            key={tag}
            className={selectedTag === tag ? 'active' : ''}
            onClick={() => {
              setSelectedTag(tag);
              setSelectedSubTags([]);
            }}
          >
            {tag}
          </button>
        ))}
      </div>

      {/* Sub-tag Filter Chips */}
      {selectedTag !== 'all' && (
        <div className="filter-chips">
          <label>Filter:</label>
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
      )}

      {/* FAQ List */}
      <div className="faq-list">
        {faqs.length === 0 ? (
          <p>No FAQs found for this filter.</p>
        ) : (
          faqs.map(faq => (
            <div key={faq.id} className="faq-item">
              <div
                className="faq-question"
                onClick={() => toggleExpanded(faq.id)}
              >
                <span>{faq.question}</span>
                <span className={expandedIds.has(faq.id) ? 'chevron expanded' : 'chevron'}>
                  ▼
                </span>
              </div>
              {expandedIds.has(faq.id) && (
                <div
                  className="faq-answer"
                  dangerouslySetInnerHTML={{ __html: faq.answer }}
                />
              )}
            </div>
          ))
        )}
      </div>

      <button onClick={loadFAQs}>Refresh FAQs</button>
    </div>
  );
}

export default FAQScreen;
```

### Vue 3 Integration

```vue
<template>
  <div class="faq-screen">
    <h1>Frequently Asked Questions</h1>
    <p>Tag: {{ currentTag }}</p>

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
    <div v-else-if="faqs.length === 0">No FAQs found.</div>
    <div v-else class="faq-list">
      <div
        v-for="faq in faqs"
        :key="faq.id"
        class="faq-item"
        :class="{ expanded: expandedIds.has(faq.id) }"
      >
        <div class="faq-question" @click="toggleExpanded(faq.id)">
          <span>{{ faq.question }}</span>
          <span class="chevron">▼</span>
        </div>
        <div v-if="expandedIds.has(faq.id)" class="faq-answer" v-html="faq.answer" />
      </div>
    </div>

    <button @click="loadFAQs">Refresh FAQs</button>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue';
import StringBoot from '@stringboot/web-sdk';

const faqs = ref([]);
const isLoading = ref(true);
const expandedIds = ref(new Set());
const selectedSubTags = ref([]);

const currentTag = 'Identity Verification';
const availableSubTags = ['Document Upload', 'Verification Failed', 'Liveness Check'];

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

function toggleExpanded(id) {
  if (expandedIds.value.has(id)) {
    expandedIds.value.delete(id);
  } else {
    expandedIds.value.add(id);
  }
  expandedIds.value = new Set(expandedIds.value); // Trigger reactivity
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

### Tag-Based Filtering

FAQs are organized by **primary tag** and optional **sub-tags** for precise categorization:

```javascript
// Get all FAQs for "payments" tag
const allPaymentFAQs = await StringBoot.FAQ.getFAQs({ tag: 'payments' });

// Get only refund-related FAQs
const refundFAQs = await StringBoot.FAQ.getFAQs({
  tag: 'payments',
  subTags: ['refunds']
});

// Get FAQs matching multiple sub-tags (OR filter)
const paymentMethodFAQs = await StringBoot.FAQ.getFAQs({
  tag: 'payments',
  subTags: ['card', 'bank_transfer', 'wallet']
});
```

**Common Tag Hierarchies:**

| Primary Tag | Sub-Tags |
|------------|----------|
| `payments` | `refunds`, `disputes`, `methods`, `failed` |
| `account` | `profile`, `security`, `deletion`, `privacy` |
| `identity_verification` | `document_upload`, `verification_failed`, `liveness` |
| `support` | `contact`, `escalation`, `feedback` |

### Language Fallback

FAQ Provider automatically falls back to English if FAQs aren't available in the requested language:

```javascript
// User's browser is set to "fr" (French)
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

**Check which language was returned:**

```javascript
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'payments', lang: 'fr' });
if (faqs.length > 0) {
  const actualLanguage = faqs[0].languageCode;
  if (actualLanguage === 'en' && 'fr' !== 'en') {
    showLanguageFallbackNotice();
  }
}
```

### Network Refresh

Manually refresh FAQs from the network (uses delta sync for efficiency):

```javascript
// Refresh FAQs for current language
await StringBoot.FAQ.refreshFromNetwork();

// Refresh for specific language
await StringBoot.FAQ.refreshFromNetwork('es');
```

### Performance & Caching

FAQ Provider uses the same three-tier caching strategy as StringBoot:

1. **Memory Cache (L1)** - LRU cache for instant access
2. **IndexedDB (L2)** - Persistent offline storage
3. **Network (L3)** - Delta sync for changed FAQs only

**Cache Performance:**
- Memory cache hit: <5ms
- IndexedDB hit: <50ms
- Network (delta sync): Downloads only changed FAQs since last sync
- Full catalog: Only on first visit or cache invalidation

### Best Practices

**1. Use Tag-Based Organization**

```javascript
// ✅ Good: Specific tag + sub-tags
await StringBoot.FAQ.getFAQs({
  tag: 'identity_verification',
  subTags: ['document_upload', 'liveness_check']
});

// ❌ Avoid: Generic tags that return too many FAQs
await StringBoot.FAQ.getFAQs({ tag: 'general' });
```

**2. Refresh on App Load**

```javascript
// On app initialization
await StringBoot.initialize({/*...*/});
await StringBoot.FAQ.refreshFromNetwork();
```

**3. Handle Empty States**

```javascript
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'payments' });
if (faqs.length === 0) {
  // Show "No FAQs available" message or contact support button
  showEmptyState();
}
```

**4. Batch FAQ Loading**

```javascript
// Load FAQs for multiple tags in parallel
const [paymentFAQs, accountFAQs, supportFAQs] = await Promise.all([
  StringBoot.FAQ.getFAQs({ tag: 'payments' }),
  StringBoot.FAQ.getFAQs({ tag: 'account' }),
  StringBoot.FAQ.getFAQs({ tag: 'support' })
]);
```

### Troubleshooting

**FAQs not appearing:**
```javascript
// 1. Check initialization
if (!StringBoot.isInitialized()) {
  console.error('StringBoot not initialized!');
}

// 2. Enable logging
StringBoot.setLogLevel('debug');

// 3. Check network sync
const success = await StringBoot.FAQ.refreshFromNetwork();
console.log('Network sync:', success);

// 4. Verify tag/subTag spelling (case-sensitive!)
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'payments' });
```

**Language fallback not working:**
```javascript
// Check actual language returned
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'payments', lang: 'fr' });
if (faqs.length > 0) {
  const actualLang = faqs[0].languageCode;
  console.log(`Requested: fr, Got: ${actualLang}`);
}
```

For complete details, see [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md)

---

## 📚 Additional Resources

- [QUICKSTART.md](QUICKSTART.md) - 10-minute integration guide
- [AB_TESTING.md](AB_TESTING.md) - Complete A/B testing guide
- [NEXTJS_INTEGRATION.md](NEXTJS_INTEGRATION.md) - Detailed Next.js integration
- [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md) - Advanced SDK features
- [API_REFERENCE.md](API_REFERENCE.md) - Complete API documentation
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Common issues & solutions
---

## 🆘 Support

- 📧 Email: support@stringboot.com
- 🐛 Issues: [GitHub Issues](https://github.com/stringboot/web-sdk/issues)
- 📚 Docs: [docs.stringboot.com](https://docs.stringboot.com)

---

**Made with ❤️ by the Stringboot Team**

SDK Version: 1.0.0 | Last Updated: December 2024 | A/B Testing & FAQ Support Added
