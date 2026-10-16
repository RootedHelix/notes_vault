# Plan React + Motion pixel wipe migration  
  
Good. Now I have everything I need to write a thorough, grounded plan.  
  
---  
  
## Architectural Recommendation: Option B (React Island)  
  
After reading every file in the site, the recommendation is unambiguously **Option B**, with one deliberate structural upgrade applied to `index.html`. Here is the full reasoning and then the concrete plan.  
  
---  
  
### Why NOT Option A (Full React Migration)  
  
The vanilla animations are not simple DOM manipulation — they are a tightly coupled system:  
  
- `initSceneCharacter` runs a hand-tuned frame-by-frame `requestAnimationFrame` loop with a 19-frame sprite sheet, a walk-in path, a timed pause, and a sit-down Y-offset ramp. It reads `--char-x-ratio` from computed CSS custom properties to determine where it walks to.  
- `initRain` fires `setInterval` callbacks per-drop, calculates bridge impact coordinates using `imageRatioToViewport()`, and creates splat elements dynamically. Each of the 120 drops has its own independent timer chain.  
- `initCityTwinkle` drives a sprite-sheet swap on a randomized `setTimeout` loop that avoids repeating frames.  
- All three systems are tightly coupled to the CSS custom property system (e.g. `--frame-idx`, `--x-offset`, `--y-offset`, `--img-size`, `--char-x-ratio`). The CSS literally computes `background-position` from these properties.  
  
Converting any of this to React/JSX would require either (a) threading all those CSS custom properties through React refs and writing useEffect loops that exactly replicate the timer logic, or (b) just calling the vanilla functions inside useEffect anyway — which provides zero benefit while adding a migration risk surface.  
  
**The existing vanilla animation system is already a well-architected, self-contained runtime. Do not touch it.**  
  
---  
  
### Why NOT Option C (Vite Hybrid with Vanilla JS Imported as Modules)  
  
The core problem with Option C is that Vite's module bundler and the vanilla JS are not actually compatible without modification:  
  
- `terminal.js` and `portfolio.js` use `window.startTerminalAnimation`, `window.__terminalOnComplete`, `window.__bridge`, `window.scrollTo()`, and `document.getElementById` freely at the module level. They are script-tag globals, not ES modules — they have no `export` statements.  
- Porting them to Vite's module system would require adding `export` / `import` everywhere, moving the `window.addEventListener('DOMContentLoaded')` call, and auditing every `window.*` global. That is a meaningful refactor of the animation code.  
- The `background-image: url("../../images/...")` paths in CSS would need to be re-expressed as Vite asset imports.  
- The font `@font-face` src paths are relative to the CSS file — Vite handles this, but only if the CSS is imported through the Vite pipeline, which means moving `main.css` into the React entry.  
  
Option C sounds like "medium effort, medium risk," but in practice it pulls the vanilla code into the Vite pipeline and every path, every global, and every DOM API call becomes a potential breakage point.  
  
---  
  
### Why Option B IS Correct  
  
The existing site has a clear architectural seam: the landing section (`#loading-wrapper`) is `position: absolute` and `height: 100%` — it fills exactly one viewport. After the animation sequence completes, nothing happens. The page is static. The character sits. Rain falls. That is the end of the current experience.  
  
The new sections live BELOW this fixed-height landing. They are a scroll-based experience that begins after the landing. This is exactly what a React island is designed for: a bounded, isolated React tree mounted into a single DOM node that lives outside the existing animation system.  
  
The key architectural insight is: **the vanilla JS never needs to know React exists, and React never needs to know the vanilla animations exist.** They share only the DOM, and they occupy different vertical regions.  
  
---  
  
## Concrete Implementation Plan  
  
### Phase 1: Project Setup  
  
The project currently has no `package.json`. Vite will be initialized at the project root but configured to point at a new React entry file that is completely separate from `src/index.html`.  
  
**Commands to run:**  
  
```bash  
cd /Users/sam/Desktop/Programming/portfolio_site  
npm create vite@latest . -- --template react  
# When prompted: "Current directory is not empty" → select "Ignore files and continue"  
# Framework: React  
# Variant: JavaScript  
  
npm install  
npm install motion  
```  
  
This creates `package.json`, `vite.config.js`, and a `src/main.jsx` entry point. The existing `src/index.html` is **not** the Vite entry. Vite's root `index.html` (generated at project root) will be a new file.  
  
**After running `npm create vite`**, Vite generates its own `index.html` at the project root. That file must be structured to:  
1. Load the vanilla CSS via `<link>` tags (pointing to `src/styles/main.css`)  
2. Contain the full existing HTML body content (terminal, loading-wrapper, etc.)  
3. Add a new `<div id="react-sections">` below `#loading-wrapper`  
4. Load `src/portfolio.js` and `src/terminal.js` as regular `<script>` tags (not modules — they rely on global scope and `window.*`)  
5. Let Vite inject the React bundle via its normal mechanism (`<script type="module" src="/src/main.jsx">`)  
  
**File structure after setup:**  
  
```  
portfolio_site/  
├── index.html                        ← Vite entry (new, replaces src/index.html)  
├── package.json                      ← new  
├── vite.config.js                    ← new  
├── src/  
│   ├── index.html                    ← KEEP (archive/reference — no longer served)  
│   ├── portfolio.js                  ← UNCHANGED  
│   ├── terminal.js                   ← UNCHANGED  
│   ├── main.jsx                      ← new React entry  
│   ├── App.jsx                       ← new  
│   ├── fonts/                        ← UNCHANGED  
│   ├── styles/  
│   │   ├── main.css                  ← UNCHANGED  
│   │   ├── terminal.css              ← UNCHANGED  
│   │   ├── components.css            ← UNCHANGED  
│   │   ├── layouts.css               ← UNCHANGED  
│   │   └── pixel-scroll-transition.css  ← new  
│   └── components/  
│       └── PixelScrollTransition.jsx ← new  
├── images/                           ← UNCHANGED  
```  
  
**Critical vite.config.js note:** By default Vite sets `root` to the project root and uses the `index.html` at the root as its entry. The CSS `@font-face` and `background-image` paths in the existing CSS files use relative paths like `../fonts/...` and `../../images/...`. After moving the served `index.html` from `src/` to the project root, these paths resolve differently.  
  
There are two clean ways to handle this:  
  
- **Preferred:** Add a `publicDir` pointing to a new `public/` folder at root and symlink or copy the images/fonts there, OR  
- **Simpler:** Set `vite.config.js` to `root: 'src'` — this keeps the original `src/index.html` as the Vite entry and preserves all relative paths. With this approach, the existing `src/index.html` IS the Vite entry, and you simply add `<div id="react-sections">` and the Vite module script to it.  
  
**The `root: 'src'` approach is strongly preferred** because it means zero path changes in any CSS or JS file.  
  
```js  
// vite.config.js  
import { defineConfig } from 'vite'  
import react from '@vitejs/plugin-react'  
  
export default defineConfig({  
  root: 'src',  
  plugins: [react()],  
  build: {  
    outDir: '../dist',  
    emptyOutDir: true,  
  },  
  publicDir: '../images',  
})  
```  
  
With this config, `src/index.html` remains the entry point. All existing `url("../../images/...")` paths in the CSS will need to reference the images correctly. Because `publicDir` is set to `../images`, all image filenames become available at `/cyberpunk_city_night_sheet.png` etc. The CSS `url()` paths in `components.css` currently read `url("../../images/filename.png")` — these would need to become `url("/filename.png")` in a Vite build. This is the one CSS change required.  
  
Alternatively, keep `publicDir` as the default (`public/`) and copy the images into `src/public/images/`. This avoids touching the CSS at all.  
  
**Cleanest resolution:** Create `src/public/images/` and `src/public/fonts/` as Vite's public directory targets, keeping all existing CSS paths intact by making Vite serve those assets at the correct relative paths. But this requires file copying.  
  
**Actual cleanest resolution with zero CSS changes:** Leave `vite.config.js` default (root at project root), move `index.html` to the project root, and update the two `url()` base paths in `components.css` from `../../images/` to `./images/` — because from the project root, `images/` is one level away, not two. Only `components.css` has image URL references (the background sprite sheet, character sprite, car smoke, bridge walker). That is 5 changed lines in one file, and it is a mechanical substitution, not a logic change.  
  
**Recommendation: go with project root as Vite root, update the image paths in components.css from `../../images/` to `../images/` (since the CSS file is at `src/styles/`, the images are at `images/`, which is `../../images/` from `src/styles/` — this does NOT change). The `index.html` moving from `src/` to root is the only concern. Vanilla `<script src="portfolio.js">` in the root `index.html` must become `<script src="src/portfolio.js">`. Font `@font-face` src in `terminal.css` (which is in `src/styles/`) points to `../fonts/...` — that is relative to the CSS file, so it stays correct.**  
  
This is getting into path arithmetic that is error-prone to get wrong. The decisive answer: **use `root: 'src'` in vite.config.js**. It is the path of least resistance and zero broken references.  
  
---  
  
### Phase 2: The Root index.html (Vite Entry at `src/index.html`)  
  
Add two things to the existing `src/index.html`:  
  
1. A `<div id="react-sections"></div>` immediately after the closing `</div>` of `#loading-wrapper`  
2. A `<script type="module" src="/main.jsx"></script>` in the `<head>` or before `</body>` (Vite injects this automatically if you use the Vite plugin, but you can also add it manually)  
  
The `<script src="portfolio.js">` and `<script src="terminal.js">` tags stay exactly as they are. They remain plain `<script>` tags — not `type="module"`. This is intentional: both files depend on global scope, `window.*` assignments, and `document.getElementById` without any module boundary. Wrapping them in modules would require adding `export`/`import` everywhere and auditing every `window.*` assignment.  
  
The `#loading-wrapper` must keep `position: absolute` and remain `height: 100%` (or `100vh`). This is what allows content below it to exist — the wrapper does not push content down; it overlays it. After the landing animation completes, the user scrolls down to reveal the React sections.  
  
One addition is needed in `portfolio.js`: after `startPortfolioStage()` finishes its setup work, it should signal the React side that scrolling is now safe. The cleanest way is to dispatch a custom event:  
  
```js  
// At the end of startPortfolioStage(), after requestAnimationFrame:  
window.dispatchEvent(new CustomEvent('portfolio:ready'));  
```  
  
The React app listens for this event and enables scroll on the `<html>` element (currently `scrollbar-width: none` hides scrollbars but scroll still works). This does NOT require modifying `portfolio.js` in any way that risks the animations — it is purely additive.  
  
---  
  
### Phase 3: PixelScrollTransition.jsx  
  
**How it works conceptually:**  
  
The component renders a grid of pixel-sized `<div>` elements (or `<canvas>`) that cover the full viewport. On scroll, `useScroll` from Motion tracks the scroll progress of a section boundary element. As `scrollYProgress` moves from 0 to 1, pixels in the grid reveal sequentially — creating the "wipe" effect. The color palette `#002029 → #00303D → #004052 → #005066 → #00607A → #8FFFE0` maps from dark to light, driving the pixel reveal in waves.  
  
**Specific Motion API usage:**  
  
`useScroll` is used with a `target` ref pointing to the section wrapper element:  
  
```jsx  
const ref = useRef(null);  
const { scrollYProgress } = useScroll({  
  target: ref,  
  offset: ["start end", "end start"]  
});  
```  
  
`scrollYProgress` is a `MotionValue` that goes 0→1 as the target scrolls from "its start is at the bottom of the viewport" to "its end is at the top of the viewport." This gives the full scroll travel range.  
  
**Pixel grid approach:**  
  
The grid uses a flat array of cells. Each cell has a `threshold` (0–1) representing at what `scrollYProgress` value it should fill in. The thresholds are assigned in a pattern (e.g., column-by-column, diagonal, or random) to create the wipe direction. On each animation frame, `scrollYProgress.get()` is compared to each cell's threshold.  
  
Two implementation paths for the grid:  
  
- **DOM div grid:** Fast to implement, but 40×30 = 1200 divs. At 100% pixel size this is fine for a viewport-filling overlay. Each div gets a color from the palette based on which "wave" it belongs to. The Motion `useTransform` hook maps `scrollYProgress` to opacity/visibility for each cell.  
- **Canvas:** Better performance for very fine grids. A single `<canvas>` element, redrawn on each `scrollYProgress` change via `useMotionValueEvent`.  
  
**Recommendation: canvas**, because it eliminates 1200+ DOM nodes, handles resize cleanly with `useEffect`, and gives pixel-perfect control over the wipe pattern.  
  
**The pixel wipe pattern:** Divide the canvas into a grid (e.g., 60×40 cells at ~16px each on a 1080p screen). Assign each cell a `revealAt` value from 0 to 1. The pattern that reads as a "wipe" is:  
  
- **Left-to-right column wipe:** `revealAt = column / totalColumns`  
- **Diagonal wipe:** `revealAt = (col + row) / (totalCols + totalRows)`  
- **Random scatter:** `revealAt = Math.random()` — feels more "pixelated" and less mechanical  
  
For a cyberpunk aesthetic matching the city scene, the recommended pattern is a **diagonal with slight randomization**: `revealAt = (col/totalCols * 0.6) + (row/totalRows * 0.4) + (Math.random() * 0.05)`. This gives a bottom-left to top-right sweep with a slightly organic edge.  
  
**Color assignment:** Map `scrollYProgress` to the palette in two passes:  
1. Fill cells where `scrollYProgress >= revealAt` with the appropriate palette color  
2. The "leading edge" cells (within ~0.05 of `scrollYProgress`) get `#8FFFE0` for a bright sweep highlight  
3. Cells behind the edge use the palette colors based on their position in the grid (darker at the leading edge side, lighter toward the trailing side — reversed from the palette order)  
  
**Full component structure:**  
  
```jsx  
// src/components/PixelScrollTransition.jsx  
import { useRef, useEffect, useCallback } from 'react';  
import { useScroll, useMotionValueEvent } from 'motion/react';  
  
const PALETTE = ['#002029', '#00303D', '#004052', '#005066', '#00607A', '#8FFFE0'];  
const CELL_SIZE = 18; // px — tune for density  
const EDGE_WIDTH = 0.06; // scrollYProgress units for the bright leading edge  
  
export default function PixelScrollTransition({ children, nextSectionBg = '#002029' }) {  
  const wrapperRef = useRef(null);  
  const canvasRef = useRef(null);  
  const cellsRef = useRef(null); // stable grid, computed once on mount  
  
  const { scrollYProgress } = useScroll({  
    target: wrapperRef,  
    offset: ['start end', 'end start'],  
  });  
  
  // Build cell grid on mount (and on resize)  
  const buildGrid = useCallback(() => {  
    const canvas = canvasRef.current;  
    if (!canvas) return;  
    const cols = Math.ceil(canvas.width / CELL_SIZE);  
    const rows = Math.ceil(canvas.height / CELL_SIZE);  
    const cells = [];  
    for (let r = 0; r < rows; r++) {  
      for (let c = 0; c < cols; c++) {  
        const base = (c / cols) * 0.55 + (r / rows) * 0.45;  
        const jitter = (Math.random() - 0.5) * 0.08;  
        cells.push({ c, r, revealAt: Math.max(0, Math.min(1, base + jitter)) });  
      }  
    }  
    cellsRef.current = { cells, cols, rows };  
  }, []);  
  
  useEffect(() => {  
    const canvas = canvasRef.current;  
    const observer = new ResizeObserver(() => {  
      canvas.width = canvas.offsetWidth;  
      canvas.height = canvas.offsetHeight;  
      buildGrid();  
    });  
    observer.observe(canvas);  
    return () => observer.disconnect();  
  }, [buildGrid]);  
  
  useMotionValueEvent(scrollYProgress, 'change', (progress) => {  
    const canvas = canvasRef.current;  
    const grid = cellsRef.current;  
    if (!canvas || !grid) return;  
    const ctx = canvas.getContext('2d');  
    ctx.clearRect(0, 0, canvas.width, canvas.height);  
  
    for (const { c, r, revealAt } of grid.cells) {  
      if (progress < revealAt) continue;  
      const distBehindEdge = progress - revealAt;  
      let color;  
      if (distBehindEdge < EDGE_WIDTH) {  
        color = PALETTE[5]; // #8FFFE0 — bright leading edge  
      } else {  
        // Map to palette[0..4] based on how far behind the edge  
        const idx = Math.min(  
          4,  
          Math.floor(((distBehindEdge - EDGE_WIDTH) / (1 - EDGE_WIDTH)) * 5)  
        );  
        color = PALETTE[idx];  
      }  
      ctx.fillStyle = color;  
      ctx.fillRect(c * CELL_SIZE, r * CELL_SIZE, CELL_SIZE, CELL_SIZE);  
    }  
  });  
  
  return (  
    <div ref={wrapperRef} className="pixel-transition-wrapper">  
      <canvas  
        ref={canvasRef}  
        className="pixel-transition-canvas"  
        aria-hidden="true"  
      />  
      <div className="pixel-transition-content">  
        {children}  
      </div>  
    </div>  
  );  
}  
```  
  
---  
  
### Phase 4: Section Structure  
  
The React app mounts into `#react-sections`. The sections that come after the landing are the actual portfolio content. Since the landing already shows the character's name ("Samuel O'Hearne") via the hologram label, the subsequent sections should answer: who is this person, what have they built, and how to reach them.  
  
**Proposed sections (in scroll order):**  
  
```  
Section 0: #landing-anchor (0px height — just a scroll target after the animation)  
Section 1: About         — short bio, tech stack, the "human" behind the terminal  
Section 2: Projects      — card grid of 3–4 projects, each with title/stack/links  
Section 3: Skills        — visual representation of tools/languages  
Section 4: Contact       — links (GitHub, LinkedIn, email), simple form or just icons  
```  
  
Each section boundary is wrapped by `<PixelScrollTransition>`. The transition occurs as you scroll from one section into the next. The canvas overlay paints over the exiting section, then the entering section appears behind it as the canvas "unpaints" (this requires a two-phase approach — wipe in, then wipe out — which maps to `scrollYProgress` going 0→0.5 for wipe-in and 0.5→1 for wipe-out on the same target).  
  
**`App.jsx` structure:**  
  
```jsx  
import PixelScrollTransition from './components/PixelScrollTransition';  
  
export default function App() {  
  return (  
    <main className="portfolio-sections">  
      <PixelScrollTransition>  
        <section id="about" className="section">  
          {/* About content */}  
        </section>  
      </PixelScrollTransition>  
  
      <PixelScrollTransition>  
        <section id="projects" className="section">  
          {/* Project cards */}  
        </section>  
      </PixelScrollTransition>  
  
      <PixelScrollTransition>  
        <section id="skills" className="section">  
          {/* Skills */}  
        </section>  
      </PixelScrollTransition>  
  
      <section id="contact" className="section">  
        {/* Contact — last section, no exit transition needed */}  
      </section>  
    </main>  
  );  
}  
```  
  
**`src/main.jsx`:**  
  
```jsx  
import { StrictMode } from 'react';  
import { createRoot } from 'react-dom/client';  
import App from './App';  
  
const container = document.getElementById('react-sections');  
if (container) {  
  createRoot(container).render(  
    <StrictMode>  
      <App />  
    </StrictMode>  
  );  
}  
```  
  
---  
  
### Phase 5: CSS for the Sections  
  
`src/styles/pixel-scroll-transition.css`:  
  
```css  
.portfolio-sections {  
  position: relative;  
  z-index: 10;  
  background: #002029; /* darkest palette color */  
}  
  
.section {  
  min-height: 100vh;  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  color: #8FFFE0;  
  font-family: 'Space Grotesk', sans-serif;  
  padding: 4rem 2rem;  
}  
  
.pixel-transition-wrapper {  
  position: relative;  
  overflow: hidden;  
}  
  
.pixel-transition-canvas {  
  position: absolute;  
  inset: 0;  
  width: 100%;  
  height: 100%;  
  pointer-events: none;  
  z-index: 20;  
}  
  
.pixel-transition-content {  
  position: relative;  
  z-index: 1;  
}  
```  
  
Import this in `main.jsx` (not in `main.css` — keep the vanilla CSS untouched):  
  
```jsx  
import './styles/pixel-scroll-transition.css';  
```  
  
---  
  
### Phase 6: The Landing-to-React Handoff  
  
The `#loading-wrapper` is `position: absolute; height: 100%` — it overlays the page but does NOT push `#react-sections` down. To make the sections appear below the landing, the `#react-sections` div needs a `margin-top: 100vh` (or the body needs `padding-top: 100vh`).  
  
The cleanest approach: add to `layouts.css` (the currently empty file):  
  
```css  
#react-sections {  
  position: relative;  
  margin-top: 100vh;  
  z-index: 5;  
}  
```  
  
This is the only addition to `layouts.css`. Nothing else changes in the vanilla CSS.  
  
The `html { scrollbar-width: none }` rule already hides scrollbars site-wide. During the terminal animation stage, scrolling should be locked so the user cannot scroll to the sections mid-animation. After `portfolio:ready` fires (added to the end of `startPortfolioStage`), the React app should unlock scroll.  
  
In `main.jsx` or `App.jsx`:  
  
```jsx  
useEffect(() => {  
  document.documentElement.style.overflow = 'hidden';  
  const unlock = () => document.documentElement.style.overflow = '';  
  window.addEventListener('portfolio:ready', unlock, { once: true });  
  return () => window.removeEventListener('portfolio:ready', unlock);  
}, []);  
```  
  
---  
  
### Phase 7: initPixelTransition — Keep or Remove?  
  
The mention of `initPixelTransition` in the prompt does not appear in the actual codebase. There is no function by that name in `portfolio.js`. The loading→cityscape transition is handled by CSS class additions (`bg-fade-in`, `ui-fade-out`, `hologram-label--visible`) and a `setTimeout` in `startLoadingBarStage`. There is no JS pixel-grid wipe in the current code.  
  
**Conclusion: there is nothing to remove.** The existing loading→cityscape transition is purely CSS animation-based (opacity fades and keyframe animations). It should be left completely unchanged. The new `PixelScrollTransition` is an entirely additive feature that lives only in the scroll-based sections.  
  
---  
  
### Phase 8: `useScroll` offset tuning  
  
The `offset: ['start end', 'end start']` configuration means:  
- `scrollYProgress = 0` when the section's **start** aligns with the viewport's **end** (section is just entering from below)  
- `scrollYProgress = 1` when the section's **end** aligns with the viewport's **start** (section has fully passed)  
  
For a `min-height: 100vh` section, this gives roughly one full viewport-height of scroll travel before the transition completes. That is appropriate — the user needs to scroll meaningfully to trigger the wipe.  
  
For the wipe-in effect specifically (fade the canvas in over the entering section), a tighter offset may feel better: `['start 0.8', 'start 0.2']` — meaning the wipe happens only during the last 60% of the section entering the viewport. Tune this based on feel.  
  
---  
  
### Summary of Changes by File  
  
| File | Change type | What changes |  
|---|---|---|  
| `src/index.html` | Additive only | Add `<div id="react-sections">` after `#loading-wrapper` |  
| `src/portfolio.js` | Additive only | Add `window.dispatchEvent(new CustomEvent('portfolio:ready'))` at end of `startPortfolioStage` |  
| `src/styles/layouts.css` | Additive only | Add `#react-sections { margin-top: 100vh; }` |  
| `src/portfolio.js` | No change | All animation functions untouched |  
| `src/terminal.js` | No change | Untouched |  
| `src/styles/components.css` | No change | Untouched |  
| `src/styles/main.css` | No change | Untouched |  
| `src/styles/terminal.css` | No change | Untouched |  
| `package.json` | New | Created by `npm create vite` |  
| `vite.config.js` | New | `root: 'src'`, react plugin |  
| `src/main.jsx` | New | React entry, mounts to `#react-sections` |  
| `src/App.jsx` | New | Section layout with `PixelScrollTransition` |  
| `src/components/PixelScrollTransition.jsx` | New | Scroll-driven pixel canvas overlay |  
| `src/styles/pixel-scroll-transition.css` | New | Styles for wrapper, canvas, sections |  
  
**Zero changes to any animation logic. Zero changes to any existing CSS rules. Two additive-only edits to existing files (one DOM element in `index.html`, one event dispatch in `portfolio.js`, one CSS rule in `layouts.css`).**  
  
---  
  
### Setup Commands in Order  
  
```bash  
cd /Users/sam/Desktop/Programming/portfolio_site  
  
# 1. Initialize Vite + React at project root  
npm create vite@latest . -- --template react  
# Select: "Ignore files and continue" when prompted about non-empty directory  
  
# 2. Install dependencies  
npm install  
  
# 3. Install Motion  
npm install motion  
  
# 4. Run dev server — existing site should still load at localhost:5173  
npm run dev  
```  
  
After running `npm create vite`, Vite generates `src/main.jsx` and `src/App.jsx` with placeholder content. These need to be replaced with the implementations described above. Vite also generates `src/index.css` — delete it, since all styles are already in `src/styles/main.css`.  
  
The Vite-generated `index.html` at the project root should be deleted. `vite.config.js` must set `root: 'src'` so Vite uses `src/index.html` as its entry instead.  
  
---  
  
### Critical Files for Implementation  
  
- `/Users/sam/Desktop/Programming/portfolio_site/src/index.html`  
- `/Users/sam/Desktop/Programming/portfolio_site/src/portfolio.js`  
- `/Users/sam/Desktop/Programming/portfolio_site/src/styles/layouts.css`  
- `/Users/sam/Desktop/Programming/portfolio_site/src/components/PixelScrollTransition.jsx`  
- `/Users/sam/Desktop/Programming/portfolio_site/vite.config.js`