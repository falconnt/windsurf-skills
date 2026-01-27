---
name: vercel-react-best-practices
description: Use when writing React/Next.js code, reviewing for performance issues, optimizing bundle size, implementing data fetching, or refactoring existing React components
---

# Vercel React Best Practices

## Overview

Comprehensive performance optimization guide for React and Next.js applications. Contains 57 rules across 8 categories, prioritized by impact to guide code generation and refactoring.

**Core principle:** Eliminate waterfalls, minimize bundle size, and optimize rendering - in that priority order.

## When to Use

- Writing new React components or Next.js pages
- Implementing data fetching (client or server-side)
- Reviewing code for performance issues
- Refactoring existing React/Next.js code
- Optimizing bundle size or load times

## Quick Reference by Priority

### 1. Eliminating Waterfalls (CRITICAL)

| Rule | Description |
|------|-------------|
| `async-defer-await` | Move await into branches where actually used |
| `async-parallel` | Use `Promise.all()` for independent operations |
| `async-dependencies` | Use better-all for partial dependencies |
| `async-api-routes` | Start promises early, await late in API routes |
| `async-suspense-boundaries` | Use Suspense to stream content |

**Pattern - Parallel fetching:**
```typescript
// ❌ Sequential (waterfall)
const user = await getUser(id);
const posts = await getPosts(id);

// ✅ Parallel
const [user, posts] = await Promise.all([
  getUser(id),
  getPosts(id)
]);
```

### 2. Bundle Size Optimization (CRITICAL)

| Rule | Description |
|------|-------------|
| `bundle-barrel-imports` | Import directly, avoid barrel files |
| `bundle-dynamic-imports` | Use `next/dynamic` for heavy components |
| `bundle-defer-third-party` | Load analytics/logging after hydration |
| `bundle-conditional` | Load modules only when feature is activated |
| `bundle-preload` | Preload on hover/focus for perceived speed |

**Pattern - Direct imports:**
```typescript
// ❌ Barrel import (pulls entire module)
import { Button } from '@/components';

// ✅ Direct import
import { Button } from '@/components/Button';
```

**Pattern - Dynamic import:**
```typescript
// ❌ Static import of heavy component
import HeavyChart from './HeavyChart';

// ✅ Dynamic import
const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <ChartSkeleton />
});
```

### 3. Server-Side Performance (HIGH)

| Rule | Description |
|------|-------------|
| `server-auth-actions` | Authenticate server actions like API routes |
| `server-cache-react` | Use `React.cache()` for per-request deduplication |
| `server-cache-lru` | Use LRU cache for cross-request caching |
| `server-dedup-props` | Avoid duplicate serialization in RSC props |
| `server-serialization` | Minimize data passed to client components |
| `server-parallel-fetching` | Restructure components to parallelize fetches |
| `server-after-nonblocking` | Use `after()` for non-blocking operations |

**Pattern - React.cache for deduplication:**
```typescript
// ✅ Cached function - called multiple times, executes once per request
const getUser = cache(async (id: string) => {
  return await db.user.findUnique({ where: { id } });
});
```

### 4. Client-Side Data Fetching (MEDIUM-HIGH)

| Rule | Description |
|------|-------------|
| `client-swr-dedup` | Use SWR for automatic request deduplication |
| `client-event-listeners` | Deduplicate global event listeners |
| `client-passive-event-listeners` | Use passive listeners for scroll |
| `client-localstorage-schema` | Version and minimize localStorage data |

### 5. Re-render Optimization (MEDIUM)

| Rule | Description |
|------|-------------|
| `rerender-defer-reads` | Don't subscribe to state only used in callbacks |
| `rerender-memo` | Extract expensive work into memoized components |
| `rerender-memo-with-default-value` | Hoist default non-primitive props |
| `rerender-dependencies` | Use primitive dependencies in effects |
| `rerender-derived-state` | Subscribe to derived booleans, not raw values |
| `rerender-derived-state-no-effect` | Derive state during render, not effects |
| `rerender-functional-setstate` | Use functional setState for stable callbacks |
| `rerender-lazy-state-init` | Pass function to useState for expensive values |
| `rerender-simple-expression-in-memo` | Avoid memo for simple primitives |
| `rerender-move-effect-to-event` | Put interaction logic in event handlers |
| `rerender-transitions` | Use `startTransition` for non-urgent updates |
| `rerender-use-ref-transient-values` | Use refs for transient frequent values |

**Pattern - Functional setState:**
```typescript
// ❌ Creates new callback reference each render
const increment = () => setCount(count + 1);

// ✅ Stable callback, no dependency on count
const increment = useCallback(() => setCount(c => c + 1), []);
```

**Pattern - Lazy state initialization:**
```typescript
// ❌ Expensive computation runs every render
const [data] = useState(expensiveComputation());

// ✅ Only runs once
const [data] = useState(() => expensiveComputation());
```

### 6. Rendering Performance (MEDIUM)

| Rule | Description |
|------|-------------|
| `rendering-animate-svg-wrapper` | Animate div wrapper, not SVG element |
| `rendering-content-visibility` | Use `content-visibility` for long lists |
| `rendering-hoist-jsx` | Extract static JSX outside components |
| `rendering-svg-precision` | Reduce SVG coordinate precision |
| `rendering-hydration-no-flicker` | Use inline script for client-only data |
| `rendering-hydration-suppress-warning` | Suppress expected mismatches |
| `rendering-activity` | Use Activity component for show/hide |
| `rendering-conditional-render` | Use ternary, not `&&` for conditionals |
| `rendering-usetransition-loading` | Prefer useTransition for loading state |

**Pattern - Conditional rendering:**
```typescript
// ❌ Can render 0 or false
{count && <Component />}

// ✅ Always renders nothing or component
{count > 0 ? <Component /> : null}
```

### 7. JavaScript Performance (LOW-MEDIUM)

| Rule | Description |
|------|-------------|
| `js-batch-dom-css` | Group CSS changes via classes or cssText |
| `js-index-maps` | Build Map for repeated lookups |
| `js-cache-property-access` | Cache object properties in loops |
| `js-cache-function-results` | Cache function results in module-level Map |
| `js-cache-storage` | Cache localStorage/sessionStorage reads |
| `js-combine-iterations` | Combine multiple filter/map into one loop |
| `js-length-check-first` | Check array length before expensive comparison |
| `js-early-exit` | Return early from functions |
| `js-hoist-regexp` | Hoist RegExp creation outside loops |
| `js-min-max-loop` | Use loop for min/max instead of sort |
| `js-set-map-lookups` | Use Set/Map for O(1) lookups |
| `js-tosorted-immutable` | Use `toSorted()` for immutability |

**Pattern - Set for lookups:**
```typescript
// ❌ O(n) lookup
const isSelected = selectedIds.includes(id);

// ✅ O(1) lookup
const selectedSet = new Set(selectedIds);
const isSelected = selectedSet.has(id);
```

### 8. Advanced Patterns (LOW)

| Rule | Description |
|------|-------------|
| `advanced-event-handler-refs` | Store event handlers in refs |
| `advanced-init-once` | Initialize app once per app load |
| `advanced-use-latest` | useLatest for stable callback refs |

## Red Flags

Watch for these anti-patterns during code review:

- **Sequential awaits** for independent data fetches
- **Barrel file imports** (`import { x } from '@/components'`)
- **Large static imports** for conditionally-used components
- **State subscriptions** for values only used in callbacks
- **Array.includes()** in loops (use Set instead)
- **useEffect for derived state** (compute during render)
- **Non-primitive effect dependencies** causing infinite loops

## Supporting Files

- **AGENTS.md** - Full compiled document with all 40+ rules expanded with detailed explanations and code examples
- **rules/*.md** - Individual rule files for detailed reference (59 files)

## References

- [Vercel Agent Skills Repository](https://github.com/vercel-labs/agent-skills)
- [React Documentation](https://react.dev)
- [Next.js Documentation](https://nextjs.org)
- [SWR Documentation](https://swr.vercel.app)
