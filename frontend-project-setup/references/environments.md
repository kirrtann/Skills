# Environment Configuration Reference

## Files

| File | Committed? | Purpose |
|---|---|---|
| `.env.local` | ❌ NEVER | Developer's local secrets and overrides |
| `.env.stage` | ✅ Yes | Staging config (no real secrets) |
| `.env.production` | ✅ Yes | Production config (no real secrets) |
| `.env.example` | ✅ Yes | Template — all keys, no values |

## .env.example (always commit this)

```env
# App
NEXT_PUBLIC_APP_NAME=MyApp
NEXT_PUBLIC_APP_ENV=local

# API
NEXT_PUBLIC_API_URL=http://localhost:4000/api

# Auth
NEXT_PUBLIC_AUTH_DOMAIN=
API_SECRET_KEY=           # Never expose — server-side only

# Monitoring
NEXT_PUBLIC_SENTRY_DSN=

# Feature Flags
NEXT_PUBLIC_FEATURE_FLAG_DARK_MODE=false
```

## .env.local (developer creates from .env.example)

```env
NEXT_PUBLIC_APP_NAME=MyApp
NEXT_PUBLIC_APP_ENV=local
NEXT_PUBLIC_API_URL=http://localhost:4000/api
API_SECRET_KEY=dev-secret-key-12345
NEXT_PUBLIC_SENTRY_DSN=
NEXT_PUBLIC_FEATURE_FLAG_DARK_MODE=true
```

## .env.stage

```env
NEXT_PUBLIC_APP_NAME=MyApp
NEXT_PUBLIC_APP_ENV=stage
NEXT_PUBLIC_API_URL=https://api.staging.myapp.com/api
NEXT_PUBLIC_SENTRY_DSN=https://xxx@sentry.io/xxx
NEXT_PUBLIC_FEATURE_FLAG_DARK_MODE=true
```

## .env.production

```env
NEXT_PUBLIC_APP_NAME=MyApp
NEXT_PUBLIC_APP_ENV=production
NEXT_PUBLIC_API_URL=https://api.myapp.com/api
NEXT_PUBLIC_SENTRY_DSN=https://xxx@sentry.io/xxx
NEXT_PUBLIC_FEATURE_FLAG_DARK_MODE=false
```

## package.json scripts using dotenv-cli

```bash
npm install --save-dev dotenv-cli
```

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",

    "start:local":  "dotenv -e .env.local -- next dev",
    "build:stage":  "dotenv -e .env.stage -- next build",
    "start:stage":  "dotenv -e .env.stage -- next start",
    "build:prod":   "dotenv -e .env.production -- next build",
    "start:prod":   "dotenv -e .env.production -- next start"
  }
}
```

For Vite projects:
```json
{
  "scripts": {
    "dev":          "vite",
    "start:local":  "dotenv -e .env.local -- vite",
    "build:stage":  "dotenv -e .env.stage -- vite build",
    "build:prod":   "dotenv -e .env.production -- vite build",
    "preview:stage":"dotenv -e .env.stage -- vite preview",
    "preview:prod": "dotenv -e .env.production -- vite preview"
  }
}
```

## Accessing env vars in code

```typescript
// src/lib/config.ts
export const config = {
  appName: process.env.NEXT_PUBLIC_APP_NAME ?? 'MyApp',
  appEnv: (process.env.NEXT_PUBLIC_APP_ENV ?? 'local') as 'local' | 'stage' | 'production',
  apiUrl: process.env.NEXT_PUBLIC_API_URL ?? 'http://localhost:4000/api',
  sentryDsn: process.env.NEXT_PUBLIC_SENTRY_DSN ?? '',
  isLocal: process.env.NEXT_PUBLIC_APP_ENV === 'local',
  isStage: process.env.NEXT_PUBLIC_APP_ENV === 'stage',
  isProd: process.env.NEXT_PUBLIC_APP_ENV === 'production',
} as const;
```

## Vite env (import.meta.env)

In Vite projects, use `import.meta.env.VITE_*` (not `process.env`):
```typescript
export const config = {
  apiUrl: import.meta.env.VITE_API_URL ?? 'http://localhost:4000/api',
  appEnv: import.meta.env.VITE_APP_ENV ?? 'local',
};
```

## .gitignore entries

```
.env.local
*.local
.env.*.local
```
