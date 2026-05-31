---
name: mobile-dev-expert
description: Expert cross-cutting mobile engineering mindset — offline-first & sync, secure storage & app hardening, app lifecycle & background work, platform integration (iOS/Android), permissions, and on-device performance/battery. Framework-agnostic mobile concerns. Pairs with flutter-expert for Flutter specifics.
---

# Mobile Dev Expert (Cross-Cutting Concerns)

The concerns that make a mobile app production-grade, independent of UI framework or specific library. This skill is the mindset for what surrounds the app: the network, the OS, the device, and the threat model. For Flutter architecture/state specifics use [flutter-expert].

## What a Mobile Expert Values

The lens to read everything below through. What separates expert work from "works on the simulator":

- **Resilience over the happy path.** A junior builds for the case where the network is up, the app stays foregrounded, and the device is trusted. An expert builds for the opposite of all three — that's where mobile actually lives.
- **Offline is a first-class state, not an error.** The UI reads from the local store and never blocks on the network. The expert designs the sync and conflict story *up front*, because retrofitting it is the hardest job in mobile.
- **The user's action must survive an app kill.** Mutations are persisted intent, not fire-and-forget. Losing a user's tap because the OS reclaimed memory is unacceptable, not unlucky.
- **The device is hostile.** It can be lost, stolen, or rooted. Secrets go in secure storage, sensitive data is encrypted at rest, and nothing sensitive is ever logged — by default, not by exception.
- **Battery and the OS are in charge.** Background work is opportunistic and throttled; the expert never promises exact timing and never holds radios/sockets/GPS open needlessly.
- **Real devices tell the truth.** Emulators hide jank, thermal throttling, and real network behavior. Verification happens on hardware.

**The bar for "expert-level work":** the app is usable with no network and reconciles cleanly on reconnect; state survives backgrounding and process death; secrets live only in secure storage; permissions degrade gracefully; background and socket work pauses and resumes politely; and it's been proven on a real device, not just the simulator.

## The Mobile Reality (design for it from line one)

A mobile app is **not** a desktop app with a small screen. Assume:
- The network is intermittent, slow, and metered. **Offline is a normal state, not an error.**
- The OS can suspend or kill the app at any moment. State must survive process death.
- The device is a hostile environment — it can be lost, stolen, or rooted/jailbroken.
- Battery, memory, and CPU are scarce and shared. Background work is rationed by the OS.

## Offline-First & Sync (the hardest part — design explicitly)

- **Local store is the source of truth for the UI.** Read from the local store; the network updates the store, the UI never waits on the network to render.
- **Outbound mutations go through a queue.** Persist the intent locally, mark it pending, sync when connectivity returns. The user's action must survive an app kill.
- **Idempotency:** every mutation carries a client-generated id so retries don't double-apply on the server.
- **Conflict policy is a decision, not an accident.** Pick per entity: last-write-wins (timestamp/version), server-wins, or merge. Write it down.
- **Reconciliation:** pull changes since a cursor/timestamp; apply with deterministic ordering. Track sync state per record (synced/pending/failed) so the UI can show it.
- React to connectivity to trigger sync, but **also** verify reachability — "connected to wifi" ≠ "server reachable".
- Show sync status honestly. Never silently drop a failed mutation.

## Security & App Hardening

- **Never store secrets in plain preferences.** Use the platform keystore/keychain for tokens, keys, credentials. Plain prefs/local DB are readable on rooted devices.
- **Tokens:** short-lived access + refresh; refresh transparently on auth failure; clear all secure storage on logout. Don't log tokens.
- **Transport:** HTTPS only. Consider certificate pinning for high-value APIs (balance against rotation pain).
- **Tamper/root detection:** detect and degrade/deny on jailbroken/rooted devices for sensitive flows — but treat it as defense-in-depth, not a guarantee.
- **Sensitive data at rest:** encrypt the local store/files when holding PII or documents. Derive keys from secure storage, never hard-code.
- Obscure sensitive screens in the app switcher; disable screenshots on secure screens where required.
- Validate deep links — treat their payload as untrusted input.

## App Lifecycle & Background Work

- Handle lifecycle transitions (resumed/inactive/paused/detached): pause timers/streams/video when backgrounded, refresh stale data on resume, persist unsaved work before backgrounding.
- **Process death is normal.** Restore UI state from persisted storage on cold start; don't assume in-memory state survives.
- Background tasks are **opportunistic and OS-throttled** — never promise exact timing. Keep them short, idempotent, and resumable. iOS background execution is especially limited.
- Long-lived connections: tear down on background, reconnect with backoff on resume; don't drain battery holding sockets in the background.

## Platform Integration

- Respect platform conventions — back navigation (Android hardware/predictive back), safe areas/notches, haptics, share sheets, file open.
- **Permissions:** request just-in-time with rationale, degrade gracefully on denial, handle "permanently denied" by routing to settings. Never block the app on an optional permission.
- Test on real devices across OS versions and screen sizes — emulators hide jank, thermal throttling, and real network behavior.

## On-Device Performance & Battery

- **Startup matters most** — defer non-critical init off the cold-start path; lazy-init heavy singletons.
- Decode/resize images to display size; cache them. Large bitmaps are the #1 memory/jank cause.
- Batch and debounce network calls; coalesce sync. Each radio wake costs battery.
- Stream/paginate large lists and large API responses; never load an unbounded set into memory.
- Watch memory: dispose controllers/streams (see [flutter-expert]), avoid retaining large objects in long-lived singletons.
- Profile with real tooling (timeline, memory, network) — measure, don't guess.

## Observability

- Structured logging with levels; **redact secrets/PII**. Ship crash + error reporting so field failures are visible.
- Log sync outcomes and network failures with enough context to diagnose, without leaking sensitive payloads.

## Anti-Patterns to Reject

- Treating offline as an error state / blocking the UI on the network.
- Fire-and-forget mutations with no local queue — lost on app kill.
- Tokens/PII in plain preferences or an unencrypted local store.
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
