---
name: audit-perf
description: Run a performance audit on specified pages or the whole app. Analyzes Core Web Vitals targets, bundle size, and Next.js optimization opportunities.
---

Run a performance audit for: $ARGUMENTS

@perf Audit the specified pages (or all main routes if none given) for:
1. Core Web Vitals: LCP, CLS, INP against targets (LCP <2.5s, CLS <0.1, INP <200ms)
2. Bundle size: check first-load JS target of <200kB
3. Image optimization: Next.js Image usage, explicit dimensions, priority attributes
4. Font loading: display swap, preload
5. Code splitting opportunities: heavy components that should use dynamic imports
6. React render performance: unnecessary re-renders, missing memo/useMemo

Produce a prioritized fix list with: severity, affected metric, exact fix with code example, projected impact.
