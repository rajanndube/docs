# Stringboot Web SDK - Quick Start Guide

Get up and running with Stringboot in **under 10 minutes**. This guide will walk you through adding dynamic string management to your web application.

## Prerequisites

- Node.js 16+ and npm/yarn
- A Stringboot API token ([Get one here](https://stringboot.com/signup))
- Basic knowledge of JavaScript/TypeScript

## Step 1: Install Package (1 minute)

```bash
# npm
npm install @stringboot/web-sdk

# yarn
yarn add @stringboot/web-sdk

# pnpm
pnpm add @stringboot/web-sdk
```

## Step 2: Initialize SDK (2 minutes)

### Vanilla JavaScript / TypeScript

```javascript
import StringBoot from '@stringboot/web-sdk';

// Initialize once at app startup
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN_HERE',
  baseUrl: 'https://api.stringboot.com',  // Optional
  defaultLanguage: 'en',                    // Optional - defaults to browser language
  debug: true                               // Optional - enable console logs
});

console.log('✅ Stringboot initialized!');
```

### React

```tsx
import React from 'react';
import { useStringBoot } from '@stringboot/web-sdk/react';

function App() {
  const { initialized, error } = useStringBoot({
    apiToken: 'YOUR_API_TOKEN_HERE',
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en',
    debug: true
  });

  if (!initialized) {
    return <div>Loading Stringboot...</div>;
  }

  if (error) {
    return <div>Error: {error}</div>;
  }

  return <MainApp />;
}
```

**⚠️ Security Note:** Store your API token in environment variables (`.env.local`) rather than hardcoding it.

## Step 3: Use Strings in Your App (5 minutes)

### Method 1: Vanilla JavaScript (Promise-based)

```javascript
// Get a single string
const welcomeText = await StringBoot.get('welcome_message');
document.getElementById('title').textContent = welcomeText;

// Get string with specific language
const spanishText = await StringBoot.get('welcome_message', 'es');

// Watch for updates (reactive)
StringBoot.watch('welcome_message', (newValue) => {
  document.getElementById('title').textContent = newValue;
});
```

### Method 2: React Hooks (Recommended)

```tsx
import { useString, useLanguage } from '@stringboot/web-sdk/react';

function WelcomeComponent() {
  // Auto-updating string (updates when language changes)
  const welcomeText = useString('welcome_message');
  const description = useString('app_description');

  // Language management
  const [currentLang, setLanguage] = useLanguage();

  return (
    <div>
      <h1>{welcomeText}</h1>
      <p>{description}</p>

      <select value={currentLang} onChange={(e) => setLanguage(e.target.value)}>
        <option value="en">English</option>
        <option value="es">Español</option>
        <option value="fr">Français</option>
      </select>
    </div>
  );
}
```

### Method 3: TypeScript (Type-Safe)

```typescript
import StringBoot, { StringBootConfig } from '@stringboot/web-sdk';

// Initialize with type safety
const config: StringBootConfig = {
  apiToken: process.env.STRINGBOOT_API_TOKEN!,
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  debug: true
};

await StringBoot.initialize(config);

// Type-safe string access
const welcomeText: string = await StringBoot.get('welcome_message');

// Type-safe language codes
type SupportedLanguage = 'en' | 'es' | 'fr' | 'de';
await StringBoot.changeLanguage('es' as SupportedLanguage);
```

## Step 4: Using FAQs (Optional - 3 minutes)

The FAQ Provider lets you deliver dynamic, multilingual FAQ content with offline-first caching.

### Initialize FAQ Provider

```javascript
import StringBoot from '@stringboot/web-sdk';

// After initializing StringBoot, initialize FAQ Provider
await StringBoot.FAQ.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  cacheSize: 200  // Optional: default is 200
});
```

### Fetch FAQs

```javascript
// Get all FAQs regardless of tag
const allFAQs = await StringBoot.FAQ.getFAQs({
  tag: 'all',
  lang: 'en',
  allowNetworkFetch: true
});

// Get FAQs by specific tag
const paymentFAQs = await StringBoot.FAQ.getFAQs({
  tag: 'payments',
  lang: 'en',
  allowNetworkFetch: true
});

// Get FAQs with sub-tag filtering (2-level filtering)
const refundFAQs = await StringBoot.FAQ.getFAQs({
  tag: 'payments',
  subTags: ['refunds', 'disputes'],
  lang: 'en',
  allowNetworkFetch: true
});

// Display FAQs
allFAQs.forEach(faq => {
  console.log(`Q: ${faq.question}`);
  console.log(`A: ${faq.answer}`);
  console.log(`Tags: ${faq.tag}, ${faq.subTags.join(', ')}`);
});
```

### React FAQ Integration

```tsx
import { useState, useEffect } from 'react';
import StringBoot from '@stringboot/web-sdk';

function FAQSection() {
  const [faqs, setFaqs] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function loadFAQs() {
      // Initialize FAQ Provider first
      await StringBoot.FAQ.initialize({
        apiToken: 'YOUR_API_TOKEN',
        baseUrl: 'https://api.stringboot.com'
      });

      // Fetch FAQs
      const fetchedFAQs = await StringBoot.FAQ.getFAQs({
        tag: 'all',
        lang: 'en',
        allowNetworkFetch: true
      });

      setFaqs(fetchedFAQs);
      setLoading(false);
    }

    loadFAQs();
  }, []);

  if (loading) return <div>Loading FAQs...</div>;

  return (
    <div>
      {faqs.map(faq => (
        <div key={faq.id}>
          <h3>{faq.question}</h3>
          <p dangerouslySetInnerHTML={{ __html: faq.answer }} />
          <span className="tag">{faq.tag}</span>
        </div>
      ))}
    </div>
  );
}
```

**Key FAQ Features:**
- **2-level filtering**: Mandatory tag + optional sub-tags
- **"All" tag**: Use `tag: 'all'` to fetch entire catalog
- **Offline-first**: Memory → IndexedDB → Network
- **Multi-language**: Automatic fallback to English
- **Delta sync**: Only downloads changed FAQs

## Step 5: Test It Out! (1 minute)

1. **Run your development server**:
   ```bash
   npm run dev
   ```

2. **Open browser console** - look for Stringboot logs:
   ```
   [StringBoot] Initializing...
   [StringBoot] ✅ Sync complete: 42 strings loaded
   [StringBoot] Current language: en
   ```

3. **Verify strings appear** in your UI

4. **Test language switching** (if implemented)

## Common Pitfalls & Troubleshooting

### ❌ Problem: Strings return empty or "undefined"

**Solution:**
- Ensure `await StringBoot.initialize()` completes before calling `.get()`
- Check browser console for initialization errors
- Verify API token is correct
- Check network tab - API calls should return 200

### ❌ Problem: TypeScript errors about module not found

**Solution:**
- Install type definitions: `npm install --save-dev @types/node`
- Add to `tsconfig.json`:
  ```json
  {
    "compilerOptions": {
      "moduleResolution": "bundler",
      "allowSyntheticDefaultImports": true
    }
  }
  ```

### ❌ Problem: "Cannot use import statement outside a module"

**Solution:**
- Use `.mjs` extension for ES modules
- Or add to `package.json`: `"type": "module"`
- Or use CommonJS import: `const StringBoot = require('@stringboot/web-sdk').default;`

### ❌ Problem: Strings don't update when changing language

**Solution:**
- Use `StringBoot.watch()` for reactive updates
- In React, use `useString()` hook (auto-updates)
- Make sure you're calling `await StringBoot.changeLanguage(lang)`

### ❌ Problem: Large bundle size

**Solution:**
- The SDK is only 6.6 kB gzipped - check if you're importing correctly
- Use tree-shaking: `import StringBoot from '@stringboot/web-sdk'` (not `import * as...`)
- For React, import hooks separately: `import { useString } from '@stringboot/web-sdk/react'`

### ❌ Problem: Offline mode not working

**Solution:**
- Strings are cached in IndexedDB after first load
- Clear browser storage to test: DevTools → Application → Clear site data
- Check IndexedDB has `stringboot_*` databases

## Next Steps

### Change Language Dynamically

```javascript
// Vanilla JS
await StringBoot.changeLanguage('es');
// All watched strings automatically update!

// React
const [lang, setLang] = useLanguage();
setLang('es'); // Automatically triggers re-render

// Vue
await StringBoot.changeLanguage('es');
```

### Get Available Languages

```javascript
const languages = await StringBoot.getActiveLanguages();

languages.forEach(lang => {
  console.log(`${lang.code}: ${lang.name} (Default: ${lang.isDefault})`);
});

// Returns:
// [
//   { code: 'en', name: 'English', isDefault: true },
//   { code: 'es', name: 'Spanish', isDefault: false },
//   ...
// ]
```

### Build Language Selector

```tsx
import { useLanguage, useActiveLanguages } from '@stringboot/web-sdk/react';

function LanguageSelector() {
  const [currentLang, setLanguage] = useLanguage();
  const { languages, loading, error } = useActiveLanguages();

  if (loading) return <div>Loading languages...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <select value={currentLang} onChange={(e) => setLanguage(e.target.value)}>
      {languages.map(lang => (
        <option key={lang.code} value={lang.code}>
          {lang.name}
        </option>
      ))}
    </select>
  );
}
```

### Multi-Tab Synchronization

The SDK automatically synchronizes across browser tabs using BroadcastChannel:

```javascript
// Tab 1: Change language
await StringBoot.changeLanguage('es');

// Tab 2: Automatically receives update!
// All watched strings update across all tabs
```

### Monitor Cache Performance

```javascript
// Get cache statistics
const stats = await StringBoot.getCacheStats();

console.log('Memory cache:', stats.memory);
console.log('Database stats:', stats.db);
```

## Learn More

| Resource | Description |
|----------|-------------|
| **[INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)** | Comprehensive integration guide for all frameworks |
| **[Backend API Reference](../docs/API_REFERENCE.md)** | Backend API endpoints documentation |
| **[Delta Sync Protocol](../docs/DELTA_SYNC_PROTOCOL.md)** | How synchronization works |

## Framework-Specific Guides

- **React:** [React Integration](INTEGRATION_GUIDE.md#react-integration)
- **Vue 3:** [Vue Integration](INTEGRATION_GUIDE.md#vue-3-integration)
- **Next.js:** [Next.js Integration](INTEGRATION_GUIDE.md#nextjs-integration)

## Need Help?

- **Email:** support@stringboot.com
- **Documentation:** https://docs.stringboot.com
- **Report Issues:** https://github.com/stringboot/web-sdk/issues
- **NPM Package:** https://www.npmjs.com/package/@stringboot/web-sdk

---

**Congratulations!** 🎉 You've integrated Stringboot into your web app. Your strings are now managed remotely and can be updated without deployments!

