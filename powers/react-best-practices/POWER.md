---
name: "react-best-practices"
displayName: "React Best Practices"
description: "Comprehensive React and Next.js performance optimization guide from Vercel Engineering. Contains 45+ rules across 8 categories for React performance, plus Web Interface Guidelines for accessibility, forms, animation, and UX best practices."
keywords:
  [
    "react performance",
    "next.js optimization",
    "react best practices",
    "bundle size",
    "waterfall",
    "re-render",
    "server components",
    "rsc",
    "suspense",
    "dynamic import",
    "usememo",
    "usecallback",
    "swr",
    "react cache",
    "hydration",
    "code splitting",
    "accessibility",
    "a11y",
    "web design",
    "ux",
    "forms",
    "animation",
  ]
author: "Salih Güler"
---

# React Best Practices

Comprehensive performance optimization guide for React and Next.js applications, maintained by Vercel Engineering. Contains 45+ rules across 8 categories, prioritized by impact to guide automated refactoring and code generation. Also includes Web Interface Guidelines for accessibility, forms, animation, and UX.

## When to Apply

Reference these guidelines when:

- Writing new React components or Next.js pages
- Implementing data fetching (client or server-side)
- Reviewing code for performance issues
- Refactoring existing React/Next.js code
- Optimizing bundle size or load times
- Reviewing UI for accessibility and UX compliance
- Implementing forms, animations, or interactive elements

## Available Steering Files

This power organizes content into focused steering files by category:

| Steering File                 | Impact      | Description                                            |
| ----------------------------- | ----------- | ------------------------------------------------------ |
| `eliminating-waterfalls.md`   | CRITICAL    | Promise.all, Suspense boundaries, defer await patterns |
| `bundle-optimization.md`      | CRITICAL    | Dynamic imports, barrel files, preload strategies      |
| `server-performance.md`       | HIGH        | React.cache, LRU caching, RSC serialization, after()   |
| `client-data-fetching.md`     | MEDIUM-HIGH | SWR deduplication, event listener patterns             |
| `rerender-optimization.md`    | MEDIUM      | Memo, transitions, functional setState, lazy init      |
| `rendering-performance.md`    | MEDIUM      | SVG animation, content-visibility, hydration           |
| `javascript-performance.md`   | LOW-MEDIUM  | Caching, loops, Set/Map, array methods                 |
| `advanced-patterns.md`        | LOW         | useLatest, event handler refs                          |
| `web-interface-guidelines.md` | HIGH        | Accessibility, forms, animation, typography, UX        |

## Quick Reference by Priority

### 1. Eliminating Waterfalls (CRITICAL)

- `async-defer-await` - Move await into branches where actually used
- `async-parallel` - Use Promise.all() for independent operations
- `async-dependencies` - Use better-all for partial dependencies
- `async-api-routes` - Start promises early, await late in API routes
- `async-suspense-boundaries` - Use Suspense to stream content

### 2. Bundle Size Optimization (CRITICAL)

- `bundle-barrel-imports` - Import directly, avoid barrel files
- `bundle-dynamic-imports` - Use next/dynamic for heavy components
- `bundle-defer-third-party` - Load analytics/logging after hydration
- `bundle-conditional` - Load modules only when feature is activated
- `bundle-preload` - Preload on hover/focus for perceived speed

### 3. Server-Side Performance (HIGH)

- `server-cache-react` - Use React.cache() for per-request deduplication
- `server-cache-lru` - Use LRU cache for cross-request caching
- `server-serialization` - Minimize data passed to client components
- `server-parallel-fetching` - Restructure components to parallelize fetches
- `server-after-nonblocking` - Use after() for non-blocking operations

### 4. Client-Side Data Fetching (MEDIUM-HIGH)

- `client-swr-dedup` - Use SWR for automatic request deduplication
- `client-event-listeners` - Deduplicate global event listeners

### 5. Re-render Optimization (MEDIUM)

- `rerender-defer-reads` - Don't subscribe to state only used in callbacks
- `rerender-memo` - Extract expensive work into memoized components
- `rerender-dependencies` - Use primitive dependencies in effects
- `rerender-derived-state` - Subscribe to derived booleans, not raw values
- `rerender-functional-setstate` - Use functional setState for stable callbacks
- `rerender-lazy-state-init` - Pass function to useState for expensive values
- `rerender-transitions` - Use startTransition for non-urgent updates

### 6. Rendering Performance (MEDIUM)

- `rendering-animate-svg-wrapper` - Animate div wrapper, not SVG element
- `rendering-content-visibility` - Use content-visibility for long lists
- `rendering-hoist-jsx` - Extract static JSX outside components
- `rendering-svg-precision` - Reduce SVG coordinate precision
- `rendering-hydration-no-flicker` - Use inline script for client-only data
- `rendering-activity` - Use Activity component for show/hide
- `rendering-conditional-render` - Use ternary, not && for conditionals

### 7. JavaScript Performance (LOW-MEDIUM)

- `js-batch-dom-css` - Group CSS changes via classes or cssText
- `js-index-maps` - Build Map for repeated lookups
- `js-cache-property-access` - Cache object properties in loops
- `js-cache-function-results` - Cache function results in module-level Map
- `js-cache-storage` - Cache localStorage/sessionStorage reads
- `js-combine-iterations` - Combine multiple filter/map into one loop
- `js-length-check-first` - Check array length before expensive comparison
- `js-early-exit` - Return early from functions
- `js-hoist-regexp` - Hoist RegExp creation outside loops
- `js-min-max-loop` - Use loop for min/max instead of sort
- `js-set-map-lookups` - Use Set/Map for O(1) lookups
- `js-tosorted-immutable` - Use toSorted() for immutability

### 8. Advanced Patterns (LOW)

- `advanced-event-handler-refs` - Store event handlers in refs
- `advanced-use-latest` - useLatest for stable callback refs

### 9. Web Interface Guidelines (HIGH)

- Accessibility - ARIA labels, semantic HTML, keyboard handlers
- Focus States - Visible focus, focus-visible patterns
- Forms - Autocomplete, validation, error handling
- Animation - prefers-reduced-motion, compositor-friendly
- Typography - Curly quotes, ellipsis, tabular-nums
- Performance - Virtualization, layout thrashing, preconnect

## How to Use

Read individual steering files for detailed explanations and code examples. Each file contains:

- Brief explanation of why it matters
- Incorrect code example with explanation
- Correct code example with explanation
- Additional context and references

## References

- [React Documentation](https://react.dev)
- [Next.js Documentation](https://nextjs.org)
- [SWR Documentation](https://swr.vercel.app)
- [better-all Library](https://github.com/shuding/better-all)
- [LRU Cache](https://github.com/isaacs/node-lru-cache)
- [Vercel Blog: Package Import Optimization](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)
- [Vercel Blog: Dashboard Performance](https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast)
