# Internal Research: Gaps & Issues

## Critical Blockers

### 1. Static Export Architecture
- `next.config.ts` line 5: `output: "export"`
- Blocks: API routes, server actions, middleware/proxy.ts, DB access
- **Must remove** for any server-side admin functionality
- Deployment must shift from GitHub Pages to Vercel or Node server

### 2. No Auth System
- Zero auth code in codebase
- No sessions, no middleware, no protected routes
- Admin route would be fully public without adding auth

### 3. No Data Persistence Layer
- All content is hardcoded TypeScript in `site.config.ts`
- Editing requires code changes + rebuild
- No DB, no API, no mutations

## Component Gaps

### 4. No Form Components
- No reusable `<Input>`, `<Select>`, `<Textarea>`, `<Toggle>`
- FreeResources and SubscribeModal use native HTML validation only
- Need to build or import a component library

### 5. No Table/List Component
- No existing admin-style list view
- CardNav uses grid layout (not table rows)
- Need sortable list with drag handles for reordering

### 6. No Drag-and-Drop
- No @dnd-kit or similar in package.json
- Cannot reorder items without adding library

### 7. No Image Upload
- Tool icons are string paths (e.g., `/icons/github.svg`)
- No upload UI, no file handling

## UX Gaps

### 8. No Undo/Redo
- No history stack in any component
- Changes can't be reverted without page reload

### 9. No Live Preview Sync
- No mechanism to sync admin edits to a preview pane
- Would need shared React state or context bridge

### 10. Missing Visibility Flags
- `SocialLink`, `Resource`, `Tool` types have no `visible: boolean` field
- Must add to types and config before toggle UI can work
