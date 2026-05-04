# SCSS / SASS Reference

## Install
```bash
npm install -D sass
```
No extra config needed — Vite and Next.js both support `.scss` natively.

## Recommended File Structure
```
src/styles/
├── _variables.scss    # Design tokens
├── _mixins.scss       # Reusable mixins
├── _reset.scss        # Custom reset (optional if using Tailwind preflight)
├── _typography.scss   # Global type styles
└── global.scss        # Imports all partials + global rules
```

## _variables.scss
```scss
// Colors
$color-primary:   #3b82f6;
$color-secondary: #8b5cf6;
$color-success:   #10b981;
$color-danger:    #ef4444;
$color-warning:   #f59e0b;
$color-bg:        #ffffff;
$color-text:      #111827;

// Spacing
$spacing-xs: 0.25rem;
$spacing-sm: 0.5rem;
$spacing-md: 1rem;
$spacing-lg: 1.5rem;
$spacing-xl: 2rem;

// Typography
$font-sans:  'Inter', sans-serif;
$font-mono:  'JetBrains Mono', monospace;
$font-size-base: 1rem;
$line-height-base: 1.5;

// Breakpoints
$breakpoint-sm: 640px;
$breakpoint-md: 768px;
$breakpoint-lg: 1024px;
$breakpoint-xl: 1280px;

// Transitions
$transition-fast:   150ms ease;
$transition-normal: 250ms ease;
$transition-slow:   400ms ease;
```

## _mixins.scss
```scss
// Responsive breakpoints
@mixin sm { @media (min-width: 640px) { @content; } }
@mixin md { @media (min-width: 768px) { @content; } }
@mixin lg { @media (min-width: 1024px) { @content; } }
@mixin xl { @media (min-width: 1280px) { @content; } }

// Flexbox helpers
@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin flex-between {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

// Text truncation
@mixin truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

// Visually hidden (accessible)
@mixin visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

## global.scss
```scss
@use './variables' as *;
@use './mixins' as *;

*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  font-family: $font-sans;
  font-size: $font-size-base;
  line-height: $line-height-base;
  color: $color-text;
  background-color: $color-bg;
  -webkit-font-smoothing: antialiased;
}

a {
  color: $color-primary;
  text-decoration: none;
  &:hover { text-decoration: underline; }
}
```

## CSS Modules with SCSS (component-level)
```tsx
// MyComponent.module.scss
.container {
  @include flex-center;
  padding: $spacing-md;
  
  &__title {
    font-size: 1.5rem;
    color: $color-primary;
  }
}
```
```tsx
// MyComponent.tsx
import styles from './MyComponent.module.scss'

export function MyComponent() {
  return (
    <div className={styles.container}>
      <h2 className={styles.container__title}>Hello</h2>
    </div>
  )
}
```

## Notes
- Use `@use` instead of `@import` (deprecated in modern SASS)
- SCSS modules auto-scope class names — great for component isolation
- With Tailwind: use SCSS for complex component styles, Tailwind for utilities
