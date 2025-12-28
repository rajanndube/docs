# Next.js Integration Guide - StringBoot Web SDK

Complete guide for integrating StringBoot Web SDK in Next.js applications (App Router and Pages Router).

## Installation

```bash
npm install @stringboot/web-sdk
```

## App Router (Next.js 13+)

### 1. Create StringBoot Provider Component

Create `app/providers/stringboot-provider.tsx`:

```tsx
'use client';

import { useStringBoot } from '@stringboot/web-sdk/react';
import { ReactNode } from 'react';

interface StringBootProviderProps {
  children: ReactNode;
}

export function StringBootProvider({ children }: StringBootProviderProps) {
  const { initialized, error } = useStringBoot({
    apiToken: process.env.NEXT_PUBLIC_STRINGBOOT_API_TOKEN!,
    baseUrl: process.env.NEXT_PUBLIC_STRINGBOOT_API_URL || 'https://api.stringboot.com',
    defaultLanguage: 'en',
    debug: process.env.NODE_ENV === 'development',
  });

  if (error) {
    console.error('StringBoot initialization error:', error);
    // Optionally show error UI or fallback to hardcoded strings
  }

  // You can show a loading state or just render children immediately
  // The SDK works offline-first, so strings will load progressively
  return <>{children}</>;
}
```

### 2. Add Provider to Root Layout

Update `app/layout.tsx`:

```tsx
import { StringBootProvider } from './providers/stringboot-provider';
import './globals.css';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <StringBootProvider>
          {children}
        </StringBootProvider>
      </body>
    </html>
  );
}
```

### 3. Use in Client Components

Create `app/components/welcome.tsx`:

```tsx
'use client';

import { useString, useLanguage } from '@stringboot/web-sdk/react';

export function Welcome() {
  const welcomeText = useString('welcome_message');
  const [lang, setLang] = useLanguage();

  return (
    <div>
      <h1>{welcomeText}</h1>

      <select value={lang} onChange={(e) => setLang(e.target.value)}>
        <option value="en">English</option>
        <option value="es">Español</option>
        <option value="fr">Français</option>
      </select>
    </div>
  );
}
```

### 4. Use in Server Components

For server components, you can fetch strings server-side:

```tsx
// app/page.tsx (Server Component)
import StringBoot from '@stringboot/web-sdk';
import { Welcome } from './components/welcome';

async function getServerStrings() {
  // Initialize for server-side usage
  await StringBoot.initialize({
    apiToken: process.env.STRINGBOOT_API_TOKEN!,
    baseUrl: process.env.STRINGBOOT_API_URL || 'https://api.stringboot.com',
    defaultLanguage: 'en',
  });

  return {
    title: await StringBoot.get('page_title'),
    description: await StringBoot.get('page_description'),
  };
}

export default async function Home() {
  const strings = await getServerStrings();

  return (
    <main>
      <h1>{strings.title}</h1>
      <p>{strings.description}</p>

      {/* Client component with reactive strings */}
      <Welcome />
    </main>
  );
}
```

## Pages Router (Next.js 12 and below)

### 1. Create StringBoot Provider

Create `components/StringBootProvider.tsx`:

```tsx
import { useStringBoot } from '@stringboot/web-sdk/react';
import { ReactNode } from 'react';

interface StringBootProviderProps {
  children: ReactNode;
}

export function StringBootProvider({ children }: StringBootProviderProps) {
  const { initialized, error } = useStringBoot({
    apiToken: process.env.NEXT_PUBLIC_STRINGBOOT_API_TOKEN!,
    baseUrl: process.env.NEXT_PUBLIC_STRINGBOOT_API_URL || 'https://api.stringboot.com',
    defaultLanguage: 'en',
    debug: process.env.NODE_ENV === 'development',
  });

  if (error) {
    console.error('StringBoot initialization error:', error);
  }

  return <>{children}</>;
}
```

### 2. Add to _app.tsx

Update `pages/_app.tsx`:

```tsx
import type { AppProps } from 'next/app';
import { StringBootProvider } from '../components/StringBootProvider';
import '../styles/globals.css';

export default function App({ Component, pageProps }: AppProps) {
  return (
    <StringBootProvider>
      <Component {...pageProps} />
    </StringBootProvider>
  );
}
```

### 3. Use in Pages

```tsx
// pages/index.tsx
import { useString, useLanguage } from '@stringboot/web-sdk/react';

export default function Home() {
  const title = useString('home_title');
  const description = useString('home_description');
  const [lang, setLang] = useLanguage();

  return (
    <div>
      <h1>{title}</h1>
      <p>{description}</p>

      <select value={lang} onChange={(e) => setLang(e.target.value)}>
        <option value="en">English</option>
        <option value="es">Español</option>
      </select>
    </div>
  );
}
```

### 4. Server-Side Rendering (getServerSideProps)

```tsx
import { GetServerSideProps } from 'next';
import StringBoot from '@stringboot/web-sdk';

interface PageProps {
  title: string;
  description: string;
}

export const getServerSideProps: GetServerSideProps<PageProps> = async () => {
  await StringBoot.initialize({
    apiToken: process.env.STRINGBOOT_API_TOKEN!,
    baseUrl: process.env.STRINGBOOT_API_URL || 'https://api.stringboot.com',
    defaultLanguage: 'en',
  });

  return {
    props: {
      title: await StringBoot.get('page_title'),
      description: await StringBoot.get('page_description'),
    },
  };
};

export default function Page({ title, description }: PageProps) {
  return (
    <div>
      <h1>{title}</h1>
      <p>{description}</p>
    </div>
  );
}
```

### 5. Static Generation (getStaticProps)

```tsx
import { GetStaticProps } from 'next';
import StringBoot from '@stringboot/web-sdk';

interface PageProps {
  strings: {
    [key: string]: string;
  };
}

export const getStaticProps: GetStaticProps<PageProps> = async () => {
  await StringBoot.initialize({
    apiToken: process.env.STRINGBOOT_API_TOKEN!,
    baseUrl: process.env.STRINGBOOT_API_URL || 'https://api.stringboot.com',
    defaultLanguage: 'en',
  });

  return {
    props: {
      strings: {
        title: await StringBoot.get('page_title'),
        subtitle: await StringBoot.get('page_subtitle'),
        cta: await StringBoot.get('cta_button'),
      },
    },
    revalidate: 60, // Revalidate every 60 seconds
  };
};

export default function Page({ strings }: PageProps) {
  return (
    <div>
      <h1>{strings.title}</h1>
      <h2>{strings.subtitle}</h2>
      <button>{strings.cta}</button>
    </div>
  );
}
```

## Environment Variables

Create `.env.local`:

```bash
# Public (exposed to browser)
NEXT_PUBLIC_STRINGBOOT_API_TOKEN=sk_live_your_token_here
NEXT_PUBLIC_STRINGBOOT_API_URL=https://api.stringboot.com

# Server-side only (not exposed to browser)
STRINGBOOT_API_TOKEN=sk_live_your_token_here
STRINGBOOT_API_URL=https://api.stringboot.com
```

**Important**:
- Use `NEXT_PUBLIC_` prefix for client-side usage
- Use regular env vars for server-side only (more secure)

## Advanced Patterns

### Language Switcher Component

```tsx
'use client'; // or regular component for Pages Router

import { useLanguage, useActiveLanguages } from '@stringboot/web-sdk/react';

export function LanguageSwitcher() {
  const [currentLang, setLanguage] = useLanguage();
  const { languages, loading, error } = useActiveLanguages();

  if (loading) return <div>Loading languages...</div>;
  if (error) return null;

  return (
    <select
      value={currentLang}
      onChange={(e) => setLanguage(e.target.value)}
      className="border rounded px-3 py-2"
    >
      {languages.map((lang) => (
        <option key={lang.code} value={lang.code}>
          {lang.name}
        </option>
      ))}
    </select>
  );
}
```

### Multiple Strings Hook

```tsx
'use client';

import { useStrings } from '@stringboot/web-sdk/react';

export function ProductCard() {
  const strings = useStrings([
    'product_name',
    'product_description',
    'add_to_cart',
    'price_label',
  ]);

  return (
    <div className="card">
      <h3>{strings.product_name}</h3>
      <p>{strings.product_description}</p>
      <div>
        <span>{strings.price_label}</span>
        <button>{strings.add_to_cart}</button>
      </div>
    </div>
  );
}
```

### Manual Sync Component

```tsx
'use client';

import { useSync } from '@stringboot/web-sdk/react';

export function SyncButton() {
  const { sync, syncing } = useSync();

  return (
    <button
      onClick={() => sync()}
      disabled={syncing}
      className="px-4 py-2 bg-blue-500 text-white rounded"
    >
      {syncing ? 'Syncing...' : 'Sync Strings'}
    </button>
  );
}
```

### With Metadata (App Router)

```tsx
// app/page.tsx
import { Metadata } from 'next';
import StringBoot from '@stringboot/web-sdk';

export async function generateMetadata(): Promise<Metadata> {
  await StringBoot.initialize({
    apiToken: process.env.STRINGBOOT_API_TOKEN!,
    baseUrl: process.env.STRINGBOOT_API_URL || 'https://api.stringboot.com',
    defaultLanguage: 'en',
  });

  return {
    title: await StringBoot.get('meta_title'),
    description: await StringBoot.get('meta_description'),
  };
}

export default function Page() {
  return <div>Page content</div>;
}
```

### Error Boundary

```tsx
'use client';

import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
}

interface State {
  hasError: boolean;
}

export class StringBootErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error: Error) {
    console.error('StringBoot error:', error);
  }

  render() {
    if (this.state.hasError) {
      return <div>Failed to load translations. Please refresh.</div>;
    }

    return this.props.children;
  }
}

// Usage in layout:
<StringBootErrorBoundary>
  <StringBootProvider>
    {children}
  </StringBootProvider>
</StringBootErrorBoundary>
```

## TypeScript Configuration

Add to `tsconfig.json`:

```json
{
  "compilerOptions": {
    "types": ["@stringboot/web-sdk"]
  }
}
```

## Troubleshooting

### Issue: "Hydration mismatch" error

**Cause**: Server-rendered content doesn't match client-rendered content.

**Solution**: Use consistent initialization on both server and client:

```tsx
// app/layout.tsx
export const metadata = {
  title: 'Loading...', // Avoid using StringBoot in metadata if using client provider
};
```

### Issue: Strings not updating after language change

**Cause**: Not using reactive hooks.

**Solution**: Use `useString` instead of direct `StringBoot.get()`:

```tsx
// ❌ Won't update
const [text, setText] = useState('');
useEffect(() => {
  StringBoot.get('key').then(setText);
}, []);

// ✅ Auto-updates
const text = useString('key');
```

### Issue: Environment variables undefined

**Cause**: Missing `NEXT_PUBLIC_` prefix for client-side usage.

**Solution**:
- Use `NEXT_PUBLIC_*` for client components
- Use regular env vars for server components

### Issue: Build fails with "Module not found"

**Cause**: Trying to import React hooks in server component.

**Solution**: Add `'use client'` directive:

```tsx
'use client';

import { useString } from '@stringboot/web-sdk/react';
```

## Performance Optimization

### 1. Preload Critical Strings

```tsx
// app/layout.tsx
import StringBoot from '@stringboot/web-sdk';

// Preload in server component
async function preloadStrings() {
  await StringBoot.initialize({
    apiToken: process.env.STRINGBOOT_API_TOKEN!,
    baseUrl: process.env.STRINGBOOT_API_URL!,
    defaultLanguage: 'en',
  });

  // Preload critical strings
  await Promise.all([
    StringBoot.get('nav_home'),
    StringBoot.get('nav_about'),
    StringBoot.get('nav_contact'),
  ]);
}

export default async function RootLayout({ children }) {
  await preloadStrings();

  return (
    <html>
      <body>
        <StringBootProvider>{children}</StringBootProvider>
      </body>
    </html>
  );
}
```

### 2. Use Static Generation When Possible

For content that doesn't change frequently:

```tsx
export const revalidate = 3600; // Revalidate every hour

export default async function Page() {
  const strings = {
    title: await StringBoot.get('title'),
    content: await StringBoot.get('content'),
  };

  return <div>{strings.title}</div>;
}
```

### 3. Code Splitting

Load StringBoot only where needed:

```tsx
import dynamic from 'next/dynamic';

const DynamicStringComponent = dynamic(
  () => import('./components/string-component'),
  { ssr: false }
);
```

## Best Practices

1. ✅ **Initialize once** in root layout/app
2. ✅ **Use `useString` hooks** for reactive updates
3. ✅ **Separate client/server env vars** for security
4. ✅ **Handle errors gracefully** with error boundaries
5. ✅ **Preload critical strings** for better performance
6. ❌ **Don't initialize in every component**
7. ❌ **Don't use `StringBoot.get()` directly in render**
8. ❌ **Don't expose server tokens** to client

## Example Project Structure

```
next-app/
├── app/                          # App Router
│   ├── layout.tsx               # Root layout with provider
│   ├── page.tsx                 # Home page
│   ├── providers/
│   │   └── stringboot-provider.tsx
│   └── components/
│       ├── language-switcher.tsx
│       └── welcome.tsx
├── .env.local                   # Environment variables
└── package.json
```

## Support

For more help:
- 📖 [Main Documentation](README.md)
- 🐛 [Report Issues](https://github.com/rajanndube/Stringboot-Client/issues)
- 💬 [Discord Community](https://discord.gg/stringboot)

---

**Last Updated**: November 2024
