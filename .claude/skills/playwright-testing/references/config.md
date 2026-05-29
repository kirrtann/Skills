# playwright.config.ts Template

## Standard config (React/Next/Vite)

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',

  use: {
    baseURL: 'http://localhost:3000', // ← change to match dev server port
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },

  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    // Mobile (add if needed)
    // { name: 'mobile-chrome', use: { ...devices['Pixel 5'] } },
  ],

  // Auto-start dev server before tests
  webServer: {
    command: 'npm run dev', // ← or 'npm start', 'npm run preview'
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120_000,
  },
});
```

## Port by stack

| Stack            | Port | Command       |
| ---------------- | ---- | ------------- |
| Next.js          | 3000 | `npm run dev` |
| Vite (React)     | 5173 | `npm run dev` |
| Create React App | 3000 | `npm start`   |
| Angular          | 4200 | `ng serve`    |
| Nuxt             | 3000 | `npm run dev` |
