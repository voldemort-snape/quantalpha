# What was fixed

## 1. Broken math rendering (content bug, affected all 23 parts)
The original `MathRenderer.tsx` only understood `$...$` and `$$...$$` LaTeX
delimiters. The actual content uses `\( ... \)` for inline math in 194
places — these were rendering as raw, literal text like `\(N(N-1)/2\)`
instead of proper typeset math.

Fixed: the renderer now handles `$...$`, `$$...$$`, `\( ... \)`, and
`\[ ... \]`, plus normalizes a smaller number of accidentally
double-escaped delimiters (`\\(` instead of `\(`) found in the data.
Verified against all 396 subsections in `parts.json` — zero unrendered
delimiters remain.

The renderer was also restructured so math is extracted and converted to
HTML *before* markdown (`**bold**`, `*italic*`) and paragraph processing
run, and spliced back in *after* — so generated KaTeX markup can never be
corrupted by the markdown regexes or double-escaped by mistake.

## 2. Homepage: replaced the WebGL book carousel
The original homepage rendered a Three.js 3D carousel of rotating "books."
Problems with it:
- Click-to-select used a different angular threshold than the visual
  "focused" state, so clicks often didn't register on the book that looked
  selected.
- Navigation dots were positioned with raw `window.innerWidth/innerHeight`
  math computed once per render, not on resize — broke on most phone
  screens and on window resize.
- No loading state; continuous WebGL render loop is heavy on battery/mobile
  and not screen-reader or keyboard accessible.
- It also pulled in `three` (~470KB) for what is, functionally, a menu.

Replaced with a CSS/DOM table-of-contents homepage in the same paper/gold/
serif visual identity: a typographic hero, a sticky live-filter search bar,
and all 23 parts grouped into 8 thematic clusters as a clean numbered
index. Uses ordinary `<Link>` navigation, real focus states, and a subtle
CSS fade-up entrance (which respects `prefers-reduced-motion`).

## 3. Removed dead weight
- Deleted unused Vite-boilerplate `Home.tsx` and `App.css` (never imported).
- Removed `three`, `@types/three` (only used by the old homepage) and
  `gsap` (never used anywhere) from `package.json`.

## Result
- JS bundle: 1,392 KB → 890 KB (414 KB → 287 KB gzipped), about 36% smaller.
- `tsc -b` and `vite build` both succeed cleanly.
- `eslint` on the changed files: 0 errors (the 10 pre-existing errors
  elsewhere are in untouched shadcn/ui boilerplate and existed before
  these changes too).
- Added: visible keyboard focus rings app-wide, smooth in-page anchor
  scrolling (`#contents` link in the hero), a calmer custom scrollbar,
  and `line-clamp` styling without needing a Tailwind plugin.

## To run it
```
npm install
npm run dev      # local dev server
npm run build    # production build -> dist/
```
