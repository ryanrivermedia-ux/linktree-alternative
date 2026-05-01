# Internal Research Findings — Admin Dashboard

_Researched: 2026-04-30. Stack: Next.js 16, React 19, Tailwind CSS 4, TypeScript 5._

---

## Executive Summary

The codebase is a **static export Next.js app** deployed to GitHub Pages. Building an admin dashboard requires one architectural change (remove `output: "export"`) and several new layers (auth, DB, API). The existing component patterns are strong and reusable.

---

## Key Findings

### Architecture Constraint (BLOCKER)

`next.config.ts` line 5: `output: "export"` — must be removed.

This single change unblocks: API routes, server actions, proxy.ts, database access.

### Data Layer

All content lives in `src/config/site.config.ts` as hardcoded TypeScript. Key types:

| Type | Fields | Count |
|---|---|---|
| `NavItem` | id, label, href | 3 |
| `SocialLink` | id, platform, title, handle, url, icon | 5 |
| `Resource` | id, title, description, badge, link, buttonLabel | 3 |
| `Tool` | id, name, icon, url, description, tags, smallIcon? | 16 |

**No `visible: boolean` field exists on any type** — must be added for visibility toggle feature.

### Reusable Components

| Component | Reusable Pattern |
|---|---|
| `SubscribeModal.tsx` | Modal shell (AnimatePresence + spring + ESC + scroll lock) |
| `FreeResources.tsx` | Card grid with badge, modal trigger |
| `CardNav.tsx` | GSAP expand/collapse, grid layout |
| `OurLinks.tsx` | `useSyncExternalStore` for responsive switching |

### Animation Stack
- **GSAP** — timelines, stagger, `power2/3.out` easing
- **Framer Motion** — `AnimatePresence`, spring `{damping:25, stiffness:300}`
- Both available for admin UI animation

### State Management
- All local `useState` — no global store
- `localStorage` for persistence (resource access, subscribe state)
- No Context API, no Zustand, no Redux

### Missing Entirely
- Auth (no sessions, no middleware, no protected routes)
- API routes (none exist)
- Drag-and-drop library
- Form component library
- Database client
- Visibility flags on content types
- Image/icon upload handling

---

## Files to Reference

- `src/config/site.config.ts` — types (lines 12-116), data (lines 133-470)
- `src/components/SubscribeModal.tsx` — modal pattern to replicate
- `src/app/layout.tsx` — root layout, dark mode class
- `next.config.ts` — static export config to modify
- `src/lib/brand-styles/theme.css` — full token system
- `src/styles/globals.css` — Tailwind 4 `@theme` block, responsive vars
