# CLAUDE.md — linktree-alternative

## Project Overview

**Purpose:** Customizable personal link portal built on Next.js 16, React 19, and Tailwind CSS 4 — a Linktree alternative with animations, CRT terminal effect, RSS blog integration, and dark mode.

---

## Skill References

Use gstack skills — don't do these manually:

| Task | Skill |
|---|---|
| Pre-PR code review | `/review` |
| Create PR + bump version + update CHANGELOG | `/ship` |
| Debug a bug (Iron Law: root cause first) | `/investigate` |
| Security audit (OWASP + STRIDE) | `/cso` |
| Product/architecture brainstorm | `/office-hours` |

---

## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately — don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction from the user: update `.claude/tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for the relevant project

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Zero context switching required from the user
- Fix failing CI tests without being told how

### 7. Search Before Building
Three layers of knowledge — prize Layer 3 above all:
- **Layer 1 (tried-and-true):** battle-tested patterns, official docs, stdlib
- **Layer 2 (new-and-popular):** recent community solutions, popular libraries
- **Layer 3 (first-principles):** derive the right answer from fundamentals

Before designing any solution: search `{runtime} {thing} built-in`, check official docs.

### 8. Long-Running Tasks: Don't Give Up
Poll until completion. Never stop checking and say "I'll be notified." Report progress at each cycle.

---

## Commit Discipline

Every commit must be a single logical change. Bisect before pushing:
- Rename/move separate from behavior changes
- Tests separate from feature implementations
- Refactors separate from new features

Never use `git add -A` or `git add .` — stage specific files by name only.

---

## Version & Release

- `VERSION` file at repo root (semver: `major.minor.patch`)
- `CHANGELOG.md` at repo root — written at ship time, not during development
- Each branch gets its own version bump and CHANGELOG entry
- CHANGELOG is for users: "You can now..." not "Refactored the..."
- Run `/ship` for the PR workflow — don't push/PR directly

---

## Task Management

1. **Plan First** — Write plan to `.claude/tasks/todo.md` with checkable items
2. **Verify Plan** — Check in before starting implementation
3. **Track Progress** — Mark items complete as you go
4. **Explain Changes** — High-level summary at each step
5. **Document Results** — Add review section to `.claude/tasks/todo.md`
6. **Capture Lessons** — Update `.claude/tasks/lessons.md` after corrections

---

## Core Principles

- **Simplicity First** — Make every change as simple as possible. Impact minimal code.
- **No Laziness** — Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact** — Changes should only touch what's necessary. Avoid introducing bugs.

---

<!-- KARIMO:START - Do not edit between markers -->
## KARIMO

This project uses [KARIMO](https://github.com/opensesh/KARIMO) for PRD-driven autonomous development.

### Quick Reference

- **Commands:** Type `/karimo:` to see all commands
- **Agent rules:** `.claude/plugins/karimo/KARIMO_RULES.md`
- **Configuration:** `.karimo/config.yaml`
- **Learnings:** `.karimo/learnings/`

### GitHub Configuration

| Setting | Value |
|---------|-------|
| Owner Type | organization |
| Owner | opensesh |
| Repository | linktree-alternative |
<!-- KARIMO:END -->
