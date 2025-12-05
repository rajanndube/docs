# A/B Testing with Stringboot Web SDK

Stringboot enables **server-side A/B testing for strings** without deployments. Run experiments on copy, messaging, and UI text to optimize user engagement and conversions.

## Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [Quick Start](#quick-start)
- [Device ID Management](#device-id-management)
- [Analytics Integration](#analytics-integration)
- [Experiment Assignment](#experiment-assignment)
- [Testing Your Integration](#testing-your-integration)
- [Framework Integration](#framework-integration)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Examples](#examples)

## Overview

### What is A/B Testing with Stringboot?

Stringboot's A/B testing allows you to:
- ✅ Test different string variations without code changes or deployments
- ✅ Run experiments on headlines, CTAs, error messages, onboarding copy
- ✅ Automatically assign users to consistent experiment variants
- ✅ Track conversions and analytics for each variant
- ✅ Make data-driven decisions on string effectiveness

### Key Benefits

- **No Deployment Required** - Create and deploy experiments instantly
- **Consistent Assignment** - Each device always sees the same variant
- **Automatic Tracking** - Optional analytics integration for conversion tracking
- **Privacy-Friendly** - Uses anonymous device IDs (not personal data)
- **Backend-Controlled** - Experiments managed from Stringboot dashboard
- **Works Offline** - Cached assignments persist in IndexedDB

## How It Works

### Architecture

```
┌─────────────────┐
│  Stringboot     │
│  Dashboard      │ ← Create experiment: "welcome_message"
│                 │   - Control: "Welcome!"
│                 │   - Variant A: "Hey there!"
│                 │   - Variant B: "Welcome back!"
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│   Backend       │ ← Assigns devices to variants
│   (Assignment   │   based on device ID hash
│    Engine)      │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│   Web SDK       │ ← Requests strings with device ID
│  (Your App)     │ ← Receives assigned variant
│                 │ ← Displays string to user
└─────────────────┘
```

### The Flow

1. **Backend receives request** with `X-Device-ID` header
2. **Checks for active experiments** for the requested string key
3. **Assigns variant** based on device ID hash (if first time)
4. **Stores assignment** in database for consistency
5. **Returns variant string** to SDK
6. **SDK caches assignment** in IndexedDB for offline use

### Consistency Guarantee

Once a device is assigned to a variant, it **always** receives the same variant, even:
- ✅ Across browser sessions
- ✅ After cache clears
- ✅ On different browsers with same device ID
- ✅ Until experiment ends

## Quick Start

### Step 1: Enable A/B Testing (30 seconds)

The SDK automatically supports A/B testing. You just need to initialize it:

```javascript
import StringBoot from '@stringboot/web-sdk';

// Device ID is automatically generated and persisted in LocalStorage
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  debug: true  // Enable logging to see experiment assignments
});
```

**That's it!** Your app now supports A/B testing. The SDK automatically:
- ✅ Generates and persists a unique device ID (UUID)
- ✅ Sends device ID with all string requests
- ✅ Receives and caches experiment assignments
- ✅ Returns assigned variants for experiment strings

### Custom Device ID (Optional)

Provide your own device ID for cross-platform consistency:

```javascript
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  defaultLanguage: 'en',
  deviceId: 'your-custom-device-id'  // e.g., from Firebase, user ID, etc.
});
```

### Step 2: Use Strings Normally

No code changes needed! Use strings exactly as before:

```javascript
// Vanilla JS
const welcomeText = await StringBoot.get('welcome_message');
document.getElementById('title').textContent = welcomeText;
// Returns "Welcome!" or "Hey there!" depending on assignment

// React
function WelcomeComponent() {
  const welcomeText = useString('welcome_message');
  return <h1>{welcomeText}</h1>;
}

// Vue
<template>
  <h1>{{ welcomeText }}</h1>
</template>

<script setup>
import { ref } from 'vue';
import StringBoot from '@stringboot/web-sdk';

const welcomeText = ref('');

StringBoot.watch('welcome_message', (value) => {
  welcomeText.value = value;
});
</script>
```

### Step 3: Create Experiment on Dashboard

1. Go to Stringboot Dashboard → Experiments
2. Click **Create Experiment**
3. Select string key (e.g., `welcome_message`)
4. Add variants:
   - Control: "Welcome!"
   - Variant A: "Hey there!"
   - Variant B: "Welcome back!"
5. Set traffic allocation (e.g., 33% / 33% / 34%)
6. Click **Start Experiment**

**Users automatically assigned** to variants based on their device ID!

## Device ID Management

### Auto-Generated Device ID

By default, the SDK generates a UUID and stores it in `localStorage`:

```javascript
// Automatic - SDK handles this internally
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com'
});

// Device ID is stored in:
// localStorage.getItem('stringboot_device_id')
```

### Custom Device ID

Provide your own device ID for cross-platform consistency:

```javascript
// Use Firebase Installation ID
import { getInstallations, getId } from 'firebase/installations';

const firebaseID = await getId(getInstallations());

await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  deviceId: firebaseID
});

// Or use authenticated user ID
const userId = auth.currentUser?.uid;
if (userId) {
  await StringBoot.initialize({
    apiToken: 'YOUR_API_TOKEN',
    baseUrl: 'https://api.stringboot.com',
    deviceId: userId
  });
}
```

#### Common Custom ID Sources

| Source | Use Case | Example |
|--------|----------|---------|
| **Firebase IID** | Cross-platform consistency | `getId(getInstallations())` |
| **User ID** | Logged-in users | `auth.currentUser.uid` |
| **Session ID** | Per-session testing | `crypto.randomUUID()` |
| **Custom UUID** | Full control | `generateCustomId()` |

⚠️ **Privacy Note:**
- Never use personally identifiable information (email, phone) as device ID
- Follow GDPR/CCPA requirements if storing device IDs
- Provide opt-out mechanism if required by law

### Retrieve Current Device ID

```javascript
// Get current device ID
const deviceId = localStorage.getItem('stringboot_device_id');
console.log('Current device ID:', deviceId);

// Or access via SDK (if exposed)
const currentDeviceId = StringBoot.getDeviceId();
```

## Analytics Integration

### Overview

Track experiment assignments and conversions by implementing a custom analytics handler.

### Implement Analytics Handler

```javascript
// analytics-handler.js
import StringBoot from '@stringboot/web-sdk';

class AnalyticsHandler {
  onExperimentAssigned(assignment) {
    // Called when device is assigned to an experiment variant

    // Send to Google Analytics 4
    if (window.gtag) {
      gtag('event', 'experiment_assigned', {
        experiment_id: assignment.experimentId,
        experiment_key: assignment.experimentKey,
        string_key: assignment.stringKey,
        variant_name: assignment.variantName,
        language: assignment.languageCode
      });
    }

    // Send to Mixpanel
    if (window.mixpanel) {
      mixpanel.track('Experiment Assigned', {
        experiment_id: assignment.experimentId,
        experiment_key: assignment.experimentKey,
        variant_name: assignment.variantName
      });
    }

    // Send to Amplitude
    if (window.amplitude) {
      amplitude.getInstance().logEvent('Experiment Assigned', {
        experiment_key: assignment.experimentKey,
        variant: assignment.variantName
      });
    }

    // Send to Segment
    if (window.analytics) {
      analytics.track('Experiment Assigned', {
        experimentId: assignment.experimentId,
        experimentKey: assignment.experimentKey,
        variantName: assignment.variantName
      });
    }
  }

  onExperimentViewed(experimentKey, variantName) {
    // Called when string is actually displayed to user
    if (window.gtag) {
      gtag('event', 'experiment_viewed', {
        experiment_key: experimentKey,
        variant_name: variantName
      });
    }
  }
}

export default AnalyticsHandler;
```

### Register Analytics Handler

```javascript
import StringBoot from '@stringboot/web-sdk';
import AnalyticsHandler from './analytics-handler';

const analyticsHandler = new AnalyticsHandler();

await StringBoot.initialize({
  apiToken: process.env.STRINGBOOT_API_TOKEN,
  baseUrl: 'https://api.stringboot.com',
  analyticsHandler: analyticsHandler
});
```

### Track Conversions

Track user actions to measure experiment effectiveness:

```javascript
// Vanilla JS
document.getElementById('cta-button').addEventListener('click', () => {
  trackConversion('button_clicked', 'welcome_experiment');
});

function trackConversion(action, experimentKey) {
  const deviceId = localStorage.getItem('stringboot_device_id');

  // Google Analytics
  gtag('event', 'experiment_conversion', {
    experiment_key: experimentKey,
    action: action,
    device_id: deviceId
  });

  // Mixpanel
  mixpanel.track('Experiment Conversion', {
    experiment_key: experimentKey,
    action: action
  });
}

// React
function CTAButton() {
  const handleClick = () => {
    const deviceId = localStorage.getItem('stringboot_device_id');

    // Track conversion
    window.gtag?.('event', 'experiment_conversion', {
      experiment_key: 'welcome_experiment',
      action: 'cta_clicked',
      device_id: deviceId
    });
  };

  return (
    <button onClick={handleClick}>
      {useString('cta_button')}
    </button>
  );
}

// Vue
<template>
  <button @click="handleClick">{{ ctaText }}</button>
</template>

<script setup>
import { ref } from 'vue';
import StringBoot from '@stringboot/web-sdk';

const ctaText = ref('');

StringBoot.watch('cta_button', (value) => {
  ctaText.value = value;
});

const handleClick = () => {
  const deviceId = localStorage.getItem('stringboot_device_id');

  window.gtag?.('event', 'experiment_conversion', {
    experiment_key: 'welcome_experiment',
    action: 'cta_clicked',
    device_id: deviceId
  });
};
</script>
```

## Experiment Assignment

### How Assignment Works

1. **Hash-Based Distribution**: Device ID is hashed to determine variant
2. **Deterministic**: Same device ID always gets same variant
3. **Weighted Random**: Traffic split (e.g., 50/50, 33/33/34) honored
4. **Sticky**: Assignment persisted in IndexedDB across sessions

### Example Assignment Flow

```javascript
// User 1 (device ID: "abc123")
const text1 = await StringBoot.get('welcome_message');
// → Backend hashes "abc123" → assigns to Variant A
// → Returns "Hey there!"
// → Assignment cached in IndexedDB

// User 2 (device ID: "xyz789")
const text2 = await StringBoot.get('welcome_message');
// → Backend hashes "xyz789" → assigns to Control
// → Returns "Welcome!"

// User 1 again (same browser, different session)
const text3 = await StringBoot.get('welcome_message');
// → IndexedDB contains cached assignment
// → Returns "Hey there!" (consistent!)
```

### Experiment Metadata

The backend may include experiment metadata in responses:

```javascript
// SDK handles this internally, but for reference:
// Response includes:
{
  "strings": {
    "en": {
      "welcome_message": "Hey there!"  // ← Assigned variant
    }
  },
  "experiments": {  // ← Optional metadata
    "welcome_message": {
      "experimentId": "exp-123",
      "experimentKey": "welcome_test",
      "variantName": "variant-a",
      "assignedAt": "2025-01-15T10:30:00Z"
    }
  }
}
```

The SDK automatically caches this metadata in IndexedDB and uses it for analytics callbacks.

## Testing Your Integration

### Test Different Variants

#### Method 1: Use Different Device IDs

```javascript
// Clear device ID and reinitialize
localStorage.removeItem('stringboot_device_id');

// Initialize with custom device ID
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  deviceId: 'test-device-001'  // Variant A
});

// Later, test another variant
localStorage.removeItem('stringboot_device_id');
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  deviceId: 'test-device-002'  // Variant B
});
```

#### Method 2: Clear Browser Storage

```javascript
// In DevTools Console:
localStorage.clear();
indexedDB.deleteDatabase('stringboot');
location.reload();

// New device ID generated → different variant assignment
```

#### Method 3: Incognito/Private Browsing

- Open incognito window → generates new device ID
- Test multiple times in different incognito windows
- Each gets different assignment

### Verify Analytics Events

Check that analytics events are firing:

```javascript
// Enable debug mode
await StringBoot.initialize({
  apiToken: 'YOUR_API_TOKEN',
  baseUrl: 'https://api.stringboot.com',
  debug: true  // Logs all events to console
});

// Watch console for:
// [StringBoot] Experiment assigned: welcome_test → variant-a
// [Analytics] Experiment Assigned: {...}
```

## Framework Integration

### React

```tsx
import { useString } from '@stringboot/web-sdk/react';
import { useEffect } from 'react';

function ExperimentComponent() {
  const welcomeText = useString('welcome_message');

  useEffect(() => {
    // Track experiment view
    const deviceId = localStorage.getItem('stringboot_device_id');

    window.gtag?.('event', 'experiment_viewed', {
      string_key: 'welcome_message',
      device_id: deviceId
    });
  }, [welcomeText]);

  const handleCTAClick = () => {
    window.gtag?.('event', 'experiment_conversion', {
      experiment_key: 'welcome_experiment',
      action: 'cta_clicked'
    });
  };

  return (
    <div>
      <h1>{welcomeText}</h1>
      <button onClick={handleCTAClick}>Get Started</button>
    </div>
  );
}
```

### Next.js

```tsx
// app/providers.tsx
'use client';

import { useStringBoot } from '@stringboot/web-sdk/react';
import AnalyticsHandler from './analytics-handler';

export function Providers({ children }: { children: React.ReactNode }) {
  const { initialized } = useStringBoot({
    apiToken: process.env.NEXT_PUBLIC_STRINGBOOT_TOKEN!,
    baseUrl: 'https://api.stringboot.com',
    analyticsHandler: new AnalyticsHandler(),
    debug: process.env.NODE_ENV === 'development'
  });

  if (!initialized) return <div>Loading...</div>;

  return <>{children}</>;
}

// app/components/ExperimentButton.tsx
'use client';

import { useString } from '@stringboot/web-sdk/react';

export function ExperimentButton() {
  const buttonText = useString('cta_button');

  const handleClick = () => {
    window.gtag?.('event', 'experiment_conversion', {
      experiment_key: 'cta_experiment',
      action: 'button_clicked'
    });
  };

  return <button onClick={handleClick}>{buttonText}</button>;
}
```

### Vue 3

```vue
<template>
  <div>
    <h1>{{ welcomeText }}</h1>
    <button @click="handleClick">{{ ctaText }}</button>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import StringBoot from '@stringboot/web-sdk';

const welcomeText = ref('Loading...');
const ctaText = ref('Loading...');

onMounted(() => {
  // Watch for string updates
  StringBoot.watch('welcome_message', (value) => {
    welcomeText.value = value;

    // Track experiment view
    window.gtag?.('event', 'experiment_viewed', {
      string_key: 'welcome_message'
    });
  });

  StringBoot.watch('cta_button', (value) => {
    ctaText.value = value;
  });
});

const handleClick = () => {
  window.gtag?.('event', 'experiment_conversion', {
    experiment_key: 'welcome_experiment',
    action: 'cta_clicked'
  });
};
</script>
```

## Best Practices

### 1. Experiment Design

✅ **DO:**
- Test one variable at a time (e.g., headline copy, not headline + CTA)
- Run experiments for sufficient sample size (typically 1-2 weeks minimum)
- Define success metrics before starting
- Use descriptive experiment keys (`homepage_hero_headline`)

❌ **DON'T:**
- Test too many variants simultaneously (3-4 max recommended)
- End experiments too early (statistical significance requires time)
- Change experiment mid-flight (invalidates results)

### 2. Device ID Strategy

✅ **DO:**
- Use auto-generated UUID for anonymous tracking
- Use Firebase Installation ID for cross-platform consistency
- Document your device ID strategy
- Follow GDPR/CCPA requirements

❌ **DON'T:**
- Use personal data (email, phone, IP address)
- Share device IDs with third parties without consent
- Change device ID generation mid-experiment

### 3. Analytics Integration

✅ **DO:**
- Track assignment events for all experiments
- Include device ID in conversion events
- Set up proper event taxonomy (consistent naming)
- Test analytics in staging before production

❌ **DON'T:**
- Send excessive events (batch when possible)
- Block UI on analytics calls (always async)
- Forget to handle analytics failures gracefully
- Track personal data without user consent

### 4. Performance

✅ **DO:**
- Use caching (`StringBoot.watch()` for reactive updates)
- Leverage IndexedDB for offline support
- Minimize network requests
- Use browser `localStorage` for device ID persistence

❌ **DON'T:**
- Make synchronous API calls
- Ignore offline scenarios
- Clear IndexedDB unnecessarily
- Fetch same string multiple times

### 5. Testing

✅ **DO:**
- Test with multiple device IDs before launch
- Verify analytics events fire correctly
- Test offline behavior (IndexedDB cache)
- Test in different browsers
- Document QA test plan

❌ **DON'T:**
- Skip testing analytics integration
- Assume experiments work without verification
- Test only on one variant
- Forget to test incognito/private browsing

## Troubleshooting

### Problem: All users see the same variant

**Solution:**
- Verify different device IDs are being generated
- Check browser console for device ID: `localStorage.getItem('stringboot_device_id')`
- Check backend experiment configuration (traffic split)
- Ensure experiment is "running" (not paused/ended)

### Problem: Variant changes between sessions

**Solution:**
- Check that device ID is persisted in `localStorage`
- Verify `localStorage` is not being cleared by browser/extension
- Ensure you're not generating a new device ID on each page load
- Check browser privacy settings (some browsers clear on exit)

### Problem: Analytics events not firing

**Solution:**
- Verify `analyticsHandler` is passed to `initialize()`
- Check browser console for errors
- Ensure analytics SDK (GA4/Mixpanel) is loaded before Stringboot
- Verify analytics SDK configuration
- Check network tab for analytics requests

### Problem: Can't test specific variant

**Solution:**
- Use custom device ID: `deviceId: 'test-variant-a'`
- Ask backend team to manually assign device ID to variant
- Clear browser storage and retry
- Check experiment traffic allocation (might be 0% for some variants)

### Problem: IndexedDB errors

**Solution:**
- Check browser supports IndexedDB: `'indexedDB' in window`
- Verify IndexedDB isn't disabled in browser settings
- Check available storage quota
- Handle errors gracefully with fallback to in-memory cache

## Examples

### Complete Integration Example

```javascript
// main.js
import StringBoot from '@stringboot/web-sdk';

// Analytics Handler
class MyAnalyticsHandler {
  onExperimentAssigned(assignment) {
    // Google Analytics 4
    gtag('event', 'experiment_assigned', {
      experiment_key: assignment.experimentKey,
      variant: assignment.variantName,
      string_key: assignment.stringKey
    });

    console.log('Experiment assigned:', assignment);
  }

  onExperimentViewed(experimentKey, variantName) {
    gtag('event', 'experiment_viewed', {
      experiment_key: experimentKey,
      variant: variantName
    });
  }
}

// Initialize
async function initializeApp() {
  await StringBoot.initialize({
    apiToken: process.env.STRINGBOOT_API_TOKEN,
    baseUrl: 'https://api.stringboot.com',
    defaultLanguage: 'en',
    analyticsHandler: new MyAnalyticsHandler(),
    debug: process.env.NODE_ENV === 'development'
  });

  console.log('StringBoot initialized with device ID:',
    localStorage.getItem('stringboot_device_id'));

  // Load strings
  await loadStrings();
}

async function loadStrings() {
  // Get experiment strings
  const welcomeText = await StringBoot.get('welcome_message');
  const ctaText = await StringBoot.get('cta_button');

  // Update DOM
  document.getElementById('welcome').textContent = welcomeText;
  document.getElementById('cta').textContent = ctaText;

  // Track conversion on click
  document.getElementById('cta').addEventListener('click', () => {
    gtag('event', 'experiment_conversion', {
      experiment_key: 'welcome_experiment',
      action: 'cta_clicked',
      device_id: localStorage.getItem('stringboot_device_id')
    });
  });
}

// Initialize on load
document.addEventListener('DOMContentLoaded', initializeApp);
```

## Related Documentation

- [QUICKSTART.md](QUICKSTART.md) - Basic SDK integration
- [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md) - Comprehensive integration guide
- [NEXTJS_INTEGRATION.md](NEXTJS_INTEGRATION.md) - Next.js-specific guide
- [ADVANCED_FEATURES.md](ADVANCED_FEATURES.md) - Advanced SDK capabilities
- [API_REFERENCE.md](API_REFERENCE.md) - Complete API documentation

## Need Help?

- **Email:** support@stringboot.com
- **Documentation:** https://docs.stringboot.com
- **Dashboard:** https://app.stringboot.com

---

**Start optimizing your web app copy with data-driven A/B tests!** 📊

