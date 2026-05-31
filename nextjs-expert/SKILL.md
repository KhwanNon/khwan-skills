---
name: nextjs-expert
description: Expert-level Next.js (App Router) and React engineering. Use when writing, reviewing, or structuring Next.js / React code — server vs client components, data fetching & caching, routing, Tailwind, performance, and project structure. Tuned for Next 15/16 + React 19.
---

# Next.js Expert (App Router)

Senior-level judgment for Next.js App Router + React 19. Bias: **server-first, minimal client JS, explicit boundaries.** Pair with [web-dev-expert] for framework-agnostic frontend craft.

## Mental Model First

Before writing code, decide three things:
1. **Where does this run?** Server Component (default) or Client (`'use client'`)? Default to server.
2. **Where does the data come from, and when?** Build time, request time, or client runtime?
3. **What is the cache lifetime?** Static, revalidated (ISR), or dynamic?

If you can't answer these, stop and ask. Most Next.js mistakes are boundary mistakes, not syntax.

## The Server/Client Boundary (most important rule)

- **Server Components are the default.** Keep `'use client'` as deep/leaf as possible. A page should be a server component that composes small client islands.
- A client component **cannot** import a server component as a child, but it **can** receive one via `children`/props. Use this to keep interactivity isolated:
  ```tsx
  // page.tsx (server) — fetches data, passes server-rendered children in
  <ClientShell><ServerContent /></ClientShell>
  ```
- Never put secrets, DB clients, or heavy deps in a file that has (or is imported by) `'use client'`. Mark server-only modules with `import 'server-only'`.
- Push `useState`/`useEffect`/event handlers down into the smallest possible component. If a whole page is `'use client'` just for one button, you did it wrong.

## Data Fetching & Caching (Next 15/16)

- Fetch in **Server Components** with `async`/`await`. No `useEffect` for initial data.
- In Next 15+, `fetch` is **uncached by default**. Be explicit:
  - `fetch(url, { cache: 'force-cache' })` — static
  - `fetch(url, { next: { revalidate: 60 } })` — ISR
  - `fetch(url, { cache: 'no-store' })` — always dynamic
- Use `next: { tags: [...] }` + `revalidateTag()` for on-demand invalidation after mutations.
- **Mutations → Server Actions** (`'use server'`), not API routes, unless you need a public/3rd-party HTTP endpoint. After a mutation call `revalidatePath`/`revalidateTag`.
- Parallelize independent fetches with `Promise.all`; don't create waterfalls. Co-locate fetches with the component that needs them and let Suspense stream.
- `await cookies()`, `await headers()`, `await params`, `await searchParams` — these are **async** in Next 15+.

## Routing & Files

- Route groups `(group)` organize without affecting the URL. Private folders `_folder` are excluded from routing — good for co-located `_components`, `_data`, `_lib`.
- Use the special files deliberately: `loading.tsx` (Suspense fallback), `error.tsx` (must be `'use client'`), `not-found.tsx`, `layout.tsx` (persists across navigation, don't refetch per-page data here unless shared).
- Prefer `<Link>` + `useRouter` from `next/navigation` (not `next/router`).
- Stream slow content with `<Suspense>` boundaries so the shell paints instantly.

## React 19 Notes

- `use()` to unwrap promises/context in render. `useActionState` for form+action state. `useOptimistic` for instant UI on mutations. `useFormStatus` for pending state inside forms.
- `ref` is now a normal prop — `forwardRef` is rarely needed.
- Don't reach for `useMemo`/`useCallback` reflexively; React Compiler (when enabled) handles most. Memoize only measured hot paths.

## Styling (Tailwind 4 + framer-motion)

- Tailwind 4 is CSS-first: config lives in `@theme` inside CSS, not `tailwind.config.js`. Define design tokens as CSS variables there.
- Extract repeated class strings into a component, not a `@apply` soup. Use `clsx`/`cva` for conditional variants.
- framer-motion components are client-only — wrap them in a leaf `'use client'` island, never make the whole page client just to animate.

## Project Structure

Your current layout (`app/(web)`, `app/modules`, `app/shared`) is sound. Reinforce it:
```
app/
  (web)/                 # route group: actual pages/routes
    <route>/
      page.tsx
      _components/        # route-local UI (private folder)
      _data/              # route-local data/constants
      _lib/               # route-local logic
  modules/                # feature modules (cross-route domain logic)
    <feature>/
  shared/                 # truly shared: ui/, lib/, hooks/, types/
```
Rules of thumb:
- **Co-locate first.** Code lives next to the route that uses it (`_components`). Promote to `modules/` only when a 2nd route needs it, to `shared/` only when it's domain-agnostic.
- One barrel `index.ts` per module max; avoid deep re-export chains (they hurt tree-shaking and create cycles).
- Keep `page.tsx` thin: fetch + compose. Logic in `_lib`, UI in `_components`.

## TypeScript Discipline

- No `any`. Use `unknown` + narrowing at boundaries. Type API responses explicitly; don't trust `fetch` shapes.
- Use `satisfies` for config objects to keep literal types. Derive types from a single source (`as const`, schema) instead of duplicating.

## Anti-Patterns to Reject

- `'use client'` at the top of a page/layout "to be safe" → cascades client-down.
- `useEffect` + `fetch` for data a Server Component could load.
- API route as a thin proxy to your own DB when a Server Action would do.
- Fetch waterfalls (sequential awaits that don't depend on each other).
- Storing server state in client state and re-syncing manually.
- `tailwind.config.js` theme edits in a Tailwind 4 project.
- Giant barrel `index.ts` re-exporting everything.

## Before Saying "Done"

- [ ] `next build` passes (catches server/client violations, type errors).
- [ ] No `'use client'` higher than necessary.
- [ ] Caching is explicit and intentional per fetch.
- [ ] Mutations revalidate the affected paths/tags.
- [ ] No secrets reachable from client bundles.
