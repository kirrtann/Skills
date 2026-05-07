# Responsive Patterns & Fixes

> Load this file for Mode D (Responsive Audit) or when fixing layout issues.

---

## Core Principle: Mobile-First Always

Write base styles for mobile. Add complexity at larger breakpoints.

```css
/* ❌ Desktop-first (avoid) */
.container { width: 1200px; }
@media (max-width: 768px) { .container { width: 100%; } }

/* ✅ Mobile-first */
.container { width: 100%; }
@media (min-width: 1024px) { .container { max-width: 1200px; margin: 0 auto; } }
```

---

## Standard Breakpoints

Always standardize to these. Flag any deviation.

```css
/* CSS */
--bp-sm:  640px;
--bp-md:  768px;
--bp-lg:  1024px;
--bp-xl:  1280px;
--bp-2xl: 1536px;
```

```js
// Tailwind config (if custom needed)
screens: { sm: '640px', md: '768px', lg: '1024px', xl: '1280px', '2xl': '1536px' }
```

---

## Common Patterns

### Responsive Grid

```css
/* CSS */
.grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}
@media (min-width: 640px)  { .grid { grid-template-columns: repeat(2, 1fr); } }
@media (min-width: 1024px) { .grid { grid-template-columns: repeat(3, 1fr); } }
@media (min-width: 1280px) { .grid { grid-template-columns: repeat(4, 1fr); } }
```

```jsx
/* Tailwind */
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
```

### Responsive Nav

```css
/* CSS */
.nav { display: none; }
.nav-toggle { display: flex; }
@media (min-width: 768px) {
  .nav { display: flex; }
  .nav-toggle { display: none; }
}
```

```jsx
/* Tailwind */
<nav className="hidden md:flex">
<button className="flex md:hidden">
```

### Responsive Typography Scale

```css
/* CSS - fluid type */
h1 { font-size: 1.5rem; }
@media (min-width: 768px)  { h1 { font-size: 2rem; } }
@media (min-width: 1024px) { h1 { font-size: 2.5rem; } }
```

```jsx
/* Tailwind */
<h1 className="text-2xl md:text-3xl lg:text-4xl">
```

### Responsive Stack → Side-by-side

```css
.card { display: flex; flex-direction: column; }
@media (min-width: 768px) { .card { flex-direction: row; } }
```

```jsx
<div className="flex flex-col md:flex-row">
```

### Responsive Spacing

```jsx
/* Tailwind - increase padding at larger screens */
<section className="px-4 sm:px-6 lg:px-8 py-8 md:py-12 lg:py-16">
```

### Responsive Images

```css
/* Always */
img { max-width: 100%; height: auto; }
```

```jsx
<img className="w-full h-auto object-cover">
/* Fixed aspect ratio */
<div className="aspect-video"> <img className="w-full h-full object-cover"> </div>
```

### Responsive Table (scroll on mobile)

```jsx
<div className="overflow-x-auto">
  <table className="min-w-full">
```

### Sidebar Layout

```jsx
/* Stack on mobile, side-by-side on desktop */
<div className="flex flex-col lg:flex-row gap-6">
  <aside className="w-full lg:w-64 shrink-0">
  <main className="flex-1 min-w-0">  {/* min-w-0 prevents overflow */}
```

---

## Anti-Patterns to Flag

### 1. Magic breakpoints
```css
/* ❌ */
@media (max-width: 743px) { }
@media (min-width: 800px) { }

/* ✅ */
@media (max-width: 767px) { }  /* or remove if mobile-first */
@media (min-width: 768px) { }
```

### 2. Redundant overrides
```css
/* ❌ */
.el { color: red; }
@media (min-width: 768px) { .el { color: red; } }  /* same value — delete */
```

### 3. Overlapping media queries
```css
/* ❌ */
@media (min-width: 768px) and (max-width: 1024px) { }
@media (min-width: 900px) and (max-width: 1200px) { }  /* overlaps */

/* ✅ Use non-overlapping ranges or just min-width */
@media (min-width: 768px) { }
@media (min-width: 1024px) { }
```

### 4. px instead of rem for font/spacing
```css
/* ❌ Doesn't respect user font settings */
font-size: 14px;

/* ✅ */
font-size: 0.875rem;
```

### 5. Fixed widths breaking on small screens
```css
/* ❌ */
.card { width: 400px; }

/* ✅ */
.card { width: min(400px, 100%); }
/* or */
.card { max-width: 400px; width: 100%; }
```

### 6. Viewport units on mobile (iOS bug)
```css
/* ❌ 100vh is broken in iOS Safari with address bar */
.hero { height: 100vh; }

/* ✅ */
.hero { height: 100svh; }  /* small viewport height */
/* or fallback */
.hero { height: 100vh; height: 100svh; }
```

---

## Container Pattern

```css
/* tokens */
.container {
  width: 100%;
  padding-inline: 1rem;
  margin-inline: auto;
}
@media (min-width: 640px)  { .container { padding-inline: 1.5rem; } }
@media (min-width: 1024px) { .container { padding-inline: 2rem; } }
@media (min-width: 1280px) { .container { max-width: 1280px; } }
```

```jsx
/* Tailwind */
<div className="w-full px-4 sm:px-6 lg:px-8 mx-auto max-w-7xl">
```

---

## Checklist — Responsive Audit

Run through each component:

- [ ] Base styles are mobile (no `max-width` media queries in new code)
- [ ] All breakpoints use standard values (640/768/1024/1280)
- [ ] No redundant overrides (same value in breakpoint)
- [ ] Font sizes use `rem`, spacing uses `rem` or Tailwind scale
- [ ] Images have `max-width: 100%; height: auto`
- [ ] Tables are scrollable on mobile
- [ ] `100vh` replaced with `100svh` for full-screen layouts
- [ ] Flex/Grid children have `min-w-0` if they contain long text
- [ ] No fixed-width elements that could overflow on small screens
