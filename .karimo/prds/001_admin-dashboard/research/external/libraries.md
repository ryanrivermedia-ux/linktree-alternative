# External Libraries Research — Admin Dashboard

_Researched: 2026-04-30. Stack: Next.js 16, React 19, Tailwind CSS 4, TypeScript 5._

---

## 1. Drag-and-Drop Libraries

### @dnd-kit/core + @dnd-kit/sortable — RECOMMENDED

| Property | Value |
|---|---|
| Package | `@dnd-kit/core`, `@dnd-kit/sortable`, `@dnd-kit/utilities` |
| Sortable version | `@dnd-kit/sortable@10.0.0` |
| React peer dep | `>=16.8.0` (React 19 works; v0.4.0 of the new `@dnd-kit/react` package fixes `useRef` readonly errors for React 19 type compat) |
| Bundle size | ~25 kB core + sortable combined |
| Maintained | Yes — active, last release April 2026 |
| Docs | https://dndkit.com |

**Architecture:** Headless, composable primitives. Three layers:
- `DndContext` — root drag context, sensor management
- `SortableContext` — supplies sorted IDs via context
- `useSortable` hook — combines draggable + droppable; returns `{attributes, listeners, setNodeRef, transform, transition, isDragging}`

**Key implementation pattern for a sortable link list:**
```tsx
// 1. Wrap list in DndContext + SortableContext
// 2. Each item uses useSortable hook
// 3. onDragEnd: use arrayMove() utility to reorder state
// 4. DragOverlay (optional) for a floating drag preview
```

**Sensors:** PointerSensor (mouse/touch) + KeyboardSensor (a11y). Custom sensor configs for drag threshold.

**Strategies:** `verticalListSortingStrategy` for a single vertical list of links.

**Known React 19 issues (open GitHub issues):**
- `removeChild` errors in some nested droppable scenarios (#1940, #1747)
- These appear in edge cases with multiple lists; single-list sortable (the admin use case) is unaffected in practice
- The new `@dnd-kit/react@0.4.0` package (rewrite) addresses React 19 type compatibility

**Note on two codebases:** dnd-kit has two parallel codebases:
- `@dnd-kit/core` (v6.x, stable) — the well-documented, widely adopted version
- `@dnd-kit/react@0.4.0` (new rewrite, alpha-quality) — framework-specific rebuild; do NOT use for production yet

**Recommendation:** Use `@dnd-kit/core` + `@dnd-kit/sortable` (stable v6/v10 line).

---

### @hello-pangea/dnd — VIABLE ALTERNATIVE

| Property | Value |
|---|---|
| Package | `@hello-pangea/dnd` |
| Latest version | 18.0.1 (February 2025) |
| React peer dep | `"^18.0.0 \|\| ^19.0.0"` — explicitly supports React 19 |
| Origin | Community-maintained fork of `react-beautiful-dnd` (Atlassian archived original Aug 2025) |
| Maintained | Yes, active (Gabriel Santerre, Reece Carolan) |
| Docs | https://github.com/hello-pangea/dnd |

**Strengths:** Drop-in replacement for react-beautiful-dnd, excellent a11y (keyboard + screen reader), virtual list support (10k items @ 60fps), familiar API for teams coming from react-beautiful-dnd.

**Weaknesses:** Larger API surface than dnd-kit, more opinionated about layout (flex/grid assumptions), slightly heavier than dnd-kit.

**React 19 note:** Package.json explicitly declares `"^18.0.0 || ^19.0.0"` and ships separate test suites for both versions.

---

### react-beautiful-dnd — DO NOT USE

| Property | Value |
|---|---|
| Status | DEPRECATED — archived August 2025, read-only |
| Last version | 13.1.1 (August 2022) |
| React 19 | Incompatible — unmaintained |

Atlassian recommends migrating to Pragmatic drag-and-drop. Do not start new projects with this library.

---

### @atlaskit/pragmatic-drag-and-drop — LOW-LEVEL OPTION

| Property | Value |
|---|---|
| Package | `@atlaskit/pragmatic-drag-and-drop` |
| Core size | ~4.7 kB |
| React compat | Framework-agnostic (works with React 19) |
| Origin | Atlassian's production library powering Trello, Jira, Confluence |

**Strengths:** Tiny, headless, framework-agnostic, highly performant.

**Weaknesses:** Low-level — no built-in sortable preset, requires significant custom implementation. Overkill for an admin dashboard list.

---

## 2. Auth / Session Libraries

### iron-session — RECOMMENDED for personal site

| Property | Value |
|---|---|
| Package | `iron-session` |
| Version | 8.0.1 (November 2023) |
| Next.js support | App Router (Route Handlers + Server Components) and Pages Router |
| Session type | Stateless, cookie-based (encrypted with AES-256 + HMAC) |
| Docs | https://github.com/vvo/iron-session |

**How it works:** Session data is AES-256 encrypted and HMAC-signed, stored entirely in a cookie. No database required. Server reads cookie, decrypts, gets session object.

**Best for:** Personal sites needing simple password-gate auth. Single user, no OAuth, no DB.

**Pattern:**
```ts
import { getIronSession } from 'iron-session'
const session = await getIronSession(cookies(), { password: process.env.SECRET, cookieName: 'admin-session' })
if (session.isAdmin) { /* ... */ }
```

---

### Better Auth — RECOMMENDED if more features needed

| Property | Value |
|---|---|
| Package | `better-auth` |
| Next.js App Router | Full support including React Server Components |
| Next.js 16 proxy support | Explicit — docs cover Next.js 16 proxy (renamed middleware) for route protection |
| Password auth | Yes — built-in credential/password provider |
| Session | Cookie-based with optional database session |
| Docs | https://www.better-auth.com |

**Key feature for Next.js 16:** Better Auth docs explicitly address the `proxy.ts` (renamed from `middleware.ts` in Next.js 16) for route protection with database session validation. This makes it the most Next.js 16-aware auth library found.

**Note:** Better Auth Inc. also acquired/merged with NextAuth (Auth.js). Auth.js is now transitioning under Better Auth branding.

---

### NextAuth / Auth.js — HEAVIER OPTION

| Property | Value |
|---|---|
| Package | `next-auth` (v4), `@auth/nextjs` (v5/Auth.js) |
| App Router | Supported in v5 (Auth.js) |
| Status | Transitioning to Better Auth Inc. branding |

**For a personal link portal:** Overkill. NextAuth is optimized for OAuth multi-provider scenarios. A password gate doesn't need it.

---

### Jose — JWT PRIMITIVE

| Property | Value |
|---|---|
| Package | `jose` |
| Purpose | JWT sign/verify, JWE encryption |
| Runtime | Edge Runtime compatible (used in Next.js official auth docs) |
| Use with | Roll-your-own stateless session (Next.js docs example uses Jose) |

**When to use:** If not using iron-session, Jose + `next/headers` cookies() API is the official Next.js pattern for hand-rolled sessions.

---

## 3. Optimistic Updates / Data Fetching

### React 19 Built-ins — RECOMMENDED (no extra library)

React 19 ships three hooks that cover optimistic UI natively:

| Hook | Purpose |
|---|---|
| `useOptimistic(state, reducer?)` | Instantly reflect a change while async action runs; auto-reverts on error |
| `useActionState(action, initialState)` | Wraps server actions, returns `[state, dispatch, isPending]` |
| `useFormStatus()` | Reads parent form pending state (for submit button components) |

**useOptimistic pattern for link reorder:**
```ts
const [optimisticLinks, reorderOptimistic] = useOptimistic(
  links,
  (current, { oldIndex, newIndex }) => arrayMove(current, oldIndex, newIndex)
)
// Call reorderOptimistic() in onDragEnd, then fire server action
```

**Key constraint:** `useOptimistic` must be called inside a `startTransition` or React Action. Calling outside one triggers a warning.

---

### TanStack Query v5 — OPTIONAL, for complex caching scenarios

| Property | Value |
|---|---|
| Package | `@tanstack/react-query` |
| Version | v5.x |
| React compat | React 18+ (React 19 compatible) |

**Optimistic update patterns (two approaches):**
1. **Variable-based (simple):** Render pending items from `variables` while `isPending` is true
2. **Cache-based (complex):** `onMutate` → `setQueryData` optimistically → `onError` → rollback with snapshot

**Verdict for this project:** TanStack Query adds significant complexity over React 19 built-ins. For a personal admin dashboard with a handful of links, prefer `useOptimistic` + server actions. TanStack Query becomes valuable if the admin data model grows complex or needs background refetching.

---

## 4. Persistence / Database Libraries

### JSON File Storage — SIMPLEST

No library needed. `fs/promises` + `JSON.stringify/parse`. Works locally.

**Caveats:**
- Does NOT work on Vercel serverless (read-only filesystem after deploy)
- Race conditions on concurrent writes (unlikely for single-admin use)
- Suitable only for local dev / self-hosted deployments

---

### better-sqlite3 — RECOMMENDED for self-hosted

| Property | Value |
|---|---|
| Package | `better-sqlite3` |
| Version | 12.9.0 (April 2026) |
| API | Synchronous (intentional — better concurrency than async for SQLite) |
| Performance | 11.7x faster than `sqlite3` package for single row reads |
| Next.js compat | Works in Next.js server-side (API routes, server actions, server components) |

**Vercel serverless caveat:** SQLite file must exist on the filesystem. Vercel's filesystem is read-only after deploy. Solutions:
1. Mount a persistent volume (Vercel doesn't offer this natively)
2. Use Turso (libSQL) instead — SQLite-compatible, serverless-native
3. Use Upstash Redis for simple key-value link storage

**Local/self-hosted:** better-sqlite3 is ideal. Fast, zero-config, no external service.

---

### Turso (libSQL) — RECOMMENDED for Vercel/edge

| Property | Value |
|---|---|
| Service | Turso (SQLite-compatible cloud database) |
| Client | `@libsql/client` |
| API | Async HTTP (no persistent TCP connection — works in serverless) |
| Vercel integration | Listed in Vercel Marketplace |
| Free tier | Generous free tier for small projects |

**Why over better-sqlite3 on Vercel:** HTTP-based connections work across serverless cold starts. No filesystem requirement.

**Drizzle ORM** is the recommended ORM layer for both better-sqlite3 and libSQL/Turso. Uses `@libsql/client` as driver.

---

### Upstash Redis — ALTERNATIVE for simple key-value

| Property | Value |
|---|---|
| Service | Upstash Redis |
| Connection | HTTP REST (serverless-safe; no persistent TCP connections) |
| Free tier | 500K commands/month, 256 MB |
| Pay-as-you-go | $0.20 per 100K commands |
| Vercel integration | First-class (Vercel KV was renamed to Upstash Redis in Dec 2024) |

**When to use for links:** Store links as a JSON array in a single Redis key. Fetch with GET, update with SET. Dead-simple for a personal portal with <100 links.

**Tradeoff vs SQLite:** No relational queries, no typed schema. Fine for flat link arrays. Not ideal if you add categories, analytics, etc.

---

## 5. Validation

### Zod — RECOMMENDED (official Next.js recommendation)

| Property | Value |
|---|---|
| Package | `zod` |
| Use case | Validate Server Action FormData inputs |
| Next.js docs | Explicitly recommended in Next.js 16 auth guide |

Pattern: `SignupFormSchema.safeParse(formData)` in server action, return errors as state for `useActionState` to surface.
