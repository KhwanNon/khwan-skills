---
name: flutter-expert
description: Expert-level Flutter engineering with Clean Architecture + BLoC. Use when writing, reviewing, or structuring Flutter/Dart code — feature layering (data/domain/presentation), BLoC/Cubit state, GetIt DI, Dartz Either error handling, Drift persistence, go_router, and widget composition. Tuned to the m_vara stack.
---

# Flutter Expert (Clean Architecture + BLoC)

Senior-level judgment for Flutter using the project's stack: **flutter_bloc, get_it, dartz, drift, go_router, dio, equatable**. Bias: strict layer boundaries, immutable state, errors as values. Pair with [mobile-dev-expert] for cross-cutting mobile concerns (offline sync, security, lifecycle).

## The Layered Mental Model

Every feature is three layers; **dependencies point inward only** (presentation → domain ← data):

```
features/<feature>/
  domain/            # PURE Dart. No Flutter, no dio, no drift imports.
    entities/        # business objects (Equatable)
    repositories/    # abstract repository contracts (return Either)
    usecases/        # one class, one verb: call() -> Future<Either<Failure, T>>
  data/
    models/          # DTOs: extend/map to entities, fromJson/toJson, fromDrift
    datasources/     # remote (dio) + local (drift) — throw exceptions here
    repositories/    # implements domain contract; catches exceptions -> Failure
  presentation/
    logic/bloc/      # Bloc/Cubit — depends on usecases only
    widgets/         # dumb widgets driven by state
    pages/           # screen composition
```

**The test of a correct boundary:** can you swap dio for GraphQL by touching only `data/`? Can you unit-test a usecase with zero Flutter imports? If not, a layer leaked.

## Error Handling — Errors as Values (Dartz)

- Datasources **throw** typed exceptions (`ServerException`, `CacheException`).
- Repositories **catch** and convert to `Either<Failure, T>` — exceptions never escape the data layer.
- Usecases and BLoC handle `Either` with `.fold(onFailure, onSuccess)`. No try/catch in presentation.
- One `Failure` hierarchy (Equatable) in `core/error`. Map exception → failure in one place.

```dart
Future<Either<Failure, User>> login(Creds c) async {
  try {
    return Right(await remote.login(c));
  } on ServerException catch (e) {
    return Left(ServerFailure(e.message));
  }
}
```

## State Management (flutter_bloc)

- **State is immutable + Equatable.** Always `copyWith`; never mutate fields. Equatable `props` must list every field that affects equality, or the UI won't rebuild.
- **Cubit for simple state, Bloc for event-driven/complex flows.** Don't reach for Bloc ceremony when a Cubit suffices.
- Model load states explicitly — prefer a sealed/union status (`initial/loading/success/failure`) over scattered booleans (`isLoading`, `hasError`). Make illegal states unrepresentable.
- BLoC contains **no UI and no Flutter widgets**; it depends on usecases, not datasources or dio.
- Emit after `await`? Guard with `if (!isClosed)` or `emit.isDone`. Never `emit` after the bloc is closed.
- In widgets: `BlocBuilder` for rebuilds, `BlocListener` for side-effects (navigation, snackbars), `BlocSelector` to rebuild on one field only. Don't navigate inside `builder`.
- `context.read<T>()` in callbacks, `context.watch<T>()`/`BlocBuilder` in build. Never `read` in build for reactive data.

## Dependency Injection (GetIt)

- Register in one composition root (`injection_container.dart` / `core`). Layers depend on **abstractions**, resolved by GetIt — never `new` a repository inside a widget.
- `registerFactory` for blocs (fresh per screen), `registerLazySingleton` for repositories/datasources/dio/db (shared, created on first use).
- Keep `get_it` calls at the edges (composition root, route builders). Don't sprinkle `sl<X>()` through business logic — inject via constructor.

## Persistence (Drift) & Offline

- Tables in `database/tables`, DAOs per aggregate in `database/daos`. Keep queries in DAOs, not in repositories.
- Map Drift rows → domain entities in the data layer. Domain must not import Drift.
- For offline-first / sync, see [mobile-dev-expert]. Repository decides cache-vs-network policy (e.g. local-first with background refresh).

## Widgets & Performance

- **Compose small `const` widgets.** A `const` constructor skips rebuilds — use it everywhere possible. Break large `build` methods into separate widget classes, not private `_buildX()` methods (those rebuild with the parent).
- Add `Key`s to items in dynamic lists; use `ListView.builder` (lazy) not `ListView(children: [...])` for long lists.
- Keep `build` pure and cheap — no I/O, no allocation of controllers/blocs. Create those in `initState`/DI and dispose in `dispose`.
- Avoid `Opacity`/`Clip` in hot paths (expensive); prefer `AnimatedOpacity`/`Container` decoration. Use `RepaintBoundary` around independently-animating subtrees.
- Always `dispose()` controllers, focus nodes, stream subscriptions, animation controllers.

## Routing (go_router)

- Central route config; type-safe params. Use `redirect` for auth gating (read auth state), not ad-hoc navigation guards in widgets.
- Drive navigation from `BlocListener` reacting to auth/flow state, not by calling `context.go` deep inside business logic.

## Dart Discipline

- `final`/`const` by default; avoid `var` for fields. Null-safety: avoid `!` — handle null explicitly or use `?.`/`??`.
- Prefer sealed classes / pattern matching (Dart 3) for state and result unions.
- Keep functions short and single-purpose. Name usecases as verbs (`LoginUser`, `FetchMeetings`).

## Anti-Patterns to Reject

- Flutter/dio/drift imports inside `domain/`.
- Business logic or `dio`/`http` calls inside widgets.
- Mutable bloc state or `props` missing a field (UI won't update).
- try/catch in presentation instead of `Either.fold`.
- `setState` for state that a Cubit/Bloc already owns; mixing both for the same data.
- Giant `build()` with nested `_buildHeader()/_buildBody()` helpers instead of widget classes.
- `sl<X>()` scattered through logic instead of constructor injection.
- Forgetting `dispose()` (memory leaks) or emitting after close.

## Before Saying "Done"

- [ ] `flutter analyze` clean; `dart format` applied.
- [ ] domain layer has zero Flutter/data imports.
- [ ] All bloc state immutable; Equatable `props` complete.
- [ ] Errors flow as `Either<Failure, T>`; no exceptions escape data layer.
- [ ] Controllers/subscriptions disposed; no `emit` after close.
- [ ] `const` constructors used; long lists use `.builder`.
