# Redux Toolkit Reference

## Install
```bash
npm install @reduxjs/toolkit react-redux
```

## Store (src/store/index.ts)
```ts
import { configureStore } from '@reduxjs/toolkit'
import counterReducer from './slices/counterSlice'
import userReducer from './slices/userSlice'

export const store = configureStore({
  reducer: {
    counter: counterReducer,
    user: userReducer,
  },
  devTools: process.env.NODE_ENV !== 'production',
})

// Types
export type RootState = ReturnType<typeof store.getState>
export type AppDispatch = typeof store.dispatch
```

## Typed Hooks (src/store/hooks.ts)
```ts
import { useDispatch, useSelector } from 'react-redux'
import type { RootState, AppDispatch } from '.'

// Use these instead of plain useDispatch/useSelector
export const useAppDispatch = useDispatch.withTypes<AppDispatch>()
export const useAppSelector = useSelector.withTypes<RootState>()
```

## Sample Slice (src/store/slices/counterSlice.ts)
```ts
import { createSlice, type PayloadAction } from '@reduxjs/toolkit'

interface CounterState {
  value: number
  status: 'idle' | 'loading' | 'failed'
}

const initialState: CounterState = { value: 0, status: 'idle' }

export const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increment: (state) => { state.value += 1 },
    decrement: (state) => { state.value -= 1 },
    incrementByAmount: (state, action: PayloadAction<number>) => {
      state.value += action.payload
    },
    reset: (state) => { state.value = 0 },
  },
})

export const { increment, decrement, incrementByAmount, reset } = counterSlice.actions
export default counterSlice.reducer
```

## Async Thunk (RTK Query preferred — but for custom async logic)
```ts
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit'

export const fetchUser = createAsyncThunk('user/fetch', async (id: string) => {
  const res = await fetch(`/api/users/${id}`)
  return res.json()
})

const userSlice = createSlice({
  name: 'user',
  initialState: { data: null, loading: false, error: null } as {
    data: any; loading: boolean; error: string | null
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => { state.loading = true })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false
        state.data = action.payload
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false
        state.error = action.error.message ?? 'Unknown error'
      })
  },
})
```

## RTK Query (preferred for API calls)
```ts
// src/store/api/baseApi.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

export const baseApi = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['User', 'Post'],
  endpoints: (builder) => ({
    getUser: builder.query<User, string>({
      query: (id) => `users/${id}`,
      providesTags: ['User'],
    }),
    createPost: builder.mutation<Post, Partial<Post>>({
      query: (body) => ({ url: 'posts', method: 'POST', body }),
      invalidatesTags: ['Post'],
    }),
  }),
})

export const { useGetUserQuery, useCreatePostMutation } = baseApi
```

## Provider Wiring
```tsx
// src/main.tsx or src/app/layout.tsx (Next.js requires 'use client' wrapper)
import { Provider } from 'react-redux'
import { store } from '@/store'

export default function RootLayout({ children }) {
  return <Provider store={store}>{children}</Provider>
}
```

## Usage in Component
```tsx
import { useAppDispatch, useAppSelector } from '@/store/hooks'
import { increment, decrement } from '@/store/slices/counterSlice'

export function Counter() {
  const count = useAppSelector((s) => s.counter.value)
  const dispatch = useAppDispatch()
  return (
    <div>
      <button onClick={() => dispatch(decrement())}>-</button>
      <span>{count}</span>
      <button onClick={() => dispatch(increment())}>+</button>
    </div>
  )
}
```
