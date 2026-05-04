# Zustand Reference

## Install
```bash
npm install zustand
```

## Basic Typed Store (src/store/useAppStore.ts)
```ts
import { create } from 'zustand'
import { devtools, persist } from 'zustand/middleware'

interface AppState {
  // Example: counter
  count: number
  increment: () => void
  decrement: () => void
  reset: () => void

  // Example: user session
  user: { name: string; email: string } | null
  setUser: (user: AppState['user']) => void
  clearUser: () => void
}

export const useAppStore = create<AppState>()(
  devtools(
    persist(
      (set) => ({
        // Counter
        count: 0,
        increment: () => set((s) => ({ count: s.count + 1 }), false, 'increment'),
        decrement: () => set((s) => ({ count: s.count - 1 }), false, 'decrement'),
        reset: () => set({ count: 0 }, false, 'reset'),

        // User
        user: null,
        setUser: (user) => set({ user }, false, 'setUser'),
        clearUser: () => set({ user: null }, false, 'clearUser'),
      }),
      { name: 'app-storage' } // localStorage key for persist
    ),
    { name: 'AppStore' } // DevTools label
  )
)
```

## Feature-Scoped Store (src/features/cart/store.ts)
```ts
import { create } from 'zustand'

interface CartItem {
  id: string
  name: string
  qty: number
  price: number
}

interface CartState {
  items: CartItem[]
  addItem: (item: Omit<CartItem, 'qty'>) => void
  removeItem: (id: string) => void
  total: () => number
}

export const useCartStore = create<CartState>()((set, get) => ({
  items: [],
  addItem: (item) =>
    set((s) => {
      const existing = s.items.find((i) => i.id === item.id)
      if (existing) {
        return { items: s.items.map((i) => i.id === item.id ? { ...i, qty: i.qty + 1 } : i) }
      }
      return { items: [...s.items, { ...item, qty: 1 }] }
    }),
  removeItem: (id) => set((s) => ({ items: s.items.filter((i) => i.id !== id) })),
  total: () => get().items.reduce((sum, i) => sum + i.price * i.qty, 0),
}))
```

## Slices Pattern (for large stores)
```ts
// src/store/slices/uiSlice.ts
import type { StateCreator } from 'zustand'
import type { RootStore } from '../index'

export interface UISlice {
  sidebarOpen: boolean
  toggleSidebar: () => void
}

export const createUISlice: StateCreator<RootStore, [], [], UISlice> = (set) => ({
  sidebarOpen: false,
  toggleSidebar: () => set((s) => ({ sidebarOpen: !s.sidebarOpen })),
})
```

## Usage in Component
```tsx
import { useAppStore } from '@/store/useAppStore'

export function Counter() {
  const { count, increment, decrement } = useAppStore()
  return (
    <div>
      <button onClick={decrement}>-</button>
      <span>{count}</span>
      <button onClick={increment}>+</button>
    </div>
  )
}
```

## Selector optimization (prevent re-renders)
```tsx
// Only re-renders when `count` changes, not the whole store
const count = useAppStore((s) => s.count)
```

## Notes
- Always use selectors in components for performance
- `devtools` middleware integrates with Redux DevTools Extension
- `persist` saves to localStorage; pass `storage: sessionStorage` for session-only
- Zustand works outside React too (call `useAppStore.getState()` / `setState()`)
