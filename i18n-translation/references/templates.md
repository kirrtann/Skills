# i18n Templates

## src/i18n/config.ts

```ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';

// ── Add/remove languages here ──────────────────────────
export const LANGUAGES = {
  en: 'English',
  hi: 'हिन्दी',
  gu: 'ગુજરાતી',
} as const;

export type LangCode = keyof typeof LANGUAGES;
export const DEFAULT_LANG: LangCode = 'en';

// ── Import all locale JSON files directly ──────────────
// No HTTP backend — bundled at build time, no network calls
import enCommon from './locales/en/common.json';
import enAuth from './locales/en/auth.json';
import enDashboard from './locales/en/dashboard.json';

import hiCommon from './locales/hi/common.json';
import hiAuth from './locales/hi/auth.json';
import hiDashboard from './locales/hi/dashboard.json';

import guCommon from './locales/gu/common.json';
import guAuth from './locales/gu/auth.json';
import guDashboard from './locales/gu/dashboard.json';

i18n.use(initReactI18next).init({
  lng: (localStorage.getItem('lang') as LangCode) ?? DEFAULT_LANG,
  fallbackLng: DEFAULT_LANG,
  defaultNS: 'common',
  ns: ['common', 'auth', 'dashboard'],
  resources: {
    en: { common: enCommon, auth: enAuth, dashboard: enDashboard },
    hi: { common: hiCommon, auth: hiAuth, dashboard: hiDashboard },
    gu: { common: guCommon, auth: guAuth, dashboard: guDashboard },
  },
  interpolation: { escapeValue: false },
});

export default i18n;
```

## src/i18n/index.ts

```ts
export { default } from './config';
export { LANGUAGES, DEFAULT_LANG } from './config';
export type { LangCode } from './config';
export { changeLang, getCurrentLang, formatDate, formatNumber, formatCurrency } from './utils';
```

## src/i18n/types.ts

```ts
import type enCommon from './locales/en/common.json';
import type enAuth from './locales/en/auth.json';
import type enDashboard from './locales/en/dashboard.json';

// Recursive dot-path extractor for full type-safety on t() calls
type Paths<T, P extends string = ''> = {
  [K in keyof T]: T[K] extends Record<string, unknown>
    ? Paths<T[K], `${P}${string & K}.`>
    : `${P}${string & K}`;
}[keyof T];

export type CommonKeys = Paths<typeof enCommon>;
export type AuthKeys = Paths<typeof enAuth>;
export type DashboardKeys = Paths<typeof enDashboard>;
```

## src/i18n/utils.ts

```ts
import i18n from './config';
import type { LangCode } from './config';

/** Switch language and persist to localStorage. No URL change. */
export async function changeLang(lang: LangCode): Promise<void> {
  await i18n.changeLanguage(lang);
  localStorage.setItem('lang', lang);
  document.documentElement.lang = lang;
  // RTL support
  document.documentElement.dir = ['ar', 'he', 'fa', 'ur'].includes(lang) ? 'rtl' : 'ltr';
}

/** Get current active language code. */
export function getCurrentLang(): LangCode {
  return (i18n.language as LangCode) ?? 'en';
}

/** Format a date in the current locale. */
export function formatDate(
  date: Date | string | number,
  options: Intl.DateTimeFormatOptions = { dateStyle: 'medium' }
): string {
  return new Intl.DateTimeFormat(getCurrentLang(), options).format(new Date(date));
}

/** Format a number in the current locale. */
export function formatNumber(value: number, options: Intl.NumberFormatOptions = {}): string {
  return new Intl.NumberFormat(getCurrentLang(), options).format(value);
}

/** Format currency in the current locale. */
export function formatCurrency(value: number, currency = 'INR'): string {
  return new Intl.NumberFormat(getCurrentLang(), { style: 'currency', currency }).format(value);
}
```
