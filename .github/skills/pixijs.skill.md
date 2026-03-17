---
name: PixiJS
description: "Build PixiJS games and prototypes with stable logical resolution, responsive viewport fitting, crisp rendering, and touch-first UI patterns."
---

# PixiJS Skill

This skill captures reusable lessons from practical PixiJS game and prototype work, especially around UI presentation, responsive fitting, asset handling, and touch ergonomics for game-first web and mobile projects.

## Capabilities
- **Application bootstrap** — configure `Application`, renderer resolution, `autoDensity`, resize handling, and scene/root containers for predictable rendering.
- **Logical resolution strategy** — define a fixed game-space resolution for layout and gameplay while fitting that space into many viewport sizes.
- **Viewport fitting & presentation** — use aspect-ratio-preserving contain scaling, full-bleed stage presentation, and mobile-first page shells.
- **Safe-area-aware HUD layout** — place UI around notches, dynamic islands, and gesture bars without distorting the game world.
- **Asset pipeline decisions** — choose when to use runtime tinting, when to regenerate source assets, and how to keep a clear source of truth.
- **Touch interaction ergonomics** — build tappable and scrollable interfaces with explicit hit areas, touch-target sizing, and predictable input zones.
- **Responsibility separation** — keep game world logic, viewport fitting, and page/chrome presentation as distinct layers.

## Best Practices
1. **Use a fixed logical resolution for layout and gameplay** — pick a stable internal size such as `1920x1080` or `390x844` and position world/UI elements in that coordinate space. Treat it as the source of truth for layout, camera math, and interaction mapping.
2. **Use `autoDensity` and device pixel ratio for crispness, not layout** — renderer resolution and DPR should improve sharpness on high-density displays, but they should not change where things are placed or how the game scales.
3. **Fit the game with contain scaling and preserved aspect ratio** — scale the logical stage uniformly so the whole intended composition stays visible. Prefer letterboxing/pillarboxing over stretching or non-uniform scaling.
4. **Separate game world, viewport fitting, and page presentation responsibilities** — the world decides what exists, the viewport decides how logical coordinates map to the screen, and the page shell decides centering, full-bleed behavior, overlays, and browser integration.
5. **Prefer game-first, mobile-first presentation** — avoid wrapping the canvas in desktop-style browser framing when the product is meant to feel like a game, kiosk, or installable mobile experience. Let the game own the screen and treat surrounding HTML as supporting chrome only when required.
6. **Make UI safe-area aware** — account for CSS safe-area insets and device cutouts when anchoring HUD, menus, and close buttons. Adjust overlay/UI padding first instead of shrinking or offsetting the entire game world.
7. **Use full-bleed presentation with disciplined overlay zones** — let backgrounds and non-critical art bleed to the screen edges while keeping important controls, text, and HUD content inside safe readable regions.
8. **Choose tinting vs asset regeneration deliberately** — runtime tinting works well for simple color variants, state changes, and lightweight theming. Regenerate or re-export source assets when variants require texture changes, lighting changes, gradients, baked text, or silhouette edits.
9. **Maintain a clear asset source of truth** — keep layered/editable originals and export rules in design/source directories so generated UI assets can be reproduced. Avoid treating hand-edited exported PNGs as canonical assets.
10. **Use explicit hit areas for touch UI** — do not rely on visual bounds alone for interactive sprites in dense or scrollable layouts. Define generous `hitArea`s and ensure targets are comfortably tappable at mobile scale.
11. **Design scrollable/touch UIs with interaction separation** — distinguish tap, drag, and scroll intent explicitly so nested buttons, lists, and carousels remain predictable. Test touch input on device-like viewports, not only with desktop mouse events.
12. **Test resize and orientation changes as first-class behavior** — verify portrait, landscape, short viewports, and tall notched devices early. A PixiJS layout that only works at one devtools size is not production-ready.

## Recommended Architecture
- **World layer** — gameplay containers, camera logic, particles, animation, and simulation in logical coordinates.
- **UI layer** — HUD, menus, modal flows, and touch controls anchored relative to the logical frame and safe areas.
- **Viewport layer** — computes scale, offset, and visible bounds using contain-fit rules.
- **Presentation layer** — HTML/CSS shell, background color, full-screen behavior, browser chrome policy, and embedding concerns.

## When to Use
- Building a new PixiJS game, prototype, vertical slice, or interactive demo
- Porting a Figma/mobile design into a PixiJS scene without distorting aspect ratio
- Making a game canvas feel native on mobile web or installable browser shells
- Reviewing PixiJS scene layout, resizing, HUD anchoring, and input ergonomics
- Deciding how to produce and manage reusable game UI assets
- Fixing blurry rendering, stretched layouts, clipped HUDs, or unsafe touch targets

## Anti-Patterns to Avoid
- ❌ Using viewport size directly as the game's logical layout size
- ❌ Mixing DPR/resolution concerns with layout math
- ❌ Stretching the stage independently on X and Y to fill the screen
- ❌ Letting browser-page framing dictate a game-first experience
- ❌ Solving safe-area issues by shrinking all content indiscriminately
- ❌ Re-exporting ad hoc image variants with no reproducible source asset workflow
- ❌ Using tiny visual sprites as the only tap target in mobile UI
