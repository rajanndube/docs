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

### Vue 3

```vue
<script setup>
import { ref, onMounted } from 'vue';
import StringBoot from '@stringboot/web-sdk';

const initialized = ref(false);

onMounted(async () => {
  await StringBoot.initialize({
    apiToken: 'YOUR_API_TOKEN_HERE',
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en'
  });
  initialized.value = true;
});
</script>

<template>
  <div v-if="initialized">
    <!-- Your app content -->
  </div>
  <div v-else>
    Loading...
  </div>
</template>
```

### Next.js (App Router)

```tsx
// app/providers.tsx
'use client';

import { useStringBoot } from '@stringboot/web-sdk/react';

export function Providers({ children }: { children: React.ReactNode }) {
  const { initialized } = useStringBoot({
    apiToken: process.env.NEXT_PUBLIC_STRINGBOOT_TOKEN!,
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en'
  });

  if (!initialized) return <div>Loading...</div>;

  return <>{children}</>;
}

// app/layout.tsx
import { Providers } from './providers';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
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

### Method 3: Vue 3 Composition API

```vue
<script setup>
import { ref, onMounted } from 'vue';
import StringBoot from '@stringboot/web-sdk';

const welcomeText = ref('');
const description = ref('');
const currentLang = ref('en');

// Load strings
onMounted(async () => {
  await StringBoot.initialize({
    apiToken: 'YOUR_API_TOKEN',
    baseUrl: 'https://api.stringboot.com'
  });

  // Watch for string updates
  StringBoot.watch('welcome_message', (value) => {
    welcomeText.value = value;
  });

  StringBoot.watch('app_description', (value) => {
    description.value = value;
  });
});

// Change language
const changeLanguage = async (lang) => {
  await StringBoot.changeLanguage(lang);
  currentLang.value = lang;
};
</script>

<template>
  <div>
    <h1>{{ welcomeText }}</h1>
    <p>{{ description }}</p>

    <select :value="currentLang" @change="changeLanguage($event.target.value)">
      <option value="en">English</option>
      <option value="es">Español</option>
      <option value="fr">Français</option>
    </select>
  </div>
</template>
```

### Method 4: TypeScript (Type-Safe)

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

## Step 4: Test It Out! (1 minute)

1. **Run your development server**:
   ```bash
   npm run dev
   ```

2. **Open browser console** - look for Stringboot logs:
   ```
   [Stringboot] Initializing...
   [Stringboot] ✅ Sync complete: 42 strings loaded
   [Stringboot] Current language: en
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
import { useLanguage } from '@stringboot/web-sdk/react';
import { useState, useEffect } from 'react';
import StringBoot from '@stringboot/web-sdk';

function LanguageSelector() {
  const [currentLang, setLanguage] = useLanguage();
  const [availableLanguages, setAvailableLanguages] = useState([]);

  useEffect(() => {
    async function loadLanguages() {
      const langs = await StringBoot.getActiveLanguages();
      setAvailableLanguages(langs);
    }
    loadLanguages();
  }, []);

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

### Server-Side Rendering (SSR)

For Next.js or other SSR frameworks:

```tsx
'use client'; // Next.js 13+

import { useEffect, useState } from 'react';
import StringBoot from '@stringboot/web-sdk';

export function ClientOnlyStringBoot({ children }) {
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    StringBoot.initialize({
      apiToken: process.env.NEXT_PUBLIC_STRINGBOOT_TOKEN!,
      baseUrl: 'https://api.stringboot.com'
    }).then(() => {
      setMounted(true);
    });
  }, []);

  if (!mounted) return <div>Loading...</div>;

  return <>{children}</>;
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

console.log('Memory cache size:', stats.memorySize);
console.log('IndexedDB entries:', stats.dbSize);
console.log('Hit rate:', stats.hitRate);
```

## Learn More

| Resource | Description |
|----------|-------------|
| **[INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)** | Comprehensive integration walkthrough |
| **[NEXTJS_INTEGRATION.md](NEXTJS_INTEGRATION.md)** | Next.js-specific integration guide |
| **[AB_TESTING.md](AB_TESTING.md)** | A/B testing and analytics integration |
| **[ADVANCED_FEATURES.md](ADVANCED_FEATURES.md)** | IndexedDB caching, multi-tab sync, FAQ provider |
| **[API_REFERENCE.md](API_REFERENCE.md)** | Complete API documentation |
| **[FAQ.md](FAQ.md)** | Frequently asked questions |
| **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** | Common issues and solutions |

## Framework-Specific Guides

- **React:** [React Integration](INTEGRATION_GUIDE.md#react-integration)
- **Next.js:** [NEXTJS_INTEGRATION.md](NEXTJS_INTEGRATION.md)
- **Vue 3:** [Vue Integration](INTEGRATION_GUIDE.md#vue-integration)
- **Angular:** [Angular Integration](INTEGRATION_GUIDE.md#angular-integration)
- **Svelte:** [Svelte Integration](INTEGRATION_GUIDE.md#svelte-integration)

## Need Help?

- **Email:** support@stringboot.com
- **Documentation:** https://docs.stringboot.com
- **Report Issues:** https://github.com/stringboot/web-sdk/issues
- **NPM Package:** https://www.npmjs.com/package/@stringboot/web-sdk

---

**Congratulations!** 🎉 You've integrated Stringboot into your web app. Your strings are now managed remotely and can be updated without deployments!

