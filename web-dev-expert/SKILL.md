---
name: web-dev-expert
description: Framework-agnostic expert frontend craft — semantic HTML, accessibility, Core Web Vitals performance, CSS architecture, state management principles, TypeScript discipline, and testing. Use for any web UI work regardless of framework. Pairs with nextjs-expert.
---

# Web Dev Expert (Frontend Craft)

Framework-agnostic principles a senior frontend engineer applies on every project. For Next.js-specific guidance use [nextjs-expert]; this skill is the layer beneath it.

## Operating Principles

1. **The platform is powerful — reach for it first.** Native `<dialog>`, `<details>`, form validation, `Intl`, `URL`, view transitions, CSS grid/container queries often beat a dependency.
2. **Ship less JavaScript.** Every KB of JS is parse + execute on the user's slowest device. Prefer HTML/CSS solutions; lazy-load the rest.
3. **Accessibility is correctness, not a feature.** If it doesn't work with a keyboard and a screen reader, it's broken.
4. **Measure before optimizing.** Use the Performance panel / Lighthouse / real-device testing. Don't guess hot paths.

## Semantic HTML & Accessibility (non-negotiable)

- Use the right element: `<button>` for actions, `<a>` for navigation, `<nav>/<main>/<header>/<footer>` landmarks, real headings in order (one `<h1>`).
- Every interactive element is keyboard-reachable and has a visible focus ring (`:focus-visible`). Never `outline: none` without a replacement.
- Label every input (`<label for>` or `aria-label`). Associate errors with `aria-describedby`.
- ARIA is a last resort — a correct native element needs none. First rule of ARIA: don't use ARIA if HTML can do it.
- Respect `prefers-reduced-motion`; gate non-essential animation behind it.
- Color contrast ≥ 4.5:1 for body text. Don't encode meaning in color alone.
- Manage focus on route change / modal open / dynamic content insertion.

## Core Web Vitals (the performance contract)

- **LCP < 2.5s** — prioritize the hero image/text. Preload the LCP image, use `priority`/`fetchpriority="high"`, avoid lazy-loading above the fold.
- **INP < 200ms** — keep main-thread tasks small; break up long tasks, debounce, move heavy work off the critical path. INP replaced FID.
- **CLS < 0.1** — always set `width`/`height` (or aspect-ratio) on media; reserve space for dynamic content; never inject content above existing content.
- Images: serve modern formats (AVIF/WebP), responsive `srcset`/`sizes`, lazy-load below the fold.
- Fonts: `font-display: swap`, preload, subset; avoid layout shift from font swap with `size-adjust`.
- Code-split by route and by interaction; defer non-critical third-party scripts.

## CSS Architecture

- **Modern layout primitives:** flexbox + grid for layout, `gap` over margins, container queries (`@container`) over global media queries for component-level responsiveness, logical properties (`margin-inline`) for i18n.
- **Design tokens as CSS variables** — single source for color/spacing/typography. Theme (dark mode) by swapping variable values, not duplicating rules.
- Keep specificity flat. Avoid `!important` and deep descendant selectors. Co-locate styles with components (CSS Modules / utility classes / scoped).
- Fluid sizing with `clamp()`; avoid a dozen breakpoints. Use `min()/max()` for responsive constraints.

## State Management Principles

- **Categorize state before choosing a tool:**
  - *Server state* (data from an API) → a query/cache library or framework data layer. Don't hand-roll caching.
  - *URL state* (filters, tabs, pagination) → the URL (searchParams). Shareable, back-button-friendly.
  - *Local UI state* → component state.
  - *Global client state* (theme, auth UI) → lightweight store (Zustand/Context), kept small.
- Don't put server data into global client state and sync manually — that's the #1 source of stale-UI bugs.
- Derive, don't duplicate. Computed values come from a single source at render time.

## TypeScript Discipline

- `strict: true`. No `any` — use `unknown` and narrow at the boundary (API responses, `JSON.parse`, form data).
- Validate external data at runtime (schema) where it enters the app; types alone don't protect against a wrong API shape.
- Prefer discriminated unions over optional-flag soup. Make illegal states unrepresentable.
- Derive types from one source (`as const`, schema inference) instead of restating shapes.

## Testing Strategy

- Test behavior, not implementation. Query by role/label (Testing Library) — if the test can't find it by role, neither can a screen reader.
- Pyramid: many fast unit tests, fewer integration, a handful of E2E (Playwright) for critical journeys.
- Cover the bug with a failing test before fixing it.

## Anti-Patterns to Reject

- `<div onClick>` instead of `<button>`; `<div>` soup with no landmarks.
- Animating layout-affecting properties (`width`, `top`) instead of `transform`/`opacity`.
- Loading a date/util library for one function the platform already provides.
- Global state for data that belongs in the URL or server cache.
- Pixel-perfect fixed breakpoints instead of fluid/container-based responsiveness.
- Disabling focus outlines for aesthetics.

## Before Saying "Done"

- [ ] Works with keyboard only; focus is visible and managed.
- [ ] No layout shift; media has dimensions.
- [ ] No unnecessary JS shipped; heavy/below-fold work is deferred.
- [ ] External data is typed and validated at the boundary.
- [ ] Responsive without a breakpoint pile (container queries / clamp).
