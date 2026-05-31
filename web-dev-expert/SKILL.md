---
name: web-dev-expert
description: Framework-agnostic expert frontend craft mindset — semantic HTML, accessibility, Core Web Vitals performance, CSS architecture, state management principles, TypeScript discipline, and testing. Use for any web UI work regardless of framework. Pairs with nextjs-expert.
---

# Web Dev Expert (Frontend Craft)

Framework-agnostic principles a senior frontend engineer applies on every project. This skill is the mindset layer — not a guide to any specific library or tool. For Next.js-specific guidance use [nextjs-expert]; this skill is the layer beneath it.

## What a Frontend Expert Values

The lens to read everything below through. What separates expert work from "it looks right on my machine":

- **The user's worst device, not yours.** An expert designs for the slow phone on a flaky network, because that's the real audience. Every dependency and every kilobyte of JS is weighed against that user.
- **Accessibility is correctness, full stop.** Not a checklist bolted on at the end — if it doesn't work with a keyboard and a screen reader, it's broken, the same way a wrong calculation is broken.
- **The platform first.** An expert knows what the browser already does well and reaches for a dependency only when the platform genuinely can't. Less code is a feature.
- **Performance is a budget, not a vibe.** Core Web Vitals are concrete numbers the expert holds the work to, and they *measure* before optimizing rather than guessing hot paths.
- **State has a home.** The expert categorizes state (server / URL / local / global) and puts it where it belongs — most "weird UI bugs" are state in the wrong place.
- **Resilient at the boundary.** External data is untrusted: typed and validated where it enters. Illegal states are made unrepresentable rather than defended against everywhere.

**The bar for "expert-level work":** fully keyboard-operable with visible, managed focus; no layout shift; no unnecessary JS on the critical path; data typed and validated at the edge; responsive without a pile of breakpoints. Works for everyone, on everything, fast.

## Operating Principles

1. **The platform is powerful — reach for it first.** Native elements, built-in form validation, internationalization, URL handling, view transitions, and modern CSS layout often beat a dependency.
2. **Ship less JavaScript.** Every KB of JS is parse + execute on the user's slowest device. Prefer HTML/CSS solutions; lazy-load the rest.
3. **Accessibility is correctness, not a feature.** If it doesn't work with a keyboard and a screen reader, it's broken.
4. **Measure before optimizing.** Use real profiling and real-device testing. Don't guess hot paths.

## Semantic HTML & Accessibility (non-negotiable)

- Use the right element: a button for actions, a link for navigation, landmark elements for structure, real headings in order (one top-level heading).
- Every interactive element is keyboard-reachable and has a visible focus ring. Never remove focus outlines without a replacement.
- Label every input. Associate errors with their input programmatically.
- ARIA is a last resort — a correct native element needs none. First rule of ARIA: don't use ARIA if HTML can do it.
- Respect reduced-motion preferences; gate non-essential animation behind them.
- Color contrast ≥ 4.5:1 for body text. Don't encode meaning in color alone.
- Manage focus on route change / modal open / dynamic content insertion.

## Core Web Vitals (the performance contract)

- **LCP < 2.5s** — prioritize the hero image/text. Preload the LCP resource, hint its priority high, avoid lazy-loading above the fold.
- **INP < 200ms** — keep main-thread tasks small; break up long tasks, debounce, move heavy work off the critical path.
- **CLS < 0.1** — always reserve space for media and dynamic content (dimensions or aspect-ratio); never inject content above existing content.
- Images: serve modern formats, responsive sources, lazy-load below the fold.
- Fonts: swap display strategy, preload, subset; avoid layout shift from font swap.
- Code-split by route and by interaction; defer non-critical third-party scripts.

## CSS Architecture

- **Modern layout primitives:** flexbox + grid for layout, gap over margins, container queries over global media queries for component-level responsiveness, logical properties for i18n.
- **Design tokens as CSS variables** — single source for color/spacing/typography. Theme (dark mode) by swapping variable values, not duplicating rules.
- Keep specificity flat. Avoid forced overrides and deep descendant selectors. Co-locate styles with components.
- Fluid sizing with clamping; avoid a dozen breakpoints. Use min/max for responsive constraints.

## State Management Principles

- **Categorize state before choosing a tool:**
  - *Server state* (data from an API) → a query/cache layer. Don't hand-roll caching.
  - *URL state* (filters, tabs, pagination) → the URL. Shareable, back-button-friendly.
  - *Local UI state* → component state.
  - *Global client state* (theme, auth UI) → a lightweight store, kept small.
- Don't put server data into global client state and sync manually — that's the #1 source of stale-UI bugs.
- Derive, don't duplicate. Computed values come from a single source at render time.

## TypeScript Discipline

- Strict mode on. No `any` — use `unknown` and narrow at the boundary (API responses, parsed JSON, form data).
- Validate external data at runtime where it enters the app; types alone don't protect against a wrong API shape.
- Prefer discriminated unions over optional-flag soup. Make illegal states unrepresentable.
- Derive types from one source instead of restating shapes.

## Testing Strategy

- Test behavior, not implementation. Query by role/label — if the test can't find it by role, neither can a screen reader.
- Pyramid: many fast unit tests, fewer integration, a handful of end-to-end tests for critical journeys.
- Cover the bug with a failing test before fixing it.

## Anti-Patterns to Reject

- A clickable non-interactive element instead of a real button; structureless markup with no landmarks.
- Animating layout-affecting properties instead of transform/opacity.
- Loading a library for one function the platform already provides.
- Global state for data that belongs in the URL or server cache.
- Pixel-perfect fixed breakpoints instead of fluid/container-based responsiveness.
- Disabling focus outlines for aesthetics.

## Before Saying "Done"

- [ ] Works with keyboard only; focus is visible and managed.
- [ ] No layout shift; media has dimensions.
- [ ] No unnecessary JS shipped; heavy/below-fold work is deferred.
- [ ] External data is typed and validated at the boundary.
- [ ] Responsive without a breakpoint pile (container queries / fluid sizing).
