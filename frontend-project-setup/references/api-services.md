# API Service Layer Reference

## Architecture

```
src/
├── lib/
│   └── api/
│       ├── client.ts        # Axios instance + interceptors
│       ├── endpoints.ts     # All URL constants
│       └── types.ts         # Shared API types (ApiResponse, PaginatedResponse, etc.)
├── services/
│   ├── userService.ts       # Domain service (pure async functions)
│   ├── authService.ts
│   └── index.ts
└── hooks/
    └── api/
        ├── useUsers.ts      # TanStack Query hooks wrapping services
        ├── useAuth.ts
        └── index.ts
```

## src/lib/api/client.ts

```typescript
import axios, { AxiosError, AxiosInstance, InternalAxiosRequestConfig } from 'axios';

const createApiClient = (): AxiosInstance => {
  const client = axios.create({
    baseURL: process.env.NEXT_PUBLIC_API_URL,
    timeout: 10000,
    headers: { 'Content-Type': 'application/json' },
  });

  // Request interceptor
  client.interceptors.request.use(
    (config: InternalAxiosRequestConfig) => {
      const token = typeof window !== 'undefined' ? localStorage.getItem('auth_token') : null;
      if (token) {
        config.headers.Authorization = `Bearer ${token}`;
      }
      return config;
    },
    (error) => Promise.reject(error)
  );

  // Response interceptor
  client.interceptors.response.use(
    (response) => response,
    (error: AxiosError) => {
      if (error.response?.status === 401) {
        // Clear token and redirect to login
        if (typeof window !== 'undefined') {
          localStorage.removeItem('auth_token');
          window.location.href = '/login';
        }
      }
      if (error.response?.status === 403) {
        console.error('Forbidden: insufficient permissions');
      }
      return Promise.reject(error);
    }
  );

  return client;
};

export const apiClient = createApiClient();
export default apiClient;
```

## src/lib/api/types.ts

```typescript
export interface ApiResponse<T> {
  data: T;
  message: string;
  success: boolean;
}

export interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    pageSize: number;
    total: number;
    totalPages: number;
  };
}

export interface ApiError {
  message: string;
  code: string;
  status: number;
  details?: Record<string, string[]>;
}

export interface QueryParams {
  page?: number;
  pageSize?: number;
  search?: string;
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
}
```

## src/lib/api/endpoints.ts

```typescript
export const ENDPOINTS = {
  AUTH: {
    LOGIN: '/auth/login',
    LOGOUT: '/auth/logout',
    REFRESH: '/auth/refresh',
    ME: '/auth/me',
  },
  USERS: {
    BASE: '/users',
    BY_ID: (id: string | number) => `/users/${id}`,
    PROFILE: '/users/profile',
  },
} as const;
```

## src/services/userService.ts

```typescript
import apiClient from '@/lib/api/client';
import { ENDPOINTS } from '@/lib/api/endpoints';
import type { ApiResponse, PaginatedResponse, QueryParams } from '@/lib/api/types';

export interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user';
  createdAt: string;
}

export interface CreateUserPayload {
  name: string;
  email: string;
  password: string;
  role?: 'admin' | 'user';
}

export interface UpdateUserPayload {
  name?: string;
  email?: string;
  role?: 'admin' | 'user';
}

export const userService = {
  getAll: async (params?: QueryParams): Promise<PaginatedResponse<User>> => {
    const { data } = await apiClient.get<PaginatedResponse<User>>(ENDPOINTS.USERS.BASE, { params });
    return data;
  },

  getById: async (id: number): Promise<User> => {
    const { data } = await apiClient.get<ApiResponse<User>>(ENDPOINTS.USERS.BY_ID(id));
    return data.data;
  },

  create: async (payload: CreateUserPayload): Promise<User> => {
    const { data } = await apiClient.post<ApiResponse<User>>(ENDPOINTS.USERS.BASE, payload);
    return data.data;
  },

  update: async (id: number, payload: UpdateUserPayload): Promise<User> => {
    const { data } = await apiClient.patch<ApiResponse<User>>(ENDPOINTS.USERS.BY_ID(id), payload);
    return data.data;
  },

  delete: async (id: number): Promise<void> => {
    await apiClient.delete(ENDPOINTS.USERS.BY_ID(id));
  },
};
```

## src/hooks/api/useUsers.ts (TanStack Query)

```typescript
import { useMutation, useQuery, useQueryClient } from '@tanstack/react-query';
import { userService, type CreateUserPayload, type UpdateUserPayload } from '@/services/userService';
import type { QueryParams } from '@/lib/api/types';

export const USER_QUERY_KEYS = {
  all: ['users'] as const,
  list: (params?: QueryParams) => ['users', 'list', params] as const,
  detail: (id: number) => ['users', 'detail', id] as const,
};

export const useUsers = (params?: QueryParams) => {
  return useQuery({
    queryKey: USER_QUERY_KEYS.list(params),
    queryFn: () => userService.getAll(params),
  });
};

export const useUser = (id: number) => {
  return useQuery({
    queryKey: USER_QUERY_KEYS.detail(id),
    queryFn: () => userService.getById(id),
    enabled: !!id,
  });
};

export const useCreateUser = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (payload: CreateUserPayload) => userService.create(payload),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: USER_QUERY_KEYS.all });
    },
  });
};

export const useUpdateUser = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: ({ id, payload }: { id: number; payload: UpdateUserPayload }) =>
      userService.update(id, payload),
    onSuccess: (_, { id }) => {
      queryClient.invalidateQueries({ queryKey: USER_QUERY_KEYS.detail(id) });
      queryClient.invalidateQueries({ queryKey: USER_QUERY_KEYS.all });
    },
  });
};

export const useDeleteUser = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (id: number) => userService.delete(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: USER_QUERY_KEYS.all });
    },
  });
};
```
