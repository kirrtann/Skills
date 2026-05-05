---
name: font
description: "Add custom fonts to your project with this skill. You'll learn how to set up custom font files, apply them globally, and manage font styles efficiently."

---
## Folder Structure
 
Place all font files under the public assets directory. Each font family gets its own subfolder:
 
```
/public/assets/fonts/
└── <font-name>/
    ├── font.woff2          ← Primary (best compression, modern browsers)
    ├── font.ttf            ← Fallback (older browser support)
    ├── font-bold.woff2     ← Weight variant (optional)
    └── font-bold.ttf       ← Weight variant fallback (optional)
```
## Font Declaration
 
### Option A: CSS (`global.css` / `globals.css`)
 
Declare each weight and style as a separate `@font-face` block. Always include both `woff2` and `truetype` formats for broad compatibility. Use `font-display: swap` to prevent invisible text during load.
 
 
:root {
  --font-custom: 'CustomFont', system-ui, sans-serif;
}

## Write @font-face Declarations
 
Create or update `src/styles/fonts.css` (or `_fonts.scss` for SCSS projects):
 
```css
 
@font-face {
  font-family: '<FontName>';
  src:
    url('/fonts/<filename>.woff2') format('woff2'),
    url('/fonts/<filename>.woff')  format('woff'),
    url('/fonts/<filename>.ttf')   format('truetype');
  font-weight: 400;          
  font-style:  normal;
  font-display: swap;        
}
 
```
## Verify & Deliver
 
1. Run a quick sanity check:
```bash
ls -lh <project-root>/public/fonts/
grep -r "font-family" <project-root>/src/styles/
```
 
2. Produce a checklist summary for the user:
```
✅ Font files copied to public/fonts/
✅ @font-face declared in src/styles/fonts.css
✅ fonts.css imported in project entry
✅ CSS variables set in global.css
✅ Tailwind fontFamily + fontSize extended   ← (if Tailwind)
✅ SCSS variables defined in _variables.scss ← (if SCSS)
```

## Manual Steps