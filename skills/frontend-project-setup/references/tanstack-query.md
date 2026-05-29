# TanStack Query (React Query v5) Reference

## Install

```bash
npm install @tanstack/react-query @tanstack/react-query-devtools
```

## QueryClient setup (src/lib/queryClient.ts)

```typescript
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,    // 5 minutes
      gcTime: 1000 * 60 * 10,       // 10 minutes (was cacheTime in v4)
      retry: 1,
      refetchOnWindowFocus: false,
      refetchOnReconnect: true,
    },
    mutations: {
      retry: 0,
    },
  },
});
```

## Provider setup

### Next.js App Router (src/app/providers.tsx)
```tsx
'use client';
import { QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { useState } from 'react';
import { QueryClient } from '@tanstack/react-query';

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient({
    defaultOptions: {
      queries: { staleTime: 60 * 1000, retry: 1, refetchOnWindowFocus: false },
    },
  }));

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

### Vite / Pages Router (main.tsx or _app.tsx)
```tsx
import { QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { queryClient } from '@/lib/queryClient';

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Router />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

## Query Key Conventions

Always use factory functions for query keys:
```typescript
export const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters?: QueryParams) => [...userKeys.lists(), filters] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: number) => [...userKeys.details(), id] as const,
};
```

## Infinite Queries (pagination)

```typescript
export const useInfiniteUsers = (pageSize = 20) => {
  return useInfiniteQuery({
    queryKey: userKeys.list({ pageSize }),
    queryFn: ({ pageParam = 1 }) => userService.getAll({ page: pageParam, pageSize }),
    getNextPageParam: (lastPage) =>
      lastPage.pagination.page < lastPage.pagination.totalPages
        ? lastPage.pagination.page + 1
        : undefined,
    initialPageParam: 1,
  });
};
```

## Optimistic Updates

```typescript
export const useUpdateUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, payload }: { id: number; payload: UpdateUserPayload }) =>
      userService.update(id, payload),
    onMutate: async ({ id, payload }) => {
      await queryClient.cancelQueries({ queryKey: userKeys.detail(id) });
      const previous = queryClient.getQueryData(userKeys.detail(id));
      queryClient.setQueryData(userKeys.detail(id), (old: User) => ({ ...old, ...payload }));
      return { previous };
    },
    onError: (_err, { id }, context) => {
      queryClient.setQueryData(userKeys.detail(id), context?.previous);
    },
    onSettled: (_data, _err, { id }) => {
      queryClient.invalidateQueries({ queryKey: userKeys.detail(id) });
    },
  });
};
```
