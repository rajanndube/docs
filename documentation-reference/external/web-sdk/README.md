# Stringboot Web SDK - Complete Integration Guide

**Verified for:** v1.0.0
**Last verified on:** 2025-11-07
**Audience:** Web developers (Vanilla JavaScript and React)

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

The **Stringboot Web SDK** provides high-performance internationalization (i18n) for web applications using the String-Sync v2 protocol. It features smart multi-layered caching, offline support, and reactive string updates with zero dependencies.

### Key Features

- **Zero Dependencies:** Lightweight with no external dependencies
- **Offline-First:** Works seamlessly without network via IndexedDB
- **Smart Caching:** Multi-layer caching (memory, IndexedDB, localStorage metadata)
- **Delta Sync:** Only downloads changed strings using String-Sync v2 protocol
- **Reactive Updates:** Watch API for automatic UI updates when strings change
- **React Integration:** 6 optimized hooks with built-in state management
- **TypeScript Support:** Fully typed with JSDoc documentation
- **Local Development:** Works with localhost backend during development

### Browser Support

| Browser | Minimum Version | Notes |
|---------|-----------------|-------|
| **Chrome** | 87+ | Full support |
| **Firefox** | 78+ | Full support |
| **Safari** | 14+ | Full support |
| **Edge** | 88+ | Full support |

---

## Installation & Setup

### Option 1: NPM Installation

```bash
npm install @stringboot/web-sdk
```

Or with yarn:
```bash
yarn add @stringboot/web-sdk
```

Or with pnpm:
```bash
pnpm add @stringboot/web-sdk
```

### Option 2: CDN Installation

For quick prototyping without a build step:

```html
<script src="https://cdn.jsdelivr.net/npm/@stringboot/web-sdk@1.0.0/dist/stringboot.umd.js"></script>

<script>
  const { StringBoot } = window.Stringboot;
</script>
```

### Local Development Setup

For development, you can run the SDK against a local backend:

1. Start your local Stringboot server on `http://localhost:8000`
2. Configure the SDK with the local URL (see initialization examples below)

---

## Initialization

### Vanilla JavaScript - Basic Initialization

Based on actual demo implementation (`index.html` lines 255-260):

```javascript
import StringBoot from '@stringboot/web-sdk';

// Initialize SDK
async function init() {
  try {
    // Local development configuration
    await StringBoot.initialize({
      apiToken: '700fdd40-8241-4035-94fe-c7e1713ba9ac',
      baseUrl: 'http://localhost:8000',
      defaultLanguage: 'en',
      debug: true
    });

    console.log('✓ SDK initialized successfully');

    // Now you can use the SDK
    await loadStrings();

  } catch (error) {
    console.error('✗ Initialization failed:', error);
  }
}

init();
```

### Vanilla JavaScript - Production Configuration

```javascript
import StringBoot from '@stringboot/web-sdk';

await StringBoot.initialize({
  apiToken: 'your-production-api-token',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  debug: false  // Disable debug logging in production
});
```

### React - Complete App Setup

Based on actual demo implementation (`react-example.tsx`):

```jsx
import React from 'react';
import {
  useString,
  useStrings,
  useLanguage,
  useActiveLanguages,
  useStringBoot,
  useSync,
} from '@stringboot/web-sdk/react';

/**
 * Main App Component with initialization
 */
export function App() {
  // Initialize SDK and handle loading/error states
  const { initialized, error } = useStringBoot({
    apiToken: 'sk_live_your_api_token_here',
    baseUrl: 'http://localhost:8000', // or 'https://api.stringboot.com' for production
    defaultLanguage: 'en',
    debug: true,
  });

  if (!initialized) {
    return (
      <div className="loading">
        <h1>Loading StringBoot SDK...</h1>
        <div className="spinner"></div>
      </div>
    );
  }

  if (error) {
    return (
      <div className="error">
        <h1>Error initializing SDK</h1>
        <p>{error}</p>
      </div>
    );
  }

  return (
    <div className="app">
      <Header />
      <LanguageSwitcher />
      <MainContent />
      <SyncControls />
    </div>
  );
}
```

---

## Usage Examples

### Example 1: Vanilla JS - Setting Up Multiple Watchers

Based on actual implementation (`index.html` lines 266-276):

```javascript
import StringBoot from '@stringboot/web-sdk';

// Set up watchers for reactive updates
// Watchers automatically update the UI when language changes
StringBoot.watch('welcome_message', (value) => {
  document.getElementById('welcome').textContent = value;
});

StringBoot.watch('goodbye_message', (value) => {
  document.getElementById('goodbye').textContent = value;
});

StringBoot.watch('app_title', (value) => {
  document.getElementById('appTitle').textContent = value;
});

// Load initial strings
await loadStrings();
```

### Example 2: Vanilla JS - Getting Strings Manually

```javascript
import StringBoot from '@stringboot/web-sdk';

async function loadStrings() {
  try {
    const welcome = await StringBoot.get('welcome_message');
    const goodbye = await StringBoot.get('faq-page-title');
    const appTitle = await StringBoot.get('toast_refresh_success');

    document.getElementById('welcome').textContent = welcome;
    document.getElementById('goodbye').textContent = goodbye;
    document.getElementById('appTitle').textContent = appTitle;
  } catch (error) {
    console.error('Failed to load strings:', error);
  }
}
```

### Example 3: Vanilla JS - Language Change with Status Updates

Based on actual implementation (`index.html` lines 303-321):

```javascript
import StringBoot from '@stringboot/web-sdk';

const languageSelect = document.getElementById('language');
const statusEl = document.getElementById('status');

languageSelect.addEventListener('change', async (e) => {
  const currentLang = e.target.value;

  try {
    statusEl.className = 'status loading';
    statusEl.textContent = `Changing language to ${currentLang}...`;

    await StringBoot.changeLanguage(currentLang);

    statusEl.className = 'status success';
    statusEl.textContent = `✓ Language changed to ${currentLang}`;

    // Strings will auto-update via watchers
  } catch (error) {
    statusEl.className = 'status error';
    statusEl.textContent = `✗ Language change failed: ${error.message}`;
  }
});
```

### Example 4: Vanilla JS - Sync with Button Disabled States

Based on actual implementation (`index.html` lines 323-340):

```javascript
import StringBoot from '@stringboot/web-sdk';

const syncBtn = document.getElementById('syncBtn');
const statusEl = document.getElementById('status');

syncBtn.addEventListener('click', async () => {
  syncBtn.disabled = true;
  syncBtn.textContent = 'Syncing...';
  statusEl.className = 'status loading';
  statusEl.textContent = 'Syncing with server...';

  try {
    await StringBoot.syncNow();
    statusEl.className = 'status success';
    statusEl.textContent = '✓ Sync completed successfully';
  } catch (error) {
    statusEl.className = 'status error';
    statusEl.textContent = `✗ Sync failed: ${error.message}`;
  } finally {
    syncBtn.disabled = false;
    syncBtn.textContent = 'Sync Now';
  }
});
```

### Example 5: Vanilla JS - Force Refresh with Complete Error Handling

Based on actual implementation (`index.html` lines 342-360):

```javascript
import StringBoot from '@stringboot/web-sdk';

const refreshBtn = document.getElementById('refreshBtn');

refreshBtn.addEventListener('click', async () => {
  refreshBtn.disabled = true;
  refreshBtn.textContent = 'Refreshing...';
  statusEl.className = 'status loading';
  statusEl.textContent = 'Force refreshing cache...';

  try {
    await StringBoot.forceRefresh();
    await loadStrings();
    statusEl.className = 'status success';
    statusEl.textContent = '✓ Cache refreshed successfully';
  } catch (error) {
    statusEl.className = 'status error';
    statusEl.textContent = `✗ Refresh failed: ${error.message}`;
  } finally {
    refreshBtn.disabled = false;
    refreshBtn.textContent = 'Force Refresh';
  }
});
```

### Example 6: React - useString Hook (Single String)

Based on actual implementation (`react-example.tsx` lines 61-71):

```jsx
import { useString } from '@stringboot/web-sdk/react';

/**
 * Header component using single string hook
 */
function Header() {
  const title = useString('app_title');
  const subtitle = useString('app_subtitle');

  return (
    <header>
      <h1>{title}</h1>
      <p>{subtitle}</p>
    </header>
  );
}
```

### Example 7: React - useStrings Hook (Multiple Strings)

Based on actual implementation (`react-example.tsx` lines 105-135):

```jsx
import { useStrings } from '@stringboot/web-sdk/react';

/**
 * Main content using multiple strings hook
 */
function MainContent() {
  const strings = useStrings([
    'welcome_message',
    'features_title',
    'feature_1',
    'feature_2',
    'feature_3',
    'cta_button',
  ]);

  return (
    <main>
      <section className="hero">
        <h2>{strings.welcome_message}</h2>
      </section>

      <section className="features">
        <h3>{strings.features_title}</h3>
        <ul>
          <li>{strings.feature_1}</li>
          <li>{strings.feature_2}</li>
          <li>{strings.feature_3}</li>
        </ul>
      </section>

      <section className="cta">
        <button className="primary">{strings.cta_button}</button>
      </section>
    </main>
  );
}
```

### Example 8: React - useLanguage Hook (Language Switcher)

Based on actual implementation (`react-example.tsx` lines 76-100):

```jsx
import { useLanguage, useActiveLanguages } from '@stringboot/web-sdk/react';

/**
 * Language switcher using useLanguage hook
 */
function LanguageSwitcher() {
  const [currentLang, setLanguage] = useLanguage();
  const { languages, loading } = useActiveLanguages();

  if (loading) {
    return <div>Loading languages...</div>;
  }

  return (
    <div className="language-switcher">
      <label htmlFor="language">Language:</label>
      <select
        id="language"
        value={currentLang}
        onChange={(e) => setLanguage(e.target.value)}
      >
        {languages.map((lang) => (
          <option key={lang.code} value={lang.code}>
            {lang.name}
          </option>
        ))}
      </select>
    </div>
  );
}
```

### Example 9: React - useSync Hook (Manual Synchronization)

Based on actual implementation (`react-example.tsx` lines 140-152):

```jsx
import { useSync, useString } from '@stringboot/web-sdk/react';

/**
 * Sync controls using useSync hook
 */
function SyncControls() {
  const { sync, syncing } = useSync();
  const syncLabel = useString('sync_now');
  const syncingLabel = useString('syncing');

  return (
    <div className="sync-controls">
      <button onClick={() => sync()} disabled={syncing}>
        {syncing ? syncingLabel : syncLabel}
      </button>
    </div>
  );
}
```

### Example 10: React - Advanced Patterns

Based on actual implementation (`react-example.tsx` lines 157-193):

```jsx
import { useString } from '@stringboot/web-sdk/react';

/**
 * Example with conditional rendering based on string value
 */
function ConditionalExample() {
  const featureEnabled = useString('feature_flag_new_ui');

  return (
    <div>
      {featureEnabled === 'true' && (
        <div className="new-feature">
          <h3>New UI is enabled!</h3>
        </div>
      )}
    </div>
  );
}

/**
 * Example with formatted strings (using template literals)
 */
function FormattedExample({ userName }: { userName: string }) {
  const greetingTemplate = useString('greeting_with_name');

  // Assuming the string is: "Hello, {name}!"
  const greeting = greetingTemplate.replace('{name}', userName);

  return <h2>{greeting}</h2>;
}

/**
 * Example with pluralization
 */
function PluralExample({ count }: { count: number }) {
  const singularString = useString('item_singular');
  const pluralString = useString('item_plural');

  const displayString = count === 1 ? singularString : pluralString;

  return <p>{count} {displayString}</p>;
}
```

---

## Error Handling

### Proper Error Handling Pattern

All SDK operations should use try/catch/finally pattern as shown in actual demos:

```javascript
import StringBoot from '@stringboot/web-sdk';

async function handleOperation() {
  const button = document.getElementById('operationBtn');

  // Disable button during operation
  button.disabled = true;
  button.textContent = 'Processing...';

  try {
    // Perform operation
    await StringBoot.syncNow();

    // Show success
    showStatus('success', '✓ Operation completed successfully');
  } catch (error) {
    // Show error
    showStatus('error', `✗ Operation failed: ${error.message}`);
    console.error('Operation error:', error);
  } finally {
    // Always re-enable button
    button.disabled = false;
    button.textContent = 'Try Again';
  }
}
```

### React Error Handling

React hooks handle errors internally:

```jsx
import { useString } from '@stringboot/web-sdk/react';

function SafeComponent() {
  const title = useString('page_title');

  // Hooks handle errors gracefully
  // Returns empty string or fallback on error
  return <h1>{title || 'Default Title'}</h1>;
}
```

### Network Error Handling

```javascript
try {
  await StringBoot.syncNow();
} catch (error) {
  if (error.message.includes('Network')) {
    console.log('Network error, using cached values');
  } else if (error.message.includes('timeout')) {
    console.log('Request timed out, retrying...');
  } else {
    console.error('Unexpected error:', error);
  }
}
```

---

## Caching & Offline Behavior

### Cache Statistics

Based on actual implementation (`index.html` lines 362-370):

```javascript
import StringBoot from '@stringboot/web-sdk';

const getCacheBtn = document.getElementById('getCacheBtn');

getCacheBtn.addEventListener('click', async () => {
  try {
    const stats = await StringBoot.getCacheStats();
    alert(`Cache Stats:\n\nMemory: ${stats.memory.size}/${stats.memory.maxSize}\nDB: ${stats.db.totalStrings} strings (${stats.db.deletedStrings} deleted)`);
    console.log('Cache stats:', stats);
  } catch (error) {
    alert(`Failed to get cache stats: ${error.message}`);
  }
});
```

### Cache Statistics Response

```javascript
{
  memory: {
    size: 42,        // Current entries in memory cache
    maxSize: 1000    // Maximum cache size
  },
  db: {
    totalStrings: 150,      // Total strings in IndexedDB
    deletedStrings: 3       // Soft-deleted strings
  }
}
```

### Offline Mode

The SDK automatically works offline using cached data:

```javascript
// Get strings works even when offline
const message = await StringBoot.get('welcome_message');
// Returns cached value or empty string

// Check online status
if (navigator.onLine) {
  await StringBoot.syncNow();
}
```

### Manual Cache Management

```javascript
// Force refresh from server (clears cache)
await StringBoot.forceRefresh();

// Manual sync (preserves cache)
await StringBoot.syncNow();
```

---

## Best Practices

### 1. Initialize Early

```javascript
// main.js - Initialize as early as possible
import StringBoot from '@stringboot/web-sdk';

await StringBoot.initialize({
  apiToken: import.meta.env.VITE_STRINGBOOT_TOKEN,
  baseUrl: import.meta.env.VITE_STRINGBOOT_URL || 'https://api.stringboot.com',
  defaultLanguage: 'en',
  debug: import.meta.env.DEV
});
```

### 2. Use Watchers for Reactive UI

```javascript
// Set up watchers once during initialization
StringBoot.watch('welcome_message', (value) => {
  document.getElementById('welcome').textContent = value;
});

StringBoot.watch('goodbye_message', (value) => {
  document.getElementById('goodbye').textContent = value;
});

// Language changes automatically trigger watchers
await StringBoot.changeLanguage('es');
// UI updates automatically via watchers
```

### 3. Always Use try/catch/finally

```javascript
// Good - Complete error handling
button.addEventListener('click', async () => {
  button.disabled = true;
  try {
    await StringBoot.syncNow();
    showSuccess();
  } catch (error) {
    showError(error.message);
  } finally {
    button.disabled = false;
  }
});

// Bad - No error handling
button.addEventListener('click', async () => {
  await StringBoot.syncNow(); // Unhandled promise rejection!
});
```

### 4. Use React Hooks for React Apps

```jsx
// Good - React hook handles everything
function MyComponent() {
  const title = useString('page_title');
  return <h1>{title}</h1>;
}

// Bad - Manual state management
function MyComponent() {
  const [title, setTitle] = useState('');
  useEffect(() => {
    StringBoot.get('page_title').then(setTitle);
  }, []);
  return <h1>{title}</h1>;
}
```

### 5. Handle Button Disabled States

```javascript
// Always disable buttons during async operations
syncBtn.addEventListener('click', async () => {
  syncBtn.disabled = true;
  syncBtn.textContent = 'Syncing...';

  try {
    await StringBoot.syncNow();
  } catch (error) {
    console.error(error);
  } finally {
    syncBtn.disabled = false;
    syncBtn.textContent = 'Sync Now';
  }
});
```

### 6. Show Loading/Error States

```jsx
// React - Handle all states
function App() {
  const { initialized, error } = useStringBoot({
    apiToken: 'token',
    baseUrl: 'url',
  });

  if (!initialized) return <Loading />;
  if (error) return <Error message={error} />;

  return <MainContent />;
}
```

### 7. Use Status Indicators

```javascript
// Provide visual feedback for all operations
statusEl.className = 'status loading';
statusEl.textContent = 'Syncing with server...';

try {
  await StringBoot.syncNow();
  statusEl.className = 'status success';
  statusEl.textContent = '✓ Sync completed successfully';
} catch (error) {
  statusEl.className = 'status error';
  statusEl.textContent = `✗ Sync failed: ${error.message}`;
}
```

### 8. Environment-Based Configuration

```javascript
// Use different configs for dev/prod
const config = {
  apiToken: import.meta.env.VITE_STRINGBOOT_TOKEN,
  baseUrl: import.meta.env.DEV
    ? 'http://localhost:8000'           // Local development
    : 'https://api.stringboot.com',     // Production
  debug: import.meta.env.DEV,
};

await StringBoot.initialize(config);
```

---

## FAQ / Troubleshooting

### Q: How do I set up local development?

**A:** Configure the SDK to use localhost:

```javascript
await StringBoot.initialize({
  apiToken: '700fdd40-8241-4035-94fe-c7e1713ba9ac',
  baseUrl: 'http://localhost:8000',
  defaultLanguage: 'en',
  debug: true
});
```

Make sure your local Stringboot server is running on port 8000.

### Q: Strings not updating when language changes

**A:** Use watchers in vanilla JS or React hooks:

```javascript
// Vanilla JS - Set up watchers
StringBoot.watch('welcome_message', (value) => {
  element.textContent = value;
});

// React - Use hooks
const title = useString('welcome_message');
```

### Q: How do I handle loading states?

**A:** Use the initialized flag in React:

```jsx
const { initialized, error } = useStringBoot({ ... });

if (!initialized) return <div>Loading...</div>;
if (error) return <div>Error: {error}</div>;
```

For vanilla JS, show status during initialization:

```javascript
statusEl.textContent = 'Initializing SDK...';
await StringBoot.initialize({ ... });
statusEl.textContent = '✓ Ready';
```

### Q: Buttons stay disabled after error

**A:** Always use finally block:

```javascript
try {
  await operation();
} catch (error) {
  console.error(error);
} finally {
  button.disabled = false;  // Always re-enable
}
```

### Q: How do I display cache statistics?

**A:** Use getCacheStats():

```javascript
const stats = await StringBoot.getCacheStats();
console.log(`Memory: ${stats.memory.size}/${stats.memory.maxSize}`);
console.log(`DB: ${stats.db.totalStrings} strings`);
```

### Q: Module not found error

**A:** Ensure correct import path:

```javascript
// Vanilla JS
import StringBoot from '@stringboot/web-sdk';

// React hooks
import { useString } from '@stringboot/web-sdk/react';
```

### Q: React hooks not working

**A:** Initialize SDK before using hooks:

```jsx
function App() {
  const { initialized } = useStringBoot({
    apiToken: 'token',
    baseUrl: 'url',
  });

  if (!initialized) return <Loading />;

  // Now hooks will work
  return <MyComponent />;
}
```

### Q: How to handle API token in production?

**A:** Use environment variables:

```javascript
// Vite
const token = import.meta.env.VITE_STRINGBOOT_TOKEN;

// Create React App
const token = process.env.REACT_APP_STRINGBOOT_TOKEN;

// Next.js
const token = process.env.NEXT_PUBLIC_STRINGBOOT_TOKEN;
```

Never commit API tokens to source control.

### Q: Sync fails with network error

**A:** Check the baseUrl configuration:

```javascript
// Make sure baseUrl is correct
await StringBoot.initialize({
  baseUrl: 'http://localhost:8000',  // Development
  // or
  baseUrl: 'https://api.stringboot.com',  // Production
  apiToken: 'your-token'
});
```

---

## Changelog

### v1.0.0 (2025-11-07)

**Initial Release:**

**Core Features:**
- Full String-Sync v2 protocol implementation
- Multi-layer caching (memory, IndexedDB, localStorage metadata)
- Offline-first architecture with automatic fallback
- Reactive watch API for automatic UI updates
- Zero dependencies

**Vanilla JavaScript API:**
- `initialize()` - SDK initialization with config
- `get()` - Get single string by key
- `watch()` - Watch string changes for reactive updates
- `changeLanguage()` - Switch languages with automatic refresh
- `syncNow()` - Manual synchronization with server
- `forceRefresh()` - Clear cache and force full refresh
- `getCacheStats()` - Get cache statistics

**React Integration:**
- `useStringBoot()` - SDK initialization with loading/error states
- `useString()` - Single string hook with automatic updates
- `useStrings()` - Multiple strings hook for batch loading
- `useLanguage()` - Language management hook
- `useActiveLanguages()` - Get available languages
- `useSync()` - Manual sync control with syncing state

**Performance:**
- Target <300ms string lookup
- Smart memory cache with LRU eviction
- IndexedDB for offline persistence
- Delta sync for minimal bandwidth usage

**Developer Experience:**
- TypeScript support with full type definitions
- Debug mode with detailed logging
- Local development support (localhost:8000)
- Comprehensive error handling
- Loading/error states in React hooks

**Browser Support:**
- Chrome 87+
- Firefox 78+
- Safari 14+
- Edge 88+

---

## Related Documentation

- **Backend API Specification:** See `/docs/BACKEND_API_SPECIFICATION.md`
- **Delta Sync Protocol:** See `/docs/DELTA_SYNC_PROTOCOL.md`
- **Android SDK:** See `/stringboot-android-sdk/INTEGRATION_GUIDE.md`
- **iOS SDK:** See `/stringboot-ios-sdk/README.md`

---

**Questions or Issues?**

For support, please contact the Stringboot team or open an issue in the repository.
