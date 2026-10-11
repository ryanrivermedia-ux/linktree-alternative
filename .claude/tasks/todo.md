# Active Tasks

## KARIMO Test Run — Admin Dashboard Feature

### Status: PRD approved, ready to execute Wave 1

---

## What's been done

- [x] Bootstrapped `.claude/` directory (bootstrap-project skill)
- [x] Installed KARIMO v9.9.1 globally (`~/.claude/commands/karimo/`)
- [x] Fixed KARIMO config (`owner_type: user`, `owner: ryanrivermedia-ux`)
- [x] Ran `/karimo:research --prd admin-dashboard` — full 4-round interview completed
- [x] Ran `/karimo:plan --prd admin-dashboard` — PRD approved, 20 tasks / 58pts / 3 waves
- [x] GitHub repo live: `ryanrivermedia-ux/linktree-alternative`
- [x] `main` branch pushed (was `master` on upstream, fixed)

---

## Before Wave 1 (manual prerequisites)

- [ ] Read `.karimo/prds/001_admin-dashboard/SETUP.md` — confirms required env vars
- [ ] Create Turso DB: `turso db create linktree-alternative`
- [ ] Get `TURSO_DATABASE_URL` and `TURSO_AUTH_TOKEN` from Turso dashboard
- [ ] Create Vercel project, link to `ryanrivermedia-ux/linktree-alternative`
- [ ] Set env vars in `.env.local` and Vercel dashboard:
  - `TURSO_DATABASE_URL`
  - `TURSO_AUTH_TOKEN`
  - `SESSION_SECRET` (random 32-char string)
  - `ADMIN_PASSWORD` (bcrypt hash or plaintext — check PRD)

---

## Wave execution

- [ ] **Wave 1** — `/karimo:run --prd admin-dashboard`
  - T001: Prerequisite setup guide (manual, already done above)
  - T002: Remove static export, configure Vercel
  - T003: Extend TS types (visible + order fields)
  - T004: Turso + @libsql/client + Drizzle schema
  - T005: DB migration + seed from site.config.ts
  - T006: Auth layer — iron-session + verifySession DAL
  - T007: Middleware — protect /admin/*
  - T008: /login page + server action
  - T009–T012: Server actions for all 4 content types (CRUD)
  - Gate: `bun run build` passes, `/admin` → 401, `/login` → 200, Turso read/write works

- [ ] **Wave 2** (run after Wave 1 gate passes)
  - T013: Admin layout + shell
  - T014: Shared components (SortableList, Drawer, Toggle, DragHandle)
  - T015–T018: SocialLinks, NavItems, Resources, Tools admin sections
  - Gate: All 4 sections render, drag/reorder/toggle/drawer edits persist to DB

- [ ] **Wave 3** (run after Wave 2 gate passes)
  - T019: Live preview pane — CRT effect + phone frame + optimistic state
  - T020: Public page cutover → reads from Turso
  - Gate: Preview updates optimistically, public page reads from DB, full CRT effect

---

## KARIMO evaluation (meta-goal)

- [ ] After Wave 1: assess code quality, assess whether KARIMO earned `--karimo` flag in bootstrap skill
- [ ] Document findings in `.claude/tasks/lessons.md`

---

## Known caveats

- T011/T012 are edit/reorder/toggle only — no create/delete for Resources and Tools (seeded from site.config.ts). Verify wave 2 doesn't build Add/Delete UI for these.
- Public page stays on static config data through waves 1 and 2. Cutover happens in T020.
