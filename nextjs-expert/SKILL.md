---
name: nextjs-expert
description: Expert-level Next.js (App Router) and React engineering mindset. Use when writing, reviewing, or structuring Next.js / React code — server vs client boundaries, data fetching & caching decisions, routing, performance, and project structure.
---

# Next.js Expert (App Router)

Senior-level judgment for Next.js App Router + React. Bias: **server-first, minimal client JS, explicit boundaries.** This skill is the mindset for reasoning about those boundaries — not a guide to any specific library. Pair with [web-dev-expert] for framework-agnostic frontend craft.

## What a Next.js Expert Values

The lens to read everything below through. What separates expert work from "it renders":

- **Server-first as a reflex.** The expert's default is the server; reaching for the client is a deliberate, justified decision, not the starting point. They feel the cost of every kilobyte shipped to the browser.
- **Boundaries are the whole game.** Most Next.js bugs are boundary mistakes (where it runs, where data comes from, what's cached) — not syntax. An expert decides these *before* writing code and keeps the client boundary as small and deep as possible.
- **Caching is an intentional decision, never a default you inherited.** For every fetch the expert can say *why* it's static, revalidated, or dynamic. "I'm not sure how this caches" is an unfinished thought.
- **No waterfalls, stream the slow parts.** Independent data loads in parallel; slow content streams behind a boundary so the shell paints instantly. Perceived speed is designed, not hoped for.
- **Secrets and server-only code never touch the client bundle.** This is a correctness/security line, not a preference.
- **Co-location and a thin page.** The expert keeps code next to the route that uses it and promotes to shared scope only when reuse demands it — structure earns its complexity.

**The bar for "expert-level work":** a production build that's clean; no `'use client'` higher than it needs to be; caching that's explicit and intentional per request; mutations that revalidate exactly what changed; no secrets reachable from the browser; fast first paint via streaming, not via shipping everything.

## Mental Model First

Before writing code, decide three things:
1. **Where does this run?** Server Component (default) or Client? Default to server.
2. **Where does the data come from, and when?** Build time, request time, or client runtime?
3. **What is the cache lifetime?** Static, revalidated (ISR), or dynamic?

If you can't answer these, stop and ask. Most Next.js mistakes are boundary mistakes, not syntax.

## The Server/Client Boundary (most important rule)

- **Server Components are the default.** Keep the client boundary as deep/leaf as possible. A page should be a server component that composes small client islands.
- A client component **cannot** import a server component as a child, but it **can** receive one via `children`/props. Use this to keep interactivity isolated while server-rendered content flows through.
- Never put secrets, DB clients, or heavy deps in a file that is (or is imported by) a client component. Keep server-only modules clearly marked so they can't leak into the client bundle.
- Push interactive state and effects down into the smallest possible component. If a whole page is a client component just for one button, you did it wrong.

## Data Fetching & Caching

- Fetch in **Server Components** with `async`/`await`. No client-side effect for initial data.
- **Be explicit about caching.** Decide per request whether it's static, revalidated on an interval (ISR), or always dynamic — don't rely on defaults you haven't verified.
- Use tag-based revalidation for on-demand invalidation after mutations.
- **Mutations belong in server-side actions**, not ad-hoc API routes, unless you genuinely need a public/3rd-party HTTP endpoint. After a mutation, revalidate the affected paths/tags.
- Parallelize independent fetches; don't create waterfalls. Co-locate fetches with the component that needs them and let streaming fill in slow parts.
- Treat request-scoped inputs (cookies, headers, params, search params) as async — await them.

## Routing & Files

- Route groups organize without affecting the URL. Private folders are excluded from routing — good for co-located components, data, and logic.
- Use the framework's special files deliberately: loading fallback, error boundary (client-side), not-found, and layout (persists across navigation — don't refetch per-page data there unless it's genuinely shared).
- Prefer the framework's navigation primitives over manual history manipulation.
- Stream slow content behind suspense boundaries so the shell paints instantly.

## React Notes

- Use the modern primitives for unwrapping promises/context, form+action state, optimistic UI, and pending state — instead of hand-rolled effects.
- Treat refs as ordinary props where the version allows it.
- Don't reach for memoization reflexively; the compiler handles most cases. Memoize only measured hot paths.

## Styling

- Keep design tokens in one place (CSS variables); theme by swapping values, not duplicating rules.
- Extract repeated class strings into a component, not a soup of inline directives. Use a small helper for conditional variants.
- Animation libraries are client-only — wrap them in a leaf client island, never make the whole page client just to animate.

## Project Structure

```
app/
  (group)/                # route group: actual pages/routes
    <route>/
      page                # thin: fetch + compose
      _components/        # route-local UI (private folder)
      _data/              # route-local data/constants
      _lib/               # route-local logic
  modules/                # feature modules (cross-route domain logic)
  shared/                 # truly shared: ui, lib, hooks, types
```
Rules of thumb:
- **Co-locate first.** Code lives next to the route that uses it. Promote to a feature module only when a 2nd route needs it; promote to shared only when it's domain-agnostic.
- One barrel per module max; avoid deep re-export chains (they hurt tree-shaking and create cycles).
- Keep the page thin: fetch + compose. Logic in `_lib`, UI in `_components`.

## TypeScript Discipline

- No `any`. Use `unknown` + narrowing at boundaries. Type external responses explicitly; don't trust fetched shapes.
- Derive types from a single source instead of duplicating shapes.

## Anti-Patterns to Reject

- Marking a page/layout as a client component "to be safe" → cascades client-down.
- A client-side effect fetching data a Server Component could load.
- An API route that's just a thin proxy to your own DB when a server action would do.
- Fetch waterfalls (sequential awaits that don't depend on each other).
- Storing server state in client state and re-syncing manually.
- Editing theme config in the wrong place for your styling setup.
- A giant barrel re-exporting everything.

## Before Saying "Done"

- [ ] Production build passes (catches server/client violations, type errors).
- [ ] No client boundary higher than necessary.
- [ ] Caching is explicit and intentional per fetch.
- [ ] Mutations revalidate the affected paths/tags.
- [ ] No secrets reachable from client bundles.
