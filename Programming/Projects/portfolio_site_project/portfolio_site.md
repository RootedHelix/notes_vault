Related to [[projects]]

#portfolio_site 

## General Notes

### How to access Different Stages of the HTML Animation
**From the dev console** you can also call any stage directly:

```js
runSite({ startAt: 'loading' });    // start at loading bar, auto-advance to portfolio
runSite({ startAt: 'loading', autoAdvance: false });  // play just the loading bar, don't advance
startPortfolioStage();             // jump straight to cityscape (no transitions)
startLoadingBarStage(() => console.log('done'));  // run loading, custom callback
```

## Memory Footprint of the Website

### AI Response - Claude
Here's the breakdown of what your portfolio site actually weighs in network transfer, decoded GPU/CPU memory, and runtime cost. (The folder on disk is 29M, but most of that is dev artifacts — heap_snaps/ alone is 8M, and several PNGs in images/ aren't referenced anywhere in the code.)

#### Network transfer (what a visitor actually downloads)

Only files referenced from `index.html` and CSS get shipped. Looking at the code, that's:

|File|Size|
|---|---|
|`cyberpunk_city_night_sheet.png` (background, 4096×1024)|**1.75 MB**|
|`carSmoke.png` (sprite sheet, 2000×724)|**1.01 MB**|
|`Orbitron Black.ttf`|48 KB|
|`portfolio.js`|16 KB|
|`components.css`|16 KB|
|`terminal.js`|12 KB|
|`Web437_IBM_VGA_8x16.woff`|12 KB|
|`index.html`|8 KB|
|`main.css` + `terminal.css`|8 KB|
|`favChar.png` (walker sprite)|7 KB|
|Google Fonts CSS (Space Grotesk)|small, external|

**Total payload: ≈ 2.9 MB**, and ~95% of it is two PNGs.

These files in `images/` are **not loaded** by the site, just sitting in the repo: `carSmoke-original.png`, `coatChar.png`, `cyberpunk_city_night.png` (the non-sheet version), `favCharClean.png`, `loading_bar.png`, `1000_F_…jpg`, `holo_label.png.jpeg`. That's ~5 MB of dead weight in the folder you could move out or `.gitignore`.

#### Decoded image memory (the bigger number)

PNGs decompress to RGBA in browser memory. That's where the real cost lives:

- City sprite sheet 4096×1024×4 bytes = **~16 MB**
- Car smoke 2000×724×4 = **~5.8 MB**
- favChar 256×40×4 = ~40 KB

**Decoded image footprint: ≈ 22 MB resident.** This is what shows up in DevTools → Memory as image data, and what the GPU has to keep around as a texture for the cover-fit background.

#### Runtime / DOM cost

What runs once `startPortfolioStage()` fires:

- **120 rain-drop `<div>`s** created up front, plus **~120 splat divs/sec** spawned and removed (one per drop, on its interval). The splat creates DOM churn but each lives only 200ms, so steady-state extra nodes are small.
- **One `requestAnimationFrame` loop** for the bridge walker (cheap — just sets two CSS vars per frame).
- **One `setTimeout` chain** for the city twinkle (fires every 550–1200ms).
- **Pure-CSS animations running constantly**: car-smoke `steps(10)` cycle, scanline overlays, header pulse, title flicker, hologram beam, 120× `rainfall` keyframes. The 120 rain-drop animations are the heaviest paint cost — they're 1px lines so it's fine, but they do run forever.
- **Hologram text shadow** stacks 9 shadows + a `drop-shadow` filter on the title. Cheap once painted, but expensive on resize/repaint.

Heap-wise, expect **~30–50 MB JS heap** for a fresh tab once the portfolio stage settles (your existing `heap_snapshot_1.heapsnapshot` would tell you exactly — I can read it if you want a precise number).

#### Summary

|Metric|Value|
|---|---|
|Folder on disk|29 MB (most is dev artifacts)|
|Network transfer to a visitor|~2.9 MB|
|Decoded image memory|~22 MB|
|DOM nodes after boot|~140 (120 rain + ~20 UI)|
|Always-running animations|~125 (mostly the rain)|

The two PNG sprite sheets are by far your biggest lever. Re-encoding `cyberpunk_city_night_sheet.png` and `carSmoke.png` as WebP (or AVIF) typically cuts those by 60–80% with no visible quality loss, which would drop the page payload to under 1 MB and cut decoded memory roughly in half. Want me to do that conversion?

## 