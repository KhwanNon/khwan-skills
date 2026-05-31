# khwan-skills

Khwan's personal [Claude Code](https://claude.com/claude-code) skills — reusable
engineering mindsets and domain experts that Claude loads automatically when a task matches.

## Install

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/KhwanNon/khwan-skills.git ~/.claude/skills
```

Already have a `~/.claude/skills` you want to keep? Clone elsewhere and symlink instead:

```bash
git clone https://github.com/KhwanNon/khwan-skills.git ~/dev/khwan-skills
ln -s ~/dev/khwan-skills/khwan-craft ~/.claude/skills/khwan-craft   # repeat per skill
```

Each skill lives in its own folder with a `SKILL.md`. Claude reads the `description` in the
frontmatter to decide when to apply it — no manual activation needed.

## Skills

| Skill | What it does |
| --- | --- |
| **khwan-craft** | Default engineering mindset and code standard for all work — *best, but simplest*. Break big problems into small obvious parts, research several approaches, synthesize, then distill. |
| **git-commit-summary** | Generates a Conventional Commits message (English) plus a Thai bullet summary from the current changes. |
| **flutter-expert** | Flutter engineering with Clean Architecture + BLoC — feature layering, GetIt DI, Dartz `Either`, Drift, go_router. Tuned to the `m_vara` stack. |
| **mobile-dev-expert** | Cross-cutting mobile concerns — offline-first & sync, secure storage, lifecycle/background work, platform integration, permissions, on-device performance. |
| **nextjs-expert** | Next.js (App Router) + React — server vs client components, data fetching & caching, routing, Tailwind, performance. Tuned for Next 15/16 + React 19. |
| **web-dev-expert** | Framework-agnostic frontend craft — semantic HTML, accessibility, Core Web Vitals, CSS architecture, TypeScript discipline, testing. |

## License

Personal use. Take what's useful.
