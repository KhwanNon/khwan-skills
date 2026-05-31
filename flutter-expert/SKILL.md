---
name: flutter-expert
description: Expert-level Flutter engineering with Clean Architecture + BLoC mindset. Use when writing, reviewing, or structuring Flutter/Dart code — feature layering (data/domain/presentation), immutable state, errors-as-values, dependency injection, persistence, routing, and widget composition.
---

# Flutter Expert (Clean Architecture + BLoC)

Senior-level judgment for Flutter. Bias: **strict layer boundaries, immutable state, errors as values.** This skill is the architectural mindset — not a guide to any specific package. Pair with [mobile-dev-expert] for cross-cutting mobile concerns (offline sync, security, lifecycle).

## What a Flutter Expert Values

The lens to read everything below through. An expert is judged less by "does it work" and more by these:

- **Boundaries over speed.** A feature that works but leaks the network/persistence into domain or UI is *not done*. The architecture is the deliverable as much as the behavior.
- **Predictable rebuilds.** The expert thinks in terms of *what triggers a rebuild and why*. Immutable state and a complete equality contract aren't ceremony — they're the difference between a UI that's correct by construction and one that's debugged by guesswork.
- **The frame budget is real.** ~16ms per frame (8ms at 120Hz). Jank is a bug, not a polish item. An expert feels the cost of work in `build` and on the main thread.
- **Errors are data, not surprises.** Every failure path is modelled and visible in the type. "It threw somewhere" is a junior outcome.
- **Testability is the proof.** If a usecase can't be unit-tested without the framework, the design is wrong — regardless of whether the screen looks fine.
- **Disposal and lifetimes are not optional.** An expert tracks every controller/subscription/stream they create and knows where it dies. Leaks are a competence signal.

**The bar for "expert-level work":** layers you could hand to another dev and they'd know exactly where each thing goes; state transitions you can reason about on paper; zero analyzer warnings; no surprise rebuilds, no leaks, no exceptions escaping the data layer. Production-grade, not demo-grade.

## The Layered Mental Model

Every feature is three layers; **dependencies point inward only** (presentation → domain ← data):

```
features/<feature>/
  domain/            # PURE Dart. No framework, no I/O, no persistence imports.
    entities/        # business objects
    repositories/    # abstract repository contracts (return a result type)
    usecases/        # one class, one verb
  data/
    models/          # DTOs: map to/from entities and to/from external shapes
    datasources/     # remote + local — throw exceptions here
    repositories/    # implements domain contract; catches exceptions -> failure
  presentation/
    logic/           # state holders — depend on usecases only
    widgets/         # dumb widgets driven by state
    pages/           # screen composition
```

**The test of a correct boundary:** can you swap the network transport (REST → GraphQL) by touching only `data/`? Can you unit-test a usecase with zero framework imports? If not, a layer leaked.

## Error Handling — Errors as Values

- Datasources **throw** typed exceptions.
- Repositories **catch** and convert to a result value (success/failure union) — exceptions never escape the data layer.
- Usecases and state holders handle the result by folding over both branches. No try/catch in presentation.
- One `Failure` hierarchy in `core/error`. Map exception → failure in one place.

## State Management

- **State is immutable.** Always copy to a new value; never mutate fields. The equality contract must cover every field that affects the UI, or the UI won't rebuild.
- **Reach for the simplest state holder that fits.** Don't add event-driven ceremony when a plain state holder suffices.
- Model load states explicitly — prefer a sealed/union status (`initial/loading/success/failure`) over scattered booleans. Make illegal states unrepresentable.
- State holders contain **no UI and no widgets**; they depend on usecases, not datasources or transport.
- Emitting after an `await`? Guard against emitting once the holder is closed/disposed.
- In widgets: rebuild on state, run side-effects (navigation, snackbars) in a listener — not in the build path. Subscribe to the narrowest slice of state needed.
- Read state in callbacks; watch state in build. Never read non-reactively in build for data that should drive rebuilds.

## Dependency Injection

- Register in one composition root. Layers depend on **abstractions**, resolved by the container — never construct a repository inside a widget.
- Fresh instances for screen-scoped state holders; shared lazy singletons for repositories/datasources/clients/db.
- Keep container lookups at the edges (composition root, route builders). Don't sprinkle service-locator calls through business logic — inject via constructor.

## Persistence & Offline

- Tables and data-access objects live in the data layer. Keep queries there, not in repositories.
- Map persistence rows → domain entities in the data layer. Domain must not import the persistence library.
- For offline-first / sync, see [mobile-dev-expert]. The repository decides cache-vs-network policy (e.g. local-first with background refresh).

## Widgets & Performance

- **Compose small immutable widgets.** A const constructor skips rebuilds — use it everywhere possible. Break large `build` methods into separate widget classes, not private `_buildX()` methods (those rebuild with the parent).
- Add keys to items in dynamic lists; use lazy/builder lists (not eager children) for long lists.
- Keep `build` pure and cheap — no I/O, no allocation of controllers/state holders. Create those in init/DI and dispose them.
- Avoid expensive compositing operations in hot paths; isolate independently-animating subtrees so they don't repaint their neighbours.
- Always dispose controllers, focus nodes, stream subscriptions, animation controllers.

## Routing

- Central route config; type-safe params. Use redirects for auth gating (read auth state), not ad-hoc navigation guards in widgets.
- Drive navigation from a listener reacting to auth/flow state, not by navigating deep inside business logic.

## Dart Discipline

- `final`/`const` by default; avoid `var` for fields. Null-safety: avoid force-unwrap — handle null explicitly.
- Prefer sealed classes / pattern matching for state and result unions.
- Keep functions short and single-purpose. Name usecases as verbs.

## Anti-Patterns to Reject

- Framework / transport / persistence imports inside `domain/`.
- Business logic or network calls inside widgets.
- Mutable state, or an equality contract missing a field (UI won't update).
- try/catch in presentation instead of folding a result value.
- Local widget state for data a state holder already owns; mixing both for the same data.
- Giant `build()` with nested `_buildHeader()/_buildBody()` helpers instead of widget classes.
- Service-locator calls scattered through logic instead of constructor injection.
- Forgetting to dispose (memory leaks) or emitting after close.

## Before Saying "Done"

- [ ] Static analysis clean; formatting applied.
- [ ] Domain layer has zero framework/data imports.
- [ ] All state immutable; equality contract complete.
- [ ] Errors flow as result values; no exceptions escape the data layer.
- [ ] Controllers/subscriptions disposed; no emit after close.
- [ ] Const constructors used; long lists are lazy.
