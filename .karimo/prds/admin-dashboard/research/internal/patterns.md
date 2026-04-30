# Internal Research: Code Patterns

## Component Patterns

### Animation: GSAP (CardNav.tsx lines 19-65)
```js
const tl = gsap.timeline();
tl.to(containerRef.current, { height: targetHeight, duration: 0.4, ease: "power2.out" })
  .fromTo(cardsRef.current.children, { opacity: 0, y: 20 }, { opacity: 1, y: 0, duration: 0.3, stagger: 0.08 }, "-=0.2");
```
Easings used: `power2.out`, `power2.in`, `power3.out`

### Animation: Framer Motion (SubscribeModal.tsx lines 169-175)
```jsx
<motion.div
  initial={{ scale: 0.95, y: 20 }}
  animate={{ scale: 1, y: 0 }}
  transition={{ type: "spring", damping: 25, stiffness: 300 }}
/>
```
Used with `AnimatePresence` for enter/exit.

### State Management Pattern (all components)
- Local `useState` per component — no global store
- `useRef` for DOM refs (GSAP targets)
- `useEffect` for GSAP triggers
- `useSyncExternalStore` for media query (OurLinks.tsx)
- `useCallback` for memoized handlers (SubscribeModal.tsx)

### localStorage Persistence (FreeResources.tsx lines 36-44, SubscribeModal.tsx lines 21-33)
```js
const hasResourceAccess = (id: string) =>
  localStorage.getItem(`resource_access_${id}`) === "true";
localStorage.setItem(STORAGE_KEY, "true");
```

### Styling Conventions
- No CSS Modules — all CSS in global files
- BEM-like class names: `.nav-card`, `.nav-card-text`, `.nav-card-arrow`
- Tailwind 4 `@theme inline` block maps CSS vars to tokens
- Responsive CSS vars via `@media` in `:root`:
  ```css
  --content-max-width: 448px;
  @media (min-width: 640px) { --content-max-width: 520px; }
  ```

### Dark Mode
- Always-on: `html className="dark"` in layout.tsx line 27
- Not user-togglable currently
- Semantic tokens in `.dark {}` block in theme.css (lines 237-370)

### Modal Pattern (SubscribeModal.tsx)
- AnimatePresence + spring animation
- Body scroll lock
- ESC key close
- Backdrop blur overlay
- No dialog/modal library — hand-rolled
