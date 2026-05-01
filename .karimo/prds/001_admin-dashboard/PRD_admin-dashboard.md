# PRD: Admin Dashboard
**Slug:** admin-dashboard
**Created:** 2026-04-30
**Status:** ready
**Priority:** must

---

## Executive Summary

Build a password-gated admin dashboard at `/admin` that allows the single site owner to manage all link portal content — SocialLinks, Resources, Tools, and NavItems — with drag-to-reorder, inline visibility toggling, a slide-over drawer for editing, and a live CRT-styled preview pane that reflects changes optimistically before they are persisted. The public page continues reading from `site.config.ts` through Waves 1 and 2; Wave 3 cuts it over to Turso.

---

## Problem Statement

All site content is hardcoded in `src/config/site.config.ts`. Updating a link URL, hiding an outdated resource, or reordering social links requires a code change and a full deployment. There is no way to manage content without touching source files.

---

## Success Criteria

- **Wave 1 done:** `bun run build` passes; `/admin` returns 401 without session cookie; `/login` returns 200; a record can be written to and read from Turso.
- **Wave 2 done:** All four content sections render in admin; drag reorder, visibility toggle, and drawer edits all persist to DB.
- **Wave 3 done:** Live preview updates optimistically on every interaction; public page reads from Turso; full CRT effect renders in preview pane.

---

## Scope

### In Scope

- Remove `output: "export"` (Vercel deployment)
- Turso (libSQL) + Drizzle ORM — 4 tables with `visible` + `order` columns
- `iron-session` auth — bcrypt password gate, stateless cookie session
- `proxy.ts` route protection + `verifySession()` DAL in every server action
- `/login` page
- `/admin` route with four content sections
- Full CRUD (create, edit, delete, reorder, toggle) for **SocialLink** and **NavItem**
- Edit + toggle + reorder (no create/delete from UI) for **Resource** and **Tool**
- `@dnd-kit/sortable` for drag-to-reorder in all four sections
- Slide-over drawer for multi-field edits (all types)
- Live preview pane — full CRT/scanline effects via FaultyTerminal composition, 390px phone frame, driven by `useOptimistic` state
- Public page data cutover from `site.config.ts` → Turso (Wave 3)

### Out of Scope

- Multi-user or role-based access
- OAuth / social login
- Image/icon upload UI (icon paths remain as strings)
- Tag management UI for Tool tags
- Analytics or access logs
- Creating new Resource or Tool entries from the UI (edit-only)

---

## Architecture Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Deployment | Vercel | Remove static export; enables server actions and proxy.ts |
| Database | Turso (libSQL) + Drizzle ORM | Serverless-safe, SQLite dialect, same Drizzle API as better-sqlite3 |
| Auth | iron-session v8 | Stateless cookie, bcrypt, no DB for sessions, simple for single user |
| Route protection | proxy.ts + verifySession() DAL | Two-layer: proxy for fast redirect, DAL for actual security |
| DnD | @dnd-kit/core + @dnd-kit/sortable | React 19 compatible, headless, works with Tailwind |
| Optimistic UI | React 19 useOptimistic + useActionState | Native, no extra library |
| Live preview | Client state mirror | Same React component, shared local state — no iframe needed |
| Public page cutover | Wave 3 (parallel read) | Keeps public page stable until admin is proven |
| Default `visible` | `true` | All existing items remain visible after type migration |
| FaultyTerminal | Compose only, never modify | CRT CSS and component are read-only for this feature |

---

## Data Model

### Type Extensions (site.config.ts)

Add to all four interfaces:
```typescript
visible: boolean   // default: true
order: number      // default: array index
```

### Turso Tables (Drizzle schema)

```
social_links  — id, platform, title, handle, url, icon, visible, order
resources     — id, title, description, badge, link, button_label, visible, order
tools         — id, name, icon, url, description, tags (JSON), small_icon, visible, order
nav_items     — id, label, href, visible, order
```

---

## Security Model

1. **proxy.ts** — checks session cookie on every request to `/admin/*`, redirects to `/login` if missing/invalid. No DB calls in proxy (edge latency).
2. **verifySession()** — called as the first line of every server action. Throws if session invalid. Server actions are public POST endpoints — proxy alone is not sufficient.
3. **Env vars required:**
   - `SESSION_SECRET` — 32-char random (`openssl rand -base64 32`)
   - `ADMIN_PASSWORD_HASH` — bcrypt hash of password (`bcryptjs.hash(password, 12)`)
   - `TURSO_DATABASE_URL` — from Turso dashboard
   - `TURSO_AUTH_TOKEN` — from Turso dashboard

---

## UI Patterns

- **List row:** item title + URL/handle (truncated) + drag handle (left) + visible toggle (inline) + edit button → opens drawer
- **Drawer:** slide-over from right, form fields, Zod validation, Save / Cancel, ESC closes
- **Preview pane:** right column, phone frame (390px), `<FaultyTerminal>` + full public page components, driven by optimistic state
- **Create (SocialLink / NavItem only):** "Add" button above list, opens drawer in create mode
- **Delete (SocialLink / NavItem only):** destructive button inside drawer, with confirmation

---

## Wave Execution Plan

### Wave 1 — Foundation (no UI)
Prerequisites handled manually by user before running Wave 1.

### Wave 2 — Admin UI
Depends on Wave 1 fully merged to main.

### Wave 3 — Preview + Public Cutover
Depends on Wave 2 fully merged to main.

---

## Research Findings

### Internal
- `next.config.ts` line 5: `output: "export"` — single line to remove
- `src/config/site.config.ts` lines 12–116: all type definitions; lines 133–470: data
- No `visible` or `order` fields exist on any type — must be added
- `SubscribeModal.tsx`: reusable modal pattern (AnimatePresence + spring + ESC + scroll lock)
- `FaultyTerminal.css` + `FaultyTerminal.tsx`: CRT effect — compose only, never modify
- No API routes, no server actions, no auth anywhere in codebase

### External
- `react-beautiful-dnd` archived August 2025 — do not use
- `@dnd-kit/sortable` stable release: React 19 compatible for single-list use case
- Next.js 16 breaking change: `middleware.ts` → `proxy.ts`, function `middleware()` → `proxy()`
- `iron-session` v8: stateless AES-256 + HMAC cookie, no DB required for sessions
- `useOptimistic` auto-reverts on server action failure — safe for all mutations
- Turso `@libsql/client`: HTTP-based, serverless-safe, SQLite dialect
