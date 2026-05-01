# Research Summary — Admin Dashboard

_Feature: Admin dashboard for managing link portal content (edit, toggle visibility, reorder, live preview)_
_Researched: 2026-04-30_

---

## TL;DR

One architectural change unblocks everything. After that, three new layers needed: auth, DB, and UI components. The codebase has strong animation and styling patterns to build on.

---

## Architecture Change Required

**Remove `output: "export"` from `next.config.ts`.** This unlocks server actions, API routes, and proxy.ts. Current static export blocks all server-side functionality.

New deployment target: Vercel (simplest) or self-hosted Node.

---

## Recommended Stack

| Concern | Recommendation | Notes |
|---|---|---|
| Remove static export | `next.config.ts` edit | One-line change, unlocks everything |
| Auth | `iron-session` | Stateless cookie, bcrypt password in env, no DB for sessions |
| Route protection | `proxy.ts` | Renamed from `middleware.ts` in Next.js 16 |
| Drag-and-drop | `@dnd-kit/core` + `@dnd-kit/sortable` | React 19 compatible; headless, works with Tailwind |
| Optimistic UI | React 19 `useOptimistic` + `useActionState` | Native, no extra library |
| Mutations | Next.js Server Actions (`'use server'`) | Stable API in v16 |
| Persistence (Vercel) | Turso (libSQL) + Drizzle ORM | SQL, serverless-safe, SQLite dialect |
| Persistence (self-hosted) | `better-sqlite3` + Drizzle ORM | Fast, zero-config, same Drizzle API |
| Form validation | `zod` | Schema validation for server actions |
| Live preview | Client state mirror | Same React component, shared local state — no iframe |
| Inline editing UX | Drawer for multi-field; inline toggle for visibility | Drawer = better keyboard + validation UX |

---

## Data Model Changes Needed

Add `visible: boolean` to existing types in `site.config.ts`:
- `SocialLink`
- `Resource`
- `Tool`
- `NavItem`

Add `order: number` for sort persistence if using DB.

---

## What Exists (Reuse)

- **Modal shell pattern** — `SubscribeModal.tsx` (AnimatePresence + spring + ESC key + scroll lock)
- **Card grid pattern** — `FreeResources.tsx`, `CardNav.tsx`
- **Full design token system** — `theme.css` has semantic tokens, dark mode, brand colors
- **Tailwind 4 custom tokens** — `globals.css` `@theme` block maps CSS vars
- **Animation libraries** — GSAP (timelines) + Framer Motion (AnimatePresence) both already installed

---

## What Needs to Be Built

1. **Auth layer** — `proxy.ts`, login page, session cookie, `verifySession()` DAL
2. **DB schema** — links, resources, tools, nav tables with `visible` + `order` columns
3. **Server actions** — create, update, delete, reorder, toggle for each content type
4. **Admin route** — `/admin` (protected)
5. **Admin UI components:**
   - Sortable list with drag handles (`@dnd-kit/sortable`)
   - Toggle switch (inline on row)
   - Slide-over drawer (multi-field edit form)
   - Live preview pane (phone-frame mockup, 390px width)
6. **Login page** — `/login` with password form

---

## Open Questions for Planning

1. **Deployment target** — Vercel or self-hosted? Determines DB choice (Turso vs better-sqlite3).
2. **Auth complexity** — Simple bcrypt password gate (iron-session) or fuller auth (better-auth)?
3. **Preview fidelity** — Full CRT/animation effects in preview, or simplified mockup?
4. **Content scope** — Manage all 4 content types (links, resources, tools, nav), or start with links only?

---

## Next Step

```
/karimo:plan --prd admin-dashboard
```
