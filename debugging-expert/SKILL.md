---
name: debugging-expert
description: Debugging mindset for any bug, test failure, crash, regression, wrong output, flaky test, or "works on my machine" mystery — any language, any framework. The iron rule — find the root cause before touching a fix. Reproduce it reliably, trace the bad value backward through the call chain to its origin, prove the cause with one minimal change, fix at the source (never the symptom), then add validation at each layer so the bug becomes impossible. Apply whenever investigating why something fails, fixing a failing or flaky test, chasing a crash or incorrect result, triaging a production incident, or whenever tempted to patch a symptom under time pressure.
---

# Debugging Expert — Find the Cause, Fix It Once

The mindset for turning "it's broken" into "it's fixed and can't break that way again." This skill is the *process* of debugging — language- and framework-agnostic.

## The North Star: The Diagnosis

> "A doctor who treats the symptom loses the patient. A doctor who finds the disease cures them — and the symptom disappears on its own."

A symptom fix is a guess wearing a lab coat. The goal is never "the error went away" — it's **understanding *why* it happened well enough that the fix is obvious and the bug can't return.** When you truly find the root cause, the fix is usually small, the special cases vanish, and you can explain the whole story in two sentences. If your fix is large, scattered, or you can't explain why it works — you found a symptom, not the disease.

## What a Debugger Values

The lens to read everything below through. What separates expert debugging from guess-and-check:

- **Understanding over speed.** Systematic debugging is *faster* than thrashing — 15 minutes of investigation beats 3 hours of random patches. The guess that "might work" is the slow path.
- **Evidence over intuition.** "It's probably X" is a hypothesis to test, not a fact to act on. The expert proves where the failure is before changing anything.
- **The source over the symptom.** The error surfaces where the bad value is *used*; the bug lives where the bad value was *born*. Trace back to the origin and fix it there.
- **One change at a time.** Multiple simultaneous fixes mean you can't tell what worked — and you've likely added new bugs. Change one variable, observe, decide.
- **The bug must die for good.** A fix that only stops *this* reproduction is half-done. Make the illegal state structurally impossible so no future code path can resurrect it.
- **Reproduce first, always.** A bug you can't trigger on demand is a bug you can't prove you fixed.

**The bar for "expert-level work":** the failure is reproduced reliably, the root cause is traced to its origin and explained in plain language, a failing test captures it, the fix targets the source, the test now passes with nothing else broken, and validation makes the bug impossible to reintroduce.

## The Iron Law

```
NO FIX WITHOUT ROOT CAUSE FIRST
```

If you can't yet explain *why* it fails, you are not ready to propose a fix — you're ready to investigate. Violating the letter of this is violating the spirit of debugging. This rule matters *most* exactly when it's tempting to skip: under time pressure, when the fix "seems obvious," and after a fix already failed.

## The Loop (every bug, turn the dial to its size)

A trivial typo runs this in seconds; a production incident runs it in full. When in doubt, treat it as real.

### 1. Reproduce & Read
- **Read the error completely** — the message, the full stack trace, line numbers, codes. It often *names* the cause. Don't skim past it.
- **Make it deterministic.** What exact steps trigger it? Every time, or sometimes? A bug you can't reproduce is data you haven't gathered yet — don't guess, instrument.
- **Check what changed.** Recent commits, new deps, config/env differences. `git diff`, `git bisect`, blame. Most bugs ride in on a recent change.

### 2. Trace to the source (don't fix where it hurts)
The error appears deep in the stack; the cause is upstream. Walk backward:
- Where is the bad value *used* (the symptom)? → What passed it in? → Keep asking "what called this?" up the chain → Where was the bad value *born* (the source)?
- When you can't trace by reading, **instrument**: log the value, the relevant state, and `new Error().stack` *before* the dangerous operation, not after it fails. In tests, print to stderr (loggers may be suppressed).
- For multi-component systems (API → service → DB, CI → build → sign): log what enters and exits **each boundary** once, to find *which* layer breaks — then investigate only that one.

### 3. Hypothesize & test minimally
- State it out loud: **"I think X is the root cause because Y."** Specific, not vague.
- Make the **smallest possible change** to confirm or kill the hypothesis — one variable.
- Worked? → Phase 4. Didn't? → form a *new* hypothesis. **Never stack another fix on top of an unconfirmed one.**
- Don't know? Say "I don't understand X" and dig — don't pretend.

### 4. Fix at the source & fortify
- **Write the failing test first** — the simplest reproduction. It proves you understood the bug and proves the fix later. An untested fix is an assertion, not a result.
- **One fix, at the root.** No "while I'm here" refactors bundled in. Address the origin you found in Phase 2, not the symptom in Phase 1.
- **Verify honestly:** the new test passes, and *nothing else broke*. Run the suite.
- **Make it impossible (defense-in-depth).** A single check gets bypassed by other paths, refactors, or mocks. Validate at every layer the bad data crosses: reject it at the entry boundary, guard the business logic, fence dangerous operations by context, and leave a breadcrumb log. Prefer making the illegal state unrepresentable (types, enums, getters that throw) over checking for it everywhere.

## When You've Tried 3+ Fixes — Stop

Three failed fixes is not a fourth-fix problem; it's a signal. If each fix reveals a new problem somewhere else, or each needs "massive refactoring," you're not facing a bug — you're facing a **wrong design**. Drowning in edge cases means the shape is wrong. Stop patching. Step back and question whether the architecture itself is sound, and raise it with the user before attempting fix #4.

## Red Flags — Stop and Return to the Loop

Catch yourself thinking any of these and you've left the process:
- "Quick fix now, investigate later" / "let me just try changing X"
- "It's probably X" — proposing a fix before tracing the data
- "I'll add the test after I confirm the fix works"
- "Let me change these few things and run it" (multiple variables at once)
- "The reference is long, I'll adapt the pattern" — partial understanding guarantees bugs
- "One more fix attempt" (after 2+ failures) — question the architecture instead

## Common Rationalizations vs. Reality

| Excuse | Reality |
| --- | --- |
| "Too simple to need a process" | Simple bugs have root causes too — and the process is fast for them. |
| "Emergency, no time" | Systematic is *faster* than guess-and-check. Rushing guarantees rework. |
| "I'll just try this first" | The first fix sets the pattern. Do it right from the start. |
| "I can see the problem" | Seeing the symptom ≠ understanding the cause. |
| "Multiple fixes save time" | You can't isolate what worked, and you add new bugs. |
| "It went away, good enough" | Gone from *this* path. Prove the cause, or it returns. |

## Anti-Patterns to Reject

- Fixing where the error surfaced instead of where the bad value originated.
- Changing several things at once, then not knowing which one mattered.
- Declaring victory on "the error stopped" without explaining *why*.
- Patching a symptom you don't understand because the deadline is close.
- Skipping the failing test — an untested fix doesn't stick.
- Swallowing the error (empty catch, broadened type) to silence it.
- A single validation point that the next code path or mock walks right past.

## Before Saying "Done"

- [ ] The failure was reproduced reliably (or instrumented when not reproducible).
- [ ] The root cause is traced to its origin and explainable in plain language.
- [ ] A failing test captured the bug *before* the fix.
- [ ] The fix targets the source, not the symptom — one focused change.
- [ ] Full suite green: the bug's test passes and nothing else broke.
- [ ] Validation added so the illegal state can't recur (defense-in-depth / unrepresentable).
- [ ] The fix is clean, minimal, and obvious in hindsight — no symptom-masking left behind.
