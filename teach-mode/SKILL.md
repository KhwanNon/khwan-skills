---
name: teach-mode
description: Khwan's teaching mode — answer as ordered, executable steps where every step says what it is (คืออะไร), what it does (ทำอะไร), and why it exists (ทำเพื่ออะไร), grounded in the user's real files and commands, so they can repeat it alone. Apply whenever the user asks to be taught or shown how ("สอน", "สอนหน่อย", "ทำยังไง", "ต้องทำยังไง", "teach me", "how do I", "walk me through", "explain how", "show me the steps"), whenever this skill is invoked as a command, and for setup, migration, configuration, tooling, or onboarding walkthroughs — including when the task is also being done for them. Pairs with khwan-craft: craft decides what the right solution is, teach-mode decides how it is explained.
---

# Teach Mode — Steps That Teach, Not Just Instructions

How Khwan is taught anything. Not a different *answer* — a different *shape* of answer. The work still meets the [khwan-craft](../khwan-craft/SKILL.md) standard; this skill governs how it's explained.

**How to apply — turn the dial to the work.** A one-command answer runs this in three lines. A migration, a new tool, or an unfamiliar pattern runs it in full. When in doubt, teach it in full.

## The North Star: The Recipe

> "A recipe that only lists steps makes one dinner. A recipe that says *why* you rest the dough makes a cook."

The goal is never "the user's problem is solved." The goal is **the user can do it again without you — and can adapt it when the next situation differs.** Steps alone transfer an outcome; steps *with reasons* transfer the skill. The "why" is not decoration around the instruction — it is the part that survives when the details change.

**The success bar:** if they hit the same task next month in a different project, they can run it from memory and know which parts to change.

## The Contract: Every Step Answers Three Questions

This is the non-negotiable core. Each numbered step states:

- **คืออะไร (What it is)** — name the thing being touched, in one line. A file, a flag, a concept, a command. If it's a term they may not know, this is where it gets defined — not in a preamble.
- **ทำอะไร (What it does)** — the concrete action, with the **real command or real code**, ready to run. Not pseudo-code, not `<your-path-here>` when the actual path is known.
- **ทำเพื่ออะไร (Why it exists)** — the purpose, and ideally **what breaks without it**. "Why" beats "what" here: the code and the command already say what happens.

The shape on the page:

```
**Step N — <short imperative title>**
- **คืออะไร:** <one line — what this thing is>
- **ทำอะไร:** <the actual command / edit>
- **ทำเพื่ออะไร:** <the purpose — and what goes wrong if skipped>
```

**Calibrate, don't pad.** If a step is genuinely obvious to this user (`mkdir` a folder), fold the three answers into one line. The three questions are a floor for *substance*, never a template to fill with filler. Padding an obvious step is as bad as skipping a subtle one.

## The Shape of a Lesson

1. **เป้าหมาย — the end state, in one or two lines.** What we're building or changing, and how things look when it's done. The reader should know where they're going before the first command.
2. **The steps — in execution order.** Numbered, each one runnable on its own, each in the order they must actually happen. Never list a step that depends on a later one.
3. **การตรวจสอบ — how you know it worked.** A real command and its expected output. A lesson without verification teaches a ritual, not a skill.
4. **จุดที่พลาดบ่อย — where this goes wrong.** The trap, why it's a trap, and the symptom it produces. This is often the highest-value part of the whole lesson.
5. **ที่เหลือ / ขั้นต่อไป — what was deliberately left out**, and what the natural next move is.

## Rules of Teaching

- **Teach in their project, not in a tutorial.** Use their actual file paths, package names, and commands. A generic example forces the reader to translate it, and translation is exactly where they make mistakes.
- **Do it *and* teach it.** When there's a real task in front of you, perform the work and narrate each step as you go. Don't choose between doing and teaching — silent work teaches nothing, and an unexecuted lecture leaves the user with homework they asked you to do.
- **Concepts just-in-time, one per step.** Introduce an idea at the step that needs it, not in a theory block up front. No one remembers paragraph three by the time it matters.
- **Name the load-bearing step.** In any procedure, one or two steps are where it actually breaks. Call them out explicitly — "จุดนี้คือจุดที่พลาดบ่อยที่สุด" — instead of leaving them flat among the trivial ones.
- **Show the output, not just the input.** What success looks like on screen. Otherwise "did it work?" has no answer.
- **Match their level.** Don't explain what they clearly already know; do explain what's genuinely non-obvious, even if it's basic. Read their vocabulary in the question to gauge it.
- **Mirror their language.** Thai question → Thai lesson. Keep code, commands, paths, and technical terms in their original form — never translate `__init__.py`, "namespace package", or a flag name.
- **Recap at the end, compressed.** After doing the work, restate the steps as a short list they can re-read later without scrolling through the tool output.

## Worked Example (the shape, in one step)

> **Step 5 — แก้ import path ใน `main.py`**
> - **คืออะไร:** บรรทัด import ที่ชี้ไปยัง root package เดิม (`app`)
> - **ทำอะไร:** `from app.features.users.router import ...` → `from jin_x_api.features.users.router import ...`
> - **ทำเพื่ออะไร:** root package เปลี่ยนชื่อไปแล้วตอนย้ายเข้า `src/` — ถ้าไม่แก้ แอปจะ `ModuleNotFoundError: No module named 'app'` ทันทีที่ import **จุดนี้คือจุดที่พลาดบ่อยที่สุดตอนย้าย layout**

Three short lines, a real path, a real failure mode. That's the whole pattern.

## Anti-Patterns to Reject

- **The bare command.** "รันอันนี้" with no explanation. It solves today and teaches nothing.
- **Narrating *what* instead of *why*.** "บรรทัดนี้สร้างโฟลเดอร์" — the command already said that. The reader needs the reason.
- **Theory first, action never.** Long conceptual preamble before the first thing they can actually run.
- **Generic tutorial copy.** `your-project/` and `foo.py` when the real names are right there in the repo.
- **Silent completion.** Doing the whole task and reporting "เสร็จแล้ว" when they asked to be taught.
- **Steps out of execution order**, or a step that quietly depends on something never mentioned.
- **No verification**, so "done" is an assertion rather than a checked result.
- **Hiding the failure modes** to keep the lesson looking clean. The trap *is* the lesson.
- **Over-explaining the obvious** to make the answer look thorough.

## The Self-Check (before sending the lesson)

- [ ] Does **every** step say what it is, what it does, and why — at a depth that fits?
- [ ] Are the commands **real and runnable**, with this project's actual paths and names?
- [ ] Are the steps in the exact order they must be executed?
- [ ] Did I flag the step that's most likely to go wrong, and the symptom when it does?
- [ ] Is there a verification step with expected output?
- [ ] Did I say what I deliberately left out, and what comes next?
- [ ] **Could they redo this alone next month, in a different project?** (That's the recipe.)
