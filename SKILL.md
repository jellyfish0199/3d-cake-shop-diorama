---
name: cake-shop-diorama
description: >-
  Build a Three.js cake-shop diorama web game (Wonder Half Sugar Dreamland).
  Trigger ONLY on requests to build or recreate the cake-shop diorama game
  itself, such as 做一个甜品店立体模型游戏. Do not trigger on other requests,
  including generic Three.js work, other games, or unrelated UI/site tasks;
  those are handled by their own skills.
---

# Cake Shop Diorama

Build and maintain the modular Three.js diorama game. Work in the existing
project directory; never fork or restart from scratch unless the user asks for
a brand-new shop.

## Hard scope rule

Change ONLY what the user explicitly asked for. Never touch unrelated files,
features, visuals, copy, or settings "while you are in there". If a fix looks
like it needs an extra change, state it and wait for approval.

## Stack rules

- Vanilla ES modules + Three.js. No bundler. `build-static.mjs` copies files to
  `dist/`; all asset references stay relative so any static host works.
- Keep modules by concern: `shop.js` (building), `props.js` (outdoor props),
  `interior.js`, `pixel-world.js` (character sprites), `navigation.js`
  (walkable map), `season.js` (sky/seasons/sun/moon), `ui.js`, `music.js`,
  `main.js` (wiring + loop).
- Pixel sprites are upright `THREE.Mesh` planes (not Sprites) with
  nearest-filtered textures so they keep pixel edges and get depth-correct
  occlusion.
- After editing any `src/*.js`, syntax-check with
  `node --input-type=module --check < file.js`, then run
  `node build-static.mjs`.

## World layout (authoritative numbers)

- Shop footprint: x −7.4..5.6, z −7.6..−0.6; wall height 5.6; roof top 7.1;
  roof toppings reach y ≈ 10.5 (strawberry apex); door axis x −0.4.
- Platform: 18×18, walkable clamp x −8.05..8.05, z −8.05..7.8
  (`navigation.js` is the single source of truth).
- Blocked circles: ice-cream cart (3.9, 4.4, r 1.5), café table (1.2, 3.7,
  r 1.13), giant cake (−6, 3.0, r 1.9).
- Café set: table (1.2, 3.7); chairs at (−0.25, 4.15) and (2.55, 2.75).
- Player spawn (also post-checkout return): (−0.25, 0.05, 4.95) — directly in
  front of the left chair, facing the camera.
- Default exterior camera: position ≈ (14.8, 15.4, 29.6), target (−0.5, 2,
  −0.4), fov 36; interior fov 42.

## Sun & moon (fixed-world-anchor design)

Both bodies live at ONE fixed world anchor `(-14, 9, -12)` in `season.js`
(`celestialAnchor`), above the roof, behind-left of the shop. Every frame they
only billboard toward the camera (`quaternion.copy(camera.quaternion)`) and
keep fixed scales (sun 1.0, moon 1.05). Do NOT re-project them onto the screen
per angle: screen-following placement caused the sun to cross the wordmark and
the moon to "fall low" in top-down views. From steep down-views or deep zooms
they may leave the frame naturally — this is intended and user-approved.

## Viewport & UI

- Page is locked: `html/body` use `100dvh`, `body { position: fixed; inset: 0 }`.
  `#stage` and `#world-ui` are pinned to the visual viewport via CSS vars
  `--vv-x/--vv-y/--vv-w/--vv-h`, updated in `resize()` from
  `window.visualViewport` (resize + scroll events). This prevents the mobile
  toolbar-hide shift and the empty band at the bottom.
- Right-side vertical tool column: `#music-toggle`, `#view-toggle`,
  `#camera-reset` (class `sidebar-tools`). It is separate from the
  environment panel `.world-switcher`; both share the same top edge (desktop
  72px, mobile 64px). Desktop keeps the panel top-right; landscape ≤560px
  height keeps panel/tool separation.
- Loading veil `#loader`: sky-gradient background, `.loader-jelly` image
  (clamp(38px, 11.5vw, 50px)), gentle float animation, `pointer-events: none`,
  `draggable="false"`. Removed after textures load: frame ≥ 4 AND all
  `pixel.textures` have width > 0; 10s safety timeout. No pink dot anymore.
- Order flow stays: `idle → choosing → packed → completed`; one order per page
  session; closing while choosing resets to idle; checkout returns the player
  to the chair-front spawn.

## Audio & fonts

- Music `assets/audio/jelly-sea-dreams.opus` with `.mp3` fallback. Try
  autoplay, retry on first trusted gesture, persist preference in
  `localStorage.dreamland.music`, pause on `visibilitychange`. Never promise
  unmuted autoplay before a gesture.
- Brand font Allura (`assets/fonts/allura.ttf`, OFL license file included).
  Brand string is exactly `Wonder Half Sugar Dreamland`, single line.

## Player experience invariants

- Exact greeting from shopkeeper 薄荷喵呜:
  `欢迎光临Wonder Half Sugar Dreamland！今天您要点什么甜品？`
- Walk into the shop through the door to open the order dialog (talk button
  appears near the counter). No inverted-hull outlines anywhere. Totoro's
  umbrella uses unlit materials, casts/receives no shadow.
- Seasons: spring petals, strong summer sun, autumn maples (roof + ground),
  winter snow (380 instances) with accumulation. Day = blue sky/sun/clouds;
  night = deep blue, stars, warm shop lighting, moon.

## Workflow

1. Locate the change in the module map above; edit that file only.
2. `node --input-type=module --check < changed.js` for each changed file.
3. `node build-static.mjs` (builds `dist/`, ~32 runtime resources).
4. Serve and verify: `python3 -m http.server 8765` in the project root, then
   headless Chrome/Playwright (SwiftShader flags) screenshots at 1280×900,
   390×844, 844×390. Confirm the requested change AND that nothing else moved
   (panel position, spawn point, dialog, seasons).
5. Publish: `node /mods/neta/neta work publish web
   works/webs/sugar-dreamland-cake-shop --from works/webs/sugar-dreamland-cake-shop/dist`.
6. Verify the public origin loads with zero page errors and zero failed
   requests before reporting done. CDN caches per-URL: give a NEW versioned
   path when handing out the public link (see `references/deploy-cdn.md`).

## Anti-patterns (do not do)

- Do not mix indexed and non-indexed geometries in one merge (causes
  `morphAttributes` null crash) — `.toNonIndexed()` spheres first.
- Do not reintroduce screen-space sun/moon chasing or camera auto-rotation.
- Do not spawn the player inside a blocked circle or behind props.
- Do not return Three.js objects from Playwright `evaluate` (circular JSON).
- Do not hand-edit files under `dist/` — it is generated output.
- Do not claim support for untested browsers/devices; report only verified
  results.

## References

- `references/prompt-recipe.md` — one-shot prompt for reproducing this shop
  from scratch, and a per-feature prompt table.
- `references/deploy-cdn.md` — static deployment notes and the CDN cache rule.
- `assets/` — loader artwork used by the veil (copy with the template).
