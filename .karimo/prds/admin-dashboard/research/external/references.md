# References — Admin Dashboard

_Researched: 2026-04-30._

---

## Next.js 16 Documentation

### Core Features Used

| Topic | URL |
|---|---|
| Mutating Data (Server Actions) | https://nextjs.org/docs/app/getting-started/mutating-data |
| Authentication Guide | https://nextjs.org/docs/app/guides/authentication |
| Data Security | https://nextjs.org/docs/app/guides/data-security |
| Proxy (formerly Middleware) | https://nextjs.org/docs/app/api-reference/file-conventions/proxy |
| revalidatePath | https://nextjs.org/docs/app/api-reference/functions/revalidatePath |
| revalidateTag | https://nextjs.org/docs/app/api-reference/functions/revalidateTag |
| redirect | https://nextjs.org/docs/app/api-reference/functions/redirect |
| cookies() API | https://nextjs.org/docs/app/api-reference/functions/cookies |
| Server Components | https://nextjs.org/docs/app/getting-started/server-and-client-components |
| Fetching Data | https://nextjs.org/docs/app/getting-started/fetching-data |

### Key Next.js 16 Breaking Change

Middleware (`middleware.ts`) is deprecated in Next.js 16.0.0 and renamed to Proxy (`proxy.ts`). The exported function changes from `middleware()` to `proxy()`. Migration codemod:
```bash
npx @next/codemod@canary middleware-to-proxy .
```

Source: https://nextjs.org/docs/app/api-reference/file-conventions/proxy#migration-to-proxy

---

## React 19 Documentation

| Topic | URL |
|---|---|
| React 19 Blog Post | https://react.dev/blog/2024/12/05/react-19 |
| useOptimistic | https://react.dev/reference/react/useOptimistic |
| useActionState | https://react.dev/reference/react/useActionState |
| startTransition | https://react.dev/reference/react/startTransition |
| Server Functions (use server) | https://react.dev/reference/rsc/server-functions |
| Actions concept | https://react.dev/blog/2024/12/05/react-19#actions |

### Key React 19 Features for Admin Dashboard

- `useOptimistic` — instant UI updates with auto-revert on server error
- `useActionState` — wraps server actions, gives `[state, dispatch, isPending]`
- `useFormStatus` — pending state for submit buttons without prop drilling
- `<form action={serverAction}>` — native server action invocation

---

## Drag-and-Drop

| Resource | URL |
|---|---|
| dnd-kit official site | https://dndkit.com |
| dnd-kit Sortable preset | https://dndkit.com/presets/sortable |
| dnd-kit GitHub | https://github.com/clauderic/dnd-kit |
| dnd-kit releases (v0.4.0 React 19 fix) | https://github.com/clauderic/dnd-kit/releases |
| @hello-pangea/dnd GitHub | https://github.com/hello-pangea/dnd |
| react-beautiful-dnd (archived) | https://github.com/atlassian/react-beautiful-dnd |
| Pragmatic drag and drop (Atlassian) | https://github.com/atlassian/pragmatic-drag-and-drop |

---

## Authentication & Session Management

| Resource | URL |
|---|---|
| iron-session GitHub | https://github.com/vvo/iron-session |
| Better Auth Next.js integration | https://www.better-auth.com/docs/integrations/next |
| Auth.js (NextAuth v5) | https://authjs.dev |
| Jose (JWT library) | https://github.com/panva/jose |
| Next.js Security Blog Post | https://nextjs.org/blog/security-nextjs-server-components-actions |
| The Copenhagen Book (auth patterns) | https://thecopenhagenbook.com |

---

## Persistence / Database

| Resource | URL |
|---|---|
| better-sqlite3 GitHub | https://github.com/WiseLibs/better-sqlite3 |
| Turso (libSQL cloud) | https://turso.tech |
| @libsql/client npm | https://github.com/tursodatabase/libsql-client-ts |
| Drizzle ORM with SQLite | https://orm.drizzle.team/docs/get-started/sqlite-existing |
| Upstash Redis | https://upstash.com |
| Upstash pricing | https://upstash.com/pricing |
| Redis on Vercel (Upstash integration) | https://vercel.com/docs/redis |

---

## TanStack Query

| Resource | URL |
|---|---|
| Overview | https://tanstack.com/query/v5/docs/framework/react/overview |
| Optimistic Updates | https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates |

---

## UX / Patterns

| Resource | URL |
|---|---|
| Zod validation | https://zod.dev |
| use-debounce (for email uniqueness checks) | https://www.npmjs.com/package/use-debounce |
| Understanding XSS Attacks (Vercel) | https://vercel.com/guides/understanding-xss-attacks |
| Understanding CSRF Attacks (Vercel) | https://vercel.com/guides/understanding-csrf-attacks |
