# LocaleSwitcher — No Routing, No URL Change

## src/components/LocaleSwitcher.tsx

```tsx
import { useTranslation } from 'react-i18next';
import { LANGUAGES, changeLang } from '@/i18n';
import type { LangCode } from '@/i18n';

interface Props {
  className?: string;
}

export function LocaleSwitcher({ className }: Props) {
  const { i18n } = useTranslation();
  const current = i18n.language as LangCode;

  return (
    <select
      value={current}
      onChange={(e) => changeLang(e.target.value as LangCode)}
      className={className}
      aria-label="Select language"
    >
      {(Object.entries(LANGUAGES) as [LangCode, string][]).map(([code, label]) => (
        <option key={code} value={code}>
          {label}
        </option>
      ))}
    </select>
  );
}
```

**How it works:**

- `changeLang()` calls `i18n.changeLanguage()` → all components using `useTranslation()` re-render instantly
- Saves choice to `localStorage` → persists on page reload
- Sets `document.documentElement.lang` for accessibility
- No URL change, no page reload, no router dependency

**Usage — drop anywhere:**

```tsx
import { LocaleSwitcher } from '@/components/LocaleSwitcher';

// In your Navbar, Header, Settings page, etc:
<LocaleSwitcher className="your-styles-here" />;
```

**To read current language anywhere:**

```tsx
import { getCurrentLang } from '@/i18n';
const lang = getCurrentLang(); // 'en' | 'hi' | 'gu'
```

**To switch language programmatically:**

```tsx
import { changeLang } from '@/i18n';
await changeLang('hi'); // switches + saves, no redirect
```

**To translate in a component:**

```tsx
import { useTranslation } from 'react-i18next';

export function MyComponent() {
  const { t } = useTranslation('common'); // namespace
  const { t: tAuth } = useTranslation('auth'); // different namespace

  return (
    <div>
      <h1>{t('nav.home')}</h1>
      <p>{t('greeting', { name: 'Raj' })}</p>
      <button>{tAuth('login.submit')}</button>
    </div>
  );
}
```
