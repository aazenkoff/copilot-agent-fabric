# Project Profile: dark-knight

> **Practical reference for agents working on this project.**  
> Keep this document updated when architecture, conventions, or assets change.

---

## Overview

| Field | Value |
|-------|-------|
| **Name** | dark-knight |
| **GitHub** | https://github.com/AZenkIO/dark-knight |
| **Local path** | `/Users/azenkov/Develop/projects/dark-knight` |
| **Description** | Dark Knight — a PixiJS v8 mobile-first RPG game prototype (360×640 px canvas) |
| **Canvas size** | 360×640 px (mobile portrait) |
| **Status** | Active prototype |

---

## Tech Stack

| Technology | Version | Role |
|------------|---------|------|
| [PixiJS](https://pixijs.com/) | `^8.17.1` | 2D rendering engine (WebGL/WebGPU) |
| TypeScript | `~5.9.3` | Primary language |
| Vite | v8 | Dev server & bundler |
| Node.js | (project default) | Build toolchain |
| Python 3 | system | Asset generation scripts |
| OpenAI `gpt-image-1` | API | AI asset generation |

---

## Dev Commands

```bash
# Start development server (usually resolves to port 5173–5177)
npm run dev

# Type-check + production build
npm run build

# Regenerate all 27 AI-generated PNG assets
python3 scripts/generate_assets_ai.py

# Regenerate only specific broken/updated assets
python3 scripts/fix_assets.py
```

> **Asset scripts** require `OPENAI_API_KEY` set in:
> - `/Users/azenkov/Develop/AI/ai-env-public0/.env`, **or**
> - `<project>/.env`

---

## Source Structure

```
dark-knight/
├── src/
│   ├── main.ts                  # App init, asset preloading, SceneManager bootstrap
│   ├── types/
│   │   └── index.ts             # Core types: SceneKey, Hero, Item, Stage, Act
│   ├── data/
│   │   ├── playerState.ts       # Mock player state (hero: Ironhide Lv15, coins, gems)
│   │   └── mockTaskRoll.ts      # Mock task roll data
│   ├── scenes/
│   │   ├── BaseScene.ts         # Abstract base class for all scenes (extends Container)
│   │   ├── SceneManager.ts      # Singleton scene router with fade transitions
│   │   ├── SplashScene.ts       # Splash screen scene
│   │   └── MainMenuScene.ts     # Main menu scene (HUD, nav bar, hero info)
│   ├── theme/
│   │   ├── colors.ts            # Color palette constants
│   │   └── typography.ts        # Text style constants
│   └── ui/
│       └── TaskRollModal.ts     # Task Roll overlay modal component
├── public/
│   ├── assets/                  # 27 PNG game assets (AI-generated via gpt-image-1)
│   └── icons.svg                # SVG icon sprite
└── scripts/
    ├── generate_assets_ai.py    # Full 27-asset AI generation script (gpt-image-1)
    ├── fix_assets.py            # Focused regeneration of 10 specific assets
    └── generate_assets.py       # Original PIL-based asset generator (legacy)
```

---

## Scene Architecture

### Scene Flow

```
SplashScene
    │
    └──(tap play button)──► MainMenuScene
                                 │
                                 └──(tap scroll icon)──► TaskRollModal (overlay)
```

### SceneManager Pattern

`SceneManager` is a **singleton** that owns all scene transitions.

```typescript
// Bootstrap (in main.ts)
const sceneManager = SceneManager.getInstance(app);
sceneManager.register('splash',   async () => new SplashScene(...));
sceneManager.register('mainMenu', async () => new MainMenuScene(...));
sceneManager.navigate('splash');

// Navigate from within a scene
SceneManager.getInstance().navigate('mainMenu');
```

**Key behaviours:**
- `register(key, factory)` — lazy-instantiates scenes on first navigation
- `navigate(key)` — fade out current scene → instantiate/enter next scene → fade in
- Fade transitions are driven by the **PixiJS ticker** (not `setTimeout`)
- `TaskRollModal` is an **overlay**, not a scene — it is added above the current scene's container and removed on close

### Scene Lifecycle

```
init()       ← called once after construction; build display objects here
onEnter()    ← called each time the scene becomes active; start animations
update(delta) ← called every ticker tick; game loop logic
onExit()     ← called before leaving; stop animations, clean up listeners
```

All scenes extend `BaseScene`, which extends PixiJS `Container`.

---

## Core Types (`src/types/index.ts`)

```typescript
type SceneKey = 'splash' | 'mainMenu'

interface Hero {
  id: string
  name: string
  level: number
  hp: number
  maxHp: number
  attack: number
  defense: number
  class: string
  rarity: string
}

interface Item {
  id: string
  name: string
  type: string
  rarity: string
  level: number
  stats: Record<string, number>
  slot: string
}

interface Stage {
  id: string
  name: string
  actId: string
  position: number
  unlocked: boolean
  completed: boolean
}

interface Act {
  id: string
  name: string
  description: string
  stages: Stage[]
  unlocked: boolean
}
```

---

## Mock Player State (`src/data/playerState.ts`)

```typescript
{
  hero: {
    name: 'Ironhide',
    level: 15,
    hp: 120,
    maxHp: 120,
    power: 7842,
  },
  coins: 3250,
  gems: 42,
}
```

---

## Asset Pipeline

All 27 PNGs in `public/assets/` are AI-generated via the OpenAI `gpt-image-1` API.

### Asset Groups

| Group | Count | Description | Gen Size | Typical Game Size |
|-------|------:|-------------|----------|-------------------|
| **A — Backgrounds** | 2 | Full-screen environment art (no UI chrome) | 1024×1536 | 360×640 |
| **B — Icons / Portraits** | 11 | Square icons and hero portrait | 1024×1024 | 20×20 – 120×120 |
| **C — UI Panels / Frames** | 10 | Nav bar, HUD bar, task cards, buttons | 1024×1536 or 1536×1024 | varies |
| **D — PIL Gradients** | 4 | Tiny shadows / vignettes (PIL-generated) | n/a | varies |

### Regenerating Assets

```bash
# Full regeneration (all 27 assets, costs API credits)
python3 scripts/generate_assets_ai.py

# Targeted fix (regenerate specific broken assets only)
python3 scripts/fix_assets.py
```

> ⚠️ Background images must be **pure environment art** — no HUD, text, or UI chrome baked in.

---

## Visual Design Language

### Aesthetic
- **Theme**: Dark fantasy — iron, steel, cold stone
- **Palette**: Dark blue, silver, charcoal; accent: steel blue (`#3A6EA8` range)
- **Feel**: Gritty, high-contrast, mobile-optimised

### Rules for AI Asset Prompts

| Element | Do ✅ | Avoid ❌ |
|---------|--------|---------|
| Backgrounds | Pure environment art, atmospheric depth | Embedded UI, HUD, text, watermarks |
| Icons | Isolated on dark bg, bold outlines, high contrast | Warm amber/gold tones, soft gradients |
| UI Panels | Dark stone/iron texture, silver beveled borders | Light or wooden textures |
| Accent colour | Steel blue (`#3A6EA8`) for active states, power bar, claim button | Warm yellows/reds as primary accent |
| Nav icons | Cold silver/steel metallic look | Amber or golden tones |

### PixiJS Rendering Notes

```typescript
// main.ts bootstrap — HiDPI / Retina support
const app = new Application();
await app.init({
  width: 360,
  height: 640,
  autoDensity: true,              // scales canvas CSS size automatically
  resolution: devicePixelRatio,   // crisp rendering on Retina/HiDPI screens
  backgroundColor: 0x0a0a0f,
});
```

---

## Key Development Conventions

1. **All scenes extend `BaseScene`** — never add a scene as a bare `Container`.
2. **Asset preloading** happens in `main.ts` via `Assets.load([...])` **before** `SceneManager` starts — scenes can assume assets are ready.
3. **Overlays (e.g. `TaskRollModal`)** are not scenes; they are added directly to `app.stage` above the current scene container and must clean up after themselves on close.
4. **Theme constants** live in `src/theme/` — never hardcode colours or text styles inline in scene/UI files.
5. **Types** are centralised in `src/types/index.ts` — extend there, not in individual scene files.
6. **Ticker-based transitions** — use the PixiJS ticker for all animations and fade effects; avoid `setTimeout`/`setInterval`.
7. **Mobile-first** — design and test at 360×640 px; the canvas does not resize.

---

## Related PRs / History

| PR | Status | Description |
|----|--------|-------------|
| #1 | Merged | Initial PixiJS v8 prototype scaffolding |
| #2 | Merged | Replace PIL assets with AI-generated images (`gpt-image-1`) |

---

## Quick Reference Card

```
Repo:       https://github.com/AZenkIO/dark-knight
Local:      /Users/azenkov/Develop/projects/dark-knight
Dev:        npm run dev  →  http://localhost:5173 (or 5174–5177)
Build:      npm run build
Canvas:     360×640 px  (mobile portrait, fixed)
Engine:     PixiJS v8  (TypeScript, Vite)
Assets:     public/assets/  (27 PNGs, AI-generated)
Regen:      python3 scripts/generate_assets_ai.py
Entry:      src/main.ts
Scenes:     src/scenes/
Types:      src/types/index.ts
Theme:      src/theme/colors.ts  +  src/theme/typography.ts
```
