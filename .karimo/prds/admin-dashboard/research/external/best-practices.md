# Best Practices — Admin Dashboard

_Researched: 2026-04-30. Stack: Next.js 16, React 19, Tailwind CSS 4, TypeScript 5._

---

## 1. Drag-and-Drop Reordering

### Core Pattern

Use `@dnd-kit/core` + `@dnd-kit/sortable` with:
- `DndContext` at the list root with `PointerSensor` + `KeyboardSensor`
- `SortableContext` wrapping the list, `verticalListSortingStrategy`
- Each link row: `useSortable` hook → apply `transform`, `transition`, `setNodeRef`
- `onDragEnd`: use `arrayMove` to reorder local state, then fire server action

### Accessibility

Configure `KeyboardSensor` with `sortableKeyboardCoordinates` for keyboard-driven reordering. dnd-kit handles ARIA `role="button"` and live announcements via `screenReaderInstructions`.

### Drag Handle vs. Entire Row

Best practice: use a drag handle icon on each row (e.g., `⠿` or hamburger grip) rather than making the entire row draggable. This prevents accidental drags when clicking toggle switches or edit buttons.

### Visual Feedback During Drag

- Set `opacity: 0.4` on the source item (`isDragging` from `useSortable`)
- Use `DragOverlay` to render a floating clone of the dragged item outside the DOM flow — avoids layout shift
- Add `box-shadow` / `scale(1.02)` on the overlay for depth

### Collision Detection

Use `closestCenter` algorithm (not `closestCorners`) for vertical lists — more forgiving hit detection when items are equal-height rows.

---

## 2. Auth / Route Protection

### Next.js 16 Key Change: `middleware.ts` → `proxy.ts`

In Next.js 16, `middleware.ts` is deprecated and renamed to `proxy.ts`. The exported function changes from `middleware` to `proxy`. A codemod exists:
```bash
npx @next/codemod@canary middleware-to-proxy .
```

**Critical note from Next.js 16 docs:** Proxy (formerly middleware) should NOT be your only line of defense. Always verify authentication inside each Server Function as well.

### Two-Layer Security Model

**Layer 1 — Proxy (proxy.ts):** Optimistic redirect. Read session cookie, decrypt with Jose, redirect unauthenticated users. Fast, runs at edge. Cookie-only check (no DB hit).

**Layer 2 — Data Access Layer (DAL):** `verifySession()` called inside every server action and server component that touches sensitive data. This is the secure check.

```
proxy.ts          → fast redirect for obvious cases (cookie check only)
verifySession()   → called in every server action and server component
```

### Simple Password Gate Pattern (No OAuth)

For a single-admin personal site, the simplest secure pattern:

1. Login form → server action validates password against `process.env.ADMIN_PASSWORD` (bcrypt hash)
2. On success: create encrypted session cookie with `iron-session` or `jose`
3. `proxy.ts` reads cookie → redirects `/admin/*` to `/login` if no valid session
4. Every server action calls `verifySession()` to re-check

### Session Cookie Configuration

Follow the Next.js official docs exactly:
```ts
cookieStore.set('session', encryptedJWT, {
  httpOnly: true,   // prevent XSS cookie theft
  secure: true,     // HTTPS only in production
  sameSite: 'lax',  // CSRF protection
  expires: expiresAt,
  path: '/',
})
```

### CSRF

Server actions use POST with CSRF token validation built into Next.js (automatic). Do not add manual CSRF middleware for server actions.

---

## 3. Server Actions for CRUD

### File Structure

```
app/
  admin/
    actions.ts   ← 'use server' at top, all mutation functions
```

### Anatomy of a Server Action

```ts
'use server'
import { verifySession } from '@/lib/dal'
import { revalidatePath } from 'next/cache'

export async function updateLink(id: string, data: LinkUpdate) {
  await verifySession()           // auth check — always first
  // validate with Zod
  // mutate DB
  revalidatePath('/admin')        // bust cache
  revalidatePath('/')             // bust public page cache too
}
```

### Cache Invalidation

- `revalidatePath('/path')` — invalidates all data cached for a route
- `revalidateTag('links')` — tag-based invalidation (more granular)
- `refresh()` from `next/cache` — client router refresh without full revalidation

### Reorder Action

Reordering is a bulk update. Accept the full ordered array of IDs:
```ts
export async function reorderLinks(orderedIds: string[]) {
  await verifySession()
  // Update `sort_order` column for each ID
  // Single transaction, not N individual updates
  revalidatePath('/admin')
  revalidatePath('/')
}
```

### Error Handling Pattern

Return errors as state (not throw), so `useActionState` can surface them:
```ts
export async function updateLink(prevState, formData) {
  try {
    // ... mutation
    return { success: true }
  } catch (e) {
    return { success: false, error: 'Failed to update link.' }
  }
}
```

---

## 4. Optimistic UI

### React 19 Approach (No Extra Library)

React 19 `useOptimistic` is the canonical approach. No need for TanStack Query or SWR for this use case.

**Pattern for toggle visibility:**
```ts
const [optimisticLinks, toggleOptimistic] = useOptimistic(
  links,
  (current, { id }) =>
    current.map(l => l.id === id ? { ...l, visible: !l.visible } : l)
)

async function handleToggle(id: string) {
  startTransition(async () => {
    toggleOptimistic({ id })         // instant UI update
    await toggleLinkVisibility(id)   // server action
  })
}
```

**Pattern for drag reorder:**
```ts
const [optimisticLinks, reorderOptimistic] = useOptimistic(
  links,
  (current, { from, to }) => arrayMove(current, from, to)
)

function onDragEnd(event) {
  const { active, over } = event
  if (!over || active.id === over.id) return
  const from = links.findIndex(l => l.id === active.id)
  const to   = links.findIndex(l => l.id === over.id)
  startTransition(async () => {
    reorderOptimistic({ from, to })
    await reorderLinks(arrayMove(links, from, to).map(l => l.id))
  })
}
```

### Error / Rollback

`useOptimistic` automatically reverts to the base `state` when the action completes (success or error). For explicit error display, combine with `useActionState` error state.

### Avoid Flickering

Ensure server actions call `revalidatePath` AFTER the mutation is committed. The optimistic state shows immediately; the real state arrives when Next.js revalidates. With proper revalidation, there is no flicker — the server returns the same value the optimistic state already showed.

---

## 5. Live Preview

### Recommended Pattern: Client State Mirror

The admin panel maintains a local `links` state. A preview panel (iFrame or inline React component) reads from the same state. Changes in the editor propagate instantly via React state — no server round-trip needed for preview.

```
[Edit Panel]  →  setState(links)  →  [Preview Panel]
              ←  server action (background persist)
```

**Two implementation options:**

**Option A — Inline React Preview (simplest):**
Render a `<LinkPortal links={optimisticLinks} preview />` component alongside the editor in a split-pane layout. This is the same component as the public page, driven by local state.

**Option B — iFrame Preview:**
Embed `<iframe src="/?preview=1" />`. Pass changes via `postMessage` from editor to iframe. More isolation but more complexity. Only worth it if the public page has heavy third-party scripts you don't want running in the admin.

**Recommendation:** Option A for this project. The portal is a React app — reuse the component tree.

### Split-Pane Layout

```
┌─────────────────┬─────────────────┐
│   Admin Panel   │  Live Preview   │
│  (editor, list) │  (phone frame)  │
└─────────────────┴─────────────────┘
```

Use CSS Grid (`grid-cols-[1fr_400px]`) or `flex`. The preview column can be a phone-frame mockup (rounded border, fixed 390px width) to simulate mobile viewing.

**Responsive:** Collapse to tabs on mobile (`<768px`). Show "Editor" / "Preview" tabs, not side by side.

---

## 6. Inline Editing UX

### When to Use Inline Editing

Inline editing is appropriate when:
- The field is simple (short text, URL, toggle)
- The user's context is the list itself (they can see surrounding items)
- Changes are frequent and lightweight
- Immediate feedback matters

### When to Use a Modal/Drawer

Use a modal or slide-over drawer when:
- Multiple related fields need to be edited together (title + URL + icon + tags)
- The edit requires validation feedback across multiple fields
- Adding a new link (creation form is more complex than editing one field)

### Recommended Pattern for this Project

**Hybrid approach:**
- **Toggle visible:** Inline (a single switch on the row — no confirmation needed)
- **Drag reorder:** Inline on the list (drag handle)
- **Edit title/URL:** Either (a) double-click opens inline text input per field, or (b) single "Edit" button opens a slide-over drawer with the full link form

Drawer is simpler to implement correctly (keyboard trap, focus management, Escape to close) and handles validation display better than inline fields.

### Inline Editing Interaction Details

If implementing inline text editing:
- Click text → transforms to `<input>` with current value selected
- `Enter` or blur → save (fire server action)
- `Escape` → cancel, revert to original
- Show a subtle "save" indicator (checkmark) while pending

---

## 7. Security

### Defense in Depth

Three layers, all required:

1. **Proxy (proxy.ts)** — redirect unauthenticated users at the network edge (cookie check)
2. **DAL verifySession()** — server-side session validation in every server action
3. **Input validation (Zod)** — validate and sanitize all inputs before DB write

### Server Actions Are Public Endpoints

Next.js docs warning: "Server Functions are reachable via direct POST requests, not just through your application's UI. Always verify authentication and authorization inside every Server Function."

Do not rely on UI hiding (e.g., not rendering the form) as a security control.

### Cookie Security Checklist

- `httpOnly: true` — JavaScript cannot read session cookie
- `secure: true` — HTTPS only (enforce in production via env check)
- `sameSite: 'lax'` — protects against CSRF while allowing normal navigation
- Set `maxAge` or `expires` — don't use session cookies that expire on browser close

### Rate Limiting

For the login endpoint (the server action that checks password), add basic rate limiting. Options:
- Upstash Rate Limit (`@upstash/ratelimit`) — serverless-safe, uses Redis counters
- Simple in-memory Map (works locally; not persistent across serverless instances)

### Environment Variables

- Never commit `ADMIN_PASSWORD` (store hashed with bcrypt)
- Never commit `SESSION_SECRET` (32-char random, `openssl rand -base64 32`)
- Use `.env.local` locally, Vercel environment variables in production
