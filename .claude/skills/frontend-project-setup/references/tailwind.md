# Tailwind CSS Reference

## Install (Vite)
```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

## Install (Next.js)
Already included if selected during `create-next-app`. Otherwise same as above.

## tailwind.config.ts (v3 — most common)
```ts
import type { Config } from 'tailwindcss'

const config: Config = {
  content: [
    './src/**/*.{js,ts,jsx,tsx,mdx}',
    // For Next.js App Router:
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        brand: {
          50:  '#eff6ff',
          500: '#3b82f6',
          900: '#1e3a8a',
        },
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
      },
    },
  },
  plugins: [],
  // When combining with Ant Design — prevents CSS reset conflicts:
  // corePlugins: { preflight: false },
}

export default config
```

## Tailwind v4 (CSS-based config — newer projects)
```css
/* src/styles/global.css */
@import "tailwindcss";

@theme {
  --color-brand-500: #3b82f6;
  --font-sans: Inter, sans-serif;
}
```
No `tailwind.config.ts` needed in v4.

## Base CSS import
```css
/* src/styles/global.css or src/app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom layer utilities */
@layer components {
  .btn-primary {
    @apply px-4 py-2 bg-brand-500 text-white rounded-lg hover:bg-brand-600 transition-colors;
  }
}
```

## Combining Tailwind + Ant Design
Add to tailwind.config.ts to prevent style conflicts:
```ts
corePlugins: {
  preflight: false,
}
```
And import Ant Design's CSS reset manually:
```ts
import 'antd/dist/reset.css' // Only needed if preflight is disabled
```

## Recommended Tailwind plugins
- `@tailwindcss/forms` — better form defaults
- `@tailwindcss/typography` — prose/rich text styling
- `@tailwindcss/aspect-ratio` — aspect ratio utilities
- `tailwind-merge` — safely merge Tailwind class strings (use with `clsx`)

## clsx + tailwind-merge pattern
```ts
// src/lib/utils.ts
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```
