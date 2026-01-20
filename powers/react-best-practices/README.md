# React Best Practices Power

A Kiro Power providing comprehensive React and Next.js performance optimization guidelines and Web Interface Guidelines for accessibility and UX from Vercel Engineering.

> This is not an official Kiro Power by the Kiro team or Vercel team.

## Installation

### Option 1: Local Installation

Copy the `react-best-practices` folder to your Kiro powers directory:

```bash
cp -r react-best-practices ~/.kiro/powers/
```

### Option 2: From Repository

Clone this repository and copy the power:

```bash
git clone <repository-url>
cp -r react-best-practices ~/.kiro/powers/
```

## What's Included

### React Performance (45+ Rules)

| Category               | Impact      | Rules                                  |
| ---------------------- | ----------- | -------------------------------------- |
| Eliminating Waterfalls | CRITICAL    | Promise.all, Suspense, defer await     |
| Bundle Optimization    | CRITICAL    | Dynamic imports, barrel files, preload |
| Server Performance     | HIGH        | React.cache, LRU, RSC serialization    |
| Client Data Fetching   | MEDIUM-HIGH | SWR, event listener dedup              |
| Re-render Optimization | MEDIUM      | Memo, transitions, functional setState |
| Rendering Performance  | MEDIUM      | SVG, content-visibility, hydration     |
| JavaScript Performance | LOW-MEDIUM  | Caching, loops, Set/Map                |
| Advanced Patterns      | LOW         | useLatest, event handler refs          |

### Web Interface Guidelines

- Accessibility (ARIA, semantic HTML, keyboard)
- Focus States (visible focus, focus-visible)
- Forms (autocomplete, validation, errors)
- Animation (prefers-reduced-motion, compositor)
- Typography (curly quotes, ellipsis, tabular-nums)
- Performance (virtualization, preconnect)
- Navigation & State (URL sync, deep-linking)
- Dark Mode & Theming

## Usage

Once installed, the power activates when you discuss:

- React performance optimization
- Next.js best practices
- Bundle size reduction
- Re-render issues
- Server components
- Accessibility reviews
- UI/UX compliance

### Example Prompts

```
Review this React component for performance issues
```

```
How can I optimize this Next.js page?
```

```
Check this form for accessibility
```

```
Help me reduce bundle size
```

## Keywords

`react performance`, `next.js optimization`, `react best practices`, `bundle size`, `waterfall`, `re-render`, `server components`, `rsc`, `suspense`, `dynamic import`, `usememo`, `usecallback`, `swr`, `react cache`, `hydration`, `code splitting`, `accessibility`, `a11y`, `web design`, `ux`, `forms`, `animation`

## Credits

- React Best Practices: Originally created by [Vercel](https://vercel.com) can be found at [GitHub](https://github.com/vercel-labs/agent-skills)
- Power Author: Salih Güler

## License

MIT
