# Project Memory

## Stack

- Next.js 16 + React 19 + TypeScript + Tailwind CSS 4
- Package manager: Bun
- DB: Turso (libSQL edge) + Drizzle ORM
- Auth: iron-session (stateless cookie, bcrypt)
- Drag-and-drop: @dnd-kit/sortable (react-beautiful-dnd archived Aug 2025)
- Optimistic UI: React 19 `useOptimistic` + `useActionState`
- Deploy target: Vercel (dynamic, NOT static export)

## KARIMO setup

- Version: v9.9.1
- Config: `.karimo/config.yaml`
- Commands live globally: `~/.claude/commands/karimo/`
- PRD in progress: `.karimo/prds/001_admin-dashboard/`
- `owner_type: user`, `owner: ryanrivermedia-ux`
- `review.enabled: false` (manual review mode)

## Admin dashboard feature decisions

- 4 content types: SocialLinks, NavItems, Resources, Tools
- Resources and Tools: edit/reorder/toggle only (no add/delete — seeded from site.config.ts)
- SocialLinks and NavItems: full CRUD
- CRT effect in live preview pane (wave 3)
- Auth: single admin user, bcrypt password, iron-session cookie

## Lessons

- KARIMO commands must be in `~/.claude/commands/karimo/` globally to be discoverable in terminal sessions. Project-level `.claude/commands/karimo/` does NOT work as of Claude Code v2.1.105.
- `$schema` URL in Claude Code settings files must end with `.json`: `https://json.schemastore.org/claude-code-settings.json`
- KARIMO `/configure` auto-detects GitHub owner from git remote — verify it wrote the correct owner (not upstream) before running `/karimo:plan`.
- When pushing to GitHub from a cloned repo, original branch may be `master` not `main`. Create `main` explicitly: `git checkout -b main && git push -u origin main`.
