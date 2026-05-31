---
name: git-commit-summary
description: Generate a Conventional Commits message (English) and a Thai bullet-point change summary from current file changes. Use when the user asks to commit, summarize a branch, describe what was done, or prepare a message for chat/PR.
---

# Git Commit & Summary

Two outputs derived from the same git diff:

1. **Commit message** — English, Conventional Commits, ready for `git commit`
2. **Change summary** — Thai, plain bullet list, optimized for copy-paste into Line/Slack/PR

## Invocation Modes

| User intent | Output |
|------------|--------|
| "commit", "ช่วย commit", "เขียน commit ให้" | Mode 1 only |
| "สรุป", "สรุปการเปลี่ยนแปลง", "ทำอะไรไปบ้าง" | Mode 2 only |
| "commit + สรุป", "เตรียม PR", or unclear | Both, labeled clearly |

If arg passed: `commit` → Mode 1, `summary` → Mode 2, otherwise both.

## Inputs to Inspect

Run these first to know what changed:

| Need | Command |
|------|---------|
| Files changed (working tree) | `git status` |
| Unstaged diff | `git diff` |
| Staged diff | `git diff --staged` |
| Branch-level diff vs base | `git diff <base>...HEAD` |
| Commits on branch | `git log <base>..HEAD --oneline` |

Base branch: check `git symbolic-ref refs/remotes/origin/HEAD` or default to `main` / `master` / `dev` per the project.

---

## Mode 1 — Commit Message (English)

### Format

```
[type]([scope]): [short description]
```

### Allowed types

`feature`, `update`, `fix`, `refactor`, `docs`, `test`, `chore`, `style`, `perf`

- `feature` — wholly new functionality
- `update` — enhancement to existing functionality
- `fix` — bug fix
- `refactor` — code restructure, no behavior change
- `docs` — documentation only
- `test` — adding/fixing tests only
- `chore` — tooling, deps, config
- `style` — formatting, whitespace
- `perf` — performance improvement

### Rules

- **Scope** is optional but encouraged. Pick from affected module: `login`, `api`, `core`, `ui`, `auth`, `<feature-name>`, etc.
- **Subject** ≤ 50 chars, lowercase, imperative present tense ("add" not "added" or "adds"), no trailing period.
- One commit = one intent. If the diff spans multiple unrelated intents, recommend splitting instead of forcing one message.
- For non-trivial changes, add a body explaining **the why**, not the what. Separate with a blank line.
- Never include ticket IDs in the subject — put them in the body if needed.

### Examples (from project history)

```
fix(api): handle 401 response and refresh token
feature(ui): add dark mode toggle
refactor(core): move widgets to shared module
docs(readme): update installation instructions
```

### Process

1. Read `git status` + `git diff --staged` (or full diff if nothing staged).
2. Identify dominant intent. When multiple types apply, pick by priority: `fix` > `feature` > `refactor` > `update` > others.
3. Pick scope from the most-touched feature/module.
4. Write the subject. Verify ≤ 50 chars.
5. If diff > ~5 files or touches multiple concerns, draft a body.

### Output format

Wrap in a fenced block so the user can copy directly:

````
```
feature(ui): add dark mode toggle
```
````

---

## Mode 2 — Thai Change Summary

For pasting into Line/Slack/PR description. Optimized for copy-paste cleanliness.

### Format

```
- [การเปลี่ยนแปลงข้อที่ 1]
- [การเปลี่ยนแปลงข้อที่ 2]
- [การเปลี่ยนแปลงข้อที่ 3]
```

### Rules

- ทุกบรรทัดเริ่มด้วย `-` (dash) ตามด้วย space เดียว
- ภาษาไทย กระชับ ใช้คำกริยานำ ("เพิ่ม...", "แก้ไข...", "ปรับปรุง...", "ลบ...", "ย้าย...", "รีแฟคเตอร์...")
- **1 ข้อ = 1 logical change** ไม่ใช่ 1 ไฟล์ — ไฟล์หลายไฟล์ที่ทำเรื่องเดียวกันรวมเป็นข้อเดียว
- เรียงจากสำคัญที่สุดไปน้อยสุด (feature > fix > refactor > chore)
- เขียนให้คนที่ไม่เห็นโค้ดเข้าใจได้ — อย่าใช้ชื่อ class/function ที่คนอื่นไม่รู้
- **ห้ามใส่ markdown formatting อื่นเด็ดขาด** เช่น `**bold**`, `` `code` ``, headers, nested bullets — เพราะ Line/Slack render ไม่ครบและจะเสียตอนก้อปวาง
- ห้ามมี header, blank line คั่นกลาง, หรือบรรทัดอื่นนอกจาก bullet
- ห้ามใส่ emoji เว้นแต่ user ขอ

### Process

1. อ่าน diff ทั้งหมด (working tree + staged หรือ branch diff)
2. จัดกลุ่ม changes ตาม **logical concern**:
   - ไฟล์หลายไฟล์ที่ทำเรื่องเดียวกัน → รวมเป็น 1 ข้อ
   - ไฟล์เดียวที่ทำหลายเรื่อง → แยกเป็นหลายข้อ
3. เขียนแต่ละข้อให้คนอ่านเข้าใจ "ทำอะไร" และ "ผลคืออะไร"
4. เรียงตามความสำคัญ
5. ตรวจสุดท้าย: ก้อปวางใน plain text แล้วยังอ่านรู้เรื่องไหม

### Example output

````
```
- เพิ่มฟีเจอร์ dark mode พร้อม toggle ใน settings
- แก้ bug ที่ token หมดอายุแล้วไม่ refresh อัตโนมัติ
- ย้าย shared widgets ไปไว้ใน core module
- อัพเดท README ขั้นตอนการติดตั้ง
```
````

---

## Combined Output (both modes)

เมื่อ user ขอทั้งสอง ให้ output แบบนี้ ครอบ code block แยกกัน:

````
**Commit message:**

```
feature(ui): add dark mode toggle
```

**สรุป (ภาษาไทย):**

```
- เพิ่มฟีเจอร์ dark mode พร้อม toggle ใน settings
- ปรับ ThemeProvider ให้รองรับการสลับ theme runtime
- อัพเดท default theme color ใน design tokens
```
````

---

## Anti-patterns to Reject

**Commit message:**
- Subject > 50 chars
- Past tense ("added", "fixed") or third person ("adds", "fixes")
- Multiple intents jammed together with " and "
- Ticket IDs in the subject (`fix(api): JIRA-123 handle 401`)
- Trailing period
- Combining unrelated changes — recommend splitting instead

**Thai summary:**
- Bullet per file แทนที่จะเป็น bullet per logical change
- Markdown formatting (`**`, `` ` ``, `#`, nested `-`)
- ใช้ชื่อ technical (class/function) ที่ผู้อ่านไม่รู้จัก
- Header หรือ intro line ก่อน bullet
- เขียนเป็น paragraph แทนที่จะเป็น bullet
- ผสมไทย-อังกฤษเกินจำเป็น (ใช้อังกฤษเฉพาะ proper noun เช่นชื่อ feature/library)

## Edge Cases

- **No changes**: บอก user ว่าไม่มีการเปลี่ยนแปลง อย่าสร้าง commit เปล่า
- **Only whitespace/formatting**: ใช้ type `style`
- **Multiple unrelated intents in diff**: เสนอให้แยก commit แทนที่จะรวม — list groups และให้ user เลือก
- **Branch-level summary** (หลาย commits): อ่าน `git log` ด้วย ไม่ใช่แค่ diff เดียว เพื่อให้ summary ครอบคลุมงานทั้ง branch