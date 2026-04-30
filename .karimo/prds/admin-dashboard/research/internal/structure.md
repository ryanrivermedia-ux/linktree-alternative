# Internal Research: Project Structure

## Directory Layout

```
src/
├── app/
│   ├── layout.tsx        # Root layout — always dark mode (html className="dark")
│   └── page.tsx          # Main page — fixed/clipper scroll pattern
├── components/
│   ├── CardNav.tsx       # Header + hamburger + card grid (GSAP animated)
│   ├── OurLinks.tsx      # Social links — carousel mobile, grid desktop
│   ├── FreeResources.tsx # Resource cards with modal trigger
│   ├── SubscribeModal.tsx # Framer Motion modal with email + copy link
│   └── Footer.tsx
├── config/
│   └── site.config.ts    # SINGLE SOURCE OF TRUTH — all content + types (lines 1-491)
└── lib/
    └── brand-styles/     # CSS tokens, typography
```

## Deployment Constraint (CRITICAL)

`next.config.ts` line 5: `output: "export"` — static export for GitHub Pages.

**Consequences:**
- No API routes (`/src/app/api/`)
- No server actions (`'use server'`)
- No middleware / proxy.ts
- No runtime database access
- All content must be in static files at build time

**Required change for admin dashboard:** Remove `output: "export"`, deploy to Vercel or Node server.

## CSS Files

- `globals.css` — base styles, Tailwind 4 `@theme` block, responsive CSS vars
- `theme.css` — full design token system (brand, gray, semantic, dark mode)
- `brand.css` — brand-specific overrides
- `FaultyTerminal.css` — CRT/scanline animation CSS
