---
name: mobile-dev-expert
description: Expert cross-cutting mobile engineering — offline-first & sync, secure storage & app hardening, app lifecycle & background work, platform integration (iOS/Android), permissions, and on-device performance/battery. Framework-agnostic mobile concerns. Pairs with flutter-expert for Flutter specifics.
---

# Mobile Dev Expert (Cross-Cutting Concerns)

The concerns that make a mobile app production-grade, independent of UI framework. For Flutter architecture/state specifics use [flutter-expert]; this skill covers what surrounds the app: the network, the OS, the device, and the threat model.

## The Mobile Reality (design for it from line one)

A mobile app is **not** a desktop app with a small screen. Assume:
- The network is intermittent, slow, and metered. **Offline is a normal state, not an error.**
- The OS can suspend or kill the app at any moment. State must survive process death.
- The device is a hostile environment — it can be lost, stolen, or rooted/jailbroken.
- Battery, memory, and CPU are scarce and shared. Background work is rationed by the OS.

## Offline-First & Sync (the hardest part — design explicitly)

- **Local store is the source of truth for the UI.** Read from local DB (Drift/SQLite); the network updates the store, the UI never waits on the network to render.
- **Outbound mutations go through a queue.** Persist the intent locally, mark it pending, sync when connectivity returns. The user's action must survive an app kill.
- **Idempotency:** every mutation carries a client-generated id (UUID) so retries don't double-apply on the server.
- **Conflict policy is a decision, not an accident.** Pick per entity: last-write-wins (timestamp/version), server-wins, or merge. Write it down.
- **Reconciliation:** pull changes since a cursor/`updatedAt`; apply with deterministic ordering. Track sync state per record (`synced/pending/failed`) so the UI can show it.
- React to connectivity (`connectivity_plus`) to trigger sync, but **also** verify reachability — "connected to wifi" ≠ "server reachable".
- Show sync status honestly. Never silently drop a failed mutation.

## Security & App Hardening

- **Never store secrets in plain prefs.** Use the platform keystore/keychain (`flutter_secure_storage`) for tokens, keys, credentials. Prefs/SQLite are world-readable on rooted devices.
- **Tokens:** short-lived access + refresh; refresh transparently on 401; clear all secure storage on logout. Don't log tokens.
- **Transport:** HTTPS only. Consider certificate pinning for high-value APIs (balance against rotation pain).
- **Tamper/root detection** (`flutter_jailbreak_detection`): detect and degrade/deny on jailbroken/rooted devices for sensitive flows — but treat it as defense-in-depth, not a guarantee.
- **Sensitive data at rest:** encrypt local DB/files (`encrypt`/`cryptography`) when storing PII or documents. Derive keys from secure storage, never hard-code.
- Obscure sensitive screens in the app switcher; disable screenshots on secure screens where required.
- Validate deep links (`app_links`) — treat their payload as untrusted input.

## App Lifecycle & Background Work

- Handle lifecycle transitions (`AppLifecycleState` resumed/inactive/paused/detached): pause timers/streams/video on `paused`, refresh stale data on `resumed`, persist unsaved work before backgrounding.
- **Process death is normal.** Restore UI state from persisted storage on cold start; don't assume in-memory state survives.
- Background tasks (`workmanager`) are **opportunistic and OS-throttled** — never promise exact timing. Keep them short, idempotent, and resumable. iOS background execution is especially limited.
- Long-lived connections (`web_socket_channel`): tear down on background, reconnect with backoff on resume; don't drain battery holding sockets in the background.

## Platform Integration

- Respect platform conventions — back navigation (Android hardware/predictive back), safe areas/notches, haptics, share sheets (`share_plus`), file open (`open_filex`).
- **Permissions:** request just-in-time with rationale, degrade gracefully on denial, handle "permanently denied" by routing to settings. Never block the app on an optional permission.
- Test on real devices across OS versions and screen sizes — emulators hide jank, thermal throttling, and real network behavior.

## On-Device Performance & Battery

- **Startup matters most** — defer non-critical init off the cold-start path; lazy-init heavy singletons (you already do via GetIt lazy singletons).
- Decode/resize images to display size; cache them. Large bitmaps are the #1 memory/jank cause.
- Batch and debounce network calls; coalesce sync. Each radio wake costs battery.
- Stream/paginate large lists and large API responses; never load an unbounded set into memory.
- Watch memory: dispose controllers/streams (see [flutter-expert]), avoid retaining large objects in long-lived singletons.
- Profile with real tooling (Flutter DevTools timeline, memory, network) — measure, don't guess.

## Observability

- Structured logging (`talker`) with levels; **redact secrets/PII**. Ship crash + error reporting so field failures are visible.
- Log sync outcomes and network failures with enough context to diagnose, without leaking sensitive payloads.

## Anti-Patterns to Reject

- Treating offline as an error state / blocking the UI on the network.
- Fire-and-forget mutations with no local queue — lost on app kill.
- Tokens/PII in SharedPreferences or plain SQLite.
- Assuming in-memory state survives backgrounding or process death.
- Promising precise timing from background tasks.
- Holding sockets/timers/GPS active in the background.
- Loading full datasets/images into memory instead of paging/resizing.
- Logging tokens, request bodies with PII, or full responses.

## Before Saying "Done"

- [ ] App renders and is usable with no network; actions queue and sync on reconnect.
- [ ] Secrets only in secure storage; nothing sensitive logged.
- [ ] State survives backgrounding and cold start (process death).
- [ ] Permissions degrade gracefully when denied.
- [ ] Background/socket work pauses in background and resumes with backoff.
- [ ] Verified on a real device, not just the simulator.
