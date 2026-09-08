# Token System Examples by App Category (Mobile)

Each example is a complete mobile token block: colors (with dark override + glass), spacing, radius, typography, elevation, motion/haptics, and safe-area. Copy-paste and tune. All pass WCAG AA 4.5:1 for body and 3:1 for large text. All honor `prefers-reduced-transparency` and `prefers-reduced-motion`.

> Rule: Dark mode is a second semantic token set — not an invert. Glass tokens are functional-layer only (tab bar, top bar, sheet, FAB). Never on cards/list rows.

---

## Category: Fintech / Banking

### Subject: Neobank "Vault" — everyday spending + savings

- **Audience**: 22-38, urban professionals, iOS-first but adaptive to M3 Expressive
- **Job**: Check balance, move money, understand spending in <5s
- **Mode**: Trust / Finance
- **Platform soul**: iOS 26 Liquid Glass primary, M3 Expressive adaptive shell
- **Signature**: Drag-to-pay slider with haptic ticks per $50 step + balance card that fans on long-press to reveal vaults. On Android, same interaction with Material 3 shape morph (pill → expanded).

```css
/* Token System — Vault (Midnight Pro + Liquid Glass) */
:root {
  --bg: #08080c;              /* near-black, not #000 */
  --surface: #121214;
  --surface-raised: #1c1c1f;
  --surface-container: #1e1e22;
  --text: #f1f1f3;
  --muted: rgba(255,255,255,0.58); /* 4.6:1 on surface → AA */
  --border: rgba(255,255,255,0.08);
  --overlay-scrim: rgba(0,0,0,0.4);

  --primary: #6366f1;         /* indigo — trustworthy, not generic blue */
  --primary-pressed: #4f46e5;
  --on-primary: #ffffff;
  --accent: #22d3ee;          /* cyan secondary — highlights, not decoration */
  --success: #22c55e;
  --warning: #fbbf24;
  --error: #f87171;

  --glass-regular: rgba(28,28,31,0.68); /* functional layer only */
  --glass-clear: rgba(28,28,31,0.42);
  --glass-border: rgba(255,255,255,0.08);
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.4);
  --shadow-md: 0 8px 24px rgba(0,0,0,0.5);
  --shadow-glass: 0 8px 32px rgba(0,0,0,0.45), inset 0 1px 0 rgba(255,255,255,0.06);

  --space-1: 4px; --space-2: 8px; --space-3: 12px; --space-4: 16px;
  --space-6: 24px; --space-8: 32px; --space-12: 48px;
  --gutter: 16px; --section-gap: 32px;
  --radius-card: 16px; --radius-sheet: 20px; --radius-full: 999px;

  --font-display: "SF Pro Display", system-ui;
  --font-body: "SF Pro Text", system-ui;
  --font-mono: "SF Mono", ui-monospace;
  --text-display-lg: clamp(28px, 7vw, 34px); /* balance figure */
  --text-body-lg: 17px; --text-label-md: 13px;
  --tracking-tight: -0.02em;

  --duration-fast: 150ms; --duration-base: 250ms; --ease-out: cubic-bezier(0,0,0.2,1);
  --ease-spring: cubic-bezier(0.34,1.56,0.64,1);
  --safe-top: env(safe-area-inset-top); --safe-bottom: env(safe-area-inset-bottom);
}
/* Light alt for screenshots/marketing — not default */
@media (prefers-color-scheme: light) {
  :root {
    --bg: #f5f5f7; --surface: #ffffff; --surface-raised: #ffffff;
    --text: #1d1d1f; --muted: #6e6e73; --border: rgba(0,0,0,0.06);
    --glass-regular: rgba(255,255,255,0.72); --glass-border: rgba(255,255,255,0.5);
  }
}
@media (prefers-reduced-transparency: reduce) {
  :root { --glass-regular: var(--surface); --glass-clear: var(--surface); }
}
```

**Typography**: SF Native (SF Pro Display + SF Pro Text + SF Mono). Numbers tabular: `font-variant-numeric: tabular-nums`. Balance 34px / 600, tracking -0.02em.
**Layout**: Bento Mobile (balance hero full-span + 2×2 quick-action pills) + Grouped List for transactions. Bottom sticky CTA for transfers.
**Why it works**: Near-black OLED saves battery, indigo is fintech-trusted (avoids 200-290° violet slop band by leaning indigo-cyan), glass only on tab bar/top bar.

---

### Subject: Trading app "Tide" — charts-first

- **Audience**: Active traders
- **Job**: Scan chart, place order without mis-tap
- **Mode**: Trust / Finance (data-dense)
- **Platform soul**: M3 Expressive (dynamic chart accents) but iOS build uses Liquid Glass for chart toolbars

```css
:root {
  --bg: #0a0a0f; --surface: #14141a; --surface-raised: #1f1f27;
  --text: #e8e8f0; --muted: rgba(232,232,240,0.6);
  --border: rgba(255,255,255,0.08);
  --primary: #0ea5e9; /* sky — price action */
  --accent: #10b981;  /* green only for gain, not generic accent */
  --loss: #ef4444;
  --glass-regular: rgba(20,20,26,0.72);
  --radius-card: 12px; /* tighter for dense data */
  --text-body-lg: 17px; --text-mono: 13px;
}
```

**Signature**: Full-bleed chart canvas with floating glass period chips (1D/1W/1M) that snap with haptic `selection`.
**Layout**: Map/Table Canvas — chart floats under glass toolbar + bottom sheet for order ticket.

---

## Category: Health & Fitness

### Subject: Running coach "Pace" — daily training

- **Audience**: Runners 25-44, one-handed logging mid-run
- **Job**: Start run with one thumb, review progress
- **Mode**: Health & Fitness
- **Platform soul**: Liquid Glass (iOS) + M3 tonal (Android) — shared Nordic Frost semantic

```css
:root {
  --bg: #f0f2f5; --surface: #ffffff; --surface-raised: #ffffff;
  --text: #1d2939; --muted: #667085; /* 4.7:1 */
  --border: #e2e8f0;
  --primary: #0ea5e9; --primary-pressed: #0284c7;
  --accent: #6366f1; --success: #16a34a;
  --glass-regular: rgba(255,255,255,0.72);
  --radius-card: 20px; --radius-sheet: 24px;
  --font-display: "General Sans", system-ui; --font-body: "Inter", system-ui;
  --text-display-lg: clamp(28px, 7vw, 32px);
  --space-4: 16px; --gutter: 16px;
}
@media (prefers-color-scheme: dark) {
  :root {
    --bg: #0b1220; --surface: #121a2b; --surface-raised: #1a2642;
    --text: #e8eef8; --muted: rgba(232,238,248,0.62);
    --border: rgba(255,255,255,0.08);
    --glass-regular: rgba(18,26,43,0.68);
  }
}
```

**Typography**: Geometric (General Sans / Inter) — warm but precise. Body 17px for outdoor legibility.
**Signature**: Activity ring hero that breathes (2s ease-in-out, paired with soft haptic pulse on complete) + log buttons 48dp with `light` tick per +1.
**Layout**: Bento Mobile for Today (ring full + 2 stats half) + Stack & Card for history. All CTAs bottom-anchored for glove/thumb use.

---

### Subject: Meditation "Haven" — calm streaks

```css
:root {
  --bg: #faf8f3; --surface: #ffffff; --text: #2c2825; --muted: #8a8078;
  --border: #e7e5e4;
  --primary: #b45309; /* amber — warm, not clinical */
  --accent: #a16207; --glass-regular: rgba(250,248,243,0.80);
  --radius-card: 24px; /* softer */
  --text-display-lg: 32px; --leading-relaxed: 1.625;
}
```

**Signature**: Breathing circle that scales 0.85→1.05 with 4-7-8 timing + haptic inhale/exhale cues.
**Layout**: Centered Column with generous section-gap (48px), single-column, no bento.

---

## Category: Social / Community

### Subject: Audio social "Campfire" — live rooms + async clips

- **Audience**: Creators 18-34
- **Job**: Discover rooms, tap to listen, react without typing
- **Mode**: Expressive Consumer
- **Platform soul**: Midnight Pro glass — dark listening UI

```css
:root {
  --bg: #0c0a14; --surface: #16131f; --surface-raised: #1f1b2e;
  --text: #f0ebff; --muted: rgba(240,235,255,0.58);
  --border: rgba(255,255,255,0.08);
  --primary: #8b5cf6; --primary-pressed: #7c3aed;
  --accent: #f472b6; --accent-2: #22d3ee;
  --glass-regular: rgba(22,19,31,0.68);
  --glass-clear: rgba(22,19,31,0.42);
  --radius-card: 20px; --radius-full: 999px;
  --font-display: "Space Grotesk", system-ui; --font-body: "Inter", system-ui;
}
```

**Signature**: Mini-player glass bar that expands via shared-element into full room with waveform reacting to audio (parallax + `selection` haptic per speaker change).
**Layout**: Stack & Card feed + horizontal snap carousel for live rooms (peek 16px).

---

### Subject: Neighborhood app "Porch" — trust-first community

```css
:root {
  --bg: #fefcf8; --surface: #ffffff; --text: #1e1b4b; --muted: #6b7280;
  --border: #e5e7eb;
  --primary: #f97316; --accent: #34d399;
  --glass-regular: rgba(255,255,255,0.75);
  --radius-card: 16px;
}
```

**Signature**: Pull-to-refresh *becomes* the composer (sheet peeks at 15% with grabber, spring to 50% on commit).
**Layout**: List-First (grouped posts) + bottom sheet for comments. Verified badge is semantic token, not color-only.

---

## Category: Commerce / Marketplace

### Subject: Fashion marketplace "Atelier" — editorial shopping

- **Audience**: 24-40, style-driven, pays for taste
- **Job**: Browse → love → buy without friction
- **Mode**: Commerce
- **Platform soul**: iOS Liquid Glass over editorial imagery — content stays opaque

```css
:root {
  --bg: #f8f6f2; --surface: #ffffff; --surface-raised: #ffffff;
  --text: #1c1917; --muted: #78716c;
  --border: #e7e5e4;
  --primary: #1c1917; --on-primary: #ffffff; /* luxury: near-black primary */
  --accent: #b45309; /* caramel */
  --glass-regular: rgba(248,246,242,0.75);
  --glass-clear: rgba(255,255,255,0.45); /* only over lookbook photos */
  --radius-card: 12px; --radius-sheet: 20px;
  --font-display: "Playfair Display", serif; --font-body: "General Sans", sans-serif;
  --text-display-lg: clamp(24px, 6vw, 28px);
}
```

**Signature**: Product scrub-to-rotate (drag on image) + double-tap zoom that expands from thumb point; variants live in bottom sheet so image stays full-bleed.
**Layout**: 2-col product grid (12px gap, 1:1 images) + sticky bottom checkout bar (56px, full-width) with haptic `medium` on add.

---

### Subject: Food delivery "Supper" — 20-min habit

```css
:root {
  --bg: #14100d; --surface: #1f1a16; --surface-raised: #2a231e;
  --text: #f5efe8; --muted: rgba(245,239,232,0.58);
  --border: rgba(255,255,255,0.08);
  --primary: #ef4444; --accent: #f59e0b;
  --glass-regular: rgba(31,26,22,0.68);
  --radius-card: 16px; --font-display: "Nunito", sans-serif;
}
```

**Signature**: Cart bottom sheet that bounces (`spring` 300ms) on add + live rider map as floating glass pill.
**Layout**: Search glass bar floating over restaurant hero + carousel sections (cuisines horizontal) + list-first for menu.

---

## Category: Productivity / Crafted Utility

### Subject: Notes "Field" — fast capture + triage

- **Audience**: Power users, keyboard + thumb equally
- **Job**: Capture thought in <2s, find it later
- **Mode**: Crafted Utility
- **Platform soul**: Aurora (light default, respectful of system light/dark)

```css
:root {
  --bg: #f8f7f4; --surface: #ffffff; --surface-raised: #ffffff;
  --text: #0f172a; --muted: #64748b;
  --border: rgba(15,23,42,0.08);
  --primary: #7c3aed; --accent: #06b6d4;
  --glass-regular: rgba(255,255,255,0.72);
  --radius-card: 16px; --radius-full: 999px;
  --font-body: "Inter", system-ui; --font-mono: "JetBrains Mono", monospace;
  --text-body-lg: 17px; --text-label-md: 13px;
}
```

**Signature**: Card expands into detail via shared-element (FLIP 260ms) + swipe actions with icon morph + `light` haptic. Command palette is a bottom sheet with blur scrim.
**Layout**: List-First (grouped by Today/This Week) + sticky bottom composer 56dp. Empty = ghost row preview, not illustration.

---

### Subject: Calendar "Sundial" — time-block planner

```css
:root {
  --bg: #fefcf8; --surface: #ffffff; --text: #1e1b4b; --muted: #6b7280;
  --primary: #6366f1; --accent: #f59e0b;
  --glass-regular: rgba(255,255,255,0.72);
  --radius-card: 16px;
}
```

**Signature**: Time-grid with drag-to-create block (haptic tick per 15-min snap, spring settle).
**Layout**: Stack with horizontal day snap + bottom sheet for event details.

---

## Category: AI-Native

### Subject: Research assistant "Margin" — read, ask, synthesize

- **Audience**: Knowledge workers
- **Job**: Ask question → get cited answer with sources
- **Mode**: Expressive Consumer (but trust-constrained)
- **Platform soul**: Liquid Glass for answer cards (depth) — restrained to one surface

```css
:root {
  --bg: #0c0a1a; --surface: #14112a; --surface-raised: #1e1a3a;
  --text: #e8e4f5; --muted: rgba(232,228,245,0.58);
  --border: rgba(255,255,255,0.08);
  --primary: #8b5cf6; --accent: #d946ef; --accent-2: #06b6d4;
  --glass-regular: rgba(20,17,42,0.68); /* only on answer cards floating over source list */
  --radius-card: 20px; --radius-sheet: 24px;
  --font-display: "Satoshi", system-ui; --font-body: "Inter", system-ui;
}
```

**Signature**: Adaptive prompt-chip row that reorders by context (uses `selection` haptic on reorder) + answer card with subtle parallax tilt (1-2 layers, respects reduced-motion).
**Layout**: Stack & Card (query at bottom thumb zone, answers above, sources as chips). Privacy dots green/red per capability inline, not buried in settings.

---

## Category: Gaming / Playful

### Subject: Daily puzzle "Grain" — one puzzle a day

- **Audience**: Casual, streak-driven
- **Job**: Play today, keep streak, miss nothing
- **Mode**: Game / Playful
- **Platform soul**: Pastel Pop (light) — haptics as UI

```css
:root {
  --bg: #fefcf8; --surface: #ffffff; --text: #1e1b4b; --muted: #7c78a0;
  --border: #e9e5f5;
  --primary: #f472b6; --accent: #a78bfa; --accent-2: #34d399;
  --glass-regular: rgba(255,255,255,0.72);
  --radius-card: 24px;
  --font-display: "Nunito", sans-serif; --font-body: "Poppins", sans-serif;
  --text-display-lg: 32px; --text-body-lg: 18px; /* larger for play */
}
```

**Signature**: Streak flame that *licks* on win + confetti with `success` haptic burst, clear variant sheets over puzzle art.
**Layout**: Centered Column (puzzle canvas) + bottom toolbar (glass, 3 actions max, 48dp targets).

---

---

## Template: Custom Category (use when no predefined example fits — mandatory after research)

Copy this scaffold. Do **not** invent hues from memory — fill every comment with a 1-line source from your 5-8 web searches (see `SKILL.md` Custom Category Research Protocol + `workflows/index.md` unlisted branch).

- **Candidate**: e.g., Weather "Cirrus", Real Estate "Found", Education "Sprout"
- **Base mode remapped**: closest of 7 + how it diverges (e.g., Weather = Health & Fitness (glanceable) + Data-Dense (radar/charts) — needs atmospheric palette, time-scrub, not breathing ring)
- **Research queries executed**: list 5-8 + 1-line finding each

```css
/* Token System — Custom: [App Name] — [Domain] */
/* Research synthesis: [query 1 → finding], [query 2 → finding] ... */
:root {
  /* Surfaces — choose OLED vs light based on domain context (outdoor glanceable = light default; night/trading = dark) */
  --bg: # /* e.g., #f0f5ff for weather — atmospheric, not fintech #08080c */;
  --surface: #ffffff; --surface-raised: # /* tonal step for sheets/cards */;
  --text: # /* 4.5:1 on --bg verified */; --muted: # /* 4.5:1 verified on surface */;
  --border: rgba( , , ,0.08);
  --overlay-scrim: rgba(0,0,0,0.4);

  /* Brand hues — ≤3, named by domain purpose after research. Example for weather: */
  --primary: # /* --sky-now: research said atmospheric blue-teal */; 
  --primary-pressed: # ;
  --on-primary: #ffffff;
  --accent: # /* --alert-severe / --sun-accent from teardown */;
  --accent-2: # /* optional second accent if justified by research */;
  --success: # /* or domain semantic e.g., --aqi-good */;
  --warning: # /* e.g., --alert-watch */;
  --error: # /* e.g., --alert-severe */;

  /* Glass — functional layer ONLY. Choose clear only if research shows rich media behind (radar map, property photos) */
  --glass-regular: rgba( , , ,0.68);
  --glass-clear: rgba( , , ,0.42); /* use only over photos/video/maps */
  --glass-border: rgba(255,255,255,0.08);
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.06);
  --shadow-md: 0 8px 24px rgba(0,0,0,0.10);
  --shadow-glass: 0 8px 32px rgba(0,0,0,0.12), inset 0 1px 0 rgba(255,255,255,0.4);

  --space-1: 4px; --space-2: 8px; --space-3: 12px; --space-4: 16px;
  --space-6: 24px; --space-8: 32px; --space-12: 48px;
  --gutter: 16px; --section-gap: 32px;
  --radius-card: 16px; --radius-sheet: 20px; --radius-full: 999px;

  --font-display: "" /* research: e.g., Satoshi for weather's Swiss clarity */;
  --font-body: "" /* research: e.g., Inter for dense data legibility */;
  --font-mono: "" /* for numbers: tabular-nums required for metrics */;
  --text-display-lg: clamp(28px, 7vw, 34px); /* domain figure: e.g., 64px temp needs tighter tracking */
  --text-body-lg: 17px; --text-label-md: 13px;
  --tracking-tight: -0.02em;

  --duration-fast: 150ms; --duration-base: 250ms; --ease-out: cubic-bezier(0,0,0.2,1);
  --ease-spring: cubic-bezier(0.34,1.56,0.64,1);
  --safe-top: env(safe-area-inset-top); --safe-bottom: env(safe-area-inset-bottom);
}
/* Dark semantic override — not invert. Glass opacity shifts for dark. */
@media (prefers-color-scheme: dark) {
  :root { --bg: #08080c; --surface: #121214; /* ... */ }
}
@media (prefers-reduced-transparency: reduce) {
  :root { --glass-regular: var(--surface); --glass-clear: var(--surface); }
}
```

**Typography after research**: e.g., Weather needs tabular mono for temps (38°), large display for current (56-64px, -0.03em), smaller 11px labels for hourly. Document why.
**Layout after teardown**: e.g., Weather = Canvas (radar) + Stack&Card (hourly/daily) + Bottom Sheet for details; Real Estate = Carousel peek (listings) + Map+Sheet. Cite competitor.
**Signature after research**: must be domain-native and tactile — e.g., Weather: horizontal time-scrub over hourly strip with live gradient/background shift + `selection` tick per hour + radar loop playhead. Not generic lift.
**IA after research**: list jobs you found (Weather: Current + Hourly + 10-day + Air Quality + Radar + Alerts + Location manage). Include empty (no saved location), loading (skeleton hourly rows), error (alert fetch failed), offline (cached last forecast).

### Worked Example: Weather "Cirrus" (from live research synthesis)

- **Audience**: Commuters + outdoor planners, outdoor glanceable (sunlight), one-handed
- **Job**: Glance now, plan hours, check radar/alerts without typing
- **Base mode**: `Custom — Health & Fitness (glanceable, calm) + Enterprise/Data-Dense (radar/charts)` → diverges: needs atmospheric palette + time as primary dimension + alert urgency system
- **Research**: `Dribbble best weather app 2025 2026` → glassmorphic radar sheets + atmospheric gradients winning; `Carrot Weather teardown` → personality copy + map-first + hourly scrub; `weather app UX best practices 2026` → glanceable current (72pt), swipe hours, pull radar, severe alert as inline banner (not toast)
- **Tokens**: `--sky-now: #3a8dde` (research: atmospheric blue, not generic #7c3aed), `--sky-night: #1e3a5f`, `--alert-severe: #dc2626`, `--aqi-good: #16a34a`/`--aqi-unhealthy: #f97316`; surface warm white for daytime legibility outdoors (contrast outdoors > indoor)
- **Layout**: Hero 40vh current (56px temp, tabular) + hourly horizontal snap (peek 16px) + daily stack cards + full-bleed radar canvas with `glass-clear` sheet (peek 24% → medium 50% for details)
- **Signature**: Time-scrub bar (0800—2200) drags background gradient day→dusk→night live + haptic `selection` per hour tick + radar loop `play` spring.

## Token Checklist (use before shipping any example above)

- [ ] Colors derive from ≤3 brand hues (dominant 60% / neutral 30% / accent 10%) + 4 semantic (success/warning/error/info) kept separate
- [ ] Text/muted contrast checked with APCA or WCAG (muted ≥4.5:1) — especially over glass
- [ ] No pure #fff/#000 — use offset (#08080c/#f5f5f7 etc.)
- [ ] Glass tokens only on functional layer; cards/rows opaque
- [ ] Spacing on 4/8 grid; section-gap > stack-md; gutter 16px phone
- [ ] Type scale is one calc (fluid `clamp`) and supports Dynamic Type 135% / fontScaling
- [ ] Motion: `--duration-fast/base`, `--ease-out/spring`; haptics mapped per interaction
- [ ] Dark variant is semantic override, not invert; test Reduced Transparency + Increase Contrast
