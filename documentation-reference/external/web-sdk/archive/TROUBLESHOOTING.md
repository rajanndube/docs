# Stringboot Web SDK - Troubleshooting Guide

## 🔧 Installation Issues

### Problem: Module Not Found Error

**Error:**
```
Cannot find module 'stringboot-web-sdk' or its corresponding type declarations.
```

**Solutions:**

#### 1. **Verify Package Installation**

```bash
# Check if package is installed
npm list stringboot-web-sdk

# Reinstall if needed
npm install stringboot-web-sdk
```

#### 2. **Check Package.json**

```json
{
  "dependencies": {
    "stringboot-web-sdk": "^1.0.0"
  }
}
```

#### 3. **Clear Cache and Reinstall**

```bash
# Clear npm cache
npm cache clean --force

# Remove node_modules
rm -rf node_modules package-lock.json

# Reinstall
npm install
```

#### 4. **Verify TypeScript Configuration**

```json
// tsconfig.json
{
  "compilerOptions": {
    "moduleResolution": "node",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true
  }
}
```

### Problem: Build Errors with Webpack/Vite

**Error:**
```
Module parse failed: Unexpected token
```

**Solution for Webpack:**

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        include: /node_modules\/stringboot-web-sdk/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      }
    ]
  }
};
```

**Solution for Vite:**

```javascript
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  optimizeDeps: {
    include: ['stringboot-web-sdk']
  }
});
```

## ⚙️ Initialization Issues

### Problem: SDK Not Initialized Error

**Error:**
```
StringBoot SDK is not initialized. Call StringBoot.initialize() first.
```

**Cause:** Trying to use SDK before initialization completes.

**Solution:**

```javascript
// ✅ Correct - Wait for initialization
import StringBoot from 'stringboot-web-sdk';

async function initApp() {
  try {
    await StringBoot.initialize({
      apiToken: 'YOUR_API_TOKEN',
      baseUrl: 'https://api.stringboot.com',
      defaultLanguage: 'en'
    });

    console.log('✅ SDK initialized');

    // Now safe to use SDK
    const text = await StringBoot.getString('welcome_message');
  } catch (error) {
    console.error('Failed to initialize:', error);
  }
}

initApp();
```

```javascript
// ❌ Wrong - Not waiting for initialization
StringBoot.initialize({ /* config */ });
const text = await StringBoot.getString('welcome_message'); // Error!
```

### Problem: Invalid API Token

**Error:**
```
401 Unauthorized: Invalid API token
```

**Solutions:**

#### 1. **Verify Token is Correct**

```javascript
// Check your token from Stringboot dashboard
await StringBoot.initialize({
  apiToken: 'YOUR_ACTUAL_TOKEN_HERE',  // Not placeholder!
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en'
});
```

#### 2. **Test Token with curl**

```bash
curl -H "X-API-TOKEN: YOUR_TOKEN" \
  https://api.stringboot.com/api/v1/client/health
```

Expected response:
```json
{"status": "ok"}
```

#### 3. **Check Token Environment Variable**

```javascript
// Use environment variable
await StringBoot.initialize({
  apiToken: process.env.REACT_APP_STRINGBOOT_TOKEN,
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en'
});
```

```bash
# .env
REACT_APP_STRINGBOOT_TOKEN=your-actual-token
```

### Problem: CORS Errors

**Error:**
```
Access to fetch at 'https://api.stringboot.com' has been blocked by CORS policy
```

**Cause:** CORS misconfiguration on backend or incorrect URL.

**Solutions:**

#### 1. **Verify Base URL**

```javascript
// ✅ Correct - HTTPS in production
await StringBoot.initialize({
  apiToken: 'YOUR_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en'
});

// ❌ Wrong - Mixed content (HTTP from HTTPS page)
await StringBoot.initialize({
  apiToken: 'YOUR_TOKEN',
  baseUrl: 'http://api.stringboot.com',  // HTTP not allowed from HTTPS page
  defaultLanguage: 'en'
});
```

#### 2. **Local Development CORS**

```javascript
// For local development
await StringBoot.initialize({
  apiToken: 'YOUR_TOKEN',
  baseUrl: 'http://localhost:8000',
  defaultLanguage: 'en'
});
```

Backend must allow CORS:
```javascript
// Express.js example
app.use(cors({
  origin: 'http://localhost:3000',
  credentials: true
}));
```

## 📝 String Display Issues

### Problem: Strings Showing as ??key??

**Symptoms:**
- Strings display as `??welcome_message??`
- No actual string content shown

**Solutions:**

#### 1. **Wait for Initialization**

```javascript
// React example
import { useEffect, useState } from 'react';
import StringBoot from 'stringboot-web-sdk';

function App() {
  const [isReady, setIsReady] = useState(false);
  const [text, setText] = useState('');

  useEffect(() => {
    async function init() {
      await StringBoot.initialize({
        apiToken: 'YOUR_TOKEN',
        baseUrl: 'https://api.stringboot.com',
        defaultLanguage: 'en'
      });
      setIsReady(true);
    }
    init();
  }, []);

  useEffect(() => {
    if (isReady) {
      StringBoot.getString('welcome_message').then(setText);
    }
  }, [isReady]);

  return <div>{text || 'Loading...'}</div>;
}
```

#### 2. **Enable Auto-Sync**

```javascript
await StringBoot.initialize({
  apiToken: 'YOUR_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  autoSync: true  // Enable auto-sync
});
```

#### 3. **Manual Sync Before Using**

```javascript
// Initialize
await StringBoot.initialize({ /* config */ });

// Sync strings
await StringBoot.syncStrings('en');

// Now get strings
const text = await StringBoot.getString('welcome_message', 'en');
```

#### 4. **Verify Key Exists in Backend**

```bash
# Check if key exists
curl -H "X-API-TOKEN: your-token" \
  "https://api.stringboot.com/api/v1/client/strings/catalog?languageCode=en"
```

Look for your key in the response.

### Problem: Strings Not Auto-Updating

**Symptoms:**
- React components don't re-render when language changes
- Strings stuck on initial value

**Solution - Use React Hook:**

```javascript
import { useStringBoot } from 'stringboot-web-sdk/react';

function MyComponent() {
  // ✅ Auto-updates when language changes
  const text = useStringBoot('welcome_message');

  return <div>{text}</div>;
}
```

**Solution - Manual Subscription:**

```javascript
import { useEffect, useState } from 'react';
import StringBoot from 'stringboot-web-sdk';

function MyComponent() {
  const [text, setText] = useState('');

  useEffect(() => {
    // Subscribe to updates
    const unsubscribe = StringBoot.onLanguageChange(() => {
      StringBoot.getString('welcome_message').then(setText);
    });

    // Initial load
    StringBoot.getString('welcome_message').then(setText);

    return unsubscribe;
  }, []);

  return <div>{text}</div>;
}
```

## 🌍 Language Switching Issues

### Problem: Language Not Changing

**Symptoms:**
- Called `setLanguage()` but strings still in old language
- UI not updating after language change

**Solutions:**

#### 1. **Wait for Language Sync**

```javascript
async function changeLanguage(lang) {
  try {
    // Change language
    await StringBoot.setLanguage(lang);

    // Wait for sync to complete
    await StringBoot.syncStrings(lang);

    console.log(`✅ Language changed to ${lang}`);
  } catch (error) {
    console.error('Language change failed:', error);
  }
}
```

#### 2. **Force Component Re-render**

```javascript
import { useEffect, useState } from 'react';
import StringBoot from 'stringboot-web-sdk';

function LanguageSwitcher() {
  const [currentLang, setCurrentLang] = useState('en');

  async function handleLanguageChange(lang) {
    await StringBoot.setLanguage(lang);
    await StringBoot.syncStrings(lang);
    setCurrentLang(lang);  // Force re-render
  }

  return (
    <div>
      <button onClick={() => handleLanguageChange('en')}>English</button>
      <button onClick={() => handleLanguageChange('es')}>Español</button>
      <p>Current: {currentLang}</p>
    </div>
  );
}
```

#### 3. **Use Language Change Event**

```javascript
useEffect(() => {
  const unsubscribe = StringBoot.onLanguageChange((newLang) => {
    console.log(`Language changed to: ${newLang}`);
    // Refresh your UI here
    forceUpdate();
  });

  return unsubscribe;
}, []);
```

### Problem: Invalid Language Code

**Error:**
```
Language 'xx' not found
```

**Solution:**

```javascript
// Get available languages first
const languages = await StringBoot.getLanguages();
console.log('Available languages:', languages);

// Check if language exists
if (languages.includes('es')) {
  await StringBoot.setLanguage('es');
} else {
  console.warn('Spanish not available');
}
```

## 🌐 Network & Sync Issues

### Problem: Network Requests Failing

**Symptoms:**
- Fetch errors in console
- Strings not syncing
- Network tab shows failed requests

**Solutions:**

#### 1. **Check Network Connection**

```javascript
// Add error handling
try {
  await StringBoot.syncStrings('en');
} catch (error) {
  if (error.message.includes('Failed to fetch')) {
    console.error('Network error - check connection');
  }
}
```

#### 2. **Verify Backend is Running**

```bash
# Test health endpoint
curl https://api.stringboot.com/api/v1/client/health

# Expected: {"status": "ok"}
```

#### 3. **Check for Rate Limiting**

```javascript
// Too many requests
// Error: 429 Too Many Requests

// Solution: Implement retry with backoff
async function syncWithRetry(lang, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      await StringBoot.syncStrings(lang);
      return;
    } catch (error) {
      if (error.status === 429 && i < retries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
}
```

### Problem: IndexedDB Quota Exceeded

**Error:**
```
QuotaExceededError: The quota has been exceeded
```

**Solutions:**

#### 1. **Clear Old Cache**

```javascript
// Clear cache periodically
await StringBoot.clearCache();

// Re-sync
await StringBoot.syncStrings('en');
```

#### 2. **Reduce Cache Size**

```javascript
await StringBoot.initialize({
  apiToken: 'YOUR_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  cacheSize: 500  // Reduce from default 1000
});
```

#### 3. **Request More Storage (PWA)**

```javascript
if (navigator.storage && navigator.storage.persist) {
  const isPersisted = await navigator.storage.persist();
  console.log(`Persistent storage: ${isPersisted}`);
}
```

## ❓ FAQ Provider Issues

### Problem: FAQProvider Not Initialized

**Note:** Unlike Android/iOS, Web SDK auto-initializes FAQ Provider when you initialize StringBoot.

**If FAQs not working:**

```javascript
// ✅ Correct - FAQ Provider auto-initialized
await StringBoot.initialize({
  apiToken: 'YOUR_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en'
});

// FAQ Provider ready to use
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'payments' });
```

### Problem: FAQs Not Appearing

**Symptoms:**
- `getFAQs()` returns empty array
- No FAQs displayed

**Solutions:**

#### 1. **Verify Backend Has FAQs**

```bash
curl -H "X-API-TOKEN: your-token" \
  "https://api.stringboot.com/api/v1/client/faqs?lang=en"
```

#### 2. **Wait for Auto-Sync**

```javascript
// Wait for initialization to complete
await StringBoot.initialize({ /* config */ });

// Give time for auto-sync
await new Promise(resolve => setTimeout(resolve, 1000));

// Now fetch FAQs
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'payments' });
console.log(`Found ${faqs.length} FAQs`);
```

#### 3. **Manual Sync**

```javascript
// Force sync FAQs
await StringBoot.FAQ.syncFAQs('en');

// Then fetch
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'payments' });
```

#### 4. **Check Tag Spelling**

```javascript
// ❌ Wrong - typo
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'paymnt' });

// ✅ Correct
const faqs = await StringBoot.FAQ.getFAQs({ tag: 'payments' });
```

### Problem: FAQs Not Updating in React

**Symptoms:**
- FAQ list not re-rendering
- Stale FAQ data

**Solution - Use State:**

```javascript
import { useEffect, useState } from 'react';
import StringBoot from 'stringboot-web-sdk';

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

### Problem: Sub-Tags Not Filtering

**Issue:** Getting all FAQs instead of filtered by sub-tag.

**Solution:**

```javascript
// Get all payment FAQs
const allPaymentFAQs = await StringBoot.FAQ.getFAQs({ tag: 'payments' });

// Filter by sub-tag in JavaScript
const refundFAQs = allPaymentFAQs.filter(faq =>
  faq.subTags?.includes('refunds')
);

console.log(`Refund FAQs: ${refundFAQs.length}`);
```

### Problem: Performance with Large FAQ Lists

**Symptoms:**
- Slow rendering
- UI lag
- High memory usage

**Solutions:**

#### 1. **Implement Pagination**

```javascript
import { useState } from 'react';

function PaginatedFAQList() {
  const [faqs, setFaqs] = useState([]);
  const [page, setPage] = useState(0);
  const pageSize = 20;

  const paginatedFAQs = faqs.slice(
    page * pageSize,
    (page + 1) * pageSize
  );

  return (
    <div>
      {paginatedFAQs.map(faq => (
        <FAQItem key={faq.id} faq={faq} />
      ))}
      <button onClick={() => setPage(p => p - 1)} disabled={page === 0}>
        Previous
      </button>
      <button onClick={() => setPage(p => p + 1)}>
        Next
      </button>
    </div>
  );
}
```

#### 2. **Lazy Loading with React**

```javascript
import { useState, useEffect, useRef } from 'react';

function LazyFAQList() {
  const [faqs, setFaqs] = useState([]);
  const [displayCount, setDisplayCount] = useState(20);
  const observerRef = useRef();

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
      <div ref={observerRef}>Loading more...</div>
    </div>
  );
}
```

#### 3. **Virtual Scrolling**

```javascript
import { FixedSizeList } from 'react-window';

function VirtualizedFAQList({ faqs }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      <FAQItem faq={faqs[index]} />
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

### Debug FAQ Provider

```javascript
// Enable debug logging
StringBoot.setLogLevel('debug');

// Check FAQ cache
const cacheStats = StringBoot.FAQ.getCacheStats();
console.log('FAQ Cache:', cacheStats);

// Test manual sync
const success = await StringBoot.FAQ.syncFAQs('en');
console.log('FAQ sync result:', success);

// Check FAQ count
const count = await StringBoot.FAQ.getFAQCount('en');
console.log(`FAQs in database: ${count}`);
```

## 🧪 A/B Testing Issues

### Problem: Experiment Not Assigned

**Symptoms:**
- `getVariant()` returns `null`
- User not assigned to experiment
- Always getting control variant

**Solutions:**

#### 1. **Verify Experiment Exists**

```bash
curl -H "X-API-TOKEN: your-token" \
  "https://api.stringboot.com/api/v1/client/experiments/button_color"
```

Expected response:
```json
{
  "experimentId": "button_color",
  "variants": [
    {"id": "control", "weight": 50},
    {"id": "blue", "weight": 50}
  ],
  "status": "active"
}
```

#### 2. **Check Experiment Name Spelling**

```javascript
// ❌ Wrong - typo
const variant = StringBoot.Experiment.getVariant('buton_color');

// ✅ Correct
const variant = StringBoot.Experiment.getVariant('button_color');
```

#### 3. **Set User ID**

```javascript
// Must set user ID before getting variant
StringBoot.Experiment.setUserId('user-12345');

// Then get variant
const variant = StringBoot.Experiment.getVariant('button_color');
console.log('Variant:', variant);
```

#### 4. **Verify Experiment is Active**

```javascript
const experiment = await StringBoot.Experiment.getExperiment('button_color');
if (experiment?.status === 'active') {
  console.log('✅ Experiment is active');
} else {
  console.log('❌ Experiment not active:', experiment?.status);
}
```

### Problem: Same Variant Every Time

**Symptoms:**
- User always gets same variant
- Assignment seems not random

**Cause:** Consistent hashing ensures same user always gets same variant (by design).

**To Test Different Variants:**

```javascript
// Test with different user IDs
StringBoot.Experiment.setUserId('user-12345');
const variant1 = StringBoot.Experiment.getVariant('button_color');
console.log('User 12345:', variant1);

StringBoot.Experiment.setUserId('user-67890');
const variant2 = StringBoot.Experiment.getVariant('button_color');
console.log('User 67890:', variant2);

// Clear assignments to reset
StringBoot.Experiment.clearAssignments();
```

### Problem: Variant Not Applying to UI

**Symptoms:**
- `getVariant()` returns correct variant
- UI not updating

**Solution - React:**

```javascript
import { useEffect, useState } from 'react';
import StringBoot from 'stringboot-web-sdk';

function ExperimentButton() {
  const [variant, setVariant] = useState(null);
  const [buttonStyle, setButtonStyle] = useState({});

  useEffect(() => {
    // Set user ID
    StringBoot.Experiment.setUserId('user-12345');

    // Get variant
    const assignedVariant = StringBoot.Experiment.getVariant('button_color');
    setVariant(assignedVariant);

    // Apply styling based on variant
    switch (assignedVariant) {
      case 'blue':
        setButtonStyle({ backgroundColor: 'blue', color: 'white' });
        break;
      case 'green':
        setButtonStyle({ backgroundColor: 'green', color: 'white' });
        break;
      default:
        setButtonStyle({ backgroundColor: 'red', color: 'white' });
    }
  }, []);

  return (
    <button style={buttonStyle}>
      Click Me (Variant: {variant})
    </button>
  );
}
```

**Solution - Vue:**

```vue
<template>
  <button :style="buttonStyle">
    Click Me (Variant: {{ variant }})
  </button>
</template>

<script>
import StringBoot from 'stringboot-web-sdk';

export default {
  data() {
    return {
      variant: null,
      buttonStyle: {}
    };
  },
  mounted() {
    StringBoot.Experiment.setUserId('user-12345');
    this.variant = StringBoot.Experiment.getVariant('button_color');

    switch (this.variant) {
      case 'blue':
        this.buttonStyle = { backgroundColor: 'blue', color: 'white' };
        break;
      case 'green':
        this.buttonStyle = { backgroundColor: 'green', color: 'white' };
        break;
      default:
        this.buttonStyle = { backgroundColor: 'red', color: 'white' };
    }
  }
};
</script>
```

### Problem: Analytics Not Firing

**Symptoms:**
- Experiment assignments not tracked
- No analytics events

**Solutions:**

#### 1. **Enable Analytics**

```javascript
// Enable analytics tracking
StringBoot.Experiment.enableAnalytics(true);

// Verify in console
StringBoot.setLogLevel('debug');
// Look for: [Experiment] Analytics event: experiment_assigned
```

#### 2. **Manual Tracking**

```javascript
const variant = StringBoot.Experiment.getVariant('button_color');

// Track assignment manually
StringBoot.Experiment.trackAssignment(
  'button_color',
  variant,
  'user-12345'
);
```

#### 3. **Integrate with Analytics Provider**

```javascript
// Google Analytics 4
StringBoot.Experiment.onAssignment((experimentId, variant) => {
  gtag('event', 'experiment_assigned', {
    experiment_id: experimentId,
    variant: variant
  });
});

// Mixpanel
StringBoot.Experiment.onAssignment((experimentId, variant) => {
  mixpanel.track('Experiment Assigned', {
    experiment_id: experimentId,
    variant: variant
  });
});

// Amplitude
StringBoot.Experiment.onAssignment((experimentId, variant) => {
  amplitude.getInstance().logEvent('Experiment Assigned', {
    experiment_id: experimentId,
    variant: variant
  });
});
```

### Problem: Experiments Not Syncing

**Symptoms:**
- New experiments not appearing
- Variant weights not updating

**Solution:**

```javascript
// Force sync experiments
await StringBoot.Experiment.syncExperiments();

// Verify sync
const experiments = StringBoot.Experiment.getAllExperiments();
console.log(`Synced ${experiments.length} experiments`);

// Or enable auto-sync during initialization
await StringBoot.initialize({
  apiToken: 'YOUR_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  autoSyncExperiments: true
});
```

### Problem: Multiple Tabs Conflicting

**Symptoms:**
- Different variants in different tabs
- Assignments not consistent across tabs

**Solution - Use localStorage:**

```javascript
// Get or create consistent user ID across tabs
function getUserId() {
  let userId = localStorage.getItem('stringboot_user_id');
  if (!userId) {
    userId = `user-${Date.now()}-${Math.random()}`;
    localStorage.setItem('stringboot_user_id', userId);
  }
  return userId;
}

// Set user ID
StringBoot.Experiment.setUserId(getUserId());
```

### Debug A/B Testing

```javascript
// Enable debug logging
StringBoot.setLogLevel('debug');

// Check all experiments
const experiments = StringBoot.Experiment.getAllExperiments();
console.log('Available experiments:', experiments.length);
experiments.forEach(exp => {
  console.log(`  - ${exp.id}: ${exp.variants.length} variants`);
});

// Check user ID
const userId = StringBoot.Experiment.getUserId();
console.log('User ID:', userId);

// Test variant assignment
const variant = StringBoot.Experiment.getVariant('button_color');
console.log('Assigned variant:', variant);

// Check experiment details
const experiment = await StringBoot.Experiment.getExperiment('button_color');
console.log('Experiment status:', experiment?.status);
console.log('Variants:', experiment?.variants);
```

## 🔍 Performance Issues

### Problem: Slow String Retrieval

**Symptoms:**
- `getString()` takes too long
- UI lag when loading strings

**Solutions:**

#### 1. **Increase Cache Size**

```javascript
await StringBoot.initialize({
  apiToken: 'YOUR_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  cacheSize: 2000  // Increase cache
});
```

#### 2. **Preload Strings**

```javascript
// Preload commonly used strings
async function preloadStrings() {
  const keys = [
    'welcome_message',
    'login_button',
    'signup_button',
    'home_title'
  ];

  await Promise.all(
    keys.map(key => StringBoot.getString(key))
  );

  console.log('✅ Strings preloaded');
}

// Call on app init
await StringBoot.initialize({ /* config */ });
await preloadStrings();
```

#### 3. **Batch String Fetching**

```javascript
// ❌ Slow - Sequential requests
const str1 = await StringBoot.getString('key1');
const str2 = await StringBoot.getString('key2');
const str3 = await StringBoot.getString('key3');

// ✅ Fast - Parallel requests
const [str1, str2, str3] = await Promise.all([
  StringBoot.getString('key1'),
  StringBoot.getString('key2'),
  StringBoot.getString('key3')
]);
```

### Problem: High Memory Usage

**Symptoms:**
- Browser tab using too much memory
- Page crashes on mobile

**Solutions:**

#### 1. **Reduce Cache Size**

```javascript
await StringBoot.initialize({
  apiToken: 'YOUR_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  cacheSize: 500  // Reduce cache
});
```

#### 2. **Clear Cache Periodically**

```javascript
// Clear cache when navigating away
window.addEventListener('beforeunload', () => {
  StringBoot.clearCache();
});

// Or clear on memory pressure
if (performance.memory) {
  setInterval(() => {
    const usedMemory = performance.memory.usedJSHeapSize;
    const totalMemory = performance.memory.totalJSHeapSize;

    if (usedMemory / totalMemory > 0.9) {
      console.warn('High memory usage, clearing cache');
      StringBoot.clearCache();
    }
  }, 60000);  // Check every minute
}
```

## 📱 Framework-Specific Issues

### React Issues

**Problem: Hook Dependencies Warning**

```javascript
// ❌ Wrong - Missing dependency
useEffect(() => {
  StringBoot.getString('key').then(setText);
}, []);  // Missing 'setText' dependency

// ✅ Correct - Include all dependencies
useEffect(() => {
  StringBoot.getString('key').then(setText);
}, [setText]);

// Or use useCallback
const setText = useCallback((text) => {
  setTextState(text);
}, []);
```

### Next.js Issues

**Problem: Window is not defined (SSR)**

```javascript
// ❌ Wrong - Breaks SSR
import StringBoot from 'stringboot-web-sdk';
StringBoot.initialize({ /* config */ });

// ✅ Correct - Initialize client-side only
import { useEffect } from 'react';

function MyApp() {
  useEffect(() => {
    StringBoot.initialize({ /* config */ });
  }, []);
}
```

**Or use dynamic import:**

```javascript
import dynamic from 'next/dynamic';

const StringBootProvider = dynamic(
  () => import('../components/StringBootProvider'),
  { ssr: false }
);
```

### Vue Issues

**Problem: Reactivity Not Working**

```javascript
// ❌ Wrong - Not reactive
export default {
  data() {
    return {
      text: StringBoot.getString('key')  // Returns Promise!
    };
  }
};

// ✅ Correct - Async in mounted
export default {
  data() {
    return {
      text: ''
    };
  },
  async mounted() {
    this.text = await StringBoot.getString('key');
  }
};
```

## 📞 Support

- GitHub Issues: https://github.com/your-org/stringboot-web-sdk/issues
- Documentation: [README.md](README.md)
- Integration Guide: [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)
- FAQ Provider Guide: [FAQ_PROVIDER.md](FAQ_PROVIDER.md)
- A/B Testing Guide: [AB_TESTING.md](AB_TESTING.md)
- API Reference: [API_REFERENCE.md](API_REFERENCE.md)
