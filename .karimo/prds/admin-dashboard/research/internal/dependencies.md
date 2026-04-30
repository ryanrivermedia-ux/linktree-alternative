# Internal Research: Dependencies

## Core Stack
- Next.js 16 (App Router)
- React 19
- TypeScript 5
- Tailwind CSS 4
- Framer Motion v12.29.0
- GSAP v3.14.2

## Data Types (site.config.ts)

```typescript
interface NavItem { id: string; label: string; href: string; }

interface SocialLink {
  id: string; platform: string; title: string;
  handle: string; url: string; icon: string;
}

interface Resource {
  id: string; title: string; description: string;
  badge: "live" | "coming-soon"; link: string; buttonLabel: string;
}

interface TagData { label: string; bg: string; text: string; }

interface Tool {
  id: string; name: string; icon: string; url: string;
  description: string; tags: TagData[]; smallIcon?: boolean;
}

interface SiteConfig {
  metadata: {...}; branding: {...}; features: {...};
  theme: {...}; nav: NavItem[]; socialLinks: SocialLink[];
  resources: Resource[]; tools: Tool[]; blog: {...};
}
```

## Current Data Counts
- `siteConfig.nav` — 3 NavItems
- `siteConfig.socialLinks` — 5 SocialLinks
- `siteConfig.resources` — 3 Resources
- `siteConfig.tools` — 16 Tools

## Missing Dependencies (for admin dashboard)
- No auth library (NextAuth, iron-session, etc.)
- No drag-and-drop library (@dnd-kit, react-beautiful-dnd)
- No form validation (zod, react-hook-form)
- No database client (better-sqlite3, Turso, Upstash)
- No ORM (Drizzle, Prisma)

## No Existing API Routes
Zero files in `src/app/api/`. No server actions anywhere in codebase.
