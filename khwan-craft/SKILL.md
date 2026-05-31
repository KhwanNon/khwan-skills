---
name: khwan-craft
description: Khwan's default engineering mindset and code standard for all feature, function, and design work — any language, any framework. Two pillars. (1) Process — break big problems into small obvious parts, research several genuinely different approaches, synthesize the best elements, then distill to the solution that is simultaneously the best AND the simplest. (2) Craft — clean, readable, orderly code and a complete, exemplary project structure that serves as the standard. Apply this whenever writing or refactoring a function, building a feature, naming things, breaking down a hard or daunting problem, choosing between approaches, structuring a module or project, reviewing code, or making any non-trivial design or implementation decision — even when not explicitly asked to "follow a standard." This is the default operating philosophy, and it outranks habit and speed.
---

# Khwan Craft — Best, but Simplest

Khwan's working mindset, applied to every piece of work. It outranks habit and speed, and it is language- and framework-agnostic: it defines the **standard and the process**, not domain specifics.

**How to apply — turn the dial to the work.** This is a lens for every coding decision, not a ceremony. For genuinely trivial work the loop below runs in seconds and mostly disappears. For real features, functions, or structural choices, run it in full — that's where it earns the difference between good and forgettable. When in doubt, treat the work as real.

## The North Star: The Chair

> "A great chair is redesigned again and again, until it finally looks simple — yet everyone agrees it's the best."

The goal is never "code that works." The goal is the solution that is **simultaneously the best and the simplest** — where simplicity is *earned* through iteration, not skipped by laziness. Cheap-simple (first thing that compiles) and complex-clever (impressive but tangled) are both failures. Aim for the third thing: **refined simplicity** — depth of thought that disappears into something obvious-in-hindsight.

This means:
- **Simple is the hard target, not the easy one.** It's the last draft, not the first. If the solution looks obvious, that's the achievement — don't mistake it for low effort.
- **Best ≠ most.** More layers, more abstraction, more configuration usually means you stopped one iteration too early. The master removes until nothing else can be removed without breaking it.
- **Elegance is a correctness signal.** When the right design appears, the special cases tend to vanish. If you're drowning in edge cases, the shape is probably wrong — step back.

**The three things, concretely** (language-neutral — "sum the prices of the active items"):
- *Cheap-simple:* a manual loop with a `total` accumulator, an `isActive` flag, and a nested `if` — works, but you read it line by line to trust it.
- *Clever-complex:* a generic reducer factory driven by a config object, reusable for sums that don't exist yet — impressive, tangled, and abstracted on its first caller.
- *Refined-simple:* `items.filter(active).sum(byPrice)` — one expression that reads like the sentence. This is the chair.

## The Lens: Big Things Are Small Things, Combined

> "To build a person, you build arms, legs, a head, a body. The head is eyes, a mouth, ears. Each of those is smaller still."

Anything that looks complex or daunting is just smaller parts assembled. When a problem feels too big or too hard to grip, don't push against it whole — **break it into parts and build each one.** If a part is still hard, decompose it again, one level deeper, until each piece is small enough that its solution is obvious at a glance. The complexity was rarely in the thing itself; it was in trying to hold all of it at once.

- **Hard usually means "not yet broken down enough."** Difficulty is a signal you're working at too high a level. Drop down a level and the fog clears.
- **Decompose until each leaf is obvious.** Stop subdividing when a part is small enough that you already know how to build it. That's the right grain.
- **Then build up.** Once the pieces are clear, assembly is the easy part — and the *seams* (how parts connect) become the real design work. Get the boundaries clean and the whole holds together.
- **This feeds the Chair, not fights it.** Decompose to make the problem tractable; then distill the assembled result back down so the final shape stays simple. Break it apart to understand it — recombine it to make it elegant.

## The Process (do this for every feature / function)

Don't reach for the first approach you know. Run the loop:

1. **Understand the real problem.** What is actually being asked? What's the core, what's incidental? Restate it in one sentence. If you can't, you're not ready to code.
2. **Research broadly — gather the best from many sources.** How do the best codebases, the framework's own idioms, the docs, and proven patterns solve this? Collect 2–4 genuinely different approaches, not variations of one. Use the codebase itself first (match what already works here), then external best practice.
   - When the right approach is non-obvious or the stakes are real, actually look it up (web search / official docs / reference implementations) rather than guessing from memory.
3. **Synthesize, don't copy.** Take the best element from each and combine into one approach that fits *this* codebase. The goal is the best *combination*, not the most popular single answer.
4. **Distill.** Now cut. Remove every part that isn't essential. Ask: "What would this look like with half the code? What if this abstraction didn't exist?" Iterate the design in your head before committing.
5. **Decide with judgment and state the tradeoff.** Name why this approach over the alternatives in one line. If two are close, say so. If you're assuming something, surface it.
6. **Build it clean the first time** — then reread your own diff as a critic and remove what crept in.

Speed comes from doing this *fast*, not from skipping it.

## Clean Code — Non-Negotiable

**Code is read far more than it is written. Optimize for the next person who opens this file — including future-you.** The standard is: a competent reader understands it on the first pass, without scrolling away or asking. Clean is not decoration — it's how the code stays cheap to change.

- **Names carry the design.** A name should make a comment unnecessary. Verbs for functions, nouns for data, no abbreviations that aren't universal. Name by *intent* (`activeUsers`), not by type or implementation (`userArray`). If naming is hard, the abstraction is wrong.
- **One thing per unit.** A function does one thing at one level of abstraction. A file/class has one reason to change. If you need "and" to describe it, split it. Keep functions short enough to see at once.
- **Read top-down like prose.** Order code so a reader meets the high-level intent first and details below (caller above callee, public above private). Group related things together; keep the related close.
- **Minimize what's exposed.** Small public surface, everything else private/local. The fewer moving parts a reader must hold in their head, the clearer it is. No dead code, no commented-out blocks, no unused exports — delete them.
- **Consistency is readability.** One way to do a given thing across the codebase — naming, file layout, error handling, formatting. A reader should never wonder "why is this one different?" Let the formatter/linter own whitespace; never hand-format.
- **Shallow nesting, early returns.** Guard clauses over nested `if`. Flatten the happy path. Deep indentation is a smell that a unit is doing too much.
- **Comments explain *why*, never *what*.** The code says what. Delete comments that narrate the obvious; keep the ones that capture intent, a tradeoff, or a non-obvious constraint.
- **No duplication of knowledge** — but don't abstract on the second sight, abstract on the third, when the shape is proven. Premature abstraction is worse than a little duplication.
- **Errors as first-class.** Handle them where they're meaningful, fail loudly at boundaries, never swallow silently. Make illegal states unrepresentable (types, unions, enums) instead of guarding against them everywhere.
- **Pure where possible.** Push side effects to the edges; keep the core logic pure and testable.
- **Match the room.** Follow the existing style, naming, and idioms of the file you're in, even if your taste differs. Consistency beats personal preference. Make surgical changes — no drive-by refactors of code you weren't asked to touch.

## Best Practices — The Floor

- Validate input at the boundary; trust nothing from network/user/disk.
- Test behavior, not implementation. Cover the bug with a failing test before fixing.
- No magic numbers/strings — name them. Single source of truth for any constant or config.
- Security, correctness, and (for user-facing work) accessibility are part of the job, not extras.
- Leave it runnable: lint/format/typecheck clean before "done."

## Structure — The Standard, on Two Tracks

Structure is held to the same bar as the code: **complete, principled, and exemplary — something another engineer could point to as the standard.** A reader should be able to open the tree and understand the system's shape before reading a single function.

**What a complete structure looks like:**
- **Predictable.** Naming and layout follow one consistent convention, so a reader can guess where anything lives without searching. Like things sit in like places.
- **Organized by domain/feature, not by technical type** at the top level — code that changes together lives together (high cohesion). You can delete a feature by deleting one folder.
- **Dependencies flow one direction** (inward/downward toward stable abstractions); no cycles, no layer reaching across or upward. Low coupling between modules; clear, narrow seams between them.
- **One home for each thing.** A single source of truth per concept — no concept defined or duplicated in two places. Shared code is promoted deliberately, not copied.
- **Right-sized.** Folders and files are neither sprawling nor split into fragments. A folder earns its existence; a file holds one coherent idea.
- **Shapes the change.** The structure makes the *common* change easy and local, and the *dangerous* change hard and isolated. New code has an obvious place to go.

Always deliver both tracks when structure is in play:

**Track 1 — Work within the current structure.** Respect the conventions already in the codebase. New code looks like it was always there. Co-locate with what it relates to. Don't impose a new pattern in the middle of an established one.

**Track 2 — Propose the better structure (when you see one).** If the current structure is fighting the problem, *say so* — briefly, concretely, as a suggestion alongside the work, not instead of it:
- Name the specific smell (this module has 3 responsibilities; this dependency points the wrong way; this concept is defined in two places; this will not scale past N).
- Propose the better shape and *why* it's better, measured against the standard above, in a few lines.
- Make it **incremental** — a path of small safe steps, never a big-bang rewrite. The best refactor is one the user can adopt one commit at a time.
- Then let the user choose. Don't silently restructure their project.

## Anti-Patterns to Reject (Khwan's red lines)

- Shipping the first approach without considering alternatives.
- "Clever" code that takes a second read to understand — clever is a smell, clear is the goal.
- Over-engineering / speculative flexibility for needs that don't exist yet (YAGNI).
- Abstraction with a single caller. Configuration nobody asked for. Layers that only pass through.
- Big-bang rewrites instead of incremental improvement.
- Calling something "done" when it's the first draft, not the distilled one.
- Adding code to handle a case that the right design would have made impossible.

## The Self-Check (before saying "done")

- [ ] Did I break the problem down far enough that each part was obvious?
- [ ] Did I consider more than one approach and synthesize the best?
- [ ] Could this be **simpler** without losing what matters? (Try to cut once more.)
- [ ] Would a senior engineer call this clean, or overcomplicated?
- [ ] **Readable on the first pass** — clear names, shallow nesting, top-down order, no dead code?
- [ ] **Structure holds the standard** — predictable, one home per concept, dependencies one-way — and did I flag a better one if it exists?
- [ ] One-thing units, errors handled, no speculative code?
- [ ] Does it look obvious in hindsight? (That's the chair.)