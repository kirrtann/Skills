# Tailwind CSS Property → Class Mapping

> Load this file when doing Mode A (CSS → Tailwind) for complex conversions.

## Layout

| CSS | Tailwind |
|-----|----------|
| `display: flex` | `flex` |
| `display: grid` | `grid` |
| `display: block` | `block` |
| `display: inline-block` | `inline-block` |
| `display: none` | `hidden` |
| `flex-direction: row` | `flex-row` |
| `flex-direction: column` | `flex-col` |
| `flex-wrap: wrap` | `flex-wrap` |
| `align-items: center` | `items-center` |
| `align-items: flex-start` | `items-start` |
| `align-items: flex-end` | `items-end` |
| `align-items: stretch` | `items-stretch` |
| `justify-content: center` | `justify-center` |
| `justify-content: flex-start` | `justify-start` |
| `justify-content: flex-end` | `justify-end` |
| `justify-content: space-between` | `justify-between` |
| `justify-content: space-around` | `justify-around` |
| `gap: 4px` | `gap-1` |
| `gap: 8px` | `gap-2` |
| `gap: 12px` | `gap-3` |
| `gap: 16px` | `gap-4` |
| `gap: 24px` | `gap-6` |
| `gap: 32px` | `gap-8` |
| `grid-template-columns: repeat(2, 1fr)` | `grid-cols-2` |
| `grid-template-columns: repeat(3, 1fr)` | `grid-cols-3` |
| `grid-template-columns: repeat(4, 1fr)` | `grid-cols-4` |
| `position: relative` | `relative` |
| `position: absolute` | `absolute` |
| `position: fixed` | `fixed` |
| `position: sticky` | `sticky` |
| `top: 0` | `top-0` |
| `right: 0` | `right-0` |
| `bottom: 0` | `bottom-0` |
| `left: 0` | `left-0` |
| `z-index: 10` | `z-10` |
| `z-index: 20` | `z-20` |
| `z-index: 50` | `z-50` |
| `overflow: hidden` | `overflow-hidden` |
| `overflow: auto` | `overflow-auto` |
| `overflow: scroll` | `overflow-scroll` |

## Spacing

| CSS | Tailwind |
|-----|----------|
| `padding: 4px` | `p-1` |
| `padding: 8px` | `p-2` |
| `padding: 12px` | `p-3` |
| `padding: 16px` | `p-4` |
| `padding: 20px` | `p-5` |
| `padding: 24px` | `p-6` |
| `padding: 32px` | `p-8` |
| `padding: 40px` | `p-10` |
| `padding: 48px` | `p-12` |
| `padding-top/bottom: Xpx` | `py-*` |
| `padding-left/right: Xpx` | `px-*` |
| `margin: auto` | `m-auto` |
| `margin: 0 auto` | `mx-auto` |
| `margin-top: 4px` | `mt-1` |
| `margin-bottom: 16px` | `mb-4` |
| `margin-left: 8px` | `ml-2` |
| Custom value | `p-[13px]` (JIT arbitrary) |

## Sizing

| CSS | Tailwind |
|-----|----------|
| `width: 100%` | `w-full` |
| `width: 100vw` | `w-screen` |
| `min-width: 0` | `min-w-0` |
| `max-width: 640px` | `max-w-sm` |
| `max-width: 768px` | `max-w-md` |
| `max-width: 1024px` | `max-w-4xl` |
| `max-width: 1280px` | `max-w-5xl` |
| `height: 100%` | `h-full` |
| `height: 100vh` | `h-screen` |
| `height: 48px` | `h-12` |
| `width: 48px` | `w-12` |
| Custom size | `w-[200px]` |

## Typography

| CSS | Tailwind |
|-----|----------|
| `font-size: 12px` | `text-xs` |
| `font-size: 14px` | `text-sm` |
| `font-size: 16px` | `text-base` |
| `font-size: 18px` | `text-lg` |
| `font-size: 20px` | `text-xl` |
| `font-size: 24px` | `text-2xl` |
| `font-size: 30px` | `text-3xl` |
| `font-size: 36px` | `text-4xl` |
| `font-weight: 400` | `font-normal` |
| `font-weight: 500` | `font-medium` |
| `font-weight: 600` | `font-semibold` |
| `font-weight: 700` | `font-bold` |
| `line-height: 1` | `leading-none` |
| `line-height: 1.25` | `leading-tight` |
| `line-height: 1.5` | `leading-normal` |
| `line-height: 2` | `leading-loose` |
| `text-align: center` | `text-center` |
| `text-align: left` | `text-left` |
| `text-align: right` | `text-right` |
| `text-transform: uppercase` | `uppercase` |
| `text-transform: lowercase` | `lowercase` |
| `text-transform: capitalize` | `capitalize` |
| `letter-spacing: 0.05em` | `tracking-wide` |
| `text-decoration: underline` | `underline` |
| `white-space: nowrap` | `whitespace-nowrap` |
| `word-break: break-all` | `break-all` |
| `overflow: hidden; text-overflow: ellipsis; white-space: nowrap` | `truncate` |

## Colors (map to theme or arbitrary)

| CSS | Tailwind |
|-----|----------|
| `color: #ffffff` | `text-white` |
| `color: #000000` | `text-black` |
| `background: #ffffff` | `bg-white` |
| `background: #000000` | `bg-black` |
| `background: transparent` | `bg-transparent` |
| `opacity: 0` | `opacity-0` |
| `opacity: 0.5` | `opacity-50` |
| `opacity: 1` | `opacity-100` |
| Custom color | `text-[#3d4f6a]` or add to theme |

## Borders

| CSS | Tailwind |
|-----|----------|
| `border: 1px solid` | `border` |
| `border: 2px solid` | `border-2` |
| `border: none` | `border-0` |
| `border-color: #e5e7eb` | `border-gray-200` |
| `border-radius: 4px` | `rounded` |
| `border-radius: 6px` | `rounded-md` |
| `border-radius: 8px` | `rounded-lg` |
| `border-radius: 12px` | `rounded-xl` |
| `border-radius: 9999px` | `rounded-full` |
| `border-top: 1px solid` | `border-t` |
| `outline: none` | `outline-none` |

## Effects

| CSS | Tailwind |
|-----|----------|
| `box-shadow: sm` | `shadow-sm` |
| `box-shadow: md` | `shadow-md` |
| `box-shadow: lg` | `shadow-lg` |
| `box-shadow: none` | `shadow-none` |
| `transition: all 150ms ease` | `transition` |
| `transition: colors 150ms` | `transition-colors` |
| `transition: transform 150ms` | `transition-transform` |
| `cursor: pointer` | `cursor-pointer` |
| `cursor: not-allowed` | `cursor-not-allowed` |
| `pointer-events: none` | `pointer-events-none` |
| `user-select: none` | `select-none` |
| `resize: none` | `resize-none` |

## Responsive Prefixes

```
sm:   min-width: 640px
md:   min-width: 768px
lg:   min-width: 1024px
xl:   min-width: 1280px
2xl:  min-width: 1536px
```

Usage: `class="text-sm md:text-base lg:text-lg"`

## State Prefixes

```
hover:    :hover
focus:    :focus
active:   :active
disabled: :disabled
focus-within: :focus-within
group-hover:  parent has .group, child uses group-hover:
```

## Dark Mode

```
dark:bg-gray-900   → applies when .dark class on <html> or prefers-color-scheme: dark
```
