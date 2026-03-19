# Bloodhorn — Game Design Document
### Casual Mobile RPG · Screen Mechanic Specs v2.0
**Scope:** 4 screens redesigned to match ohota.mobi reference UX  
**Stack:** PixiJS + TypeScript (frontend) · Spring Boot (backend)  
**Art style (LOCKED):** `"casual mobile RPG game, colorful anime-style art, vibrant illustration, clean bold outlines, bright dramatic lighting"`  
**Image model:** OpenAI `gpt-image-1`, quality `medium`  
**Portrait size:** 1024×1024 · **Thumbnail size:** 256×256

---

## Table of Contents
1. [Player Flow Overview](#1-player-flow-overview)
2. [Screen 1 — Landing (SplashScene + LoginScene)](#2-screen-1--landing-splashscene--loginscene)
3. [Screen 2 — Fight (TutorialFightScene)](#3-screen-2--fight-tutorialfightscene)
4. [Screen 3 — Tutorial Reward (TutorialRewardScene)](#4-screen-3--tutorial-reward-tutorialrewardscene)
5. [Screen 4 — Campaign Map (CampaignMapScene)](#5-screen-4--campaign-map-campaignmapscene)
6. [Card Mechanic Spec](#6-card-mechanic-spec)
7. [Boss Mechanic Spec](#7-boss-mechanic-spec)
8. [Attempt Counter System](#8-attempt-counter-system)
9. [Chapter I Lore](#9-chapter-i-lore)
10. [Tuning Table](#10-tuning-table)
11. [Measurable Success Criteria](#11-measurable-success-criteria)
12. [Open Questions & Tuning Variables](#12-open-questions--tuning-variables)
13. [Asset Generation Prompts](#13-asset-generation-prompts)
- [Appendix A — Scene Class Inventory](#appendix-a--scene-class-inventory)
- [Appendix B — Asset Requirements Summary](#appendix-b--asset-requirements-summary)
- [Appendix C — Backend Contract](#appendix-c--backend-contract)

---

## 1. Player Flow Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        FIRST SESSION                           │
│                                                                 │
│  [SplashScene]                                                  │
│      │  Title appears · tagline fades in                        │
│      ▼                                                          │
│  [LoginScene]                                                   │
│      │  Guest tap / social login                                │
│      │  Handoff: server issues session token                    │
│      ▼                                                          │
│  [TutorialFightScene]  ◄──── ONLY on first ever login ────      │
│      │  Tutorial boss fight (no attempt counter)                │
│      │  Player learns: tap card → deal damage → boss dies       │
│      ▼                                                          │
│  [TutorialRewardScene]                                          │
│      │  Gold burst animation · reward summary                   │
│      │  "Continuing in 5..." auto-advance                       │
│      ▼                                                          │
│  [CampaignMapScene]  ◄──── ALL subsequent sessions land here    │
│      │  Chapter I map · boss list · attempt counters            │
│      │  Player taps a boss entry                                │
│      ▼                                                          │
│  [FightScene]  ──── win ────►  [RewardScene]  ──► CampaignMap  │
│               └──── lose ───►  [DefeatScene]  ──► CampaignMap  │
└─────────────────────────────────────────────────────────────────┘

RETURNING SESSION:
  App open → SplashScene (logo + brief loading) → CampaignMapScene
```

**Handoff contracts between scenes**

| From | To | Data passed |
|---|---|---|
| LoginScene | TutorialFightScene | `userId`, `sessionToken`, `isFirstLogin: true` |
| TutorialFightScene | TutorialRewardScene | `rewardsBundle`, `xpGained`, `goldGained` |
| TutorialRewardScene | CampaignMapScene | — (navigate to chapter 1) |
| CampaignMapScene | FightScene | `bossId`, `chapterId`, `attemptIndex` |
| FightScene (win) | RewardScene | `rewardsBundle`, `newBossesUnlocked[]` |
| FightScene (lose) | DefeatScene | `bossId`, `damageDealt`, `attemptsLeft` |

---

## 2. Screen 1 — Landing (SplashScene + LoginScene)

### 2.1 Player Experience Goals
- **Feel:** "I just stepped into something epic." The first five seconds should feel cinematic and warm — not a loading screen.
- **Understand immediately:** This is a fantasy RPG about fighting monsters. The tagline and art make the genre unmistakable.
- **Zero friction to start:** One tap to play as guest; social login as optional upgrade. New players should not be stuck at a form.

### 2.2 SplashScene

**Asset:** `splash-bg.png` — served from `/images/splash-bg.png`  
*(See §13 for generation prompt)*

**Layout (portrait 390 × 844 reference)**
```
┌────────────────────────────────┐
│                                │
│   [Full-bleed BG art]          │
│   dramatic fantasy forest      │
│   clearing at dusk             │
│   warm amber/purple palette    │
│                                │
│                                │
│      🩸 BLOODHORN              │  ← game logo, centered, 72pt bold
│   "Hunt. Conquer. Ascend."     │  ← tagline, 18pt, cream/gold color
│                                │
│   [animated PixiJS particle    │
│    emitter: floating embers]   │
│                                │
│   ████ Loading... ████         │  ← slim progress bar, gold fill
│                                │
└────────────────────────────────┘
```

**Behaviour**
- Auto-advance to LoginScene once assets preloaded (or after 2 s minimum for feel)
- Ember particles: 20–30 slow-rising orange particles, alpha fade at top
- Logo entrance: scale from 0.7 → 1.0 over 600 ms, ease-out-back

### 2.3 LoginScene

**Layout**
```
┌────────────────────────────────┐
│   [same BG art, slightly       │
│    darkened overlay]           │
│                                │
│      🩸 BLOODHORN              │
│                                │
│  ┌──────────────────────────┐  │
│  │  [G] Continue with Google│  │  ← primary CTA button
│  └──────────────────────────┘  │
│  ┌──────────────────────────┐  │
│  │  🎮 Play as Guest        │  │  ← secondary CTA button
│  └──────────────────────────┘  │
│                                │
│  Already have an account?      │
│  [Email login ▸]               │  ← small text link, not prominent
│                                │
│  Terms · Privacy               │  ← legal footer, tiny
└────────────────────────────────┘
```

**UX Rules**
- "Play as Guest" is always visible and requires zero additional input
- Guest account upgraded silently in background on social-link later
- On successful auth: brief screen flash + transition to TutorialFightScene (first-time) or CampaignMapScene (returning)
- Error state: subtle red toast at top — never modal blocks

**Analytics events to fire**
- `splash_shown`, `login_screen_shown`, `login_method_selected`, `login_success`, `login_failed`

---

## 3. Screen 2 — Fight (TutorialFightScene)

> This spec covers both **TutorialFightScene** and standard **FightScene**. Tutorial-only rules are marked 🎓.  
> **Reference:** `ohota-current.png` — fight screen with "Безумный крестьянин" boss.

### 3.1 Player Experience Goals
- **Feel empowered immediately:** Three clearly distinct attack cards are visible the instant the fight begins.
- **Understand cause and effect:** Every tap on a card should produce a loud, visible, satisfying hit on the boss.
- **Feel tension:** The boss's HP bar should feel threatening even if the player is winning comfortably.
- **No dead air:** The whole fight takes 10–40 seconds; there is no waiting.

### 3.2 Layout (matches ohota.mobi reference exactly)

```
┌────────────────────────────────────────────────┐  ← 390px wide portrait
│ [boss thumb] BOSS NAME                   ♥ HP  │  ← HEADER BAR
│  64×64px     crimson text (#CC3333)      red   │    height: 64px, bg: #1A0000
├━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┤  ← red 3px separator
│                                                │
│         FULL-WIDTH LANDSCAPE ARENA ART         │  ← ARENA PANEL
│         file: arena-1.png (1536×1024 crop)     │    height: ~300px
│         (swamp/dungeon background, no UI)      │    bg fills full width
│                                                │
├────────────────────────────────────────────────┤
│ [player icon] You                       💙 HP  │  ← PLAYER BAR
│   64×64px     white text (#FFFFFF)      blue   │    height: 64px, bg: #0D0D1A
├━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┤  ← red 3px separator
│                                                │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐    │  ← CARD ROW
│   │          │  │          │  │          │    │    bg: #000000
│   │  [art]   │  │  [art]   │  │  [art]   │    │    cards evenly spaced
│   │          │  │          │  │          │    │    with flex/justify-around
│   └──────────┘  └──────────┘  └──────────┘    │
│   💀 55 power   ⚔️ 58 power   🎯 56 power      │  ← power badge below circle
│                                                │
└────────────────────────────────────────────────┘
```

**Pixel measurements (390px viewport reference)**

| Zone | Height | Width | Notes |
|---|---|---|---|
| Header bar | 64px | 100% | Dark crimson bg `#1A0000` |
| Red separator | 3px | 100% | Color `#CC2222` |
| Arena panel | 300px | 100% | Landscape crop of `arena-1.png` |
| Player bar | 64px | 100% | Dark bg `#0D0D1A` |
| Red separator | 3px | 100% | Color `#CC2222` |
| Card row | 200px | 100% | Black bg `#000000`; cards centered vertically |
| Card circle | 150px ⌀ | 150px | Touch target ≥ 88px (CSS: `min-width: 88px`) |
| Power badge | 28px tall | ~80px | Dark pill below circle, icon + number |

**Component details**

| Zone | Element | Spec |
|---|---|---|
| Header bar | Boss thumbnail | 64×64px, rounded-square 8px radius, dark border `#333` |
| Header bar | Boss name | Crimson `#CC3333`, bold, 18pt, left-aligned after thumb |
| Header bar | Boss HP value | Red ♥ icon + numeric, right-aligned, 16pt white |
| Header bar | Boss HP bar | Full-width 4px bar pinned to bottom of header, color `#CC2222` |
| Arena panel | BG art | `arena-1.png`, `object-fit: cover`, centered |
| Arena panel | Boss hit flash | White `#FFFFFF` overlay on arena at opacity 0.7, 200ms ease-out |
| Arena panel | Damage number | Floating gold `#FFD700` number (+55), rises 60px and fades over 600ms |
| Player bar | Player icon | 64×64px, rounded-square, dark border |
| Player bar | Label | "You" white `#FFFFFF`, 16pt |
| Player bar | Player HP value | Blue 💙 icon + numeric, right-aligned, 16pt white |
| Player bar | HP bar | Full-width 4px bar at bottom of player row, color `#2196F3` |
| Card row | Circle | 150px diameter, colored rim (3px stroke): blue for card 1, amber for card 2, olive for card 3 |
| Card row | Thumbnail art | `card-{n}-thumb.png` fills 90% of circle, object-fit cover, circular clip |
| Card row | Power badge | Dark pill `#1A1A1A` border `#444`, icon glyph + space + number, 14pt white |

### 3.3 Card Selection Mechanic

**Input → Process → Output loop**

```
Player TAP on one visible card slot
       │
       ├── if only that slot matches → single-card attack (consume 1 slot)
       │
       └── if 2 or 3 visible slots share that card type → combo attack
               consume every visible matching slot
               matching cards visibly merge into the tapped card before impact
       │
       ▼
Chosen card: scale pulse 0.85→1.0 (80ms), rim glow flash
Consumed combo cards: arc/slide into the chosen card, shrink, and fade
       │
       ▼
Resolve strike damage from the chosen attack card
progressTimeReductionSeconds = consumedCardCount * 2
       │
       ▼
Boss HP -= damage
Fight progress/time -= 2s per consumed card
Boss arena: white flash (200ms) + screen rumble (3px, 150ms)
Floating damage number appears above boss art
       │
       ▼
Boss HP bar animates to new value (300ms tween)
Consume matched slots from the local bag and rotate replacements from reserve
       │
       ├── if boss.HP > 0 → idle state, row returns to 3 visible slots (tutorial: instant)
       │
       └── if boss.HP ≤ 0 → VICTORY sequence
               Boss sprite: shake + fade (500ms)
               Screen flash white
               Transition to RewardScene
```

**Combo / timing rules**
- Single-card attack = consume 1 slot = **2s** progress/time reduction
- 2-card combo = consume 2 matching visible slots = **4s** progress/time reduction
- 3-card combo = consume 3 matching visible slots = **6s** progress/time reduction
- Every consumed matching slot rotates independently and draws its replacement from reserve in the same resolution step

**Tutorial guard 🎓**
- Cards are gated behind a pulsing arrow tooltip pointing at card 1
- Tooltip text: "Tap a card to attack!"
- After first tap: tooltip dismissed, all 3 visible slots freely tappable
- Tutorial uses the same 9-card local bag model as normal fights, but the boss does not counter-attack
- Tutorial boss has **reduced HP (100)** to guarantee completion in 2–4 taps

### 3.4 Win / Lose States

| State | Trigger | Visual | Next |
|---|---|---|---|
| **Win** | Boss HP reaches 0 | Gold screen flash, victory fanfare SFX, boss fade-out | → RewardScene |
| **Lose** | Player HP reaches 0 | Screen darkens, defeat overlay, "You were defeated" | → DefeatScene |
| **Tutorial Win** | Boss HP reaches 0 (tutorial) | Same as Win | → TutorialRewardScene |

> **Note:** In the initial tutorial fight, player HP does NOT decrease (boss does not counter-attack). This eliminates the possibility of tutorial failure and ensures 100% completion.

### 3.5 Standard Boss Counter-Attack (post-tutorial)

In non-tutorial fights, the boss retaliates after every player attack:

```
Player attacks
     │
     ▼
Boss counter-attack resolves after 500ms delay
     │
     ▼
Player HP -= boss.attackPower
Player HP bar animates
Screen edge red vignette flash (400ms)
     │
     └── if player.HP ≤ 0 → DEFEAT
```

---

### 3.6 Fight Hand / Card Thumbnail System

Each fight owns a **true 9-card local bag**. The bag is seeded once at fight start, then rotated locally throughout the fight.

**Fight-hand rules**
- At fight start, **3 card slots are active** in the visible row and **6 cards remain in reserve**
- Replacements rotate in from reserve whenever a slot is consumed
- Bag reset happens **only after the full 9-card cycle is exhausted**
- After every refill, the UI must return to **3 visible card slots**; consumed cards may be replaced by repeated card types, but the row must stay visually distinct as three separate slots
- On combo attacks, **all consumed matching slots rotate out** before the boss turn continues

**Fight start payload (authoritative seed)**
```
POST /api/v1/fight/start
Authorization: Bearer {sessionToken}
Body: { bossId, userId }
```

**Response DTO**
```ts
interface FightCardInstanceDto {
  instanceId: string;
  cardId: number;           // archetype id: 1, 2, or 3
  name: string;             // "Arcane Bolt", "Iron Cleave", "Piercing Shot"
  type: 'MAGIC' | 'WARRIOR' | 'RANGED';
  basePower: number;        // 55 / 58 / 56
  variance: number;         // ±5 / ±6 / ±5
  thumbnailUrl: string;     // "/images/cards/card-{cardId}-thumb.png"
}

interface FightHandDto {
  cycleSize: 9;
  active: [FightCardInstanceDto, FightCardInstanceDto, FightCardInstanceDto];
  reserve: FightCardInstanceDto[];    // length 6 at fight start
  resetWhenExhausted: true;
}
```

**Static asset locations**

| File | Path served | Size | Rim color |
|---|---|---|---|
| `card-1-thumb.png` | `/images/cards/card-1-thumb.png` | 256×256 | Cyan/blue `#00B4FF` |
| `card-2-thumb.png` | `/images/cards/card-2-thumb.png` | 256×256 | Amber/orange `#FF8C00` |
| `card-3-thumb.png` | `/images/cards/card-3-thumb.png` | 256×256 | Olive/green `#6DBE45` |

Assets reside in:  
`/Users/azenkov/Develop/projects/bloodhorn/bloodhorn-server/src/main/resources/static/images/cards/`

**Registration guarantee:** every new user account is assigned starter cards 1, 2, and 3. Tutorial fights must still initialize a full 9-card local bag and render 3 visible slots plus 6 reserve cards. Missing seed data should trigger an error overlay, not a broken UI.

---

### 3.7 Card Animate-In / Refill / Combo Merge Spec

Cards animate in from below when the fight scene first renders. After that, only the consumed slots refill from reserve; unaffected slots stay anchored so the row remains legible.

**Scene-load animate-in sequence**

```
t=0ms    Card 1: starts at y+80px, opacity 0
         Card 2: starts at y+80px, opacity 0  (80ms behind card 1)
         Card 3: starts at y+80px, opacity 0  (160ms behind card 1)
         │
t=0ms    Card 1 begins: translateY(+80 → 0) + opacity(0 → 1), 300ms ease-out-cubic
t=80ms   Card 2 begins: same tween
t=160ms  Card 3 begins: same tween
         │
t=460ms  All 3 visible slots at rest. Idle glow pulse begins (3s loop, opacity 0.7→1.0)
```

**Combo merge + refill sequence**
```
t=0ms      Tapped card pulses (80ms)
t=0–180ms  Other consumed matching cards arc into the tapped card, shrink to 0.6 scale, fade to 0
t=180–260ms Tapped card glows/brights to show combo intake
t=260ms    Strike resolves; boss HP/progress update
t=260–420ms Consumed slots pull replacement cards from reserve
t=420ms    Row is back to 3 visible slots
```

**Boss-turn re-enable (post-tutorial)**
```
All 3 visible slots snap to 0.5 opacity during boss turn (no translate offset)
On re-enable: opacity(0.5 → 1.0), 150ms ease-out
No translate — only opacity fade-in to avoid motion fatigue
```

**PixiJS implementation notes**
```ts
// Scene-load stagger
cards.forEach((card, i) => {
  card.alpha = 0;
  card.y += 80;
  gsap.to(card, {
    alpha: 1,
    y: card.y - 80,
    duration: 0.3,
    ease: 'power2.out',
    delay: i * 0.08
  });
});
```

---

## 4. Screen 3 — Tutorial Reward (TutorialRewardScene)

### 4.1 Player Experience Goals
- **Feel:** Pure celebration. The player just won their first fight and should feel rewarded before they know what anything means.
- **Understand what they earned:** Gold, XP, and any item should each have a clear label and icon.
- **Motivated to continue:** The auto-advance countdown creates curiosity ("where am I going?") rather than dread of choosing.

### 4.2 Layout

**Asset:** `victory-bg.png` — full-bleed background, served from `/images/victory-bg.png`  
*(See §13 for generation prompt)*

```
┌────────────────────────────────────┐
│                                    │
│   [FULL-BLEED victory-bg.png]      │  ← gold light burst, sparkles, runes
│   (semi-transparent dark overlay  │    overlay: rgba(0,0,0,0.35) for legibility
│    at 35% over the background)     │
│                                    │
│         ✨ VICTORY! ✨             │  ← 36pt bold, gold #FFD700, centered
│                                    │    glow shadow: 0 0 20px #FFD700
│    ┌──────────────────────────┐    │
│    │  🪙 + 50 Gold            │    │  ← reward rows, icon + value + label
│    │  ⭐ + 120 XP             │    │    slide in from bottom, 150ms stagger
│    │  🧪 [Starter Potion ×1]  │    │
│    └──────────────────────────┘    │
│                                    │
│   ┌──────────────────────────┐     │  ← CARD REWARD REVEAL
│   │  🃏 NEW CARD UNLOCKED!   │     │    fades in after reward rows settle
│   │                          │     │    (300ms after last reward row)
│   │   ┌────────────────┐     │     │
│   │   │  [card art]    │     │     │    card shown: card-1-thumb.png
│   │   │  card-1-thumb  │     │     │    150px circle, glow rim
│   │   └────────────────┘     │     │
│   │   Arcane Bolt  55 power  │     │
│   └──────────────────────────┘     │
│                                    │
│   [🪙 gold coin particle burst]    │  ← PixiJS emitter, 60–80 coins
│                                    │
│         Continuing in 5...         │  ← countdown timer, 20pt, cream
│                                    │
└────────────────────────────────────┘
```

**Component details**

| Element | Spec |
|---|---|
| Background | `victory-bg.png`, full-bleed, `object-fit: cover`; dark overlay `rgba(0,0,0,0.35)` |
| Victory header | "VICTORY!" text, gold `#FFD700`, glow shader `0 0 20px #FFD700`, 36pt bold |
| Gold coin particles | Emitter origin: center of screen, burst of 60–80 coins, arc outward then fall with gravity, fade at screen edge |
| Reward rows | Slide in from bottom (+60px → 0), opacity 0→1, staggered 150ms per row; 3 rows total |
| Card reward panel | Fade-in at t=reward_rows_done+300ms; centered; card circle 150px; rim glow matches card type |
| Card circle | `card-1-thumb.png`; circular clip; rim glow `#00B4FF` (cyan, card 1 type) |
| XP bar | Optional: thin XP progress bar below rows showing level-up fill |
| Countdown | Starts at 5, ticks down each second, auto-navigates to CampaignMapScene at 0 |
| Skip button | "Skip →" small button, top-right, allows early advance |

**Particle emitter parameters (PixiJS)**
```ts
{
  maxParticles: 80,
  lifetime: { min: 1.2, max: 2.0 },
  speed: { start: 600, end: 200 },
  rotationSpeed: { min: -180, max: 180 },
  spawnType: 'burst',
  spawnRect: { x: -10, y: -10, w: 20, h: 20 },
  gravity: 600,
  alpha: { start: 1, end: 0 }
}
```

**Card reward reveal sequence**
```
t=0ms     Victory header fades in (400ms)
t=400ms   Reward row 1 slides in (300ms)
t=550ms   Reward row 2 slides in (300ms)
t=700ms   Reward row 3 slides in (300ms)
t=1000ms  Card reward panel fades in (400ms) + coin particle burst fires
t=1400ms  Countdown timer appears: "Continuing in 5..."
t=6400ms  Auto-navigate to CampaignMapScene
```

---

## 5. Screen 4 — Campaign Map (CampaignMapScene)

### 5.1 Player Experience Goals
- **Feel:** A hub of opportunity — the player sees multiple options and feels the world has depth.
- **Clear progression:** Which boss comes next, which is locked, and why — all legible at a glance.
- **Daily loop anchor:** Attempt counters and the lore context make the player feel a pull to return tomorrow.

### 5.2 Layout (matches ohota.mobi reference — ohota-campaign.png)

**Asset:** `chapter1-bg.png` — full-bleed background behind boss entries, served from `/images/chapter1-bg.png`  
*(See §13 for generation prompt)*

```
┌──────────────────────────────────────────────────────┐  ← 390px portrait
│ [avatar] ✗511    💧61/72    🪙1196    🪙49           │  ← STATS BAR, 56px
│  40×40px  atk    blue HP   silver     gold           │    bg: #0D0D20
├──────────────────────────────────────────────────────┤  ← gold 2px divider
│                                                      │
│          ✦ CHAPTER I: ROTTING SWAMPS ✦               │  ← CHAPTER TITLE
│           decorative borders left + right            │    16pt bold, cream/gold
│                                                      │    bg: semi-dark overlay
├──────────────────────────────────────────────────────┤
│                                                      │
│  [chapter1-bg.png full bleed behind content]         │  ← SCROLLABLE BOSS LIST
│                                                      │
│  ┌──────────────────────────────────────────────┐    │  ← BOSS ENTRY 1 (unlocked)
│  │                                              │    │    full-width card
│  │  [boss portrait art, centered, ~280px tall]  │    │    bg: semi-dark gradient
│  │                                              │    │    overlaid on chapter1-bg
│  │  SERGEANT BEAR          💧 3                 │    │    boss name 18pt bold cream
│  │  🪙 1 of 7 attempts                          │    │    attempts 14pt muted gold
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  ┌──────────────────────────────────────────────┐    │  ← BOSS ENTRY 2 (unlocked)
│  │                                              │    │
│  │  [boss portrait art, centered, ~280px tall]  │    │
│  │                                              │    │
│  │  SWAMP WITCH             💧 2                │    │
│  │  🪙 1 of 2 attempts                          │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  ┌──────────────────────────────────────────────┐    │  ← BOSS ENTRY 3+ (locked)
│  │  [🔒 padlock icon, centered]                 │    │    dark overlay, 40% opacity
│  │  LOCKED LOCATION                             │    │    "Defeat [prev boss]
│  │  Defeat Swamp Witch to unlock               │    │     to unlock" tooltip
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  ┌──────────────────────────────────────────────┐    │  ← LORE TEXT BLOCK
│  │  The Rotting Swamps were once a cratered     │    │    14pt, cream/ivory
│  │  battlefield — now reclaimed by festering   │    │    line-height: 1.6
│  │  bog and ancient grudge...                   │    │    bg: rgba(0,0,0,0.5)
│  └──────────────────────────────────────────────┘    │    padding: 16px
│                                                      │
├──────────────────────────────────────────────────────┤
│ [mascot] │  HOME  │  HERO  │  CLAN  │  SHOP          │  ← BOTTOM NAV, 80px
│  sprite  │  icon  │  icon  │ 🔒icon │  icon          │    bg: #0A0A14
│          │ label  │ label  │ label  │  label         │
│          ┌──────────────────────────────────────┐    │
│          │ Forum  ·  Chat  ·  Settings           │    │  ← secondary row, 32px
│          └──────────────────────────────────────┘    │
└──────────────────────────────────────────────────────┘
```

**Pixel measurements**

| Zone | Height | Notes |
|---|---|---|
| Stats bar | 56px | Fixed top, never scrolls |
| Gold divider | 2px | Color `#C8A040` |
| Chapter title bar | 48px | Decorative ornament SVGs left + right |
| Boss entry card | ~300px | Portrait art ~280px + info row 48px |
| Gap between boss entries | 8px | Dark gap, chapter1-bg shows through |
| Locked entry card | 120px | Smaller — padlock + name only |
| Lore text block | variable | Auto-height based on text; max 180px; overflow: hidden |
| Bottom nav | 80px | Fixed bottom, never scrolls |
| Secondary nav row | 32px | Part of bottom nav, lighter text |

**Component details**

| Zone | Element | Spec |
|---|---|---|
| Stats bar | Avatar icon | 40×40px, rounded square, player faction art |
| Stats bar | Attack power | ✗ (crossed swords glyph) + numeric, white 14pt |
| Stats bar | HP | 💧 (blue orb) + `current/max` format, blue `#4FC3F7` |
| Stats bar | Silver | 🪙 (silver coin) + numeric, silver `#AAAAAA` |
| Stats bar | Gold | 🪙 (gold coin, different tint) + numeric, gold `#FFD700` |
| Chapter title | Text | Uppercase, decorative font or tracked-out caps, cream `#F5E6C8` |
| Chapter title | Ornaments | Small crossed-sword or crown SVG icons flanking the title |
| Boss entry | Portrait art | Full-width card area; boss portrait centered; see §7 for boss art desc |
| Boss entry | Boss name | 18pt bold, cream `#F5E6C8`, uppercase tracking |
| Boss entry | Power | 💧 + numeric (attack power), blue `#4FC3F7`, 14pt |
| Boss entry | Attempts | 🪙 + `used of max` format (e.g., "1 of 7 attempts"), muted gold `#C8A040`, 13pt |
| Boss entry | Glow border | Next-recommended boss: cyan glow `0 0 12px #00E5FF` border |
| Locked entry | Padlock | Large padlock SVG, centered, white 40% opacity |
| Locked entry | Overlay | Dark overlay `rgba(0,0,0,0.6)` over portrait area |
| Locked entry | Unlock hint | "Defeat [Boss Name] to unlock", 12pt muted, italic |
| Lore text | Block | Paragraph text, 14pt, cream `#E8D8C0`, centered, max-width 340px |
| Bottom nav | Mascot | ~80px tall character portrait, left-aligned, idle animation (8-frame loop) |
| Bottom nav | Nav items | 4 items: HOME · HERO · CLAN · SHOP; each: 48×48px icon + 12pt label |
| Bottom nav | Active tab | Highlighted icon, brighter label color |
| Bottom nav | CLAN locked | Show level badge "10 LV." overlay on clan icon until unlocked |
| Secondary nav | Items | Forum · Chat · Settings; 12pt, muted `#666`, centered row |

### 5.3 Boss Progression & Locking

```
CHAPTER I BOSS UNLOCK CHAIN

  [Tutorial Boss] — always unlocked, no attempt counter
       │ defeated
       ▼
  [Sergeant Bear]  — unlocked at start of Ch.1 (1/7 attempts)
       │ defeated
       ▼
  [Swamp Witch]    — unlocks after Sergeant Bear first defeat
       │ defeated
       ▼
  [Galia Durtanga] — unlocks after Swamp Witch first defeat
   (next recommended, cyan glow border)
       │ defeated
       ▼
  [Chapter I Boss] — FINAL BOSS, locked until Durtanga defeated
```

**Lock state rules**
- Locked boss: `opacity: 0.4` overlay + padlock icon + unlock hint text, tap shows tooltip "Defeat [prev boss] to unlock"
- Next-in-chain boss: cyan glow border `0 0 12px #00E5FF` + entry rendered at full brightness
- All attempts depleted: attempts row shows `0 / max · Resets in HH:MM` in red `#CC2222`; boss entry still tappable but shows blocking modal

**Attempts depleted modal (non-blocking overlay on boss entry)**
```
┌────────────────────────────────────┐
│  No attempts remaining             │
│  Resets in  04:22:15               │
│                                    │
│  [OK]      [Buy +1 Attempt 20💰]   │  ← v2 monetization hook
└────────────────────────────────────┘
```

---

## 6. Card Mechanic Spec

### 6.1 The Three Cards

| # | Name | Type | Icon | Base Power | Variance | Visual Theme |
|---|---|---|---|---|---|---|
| 1 | **Arcane Bolt** | Magic | 💀 skull glyph | 55 | ±5 | Blue circle, lightning mage art, cyan rim |
| 2 | **Iron Cleave** | Warrior | ⚔️ crossed swords | 58 | ±6 | Orange/red circle, armored warrior art, amber rim |
| 3 | **Piercing Shot** | Ranged | 🎯 target crosshair | 56 | ±5 | Yellow-green circle, target/arrow art, olive rim |

### 6.2 Fight-Hand Bag Model

```
Fight start:
  localBag.total = 9
  activeSlots = 3
  reserve = 6

On attack:
  consumeCount = number of visible matching slots used by the attack (1, 2, or 3)
  rotate each consumed slot from reserve

If reserve becomes empty after the 9-card cycle is fully consumed:
  reseed a new 9-card bag
  repopulate the row back to 3 visible slots
```

**Rules locked for v2.0**
- The fight hand is local to a single fight; it is not re-rolled after every tap
- Bag reset happens only after the full cycle is exhausted
- Refill must preserve 3 visible slots after every resolution step
- Combo attacks rotate every consumed matching slot, not just the tapped slot

### 6.3 Damage & Progress Formula

```
damage = card.basePower + Math.round((Math.random() * 2 - 1) * card.variance)
progressTimeReductionSeconds = consumedCardCount * 2

// Example: Arcane Bolt single-card attack → damage range 50–60, time reduction 2s
// Example: 2-card combo                → time reduction 4s
// Example: 3-card combo                → time reduction 6s
```

### 6.4 Card UX States

| State | Visual |
|---|---|
| **Idle** | Full opacity, subtle idle glow pulse (3s loop) |
| **Pressed** | Scale 0.85, immediate |
| **Executing** | Chosen card returns to 1.0 + rim flash (80ms) |
| **Combo Merge** | Consumed matching cards arc into the chosen card before strike resolve |
| **Refilling** | Consumed slots pull next cards from reserve; row must settle back to 3 visible slots |
| **Cooldown** (post-tutorial only) | Greyed rim + circular cooldown overlay, 0.6–1.2s |
| **Disabled** (boss turn) | All 3 visible slots: 0.5 opacity, non-interactive |

### 6.5 Tutorial Card Gating 🎓

```
Fight loads
    │
    ▼
All 3 cards: opacity 0.4, non-interactive
Pulsing arrow tooltip appears over Card 1
Tooltip text: "Tap a card to attack!"
    │
    ▼  (player taps Card 1)
Card 1 executes attack normally
Tooltip dismissed
All 3 cards: fully interactive, no further gating
```

---

## 7. Boss Mechanic Spec

### 7.1 Tutorial Boss

| Property | Value |
|---|---|
| Name | Swamp Goblin (tutorial) |
| HP | 100 |
| Attack Power | 0 (does NOT attack back) |
| Attempt limit | None (tutorial is infinitely retryable) |
| Defeat condition | Player HP ≤ 0 (impossible by design) |
| Required taps | 2–4 depending on variance |

### 7.2 Standard Boss Template

| Property | Description |
|---|---|
| `bossId` | Unique string identifier |
| `name` | Display name |
| `maxHp` | Total HP pool |
| `attackPower` | Damage dealt per counter-attack |
| `attackVariance` | ± variance on attack damage |
| `maxDailyAttempts` | Max attempts per 24h window |
| `unlocksAfter` | `bossId` of prerequisite boss, or `null` |
| `chapterId` | Chapter this boss belongs to |
| `arenaArt` | Asset key for arena background |
| `thumbnail` | Asset key for 64px header portrait |

### 7.3 Chapter I Boss Roster

| Boss | HP | Atk Power | Atk Variance | Daily Attempts | Unlocks After |
|---|---|---|---|---|---|
| Tutorial Goblin | 100 | 0 | 0 | ∞ | — |
| Sergeant Bear | 350 | 30 | ±8 | 7 | Tutorial |
| Swamp Witch | 500 | 45 | ±10 | 5 | Sergeant Bear |
| Galia Durtanga | 800 | 60 | ±12 | 3 | Swamp Witch |
| Rotting King (Ch.1 Final) | 1400 | 90 | ±15 | 2 | Galia Durtanga |

### 7.4 Fight Loop (post-tutorial)

```
Round start: row shows 3 active card slots, backed by a 9-card local bag
    │
    ▼
PLAYER TURN
    Player taps a visible card
    If 2 or 3 visible slots match that card type → combo consumes all visible matches
    Consumed combo cards merge into the chosen card
    Damage resolves → boss HP updates
    Progress/time reduces by 2s per consumed card
    Consumed slots rotate from reserve
    All visible slots disabled (0.5 opacity)
    │  500ms delay
    ▼
BOSS COUNTER-ATTACK TURN
    Boss attack resolves → player HP updates
    Red vignette flash
    Visible row re-enabled
    │
    ├── player.HP ≤ 0 → DEFEAT
    ├── boss.HP ≤ 0   → VICTORY (checked before boss attacks)
    └── else          → repeat from PLAYER TURN
```

> **Design note:** There is no per-fight "turn limit." A fight ends only by HP reaching zero. The 9-card bag adds cyclical texture without re-randomising the row after every single tap.

---

## 8. Attempt Counter System

### 8.1 Data Model

```ts
interface BossAttemptRecord {
  userId: string;
  bossId: string;
  attemptsUsedToday: number;   // incremented on FIGHT START (not fight end)
  maxAttemptsPerDay: number;   // configured per boss (see roster table)
  lastResetTimestamp: number;  // Unix ms, midnight server time (UTC)
}
```

### 8.2 Rules

1. **Deducted on entry** — one attempt is consumed when the player taps the boss card and the FightScene loads. A player who closes the app mid-fight does NOT get the attempt refunded.
2. **Reset time** — midnight UTC daily. Backend cron job resets `attemptsUsedToday` to 0.
3. **Display format** — campaign map shows `used / max` (e.g., `0 / 7`). When 0 attempts remain it shows `0 / 7 · Resets in 4h 22m`.
4. **Locked UI** — when attempts are depleted, boss card is tappable but shows a modal: "No attempts remaining. Resets in [HH:MM]." — includes optional "Buy +1 Attempt (20💰)" CTA (monetization hook, v2).
5. **Tutorial exception** — tutorial boss has no attempt record; it is always accessible.

### 8.3 API Contract (Backend)

```
POST /api/v1/fight/start
  Body: { bossId, userId }
  Response: { success, attemptsLeft, fightToken, fightHand } | { error: "NO_ATTEMPTS", resetsAt }
  Notes:
    - `fightHand` seeds the 9-card local bag for this fight
    - `fightHand.active.length === 3` and `fightHand.reserve.length === 6` at fight start

POST /api/v1/fight/end
  Body: { fightToken, outcome: "win"|"lose", finalPlayerHp }
  Response: { rewardsBundle, newUnlocks[] }
```

---

## 9. Chapter I Lore

### Chapter Title
**Chapter I: The Rotting Swamps**

### Lore Text (campaign map flavor, 2–3 sentences)

> *The Rotting Swamps were once a cratered battlefield — now reclaimed by festering bog and ancient grudge. Strange beasts have made this place their throne, grown fat on the lingering magic of old war. If you mean to forge a legend, you'll bleed for every inch of this rancid earth.*

### Boss Entry Flavor Lines (short, on boss card tap)

| Boss | Flavor line |
|---|---|
| Tutorial Goblin | *"Every hunter starts somewhere. Make it count."* |
| Sergeant Bear | *"He was a soldier once. The swamp made him into something worse."* |
| Swamp Witch | *"She doesn't curse enemies. She gives them exactly what they deserve."* |
| Galia Durtanga | *"His lair smells of rot and gold — equally intoxicating, equally deadly."* |
| Rotting King | *"He drowned once. It only made him angrier."* |

---

## 10. Tuning Table

> All numeric values are **starting estimates** for playtesting. Adjust based on session data and D1/D7 retention KPIs.

### 10.1 Fight Values

| Variable | Tuning Lever | Playtest Target |
|---|---|---|
| Tutorial boss HP | `tutorialBoss.maxHp` | Player wins in 2–4 taps; never more |
| Fight hand cycle size | `fightHand.cycleSize` | Locked at 9 local cards per fight |
| Active fight slots | `fightHand.activeSlots` | Locked at 3 visible slots |
| Reserve size at fight start | `fightHand.startingReserve` | Locked at 6 cards |
| Card 1 base power | `arcane.basePower` | ~55, feel slightly weaker than Card 2 |
| Card 2 base power | `warrior.basePower` | ~58, highest single-hit feel |
| Card 3 base power | `ranged.basePower` | ~56, mid-feel |
| Power variance | `card.variance` | ±5–6; just enough to feel "random" without frustration |
| Progress reduction / consumed card | `fight.progressSecondsPerCard` | Locked at 2s per consumed card |
| Sergeant Bear HP | `boss_sgt_bear.maxHp` | Player wins in 7–12 taps, ~45s fight |
| Sergeant Bear atk | `boss_sgt_bear.attackPower` | Reduce player HP by ~8% per round |
| Swamp Witch HP | `boss_swamp_witch.maxHp` | Extend fight to ~60–90s; escalate tension |
| Rotting King HP | `boss_rotting_king.maxHp` | Final fight: 3–4 min with full strategy |

### 10.2 Progression & Monetization Values

| Variable | Value | Notes |
|---|---|---|
| Daily attempt reset | 00:00 UTC | Keep consistent; use server time only |
| Sergeant Bear daily attempts | 7 | High — introductory boss, builds habit |
| Swamp Witch daily attempts | 5 | Moderate — mid-chapter pacing |
| Galia Durtanga daily attempts | 3 | Scarce — increase perceived value |
| Rotting King daily attempts | 2 | Rare — final boss should feel precious |
| Attempt refill cost (v2) | 20 gold / attempt | Soft monetization touch point |
| Tutorial reward — gold | 50 | Enough to feel significant; too much devalues shop |
| Tutorial reward — XP | 120 | Should get player to Level 2 |
| Reward screen countdown | 5 seconds | Long enough to read; short enough to not frustrate |

### 10.3 UX Timing Values

| Animation | Duration | Notes |
|---|---|---|
| Card scene-load animate-in | 300ms + 80ms stagger | translateY(+80→0) + opacity(0→1), ease-out-cubic, staggered per card |
| Card re-enable fade-in | 150ms | opacity(0.5→1.0) only; no translate after boss turn |
| Card tap scale pulse | 80ms | Must feel snappy, not laggy |
| Combo merge intake | 180ms | Consumed matching cards should visibly merge before strike resolve |
| Boss hit flash | 200ms | Longer = feels impactful |
| Damage float rise | 600ms | Standard mobile RPG feel |
| Screen rumble | 150ms, 3px | Subtle; disable for accessibility option |
| Boss death anim | 500ms | Fade + shake |
| Card cooldown (post-tutorial) | 600–1200ms | Adjust per boss difficulty tier |
| Reward row stagger | 150ms per row | Creates "loot drip" feel |
| Card reward reveal | 400ms fade | After all reward rows settle + 300ms |
| Coin particle burst duration | 2.0s max | Must complete before auto-advance |
| Reward auto-advance countdown | 5s | Empirically strong for "one more" behavior |

---

## 11. Measurable Success Criteria

| KPI | Target | Measurement Method |
|---|---|---|
| Tutorial completion rate | ≥ 90% of day-1 installs | `tutorial_fight_win` event / `tutorial_fight_start` event |
| Tutorial time | 30–90s median | `tutorial_fight_start` → `tutorial_reward_shown` delta |
| Campaign map reached (D0) | ≥ 88% of logged-in users | `campaign_map_shown` event |
| First real boss fought (D0) | ≥ 60% of campaign-map users | `fight_start` where `bossId ≠ tutorial` |
| First boss win rate | ≥ 70% | `fight_win` / `fight_start` for `Sergeant Bear` |
| Day-1 retention | ≥ 35% | Returning session within 24h |
| Day-7 retention | ≥ 12% | Returning session within 7 days |
| Daily attempt depletion rate (engaged users) | ≥ 50% use all attempts before midnight | `attempts_used = max_daily` events |
| Average fight duration | 30–120s | `fight_start` → `fight_end` delta |
| Reward screen interaction (skip button) | < 30% skip | `reward_skip_tapped` / `reward_shown` |

---

## 12. Open Questions & Tuning Variables

### High Priority (must resolve before implementation)

1. **Player HP pool** — What is the player's starting max HP? Reference shows 511. Tuning question: should it scale with level or stay flat in tutorial? **Assumption in this doc:** flat at 500 for tutorial, backend-determined for campaign.

2. **Boss counter-attack timing** — 500ms delay after player attack is assumed. Too fast = stressful; too slow = boring. **Playtest checkpoint:** Test 200ms, 500ms, 800ms with 5 internal players.

3. **Card cooldown post-tutorial** — Does each card have an individual cooldown, or do all 3 lock during boss counter-attack? **Recommendation:** all 3 lock during boss turn (simpler, clearer, matches reference behavior).

4. **Attempt consume timing** — Deduct on fight START or fight END? **Recommendation:** on START (prevents exploit of force-closing app), stated in §8.2. Confirm with product.

5. **Guest account persistence** — If a guest plays tutorial and closes app without linking account, are their progress and attempts saved? **Requirement for backend:** guest session must persist by device ID for ≥7 days.

### Medium Priority (can defer to first patch)

6. **Player death in tutorial** — Currently designed as impossible (boss does not attack). If this changes, add a "retry" button and explicit "You lost — try again!" message before the retry.

7. **Card upgrade path** — Cards have fixed base power in v1. Future: card upgrade system increases `basePower` per card level. Reserve `card.level` field in data model now.

8. **Chapter lore visibility** — The lore text lives on the campaign map. Should it be behind a tap ("read more")? **Recommendation:** show 2 lines collapsed, expand on tap to avoid cluttering the boss list.

9. **Daily reward content** — The daily reward banner is visible but its mechanic is not fully specced. Minimum viable: show chest on login, tap to collect [50 silver OR a card shard], 24h cooldown.

10. **Offline mode** — If the backend is unreachable, can the player still fight? **Recommendation for v1:** show "connection required" overlay; no offline fight support yet.

### Low Priority (post-launch)

11. Boss voiced lines on entry (short audio barks)
12. Victory screen shareable screenshot/GIF
13. Boss attack animations (currently just red vignette + player HP decrease)
14. Clan tab functionality (nav item exists but is locked)

---

## 13. Asset Generation Prompts

> **Model:** OpenAI `gpt-image-1` · **Quality:** `medium`  
> **Style anchor (prepend to every prompt):** `"casual mobile RPG game, colorful anime-style art, vibrant illustration, clean bold outlines, bright dramatic lighting"`  
> **Output path:** `/Users/azenkov/Develop/projects/bloodhorn/bloodhorn-server/src/main/resources/static/images/`

### 13.1 Background & Arena Assets

| File | Size | Destination path | Full prompt |
|---|---|---|---|
| `splash-bg.png` | 1024×1024 | `/images/splash-bg.png` | `casual mobile RPG game, colorful anime-style art, vibrant illustration, clean bold outlines, bright dramatic lighting, epic fantasy landscape with ancient stone castle silhouette on distant misty mountains, lush magical forest clearing at dusk, warm amber and deep purple twilight sky, glowing mystical particles floating upward, wide cinematic establishing shot, no characters or UI, rich detail, painterly` |
| `arena-1.png` | 1536×1024 | `/images/arena-1.png` | `casual mobile RPG game, colorful anime-style art, vibrant illustration, clean bold outlines, bright dramatic lighting, dark swamp dungeon battle arena, murky stagnant water reflecting torchlight, twisted dead trees with hanging moss, glowing green bioluminescent mushrooms, ancient crumbling stone pillars, eerie atmospheric fog, wide landscape orientation, battle arena environment background, no characters no UI, cinematic` |
| `victory-bg.png` | 1024×1024 | `/images/victory-bg.png` | `casual mobile RPG game, colorful anime-style art, vibrant illustration, clean bold outlines, bright dramatic lighting, triumphant victory celebration screen, radiant golden light burst from center, cascading gold coins and magical sparkles, ornate glowing rune circles, warm amber and brilliant gold color palette, celestial rays, epic fantasy reward moment, no characters no UI, full bleed` |
| `chapter1-bg.png` | 1024×1024 | `/images/chapter1-bg.png` | `casual mobile RPG game, colorful anime-style art, vibrant illustration, clean bold outlines, bright dramatic lighting, rotting swamps chapter map background, dark murky wetlands stretching to horizon, ancient battlefield partially submerged in stagnant bog, twisted mangrove trees silhouetted, eerie sickly green glow rising from water, heavy atmospheric mist, ominous and foreboding, dark blue-grey palette with green accents, top-down map aesthetic, no characters no UI` |

### 13.2 Card Thumbnail Assets

| File | Size | Destination path | Card type | Full prompt |
|---|---|---|---|---|
| `card-1-thumb.png` | 256×256 | `/images/cards/card-1-thumb.png` | Magic / Arcane | `casual mobile RPG game, colorful anime-style art, vibrant illustration, clean bold outlines, bright dramatic lighting, magic spell attack card portrait, robed sorcerer figure casting a blazing blue arcane lightning bolt, glowing cyan energy runes, dramatic electric sparks, deep blue and teal color palette, square card art format, centered composition suitable for circular crop, bold high-contrast colors, fantasy RPG card illustration` |
| `card-2-thumb.png` | 256×256 | `/images/cards/card-2-thumb.png` | Warrior / Melee | `casual mobile RPG game, colorful anime-style art, vibrant illustration, clean bold outlines, bright dramatic lighting, warrior melee strike card portrait, heavily armored warrior swinging a massive flaming sword in a powerful cleave attack, blazing orange and red fire trails, dramatic battle pose, warm amber and crimson color palette, square card art format, centered composition suitable for circular crop, bold high-contrast colors, fantasy RPG card illustration` |
| `card-3-thumb.png` | 256×256 | `/images/cards/card-3-thumb.png` | Ranged / Archer | `casual mobile RPG game, colorful anime-style art, vibrant illustration, clean bold outlines, bright dramatic lighting, ranged archer precision shot card portrait, agile archer drawing a glowing emerald energy arrow at full draw, radial target crosshair energy effect emanating from arrowhead, green and yellow-green color palette, dynamic motion lines, square card art format, centered composition suitable for circular crop, bold high-contrast colors, fantasy RPG card illustration` |

### 13.3 Asset Generation Checklist

```
[ ] splash-bg.png         1024×1024  → /images/splash-bg.png
[ ] arena-1.png           1536×1024  → /images/arena-1.png
[ ] victory-bg.png        1024×1024  → /images/victory-bg.png
[ ] chapter1-bg.png       1024×1024  → /images/chapter1-bg.png
[ ] card-1-thumb.png       256×256   → /images/cards/card-1-thumb.png
[ ] card-2-thumb.png       256×256   → /images/cards/card-2-thumb.png
[ ] card-3-thumb.png       256×256   → /images/cards/card-3-thumb.png
```

**Notes for generation:**
- All assets use `gpt-image-1` with `quality: "medium"` and `size` matching above table.
- `arena-1.png` uses size `1536x1024` (landscape, PixiJS arena panel crops to 390×300).
- Card thumbnails are generated as squares; PixiJS applies CSS `border-radius: 50%` and `object-fit: cover` at render time — center the subject in the frame.
- Do not include any text, UI chrome, or HUD elements in any generated image.

---

## Appendix A — Scene Class Inventory

> For handoff to `pixijs-prototype-specialist`:

| Scene Class | Route | Data In | Data Out |
|---|---|---|---|
| `SplashScene` | `/` | none | none |
| `LoginScene` | `/login` | none | `{ userId, token, isFirstLogin }` |
| `TutorialFightScene` | `/fight/tutorial` | `{ userId }` | `{ rewardsBundle }` |
| `TutorialRewardScene` | `/reward/tutorial` | `{ rewardsBundle }` | none (auto-navigate) |
| `CampaignMapScene` | `/campaign/1` | `{ userId, token }` | `{ selectedBossId }` |
| `FightScene` | `/fight/:bossId` | `{ bossId, fightToken, fightHand }` | `{ outcome, rewardsBundle }` |
| `RewardScene` | `/reward/:fightId` | `{ rewardsBundle }` | none |
| `DefeatScene` | `/defeat/:bossId` | `{ bossId, attemptsLeft }` | none |

---

## Appendix B — Asset Requirements Summary

> For handoff to art/technical-artist. **7 generated assets** + supporting UI assets.

**Generated assets (gpt-image-1, all in v2 scope)**

| File | Format | Generated Size | Rendered At | Path | Screen |
|---|---|---|---|---|---|
| `splash-bg.png` | PNG | 1024×1024 | full-bleed 390×844 | `/images/splash-bg.png` | SplashScene, LoginScene |
| `arena-1.png` | PNG | 1536×1024 | 390×300px crop | `/images/arena-1.png` | TutorialFightScene |
| `victory-bg.png` | PNG | 1024×1024 | full-bleed 390×844 | `/images/victory-bg.png` | TutorialRewardScene |
| `chapter1-bg.png` | PNG | 1024×1024 | full-bleed behind boss list | `/images/chapter1-bg.png` | CampaignMapScene |
| `card-1-thumb.png` | PNG | 256×256 | 150px circle | `/images/cards/card-1-thumb.png` | Fight + Reward |
| `card-2-thumb.png` | PNG | 256×256 | 150px circle | `/images/cards/card-2-thumb.png` | Fight |
| `card-3-thumb.png` | PNG | 256×256 | 150px circle | `/images/cards/card-3-thumb.png` | Fight |

**Supporting UI assets (manual / procedural)**

| Asset | Format | Size | Notes |
|---|---|---|---|
| Game logo | PNG (transparent) | 512×256 | Bold, blood-red + gold; can be SVG |
| Boss thumbnail (per boss) | PNG | 64×64 | Dark portrait style; 5 bosses = 5 files |
| Player icon | PNG | 64×64 | Player faction / avatar |
| Gold coin (particle) | PNG | 64×64 | Used in PixiJS emitter (TutorialRewardScene) |
| Boss portrait (per boss) | PNG | 800×600 | Full-body for campaign map entries; 5 files |
| Mascot (campaign nav) | PNG spritesheet | 512×512 | 8-frame idle loop; blonde female character |
| Lock icon | SVG | 48×48 | Padlock, used on locked boss entries |
| UI icon set | SVG | 48×48 each | HP drop (💧), attack (✗), silver coin, gold coin |

---

## Appendix C — Backend Contract

> Full API surface required for the 4 redesigned screens. Spring Boot endpoints.

### C.1 Fight Hand Payload

```
POST /api/v1/fight/start
  Auth:     Bearer {sessionToken}
  Body:     { bossId: string, userId: string }
  Purpose:  Start the fight, deduct attempts, and seed the per-fight 9-card local bag
  Response: FightStartDto | { error: "NO_ATTEMPTS", resetsAt: string }   // ISO-8601

interface FightCardInstanceDto {
  instanceId:   string;
  cardId:       number;   // archetype id: 1, 2, or 3
  name:         string;   // "Arcane Bolt" | "Iron Cleave" | "Piercing Shot"
  type:         'MAGIC' | 'WARRIOR' | 'RANGED';
  basePower:    number;   // 55 / 58 / 56
  variance:     number;   // 5 / 6 / 5
  thumbnailUrl: string;   // "/images/cards/card-{cardId}-thumb.png"
}

interface FightHandDto {
  cycleSize:          9;
  active:             [FightCardInstanceDto, FightCardInstanceDto, FightCardInstanceDto];
  reserve:            FightCardInstanceDto[];   // length 6 at fight start
  resetWhenExhausted: true;
}

interface FightStartDto {
  success:      true;
  attemptsLeft: number;
  fightToken:   string;
  fightHand:    FightHandDto;
}

// Registration guarantee: every new user receives cards 1, 2, 3.
// Tutorial and normal fights must still seed a full 9-card local bag.
```

### C.2 Fight Endpoints

```
POST /api/v1/fight/end
  Auth:     Bearer {sessionToken}
  Body:     { fightToken: string, outcome: "win" | "lose", finalPlayerHp: number }
  Response: { rewardsBundle: RewardsBundle, newUnlocks: string[] }

interface RewardsBundle {
  gold:    number;
  xp:      number;
  items:   ItemGrant[];
}

interface ItemGrant {
  itemId:   string;
  name:     string;
  quantity: number;
  iconUrl:  string;
}
```

### C.3 Campaign Map Endpoints

```
GET /api/v1/campaign/1/bosses
  Auth:     Bearer {sessionToken}
  Purpose:  Returns boss list for Chapter 1 with player's attempt state
  Response: BossEntryDto[]

interface BossEntryDto {
  bossId:          string;
  name:            string;
  attackPower:     number;
  maxDailyAttempts: number;
  attemptsUsedToday: number;
  isUnlocked:      boolean;
  unlocksAfter:    string | null;   // bossId of prerequisite
  portraitUrl:     string;          // "/images/bosses/boss-{n}-portrait.png"
  thumbnailUrl:    string;          // "/images/bosses/boss-{n}-thumb.png"
}
```

### C.4 Auth Endpoints (existing, unchanged)

```
POST /api/v1/auth/guest
  Body:     { deviceId: string }
  Response: { userId: string, sessionToken: string, isFirstLogin: boolean }

POST /api/v1/auth/google
  Body:     { googleIdToken: string }
  Response: { userId: string, sessionToken: string, isFirstLogin: boolean }
```

---

*Document version: 2.0 — Ready for handoff to `pixijs-prototype-specialist`*  
*Author: game-designer agent*  
*Updated: Visual redesign scope — 4 screens, 7 generated assets, card thumbnail system, exact ohota.mobi layout specs*  
*Next step: Asset generation by `technical-artist`, then scene implementation by `pixijs-prototype-specialist`*
