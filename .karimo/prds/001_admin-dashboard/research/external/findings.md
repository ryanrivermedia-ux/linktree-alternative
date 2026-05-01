# Research Findings — Admin Dashboard

_Researched: 2026-04-30. Stack: Next.js 16.2.4, React 19, Tailwind CSS 4, TypeScript 5._

---

## Executive Summary

All eight research areas have clear answers. The recommended stack:

| Concern | Recommendation |
|---|---|
| Drag-and-drop | `@dnd-kit/core` + `@dnd-kit/sortable` |
| Auth | `iron-session` (simple) or `better-auth` (richer) |
| Optimistic UI | React 19 `useOptimistic` + `useActionState` (no extra library) |
| Inline editing | Drawer for multi-field edits; toggle switch inline |
| Live preview | Client state mirror (same React component, local state) |
| Server mutations | Next.js Server Actions (`'use server'`) |
| Persistence (self-hosted) | `better-sqlite3` + Drizzle ORM |
| Persistence (Vercel) | Turso (libSQL) or Upstash Redis |
| Route protection | `proxy.ts` (formerly `middleware.ts`) + DAL `verifySession()` |

---

## Finding 1: Drag-and-Drop

### Decision: @dnd-kit/core + @dnd-kit/sortable

**Rationale:**
- Most popular maintained DnD library in the React ecosystem
- Headless — works with any Tailwind styling without fighting CSS
- `@dnd-kit/sortable@10.0.0` peer dep is `react >= 16.8.0` — React 19 works
- `verticalListSortingStrategy` + `closestCenter` is the exact pattern for a link list
- `DragOverlay` provides smooth floating drag preview

**React 19 Compatibility Status: YELLOW**
- The stable `@dnd-kit/core` / `@dnd-kit/sortable` packages list `react >= 16.8.0` and work with React 19
- Some open issues with `removeChild` errors in nested multi-list scenarios (not relevant for a single link list)
- The new `@dnd-kit/react@0.4.0` rewrite has a React 19 `useRef` readonly type fix — but this package is still alpha; do not use it
- **Verdict:** Use stable `@dnd-kit/core` + `@dnd-kit/sortable`. Single-list use case is unaffected by the known issues.

**@hello-pangea/dnd alternative:**
- Explicitly declares `"react": "^18.0.0 || ^19.0.0"` — the only library in this survey with explicit React 19 peer dep testing
- More familiar API for teams coming from react-beautiful-dnd
- Slightly heavier and more opinionated about layout
- Use if team prefers react-beautiful-dnd familiarity or if dnd-kit issues arise

**Ruled out:**
- `react-beautiful-dnd` — archived August 2025, deprecated on npm, no React 19 support
- `@atlaskit/pragmatic-drag-and-drop` — too low-level, no sortable preset, requires significant custom work

---

## Finding 2: Authentication

### Decision: iron-session (minimal) or better-auth (richer)

**Context:** This is a personal site, single admin user. No OAuth, no multi-user, no roles.

**iron-session (v8.0.1) — RECOMMENDED for simplest implementation:**
- Stateless cookie session — no database required for session storage
- Password stored as bcrypt hash in env var
- Session encrypted with AES-256 + HMAC
- Official Next.js docs recommend it for stateless sessions
- App Router (Route Handlers, Server Components) fully supported

**better-auth — RECOMMENDED if you want structured auth:**
- Explicitly supports Next.js 16 `proxy.ts` in its docs (unique among libraries found)
- Credential/password provider built in
- Database-backed sessions available
- `nextCookies` plugin handles cookie setting in server actions automatically

**Roll-your-own with Jose:**
- Official Next.js auth docs demonstrate exactly this pattern
- `jose` for JWT sign/verify, `next/headers` cookies() for cookie management
- Gives maximum control, reasonable for a personal project
- More boilerplate than iron-session

**NextAuth/Auth.js:** Ruled out for this use case — designed for multi-provider OAuth, adds significant complexity and dependency weight for a password gate.

### Next.js 16 Auth Critical Note

Next.js 16 renames `middleware.ts` to `proxy.ts` and `middleware()` to `proxy()`. Better Auth is the only major auth library found that explicitly documents `proxy.ts` support. For iron-session + hand-rolled session, the proxy implementation is purely cookie checking — framework-agnostic, works fine.

**Security model:** Two layers required:
1. `proxy.ts` — optimistic redirect (cookie check, fast)
2. `verifySession()` in every server action — actual secure check

Never rely on proxy alone. Server actions are direct POST endpoints — they must self-authenticate.

---

## Finding 3: Live Preview

### Decision: Client state mirror (inline React component)

**Pattern:** The admin panel drives a local `links` state. A preview pane renders `<LinkPortal links={optimisticLinks} preview />` — the same component used on the public page, in a narrow column.

Changes propagate instantly because they share the same React state. No server round-trip needed for preview rendering. Server action persists in background.

**Split-pane layout:**
```
CSS Grid: grid-cols-[1fr_400px] on desktop, stacked tabs on mobile
Left: editor (link list, drag handles, toggles)
Right: phone-frame mockup showing preview at ~390px width
```

**iFrame preview:** Not recommended for this project. The public page is a React app — reusing the component is simpler and avoids postMessage complexity.

---

## Finding 4: Inline Editing UX

### Decision: Hybrid (drawer for multi-field, inline for simple controls)

**Inline (no modal):**
- Visibility toggle: `<Switch>` on the row — immediate state change, no confirmation needed
- Drag handle: integrated on the row

**Slide-over Drawer for editing link details (title, URL, icon, description):**
- Multiple fields need to be edited together
- Validation error display is cleaner in a form container than inline
- Keyboard trap + focus management is standard in a drawer component
- `Escape` to cancel, `Save` button fires server action

**Why not pure inline editing for text:**
- Managing `contentEditable` or `<input>` visibility state per-row adds complexity
- Two-field edits (title + URL) require knowing which field is focused
- Validation across fields is harder inline

---

## Finding 5: Optimistic UI

### Decision: React 19 built-ins only — no TanStack Query needed

React 19 ships `useOptimistic` and `useActionState` natively. No extra library required for this use case.

**`useOptimistic` for reorder:**
- Accept `(from, to)` indices as the action payload
- Reducer applies `arrayMove(current, from, to)` synchronously
- Call inside `startTransition` wrapping the server action

**`useOptimistic` for toggle:**
- Reducer flips `visible` for the matching link ID
- Call inside `startTransition` or directly in the `<form action>` handler

**Auto-rollback:** If the server action throws or returns an error, `useOptimistic` automatically reverts the optimistic state to the base state on action completion.

**TanStack Query verdict:** Adds ~30 kB, significant setup, and a QueryClient provider. The benefit (background refetching, stale-while-revalidate, deduplication) does not justify the cost for a personal admin dashboard. `useOptimistic` + server actions + `revalidatePath` covers the use case completely.

---

## Finding 6: Next.js 16 Server Actions

### Key Patterns

**Server actions in Next.js 16** use the same API as Next.js 15 (the mutation model is stable). Key points:

- Place `'use server'` at file top or function level
- Call `verifySession()` as first line — always
- Validate input with Zod before touching DB
- Call `revalidatePath('/')` AND `revalidatePath('/admin')` after mutation to bust both caches
- Return errors as state (not throw) for `useActionState` to surface them
- For redirects: call `revalidatePath` first, then `redirect`

**Reorder action pattern:**
Accept ordered array of IDs. Write all sort orders in a single DB transaction, not N individual updates.

**Cache functions in Next.js 16:**
- `revalidatePath(path)` — invalidates all cached data for the route
- `revalidateTag(tag)` — granular tag-based invalidation
- `refresh()` (from `next/cache`) — client-side router refresh only

---

## Finding 7: Persistence Options

### Comparison Table

| Option | Works on Vercel? | Setup | Query Power | Best For |
|---|---|---|---|---|
| JSON file | No (read-only FS) | Trivial | None | Local dev only |
| better-sqlite3 | No (needs FS) | Simple | SQL | Self-hosted |
| Turso (libSQL) | Yes | Moderate | SQL (SQLite dialect) | Vercel + SQLite |
| Upstash Redis | Yes | Simple | Key-value | Vercel + simple flat data |
| Postgres (Supabase/Neon) | Yes | Moderate | Full SQL | If data grows complex |

**For self-hosted deployment:** `better-sqlite3` + Drizzle ORM. Fast, zero-config, SQLite file alongside the app.

**For Vercel deployment:**
- **Turso** if you want SQL (SQLite dialect) — `@libsql/client` connects via HTTP, serverless-safe
- **Upstash Redis** if link data stays flat (no relational queries) — store as JSON array in a single key, extremely simple

**Drizzle ORM** works with both `better-sqlite3` and `@libsql/client`. The schema and query API are identical — only the driver initialization changes. This makes it straightforward to switch from local SQLite to Turso for production.

---

## Finding 8: Security

### Next.js 16 Proxy (formerly Middleware)

**Breaking change in v16.0.0:** `middleware.ts` is deprecated, file renamed to `proxy.ts`, function renamed to `proxy`. Migration codemod:
```bash
npx @next/codemod@canary middleware-to-proxy .
```

**Proxy pattern for admin protection:**
```ts
// proxy.ts
const protectedRoutes = ['/admin']
export async function proxy(req: NextRequest) {
  const path = req.nextUrl.pathname
  if (protectedRoutes.some(r => path.startsWith(r))) {
    const session = await decrypt(req.cookies.get('session')?.value)
    if (!session?.isAdmin) {
      return NextResponse.redirect(new URL('/login', req.nextUrl))
    }
  }
  return NextResponse.next()
}
```

**Important:** Proxy runs at the edge. Do NOT hit the database from proxy — causes latency on every request. Database session validation belongs in the DAL.

### Server Actions Security

Server actions are POST endpoints accessible directly. Always:
1. Call `verifySession()` first
2. Validate input with Zod
3. Check ownership (user owns the resource being mutated)
4. Never return sensitive data in error messages

### Environment Variable Checklist

```bash
SESSION_SECRET=<32-char random>       # openssl rand -base64 32
ADMIN_PASSWORD_HASH=<bcrypt hash>     # bcryptjs.hash(password, 12)
DATABASE_URL=<turso or sqlite path>
```

---

## React 19 Compatibility Summary

| Library / Feature | React 19 Status |
|---|---|
| `@dnd-kit/core` + `@dnd-kit/sortable` | Works; minor open issues in nested multi-list edge cases; single-list fine |
| `@hello-pangea/dnd` | Explicitly tested on React 19 — peer dep `"^18.0.0 || ^19.0.0"` |
| `react-beautiful-dnd` | BROKEN — archived, unmaintained |
| `@atlaskit/pragmatic-drag-and-drop` | Works (framework agnostic) |
| `iron-session` | Works (Node.js library, no React dep) |
| `better-auth` | Works; explicitly documents Next.js 16 proxy support |
| `useOptimistic` | Native React 19 feature |
| `useActionState` | Native React 19 feature |
| `better-sqlite3` | Works in Node.js server runtime; not serverless |
| `@tanstack/react-query` v5 | React 19 compatible |
| `zod` | Framework-agnostic, no React dep |
| `jose` | Edge Runtime compatible, no React dep |

---

## Open Questions for Implementation

1. **Deployment target:** Self-hosted (better-sqlite3) or Vercel (Turso/Upstash)? This determines persistence choice.
2. **Auth complexity:** Simple bcrypt password gate (iron-session) or full auth library (better-auth)?
3. **Preview fidelity:** Does the live preview need to render the CRT/animation effects, or a simplified mockup?
4. **Link data shape:** Does each link need categories, icons, custom colors, analytics? This affects whether flat key-value (Redis) or relational (SQLite) storage is more appropriate.
