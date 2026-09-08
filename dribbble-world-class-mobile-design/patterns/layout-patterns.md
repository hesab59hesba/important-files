# Mobile Layout Patterns Library

All patterns are mobile-first (375px → large phone → tablet/foldable). Design for thumb zone first; desktop patterns are not shrunk.

---

## Core App Layouts

### 1. Stack & Card (default for 90% of apps)
Single-column stack, every block is a card or grouped list with 16px inset, 12-16px gap between cards, section-gap 32px between groups.

```css
.app-stack { display: flex; flex-direction: column; gap: var(--section-gap); padding: 16px 0 100px; /* bottom pad for tab bar */ }
.app-stack > section { padding: 0 var(--gutter); }
.card-stack { display: flex; flex-direction: column; gap: 12px; }
```

Best for: Feed, home, detail, settings — the safest mobile default. If in doubt, use this.

### 2. Bottom Sheet Architecture (the mobile modal)
Primary content is full-bleed (map, chart, photo, video, product hero). Secondary content lives in a draggable sheet anchored to bottom with 2 detents.

```css
.canvas { position: relative; height: 100dvh; overflow: hidden; }
.canvas__content { position: absolute; inset: 0; } /* map/chart/photo */
.canvas__sheet { /* see component-recipes.md .sheet */ }
```

Detents:
- `peek` 24% — glanceable summary + handle
- `medium` 50% — default, main actions visible
- `large` 92% — full scroll, scrim 40%

Best for: Maps search, product variants, filters, comments, music now-playing, checkout summary. Keeps thumb zone.

**Rule**: Background (map/chart) never shrinks when sheet expands — sheet floats above with `largestUndimmedDetentIdentifier = .medium`. Never nest sheets.

### 3. Bento Mobile (home/dashboard highlights)
2-column grid with varied spans for overview — but mobile-bento is restrained (not 4-col web bento).

```css
.bento-mobile {
  display: grid; grid-template-columns: repeat(2, 1fr); gap: 12px;
  padding: 0 var(--gutter);
}
.bento-mobile .span-full { grid-column: span 2; }
.bento-mobile .span-half { grid-column: span 1; }
.bento-mobile .tall { grid-row: span 2; }
@media (max-width: 360px) { .bento-mobile { grid-template-columns: 1fr; } }
```

Patterns:
- Balance hero: `span-full` with 34px figure
- Stats row: 2× `span-half` with sparkline
- Quick actions: 4 pills in 2×2 grid (Send, Request, Top up, More)
- Promo: `span-full` tonal card

Best for: Home dashboards, fitness today, finance overview. Limit to one bento per app (home).

### 4. List-First (grouped lists)
iOS `insetGrouped` / M3 lists — the most scannable pattern for heterogeneous content.

```css
.screen-list { padding: 16px 0 100px; }
.list-section { margin-bottom: 24px; }
.list-section__header {
  font: 600 13px var(--font-body); color: var(--muted);
  letter-spacing: 0.04em; text-transform: uppercase;
  padding: 0 var(--gutter) 8px; margin: 0;
}
```

Sticky section headers on long lists (contacts, transactions). Swipe actions reveal on row.

Best for: Settings, transactions, messages, contacts, notifications. When content is a list, be a list — don't card-ify lists.

### 5. Carousel + Snap (horizontal discovery)
Horizontal snap with peek (16px of next card visible) signals scroll.

```css
.carousel {
  display: flex; gap: 12px; overflow-x: auto;
  scroll-snap-type: x mandatory; scrollbar-width: none;
  padding: 0 var(--gutter); scroll-padding: 0 var(--gutter);
}
.carousel::-webkit-scrollbar { display: none; }
.carousel__item { flex: 0 0 280px; scroll-snap-align: start; }
.carousel__item--story { flex: 0 0 80px; } /* story chips */
.carousel--peek .carousel__item:last-child { margin-right: var(--gutter); }
```

Best for: Stories, categories, product carousels, onboarding pages. Always show peek; dots or `X / Y` counter for orientation.

### 6. Map / Chart / Camera Canvas (full-bleed + floating controls)
Canvas bleeds edge-to-edge; controls float as glass pills/circles. Content peeks behind chrome.

```
┌─────────────────────────┐
│ [← Back]    [Search ○] │  ← floating glass top bar
│                         │
│      MAP / CHART        │  ← full-bleed canvas
│                         │
│  [─ ─ ─ ─ ─ ─ ─ ─ ─ ─] │  ← bottom sheet at peek (24%)
└─────────────────────────┘
```

Best for: Maps, trading charts, camera, media viewers. Controls are `glass-clear` over rich content, `glass-regular` over busy content.

### 7. Tab & Stack (app shell)
Bottom tabs (3-5) each owning its own navigation stack. Switching tabs preserves scroll + form state.

```css
.app-shell { display: flex; flex-direction: column; height: 100dvh; }
.app-shell__content { flex: 1; overflow-y: auto; padding-bottom: 96px; /* tab bar height + safe */ }
.app-shell__tabs { flex: none; }
```

Deep link must address `tab + stack position`. M3 tablet: tabs → navigation rail (left 80dp) or adaptable sidebar.

Best for: Any app with 3-5 peer destinations (Home, Search, Library, Profile). This is the mobile default shell.

---

## Section-Level Layouts

### 8. Split Hero (detail screens)
Image top (40-45vh), content below in stack. On scroll, image parallaxes slightly, top bar becomes glass.

Best for: Product detail, profile, article.

### 9. Concentric / Radial (focus moment)
Content arranged around a central orb/visual (streak, timer, avatar). Hero orb 180-220px, surrounding chips.

Best for: Streaks, timers, voice recording, AI orbs — moments of focus.

### 10. Asymmetric Breakout (editorial)
Content breaks gutter — image bleeds to edge while text stays in gutter. 8px overlap.

```css
.breakout { margin: 0 calc(-1 * var(--gutter)); }
.breakout img { width: 100%; border-radius: 0; }
@media (min-width: 430px) { .breakout { margin: 0; } .breakout img { border-radius: var(--radius-card); } }
```

Best for: Creative portfolios, lookbooks — use once per flow as signature.

### 11. Centered Column (reading, onboarding)
Single column, max-width 36-40ch, centered, generous line-height.

Best for: Empty states, onboarding value props, error screens.

### 12. Sticky Bottom Bar (checkout, composer)
Primary action pinned bottom, content scrolls behind it. Progress indicator above bar for multi-step flows.

```css
.screen--with-sticky { padding-bottom: 80px; }
.sticky-bar {
  position: fixed; left: 0; right: 0; bottom: 0;
  padding: 12px var(--gutter) calc(12px + env(safe-area-inset-bottom));
  background: var(--surface); border-top: 1px solid var(--border);
}
```

Best for: Checkout, forms, composers — any screen where losing the CTA off-screen loses the conversion (sticky CTA lifts 5-12% on mobile per Baymard/Lift).

---

## Typography Patterns (Mobile)

### Hero Headlines (thumb-readable, not web-hero huge)

| Treatment | Technique | When |
|---|---|---|
| Solid + tracking | `clamp(24px,6vw,32px)`, weight 700, tracking -0.02em | Data figures, balances |
| Gradient word | Gradient on single word via `background-clip:text` | Brand moment in hero (one word only) |
| Large price/figure | Tabular nums, 34px, primary color | Fintech balances |
| Eyebrow + title | Mono 11px uppercase tracking 0.08em + 22px title | Card titles |

### Text Hierarchy Helpers

| Element | Style | Use |
|---|---|---|
| Eyebrow | `font-mono 11px uppercase tracking 0.08em` muted | Section label |
| Caption | `13px muted` | Timestamp, helper |
| Overline | `11px mono uppercase` primary | Category |
| Stat | `clamp(28px,7vw,40px) display` weight 800 | Counters that animate on scroll |
| Callout quote | `20px serif italic` left border 3px accent | Testimonial highlight |

---

## Interaction Patterns (Mobile-Specific)

### Card Press / Hover Replacement (touch = press)

```
On touch: scale 0.98 + shadow-md + haptic light (80ms)
On commit (swipe threshold): icon morph + haptic medium
Never rely on hover to reveal critical info — hover enhances, never gates
```

### Navigation Variants

| Pattern | Use | Don't |
|---|---|---|
| Bottom tabs (3-5, labeled) | Peer destinations, frequent switching | Exceed 5, add "More", hide behind hamburger |
| Top segmented | Filtering *within* a screen (All / Active / Done) | Use for app-level nav |
| Bottom sheet | Filters, sort, share, confirm | Use for primary nav |
| Hamburger drawer | Secondary / rare items only (Help, Legal) | Bury primary actions |
| FAB | One primary action per screen (Compose) | Stack multiples, cover content |

### Form Patterns

```
Single-col: every field full-width (no side-by-side below 768px)
Progress: Step dots + "Step 2 of 3" caption (honest count, not surprise step 4)
Inline validation: error appears on blur with 13px red + icon; scroll to first error, not top
Keyboard: correct inputmode/type + autocomplete attributes; CTA stays above keyboard
```

---

## Visual Element Patterns

### Dividers (prefer space, then line)

```
Line: 1px border-top var(--border)
Hairline card divider: border-top inside card inset 16px from leading
Space: 32px gap — most Dribbble-worthy (no line)
```

### Decorative Elements (restrained on mobile — perf matters)

```
Gradient wash: single radial at 8% opacity behind hero (not blobs everywhere)
Grid dots: 24px repeat, 4% opacity, contained to hero header
Blur: budgeted — max 2 glass surfaces on screen at once (blur is GPU cost on mid-range Android)
```

### Image Treatment (mobile)

```
Radius: 12-16px card radius, 0 on edge-to-edge breakout
Aspect lock: 1:1 products, 16:9 hero, 4:3 thumbnails — consistent per screen
Double-tap zoom: expand from thumb point, not center
```

---

## Color Treatment (Mobile)

### Backgrounds

```
Solid: preferred — fastest paint, best contrast
Tonal surface: M3 surface-container steps (0 / high / highest) for depth without shadow
Glass: ONLY functional layer (tab bar, top bar, sheet, FAB over maps) — never feed/card
```

### Accent Application

```
Single accent: indigo/primary for CTAs, active tabs, focus rings
Semantic colors separate: success/warning/error never derived from brand accent
Tint (Liquid Glass): only primary action gets tinted glass via .glassProminent
```

---

## Safe Area & Responsive Helpers

```css
/* Safe area — never hard-code top/bottom padding */
.screen { padding-top: env(safe-area-inset-top); padding-bottom: calc(96px + env(safe-area-inset-bottom)); }
.top-bar { height: calc(56px + env(safe-area-inset-top)); padding-top: env(safe-area-inset-top); }

/* Fluid widths — no fixed px widths on mobile */
.card, .list-group, .sheet { max-width: 100%; }

/* Breakpoint ladder (mobile-first) */
@media (min-width: 375px) { /* phone */ }
@media (min-width: 430px) { /* large phone / Max */ .gutter { --gutter: 20px; } }
@media (min-width: 768px) { /* tablet / foldable open */ .tab-bar { rail mode } .grid-auto { 3 cols } }
@media (min-width: 1024px) { /* desktop preview of mobile app — center phone frame */ }

/* Foldable inner (842×595-ish) — treat as tablet */
@media (min-width: 700px) and (max-width: 900px) and (min-height: 700px) { /* foldable open */ }

/* Fluid type */
.text-fluid { font-size: clamp(15px, 2vw + 0.5rem, 18px); }
```

---

## Dark / Light Strategy (Mobile)

```css
/* System preference */
@media (prefers-color-scheme: dark) {
  :root { --bg: #08080c; --surface: #121214; --text: #f1f1f3; /* semantic override, not invert */ }
}
/* Manual toggle */
html[data-theme="dark"] { --bg: #08080c; /* ... */ }
/* Hybrid recommended: default = Midnight Pro for fintech/dev, Aurora/light for wellness/commerce */
```

On OLED, dark saves battery — default dark for fintech/trading/dev, light default for lifestyle/commerce/reading. Always offer Increase Contrast + Reduced Transparency paths.

