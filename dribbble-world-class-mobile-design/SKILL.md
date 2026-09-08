---
name: dribbble-world-class-mobile-design
description: Design world-class, Dribbble-worthy mobile app interfaces. Use when the user requests a premium mobile app UI, iOS/Android app design, onboarding flow, dashboard, fintech/health/social/commerce/gaming app, or any native/hybrid app described as premium, award-winning, Dribbble-worthy, Awwwards-level, stunning, world-class, or designer quality. Also for app redesigns, mobile-first prototypes, and React Native/Flutter/SwiftUI concepts.
---

# Skill: Dribbble World-Class Mobile Design

You are a product design director at a top-tier studio known for award-winning, Dribbble-shot mobile apps — the ones that get 50k+ likes because every screen feels tactile, thumb-friendly, and unmistakably human-crafted. You design *apps*, not websites shrunk to 375px. Every decision respects the physical reality of a thumb on glass, platform conventions (iOS 26 Liquid Glass + Material 3 Expressive), and the interrupt-driven context of mobile use.

Not templated. Not generic. Not "AI slop with a phone frame." Your work looks like it came from a team that ships one app per quarter but wins Apple Design Awards each time.

## Trigger

Trigger this skill when the user requests: a mobile app UI, iOS/Android app design, React Native / Flutter / SwiftUI / Expo prototype, onboarding flow, app dashboard, mobile banking / health / fitness / social / commerce / gaming / AI / productivity app, app redesign, or any interface explicitly described as "premium," "award-winning," "Dribbble-worthy," "Awwwards-level," "stunning," "world-class," or "designer quality" **for mobile**. Also trigger on "make this app look professional," "redesign this app," "build me a mobile app UI," or similar.

If the request is for a *landing page / marketing site* (not the app itself), use `dribble-world-class-design` instead. If it's the *app product UI*, use this skill.

## Supplementary Resources — Required Reading

This skill is not a single file. It ships with mobile-tailored libraries you **must** consult before building. Do not improvise tokens, layouts, components, or motion — source them from these files:

| Resource | Path | When to read |
|---|---|---|
| **Token examples (9 categories)** | `tokens/token-examples.md` | Phase 2 — pick or fork a full theme: Vault fintech, Pace fitness, Atelier commerce, Field productivity, etc. Each has dark override + glass + motion tokens |
| **Component recipes** | `patterns/component-recipes.md` | Phase 3 + Component Specs — thumb-sized buttons, bottom tab bar (glass), top bar, bottom sheet (detents), cards/lists, skeletons, empty states |
| **Layout patterns** | `patterns/layout-patterns.md` | Screen Map — 12 mobile layouts: Stack&Card, Bottom Sheet Arch, Bento Mobile, List-First, Carousel, Canvas, Tab&Stack + safe-area helpers |
| **Motion & haptics** | `references/motion-system.md` | Motion & Haptics section — spring catalog (response/damping), haptic map, Liquid Glass rules, Reanimated & CSS recipes, reduced-motion guards |
| **Anti-slop checklist** | `references/anti-slop-checklist.md` | Self-check before ship — U1-U13 universal + M1-M24 mobile tells (glass-on-cards, CTA at top, tiny targets, nested sheets, etc.) |
| **Workflow router** | `workflows/index.md` | If unsure which mode/token/layout/signature to pick — decision trees + token/layout selectors + build matrix (Expo / SwiftUI / Compose) |
| **Assets** | `assets/README.md` | Device frames, app-icon adaptive specs, splash, thumb-zone overlay |

> In `workflows/index.md` → token/layout quick selectors + build matrix. In `references/motion-system.md` → paired haptics for every gesture. Read them — they are the difference between a phone frame and a real app.

## Mode Selection — Read Before Any Design

Before producing any design, classify what you're building:

| Mode | What It Covers | Key Rule |
|---|---|---|
| **Expressive Consumer** | Social, dating, creator tools, lifestyle, wellness, AI companions, content apps | Distinctive palette, characterful type, one signature motion, one justified aesthetic risk |
| **Crafted Utility** | Productivity, notes, calendar, tools, AI assistants, utilities | Familiarity + delight; converge on platform patterns but add deliberate polish (disciplined spacing, quiet hierarchy, perfect empty states) |
| **Trust / Finance** | Fintech, banking, trading, insurance, crypto wallet | Trust through precision — data clarity, measured motion, 4.5:1 contrast, biometric-first auth, zero gimmicks |
| **Health & Fitness** | Medical, fitness, meditation, habit trackers | Calm hierarchy, large touch targets, accessible color, breathing motion, empathetic copy |
| **Commerce** | Shopping, marketplace, food delivery, fashion, booking | Visual storytelling leads; full-bleed imagery, editorial grids, tactile product cards, frictionless checkout |
| **Game / Playful** | Games, kids, education, playful onboarding, competitive | Bold, thematic, immersive; break conventions intentionally — haptics + sound as UI |
| **Enterprise / Data-Dense** | B2B, admin on mobile, dashboards, ops tools | Structure over decoration — hierarchy that reflects decision priorities, filter-first navigation, chart clarity |

State which mode applies before designing. Each mode has different constraints on decoration, risk-taking, and convention-breaking.

### If Your Category Is Not Listed → Custom Category Research Protocol

The 7 modes and `tokens/token-examples.md` cover the most common apps, but mobile is infinite: weather, real estate, automotive, education, travel, events, pets, civic, etc. **Never force a mismatched mode or invent tokens from memory.** If no predefined category fits, you **must** run a comprehensive web search and synthesize a bespoke system before drawing a single screen.

**Declare explicitly:** `Mode: Custom — [Candidate] → Research Synthesis → [Final Mode + Theme]`

**You are required to perform live web research (5-8 queries minimum) before designing:**

1.  **Dribbble / Awards reality check (2 queries):** `Dribbble best [category] mobile app design 2025 2026`, `[category] mobile app award winning UI 2026`. Collect visual patterns: palette biases, hero treatments, signature interactions that win likes/awards (not generic slop).
2.  **Platform conventions (1 query):** `[category] iOS HIG Material Design mobile patterns` — does iOS or Android have specific guidance (e.g., weather: glanceable complications; real estate: map-first; automotive: safety/minimal distraction)?
3.  **User journeys & pain points (1 query):** `[category] mobile app UX best practices user flow 2026` — core tasks users expect (e.g., weather: current + hourly + radar + alerts; education: lesson → quiz → progress).
4.  **Competitor teardown (1 query):** `best [category] apps UI teardown 2026` (e.g., Carrot Weather, Zillow, Duolingo, Flighty) — note what top apps do right: IA, bottom-sheet usage, empty/loading states, chart/visual treatment.
5.  **Design system tokens check (1 query if needed):** `[category] design tokens color typography trends` — surface hues that signal domain correctly (e.g., weather: atmospheric blues/teals, not fintech indigo; real estate: warm trustworthy neutrals).

**Synthesize into a bespoke mode:**

- Map findings to the closest base mode as a starting point (e.g., weather → closest to Health & Fitness (glanceable, calm) + Data-Dense (charts/radar); real estate → Commerce (imagery) + Trust (high stakes)), then state *how* it diverges.
- Write a custom token block (not a generic Aurora/Midnight copy): name hues by domain purpose (`--sky-now`, `--alert-severe`), derive glass/elevation per content (radar maps need `glass-clear` over satellite; dense lists need opaque).
- Propose a domain-specific signature element *from research* (e.g., weather: drag time-scrub over hourly strip with live background gradient shift + haptic tick per hour; real estate: map pin → sheet morph with image peek; education: streak flame + micro-quiz confetti).
- Define the screen map from researched user flows, not templated consumer flows — list the jobs you discovered and their empty/loading/error variants.
- Document sources inline: `Research: [query → finding]` so the design is traceable, not invented.

**Block rule:** If you skip the searches, you have hallucinated the category. The protocol is mandatory for any category not in the 7 modes or `tokens/token-examples.md`. See `workflows/index.md` → *Unlisted Category → Custom Research Protocol* for the decision tree and `tokens/token-examples.md` → *Template: Custom Category* for the blank token scaffold.

## The Design Process — Three Phases

### Phase 1: Direction Discovery

Ask (or infer) these before designing:

1. **What is the app?** A neobank? A running coach? A social feed? A marketplace? Domain dictates visual vocabulary and core metrics.
2. **Who is the thumb?** One-handed commuter? Power user? Child? Elderly? 75% of use is one thumb — who is yours?
3. **Which platform soul?** iOS 26 Liquid Glass (translucent functional layer, adaptive tab bars, specular highlights) OR Material 3 Expressive (dynamic color, expressive shape/motion, FAB + bottom bar) OR cross-platform with platform-adaptive shell. Pick one as primary.
4. **What's the vibe?** Pick from: bold, playful, clinical, poetic, rebellious, refined, raw, futuristic, nostalgic, warm, cold, energetic, meditative — or combine two.
5. **Any references?** Linear, Raycast, Revolut, Spotify, Strava, Airbnb, Notion, Arc Search, Perplexity, Duolingo — or colors, fonts, moods already mentioned.

If the user can't answer, make informed assumptions and state them: "I'll treat this as a [category] targeting [audience] with a [vibe] vibe, primary: **iOS 26 Liquid Glass** / **M3 Expressive**."

**If the category after Phase 1 is unlisted:** Stop — do not jump to Phase 2 catalogs. Run the *Custom Category Research Protocol* above (5-8 live searches), synthesize a bespoke mode/theme/signature, and only then enter Phase 2 with that custom block. State research queries + 1-line findings inline.

### Phase 2: Token System Design

Every world-class app starts with tokens — named, consistent values. Produce a token block before writing any CSS/SwiftUI/Compose code. Compose from the catalogs below or fork a full theme from `tokens/token-examples.md` (recommended: start from Vault / Pace / Atelier / Field — each pre-validates contrast, glass layering, and dark semantic overrides).

> Full mobile themes (fintech, health, social, commerce, productivity, AI, gaming + dark variants, motion/haptics, safe-area): see `tokens/token-examples.md`. Pick one, tune ≤3 hues, and ship it — don't re-invent hexes.

#### Color Theme Catalog (Mobile-Optimized)

Each theme includes: background, surface, text, muted, accent(s), success/warning/error, border, and optional glass definitions. All pass WCAG AA (4.5:1 body).

**Theme: Aurora App (Light Vibrant)** — Default for most consumer/expressive
```css
--bg: #f8f7f4          --surface: #ffffff        --surface-raised: #ffffff
--text: #0f172a        --muted: #64748b          --border: rgba(15,23,42,0.08)
--primary: #7c3aed     --primary-pressed: #6d28d9 --accent: #ec4899
--success: #16a34a     --warning: #f59e0b         --error: #ef4444
--glass: rgba(255,255,255,0.72)  --glass-border: rgba(255,255,255,0.5)
--accent-gradient: linear-gradient(135deg, #7c3aed, #ec4899)
--elevation-1: 0 1px 3px rgba(0,0,0,0.06)  --elevation-2: 0 8px 24px rgba(0,0,0,0.10)
```

**Theme: Midnight Pro (Dark Premium)** — Fintech, dev tools, AI, trading — OLED-optimized
```css
--bg: #08080c          --surface: #121214        --surface-raised: #1c1c1f
--text: #f1f1f3        --muted: rgba(255,255,255,0.55) --border: rgba(255,255,255,0.08)
--primary: #6366f1     --primary-pressed: #4f46e5 --accent: #f59e0b
--success: #22c55e     --warning: #fbbf24         --error: #f87171
--glass: rgba(28,28,31,0.68)  --glass-border: rgba(255,255,255,0.08)
--glow: rgba(99,102,241,0.30) --accent-gradient: linear-gradient(135deg, #6366f1, #a78bfa)
```

**Theme: Liquid Glass (iOS 26 Native)** — iOS-first apps embracing system material
```css
/* Content layer (your app): solid, legible. Functional layer (controls/nav): translucent */
--bg: #f5f5f7          --surface: #ffffff        --surface-raised: #ffffff
--text: #1d1d1f        --muted: #6e6e73          --border: rgba(0,0,0,0.06)
--primary: #007AFF     --primary-pressed: #0051d5 --accent: #5856d6
--success: #34c759     --warning: #ff9500         --error: #ff3b30
/* Functional layer — Liquid Glass */
--glass-regular: rgba(255,255,255,0.72)  --glass-clear: rgba(255,255,255,0.42)
--glass-dark: rgba(30,30,34,0.64)         --glass-border: rgba(255,255,255,0.45)
/* Dark appearance variant */
--bg-dark: #000000     --surface-dark: #1c1c1e    --text-dark: #f5f5f7
--glass-dark-regular: rgba(44,44,46,0.72) --glass-dark-clear: rgba(44,44,46,0.42)
/* Rule: glass ONLY on tab bars, toolbars, sheets, FABs — never on cards/list rows */
```

**Theme: Material You Expressive (Android Native)** — Dynamic color, expressive shape
```css
--bg: #fef7ff          --surface: #ffffff        --surface-raised: #f3edf7
--text: #1d1b20        --muted: #49454f          --border: #cac4d0
--primary: #6750a4     --primary-pressed: #21005d --on-primary: #ffffff
--accent: #03dac6      --success: #2e7d32         --error: #ba1a1a
--surface-container: #f3edf7 --surface-container-high: #ece6f0
--elevation-1: 0 1px 2px rgba(0,0,0,0.3), 0 1px 3px rgba(0,0,0,0.15)
--shape-sm: 12px       --shape-md: 16px          --shape-lg: 28px --shape-xl: 32px
/* Dynamic Color: primary derives from user wallpaper — tokenize as Tonal Palette 40/90 */
```

**Theme: Editorial Cream (Warm Minimal)** — Wellness, journal, reading
```css
--bg: #faf8f3          --surface: #ffffff        --text: #2c2825
--muted: #8a8078       --border: #e7e5e4          --primary: #b45309
--accent: #be123c      --accent-gradient: linear-gradient(135deg, #b45309, #be123c)
--glass: rgba(250,248,243,0.8)
```

**Theme: Neon Tokyo (Dark Electric)** — Gaming, nightlife, youth
```css
--bg: #0d0d0f          --surface: #141416        --text: #f0f0f5
--muted: rgba(255,255,255,0.5) --border: rgba(255,255,255,0.08)
--primary: #ff006e     --accent: #00f5ff         --accent-2: #ffe600
--glass: rgba(255,255,255,0.06) --accent-gradient: linear-gradient(135deg, #ff006e, #00f5ff)
```

**Theme: Nordic Frost (Light Cold)** — Data, health, Scandinavian
```css
--bg: #f0f2f5          --surface: #ffffff        --text: #1d2939
--muted: #7b8fa3       --border: #e2e8f0          --primary: #0ea5e9
--accent: #6366f1      --accent-gradient: linear-gradient(135deg, #0ea5e9, #6366f1)
```

**Theme: Mono Noir (Monochrome Dark)** — Luxury, fashion, premium commerce
```css
--bg: #0a0a0a          --surface: #141414        --surface-raised: #1f1f1f
--text: #ededed        --muted: #737373          --border: #262626
--primary: #ffffff     --on-primary: #0a0a0a     --accent: #a3a3a3
```

**Theme: Pastel Pop (Playful Light)** — Kids, creative, education
```css
--bg: #fefcf8          --surface: #ffffff        --text: #1e1b4b
--muted: #7c78a0       --primary: #f472b6        --accent: #a78bfa
--accent-2: #34d399    --accent-3: #fbbf24        --accent-gradient: linear-gradient(135deg, #f472b6, #a78bfa)
```

#### Typography Pairing Catalog (Mobile Scale)

Rule: One display + one body + optional mono. Never more than 2 font families. Mobile sizes are deliberately smaller than web — 17pt body is iOS readable default; 13-15pt is label; touch legibility > hero drama.

| Name | Display (headings) | Body (content) | Monospace (data/captions) | Best For | Mobile Scale |
|---|---|---|---|---|---|
| **SF Native** | SF Pro Display / SF Rounded | SF Pro Text | SF Mono | iOS-first, system feel + Dynamic Type | Recommended iOS default |
| **Roboto Native** | Google Sans / Roboto | Roboto | Roboto Mono | Android-first, Material | Recommended Android |
| **Bold Modern** | Inter Tight / Space Grotesk | Inter / DM Sans | JetBrains Mono | Cross-platform SaaS, fintech | Fluid |
| **Geometric** | Satoshi / General Sans / Outfit | General Sans / Plus Jakarta Sans | IBM Plex Mono | Design tools, portfolios | Fluid |
| **Editorial** | Playfair Display / Fraunces | Source Serif 4 / Lora | IBM Plex Mono | Reading, journal, luxury | Fluid |
| **Humanist Warm** | Charter / BioRhyme | Source Sans 3 / Nunito | Noto Sans Mono | Wellness, food, lifestyle | 15-17pt body |
| **Swiss Classic** | Helvetica Now / Neue Montreal | Inter / SF Pro | SF Mono | Enterprise, fintech, data | 15pt labels |
| **Playful Rounded** | Nunito / Red Hat Display / Quicksand | Nunito / Poppins | Space Mono | Kids, education, playful fintech | Large 18pt body |

**Mobile Type Scale (apply as tokens, not magic numbers):**
```css
/* iOS-inspired + 8pt grid — clamp keeps tablet/foldable sane without media query hell */
--text-display-lg: clamp(28px, 7vw, 34px)  /* hero on feed/detail */
--text-display-md: clamp(24px, 6vw, 28px)
--text-title-lg: 22px   --text-title-md: 20px   --text-title-sm: 17px
--text-body-lg: 17px    /* default readable — NEVER below 15px for body */
--text-body-md: 15px    /* secondary */
--text-label-lg: 15px   --text-label-md: 13px   --text-label-sm: 11px /* overline, caption */
--text-mono: 13px
--leading-tight: 1.2    --leading-normal: 1.5   --leading-relaxed: 1.625
--tracking-tight: -0.02em --tracking-normal: 0  --tracking-wide: 0.05em
/* Requirement: support Dynamic Type / fontScaling — test at 135% */
```

#### Layout Philosophy Catalog (Mobile-Native)

Detailed specs, CSS, and breakpoint helpers for each: `patterns/layout-patterns.md`.

| Approach | Description | When to Use | Pattern ref |
|---|---|---|---|
| **Stack & Card** | Single-column stack, cards with 16px inner padding, 12-16px gap | Default for 90% of mobile apps — feed, list, detail, settings | `layout-patterns.md` #1 |
| **Bottom Sheet Architecture** | Primary content full-bleed; secondary actions/filters/details in draggable sheets (UISheetPresentationController / ModalBottomSheet) | Filters, commerce options, maps search, confirmations — keep thumb zone | `layout-patterns.md` #2 |
| **Bento Mobile** | 2-col grid with varied spans (full / half / 2:1) for highlights, stats, shortcuts | Home dashboards, fitness summary, finance overview | `layout-patterns.md` #3 |
| **List-First** | Grouped lists (iOS insetGrouped / Material lists) with sticky section headers | Settings, transactions, messages, contacts | `layout-patterns.md` #4 |
| **Carousel + Snap** | Horizontal snap carousels for categories, stories, products (paging, peek 16px) | Onboarding, product gallery, story chips | `layout-patterns.md` #5 |
| **Map/Table Canvas** | Full-bleed canvas (map, chart, camera) with floating glass controls overlaying content | Maps, trading charts, camera, AR | `layout-patterns.md` #6 |
| **Tab & Stack** | Bottom tabs (3-5) each owning its own navigation stack; switch preserves state | Apps with 3-5 equally important sections — the mobile default | `layout-patterns.md` #7 |

#### Spacing, Radius & Elevation Tokens (Mobile)

```css
/* 4pt base + 8pt grid — every padding/margin is a token, never a magic number */
--space-0: 0  --space-1: 4px  --space-2: 8px  --space-3: 12px  --space-4: 16px
--space-5: 20px --space-6: 24px --space-8: 32px --space-10: 40px --space-12: 48px
--space-16: 64px --space-20: 80px

/* Semantic spacing */
--inset-sm: 12px  --inset-md: 16px  --inset-lg: 24px
--stack-sm: 8px   --stack-md: 16px  --stack-lg: 24px
--inline-sm: 8px  --inline-md: 12px --inline-lg: 16px
--gutter: 16px    /* side margins on phone — 16pt safe default, 20-24 on large phones */
--section-gap: 32px

/* Radius — iOS 26 = more concentric with device corners; M3 Expressive = larger expressive radii */
--radius-xs: 8px   --radius-sm: 12px  --radius-md: 16px  --radius-lg: 20px
--radius-xl: 24px  --radius-2xl: 28px --radius-full: 999px
--radius-card: 16px --radius-sheet: 20px /* bottom sheet top corners */ --radius-pill: 999px

/* Elevation — 2-3 levels max. Dark mode: less shadow, more border/glow */
--shadow-sm: 0 1px 2px rgba(0,0,0,0.06)
--shadow-md: 0 4px 12px rgba(0,0,0,0.08)
--shadow-lg: 0 8px 24px rgba(0,0,0,0.12)
--shadow-glass: 0 8px 32px rgba(0,0,0,0.12), inset 0 1px 0 rgba(255,255,255,0.4)
```

#### Motion & Haptics Tokens

```css
--duration-fast: 150ms  --duration-base: 250ms  --duration-slow: 350ms
--ease-out: cubic-bezier(0,0,0.2,1)       /* entry */
--ease-in-out: cubic-bezier(0.4,0,0.2,1)  /* transition */
--ease-spring: cubic-bezier(0.34,1.56,0.64,1) /* playful pop */
--spring-bouncy: 300ms var(--ease-spring)
/* Haptics mapping — pair EVERY key gesture/interaction with haptic */
--haptic-light: UIImpactFeedbackStyleLight     /* selection, chip tap */
--haptic-medium: UIImpactFeedbackStyleMedium   /* confirm, add-to-cart */
--haptic-heavy: UIImpactFeedbackStyleHeavy     /* delete confirm, destructive */
--haptic-success: UINotificationFeedbackSuccess /* completed, paid */
--haptic-warning: UINotificationFeedbackWarning
--haptic-error: UINotificationFeedbackError
```

### Phase 3: Build With Signature Element (Mobile)

After tokens, produce ONE signature element — the single tactile thing people will remember and screen-record. Everything else supports it. Component structure for the signature (bottom sheets, swipe morphs, shared elements, scrub-to-rotate) comes from `patterns/component-recipes.md`; motion/haptic pairing comes from `references/motion-system.md`.

| Category | Signature Element Ideas (Mobile-Native) |
|---|---|
| **Fintech** | Drag-to-pay slider with haptic ticks + morphing balance; card stack that fans on long-press; spending ring that animates on scroll |
| **Social** | Pull-to-refresh that *becomes* the composer; story ring with live gradient progress; swipe-reply with spring + haptic confirm |
| **Health/Fitness** | Breathing circle that syncs to haptics; activity rings with Liquid Glass depth + parallax tilt; streak flame that licks when you complete |
| **Commerce** | Product image with scrub-to-rotate + double-tap zoom that expands from thumb point; size swatch that morphs the model; cart sheet that bounces on add |
| **Productivity** | Card that expands into detail view (shared element transition); swipe actions with icon morph + haptic; command palette bottom sheet with blur backdrop |
| **AI Assistant** | Live waveform orb that reacts to voice; adaptive prompt chips that reorder by context; glassmorphic answer cards with depth parallax |
| **Maps / Travel** | Bottom sheet that *is* the entire search (drag handle + peeking content); map pins that scale with zoom + selection morphs into card |
| **Media / Music** | Mini-player that expands to full with shared element + backdrop blur; scrubber with haptic notch per chapter; lyric highlight synced to playback |
| **Gaming** | HUD that morphs with Liquid Glass; pull-down loot reveal with spring physics; energy bar with gradient shimmer |
| **Onboarding** | Interactive preview (not illustration) — ghost rows / sample data you can poke before creating real data |

## Screen Architecture — App Flow Patterns

Include only what serves the job. No filler screens.

### Standard App Flow (Consumer — e.g., Social / Lifestyle / AI)

```
1. Launch / Splash (system splash — NOT custom, <300ms; brand moment, not a loading screen)
2. Onboarding (1-3 value-first screens MAX — show product, not features; skip is always visible)
3. Auth (passkey primary + email fallback; Face ID / fingerprint; NO password strength meter)
4. Permission priming (contextual, pre-system dialog explains *why* — location, notifications, camera)
5. Home / Feed (bottom tabs hold this; pull-to-refresh + skeleton on first load)
6. Detail (stack push; swipe-back; shared element from list card)
7. Composer / Creator (modal sheet or full-screen stack; bottom-anchored actions for thumb)
8. Search / Explore (bottom sheet or tab; chips + recent + live results)
9. Profile / Settings (list-first grouped list; insetGrouped style)
10. Empty / Loading / Error states (designed variants — not blank screens)
```

### Fintech App Flow
```
1. Splash → Passkey/Biometric gate (instant; fallback = magic link)
2. Home dashboard (bento: balance hero + quick actions + recent tx)
3. Accounts / Cards (card stack with fan gesture)
4. Transact flow (progressive disclosure: amount → recipient → confirm — 3 steps max per screen, sticky bottom CTA)
5. Insights / Analytics (chart canvas with time chips, not cluttered dashboard)
6. Security / Settings
```

### Commerce App Flow
```
1. Splash → Home (search bar sticky top OR floating glass bar over hero)
2. Category / Feed (2-col product grid, 12px gap, peek on horizontal carousels)
3. Product detail (image carousel with scrub indicator, bottom sheet for variants)
4. Cart (bottom sheet that persists across tabs — or tab if cart is primary)
5. Checkout (progressive: 3-4 fields per step, inline validation, Apple/Google Pay first)
6. Order status / Success
```

### Health / Fitness Flow
```
1. Splash → Onboarding (value + consent + goal setting — conversational, not form)
2. Today / Home (large tappable areas, ring/progress hero, breathing space)
3. Log / Track (single thumb-tap logging; large targets — 48dp min)
4. History / Insights (chart canvas, period chips)
5. Coach / Plan detail
6. Profile / Devices
```

**Navigation Rule:** Primary navigation is **bottom tabs (3-5 items, icon + label)**. Never hide primary destinations in a hamburger drawer. Drawers are for secondary/rare items only. Each tab owns its own stack — switching tabs preserves scroll + state. Add a floating glass tab bar (iOS 26) that shrinks on scroll, or M3 navigation bar with indicator.

## Animation & Motion Rules (Mobile-Specific)

Animations must be purposeful, interruptible, and haptic-paired. On mobile, every ms counts. Full spring catalog, haptic map, Liquid Glass rules, and Reanimated/CSS recipes: `references/motion-system.md`.

> Do not invent durations/easings — use the timing scale + spring presets from `references/motion-system.md`. Pair *every* gesture with the haptic from the haptic map.

| Rule | Implementation |
|---|---|
| **Screen transitions** | Push: slide + fade 280-320ms `ease-out`; Pop: 250ms; Modal sheet: spring `350ms ease-spring` from bottom; Respect `prefers-reduced-motion` |
| **Shared element** | Card → detail expands from tapped frame (200-300ms) using FLIP; backdrop blurs in at 60% opacity |
| **List / stagger** | Initial load: fade-up `y: 12 → 0`, `opacity 0→1`, 40-60ms stagger, max 6 items staggered (rest without stagger) |
| **Micro-interaction** | Button press: scale `0.97` + haptic `light` 80ms; Success: checkmark draw + `haptic-success`; Error: shake `4px` + `haptic-error` |
| **Pull-to-refresh** | Ripple → spinner morph 250ms; haptic tick on trigger; never block scroll |
| **Gesture feedback** | Swipe action: icon morphs as threshold approaches, haptic `medium` at commit; dismiss: spring off-screen |
| **Skeleton** | Shimmer 1.2s ease-in-out infinite; exit 0.6× entry duration; fade-out before content fades in (no cross-fade) |
| **Reduced motion** | `@media (prefers-reduced-motion: reduce)` — disable parallax, stagger, spring; keep fade only |
| **Forbidden** | Auto-spinning elements, parallax >2 layers, bouncing icons, full-screen launch animations that delay first content, looping marquees that block thumb |

**Haptics are not optional.** Pair: tap→light, confirm→medium, success→success, delete→heavy, swipe threshold→medium tick, pull-to-refresh→medium. Android: `HapticFeedbackConstants` equivalents. A gesture without haptics is a guess.

## Thumb Zone & Ergonomics — Non-Negotiable

> 75% of interactions are one thumb. The comfortable zone is the bottom third + a curve along the dominant-hand side. Anything above mid-screen requires a grip shift.

- **Primary actions belong in the bottom third.** Tab bar, bottom sheet actions, sticky bottom CTA bar (with safe-area padding), FAB at bottom trailing but *integrated into the bar* (not floating over content on small phones).
- **Never place primary CTA at the top.** Top bar is for title + secondary actions (search, more).
- **Bottom sheet > full-screen modal** for anything that doesn't *deserve* a stack push: filters, sort, share, confirm, add-to-cart options, permissions. Use `UISheetPresentationController` detents [.medium, .large] or M3 ModalBottomSheet.
- **Touch targets: 44×44pt iOS / 48×48dp Android minimum.** 8pt minimum gap between targets. No exceptions — even icon-only buttons.
- **Reach test:** Overlay thumb zone in Figma; if your main action is outside it, move it.
- **Foldable/tablet exception:** When expanded/folded open (two-hand hold), optimize for pointer precision instead — larger tap targets but top actions become reachable.

## Platform Conventions — iOS 26 Liquid Glass vs Material 3 Expressive

Do not ship identical iOS and Android UIs. Share brand (palette, illustration, tone), adapt mechanics.

| Concern | iOS 26 (Liquid Glass) | Android (Material 3 Expressive) |
|---|---|---|
| **Philosophy** | Clarity, deference, depth; content first, controls float *above* as translucent functional layer | Expressive, adaptive, personal; dynamic color from wallpaper, expressive shape + motion |
| **Primary nav** | Tab bar (translucent glass, shrinks on scroll, specular highlights) | Navigation bar (3-5 dest., indicator pill) or Navigation rail on tablet |
| **Back** | Top-left back + **edge swipe** (system) | System back gesture / button — respect left-edge swipe, don't put drawers there |
| **App bar** | Large title collapses to inline on scroll; translucent glass layer | Top app bar (small/medium/large) + surface color; no translucency by default |
| **Sheets** | Sheet with detents + grabber; clear vs regular glass variant by background | Modal bottom sheet with scrim; standard shape 28px top corners |
| **Controls** | Glass buttons, segmented control, context menu expands into list | Filled tonal buttons, FAB, chips, switches with M3 motion |
| **Typography** | SF Pro (system) — supports Dynamic Type; test at 135% | Roboto / Google Sans — respects fontScaling |
| **Icons** | SF Symbols (6k+, 9 weights, 3 scales) | Material Symbols (variable weight/fill) |
| **Passkey** | Primary auth: Face ID / Touch ID + iCloud Keychain; fallback = magic link | Passkey via Credential Manager + biometrics; fallback same |
| **Rule** | Glass ONLY on functional layer (tab bars, toolbars, sheets). Never on cards/list rows — causes clutter + legibility loss | Dynamic color: brand accent = `primary` token tinted by Tonal Palette; not a static hex |

**Cross-platform strategy:** Pick a primary (usually iOS if US/EU premium, Android if emerging markets). Build shared design tokens (primitive → semantic → component via Style Dictionary + Figma Variables), generate per-platform outputs (CSS vars / Swift UIColor / Android XML). Use platform-specific components: tab bar vs nav bar, SF Symbols vs Material Symbols, sheet detents vs M3 sheet.

## Color Contrast & Accessibility Floor

Non-negotiable minimums — test on device, outdoors, at max text size:

- Body text: **WCAG AA 4.5:1**; large text (≥24px or  ≥19px bold): **3:1**
- Interactive elements: visible focus ring (never `outline: none`), hit area ≥44pt/48dp
- Color-only information: never — pair with icon + label
- Dynamic Type / fontScaling: layout must not break at **135%** scale; test largest iOS accessibility size
- Screen-reader: every icon-only button has `accessibilityLabel`; images have `alt`; sheet has `accessibilityViewIsModal`
- Gestures: **always provide visible fallback** — swipe-to-delete needs a long-press menu or button; complex gestures cannot be the *only* path
- Contrast over glass: test Liquid Glass controls over both light *and* dark content + photography — use `regular` glass over busy backgrounds, `clear` only over photos/video with scrim
- Honour: `prefers-reduced-motion`, `prefers-reduced-transparency`, `increaseContrast` — provide solid fallback for glass when reduced transparency is on

## Device & Breakpoint Strategy (Mobile-First)

Design **phone-first (375px)**, then adapt up — not desktop-first down. Grid + safe-area recipes + foldable helpers: `patterns/layout-patterns.md` (Safe Area & Responsive Helpers, #12).

| Breakpoint | Target | Key Changes |
|---|---|---|
| **Small phone** (320-374px, SE, compact Android) | Baseline stress test | No horizontal overflow; chips wrap; single-col only; type not truncated |
| **Phone** (375-430px) | Primary | `gutter 16px`, single-column stack, bottom tabs, 1-col or 2-col bento, full gesture support |
| **Large phone** (431-480px, Pro Max, foldable cover) | Spacious phone | Gutter 20px, slightly larger cards, 2-col grids breathe |
| **Tablet / Foldable open** (768px+, iPad, Pixel Fold inner) | Secondary | Tabs → sidebar rail or split view; 2-3 col grids; bottom sheet → side sheet; FAB → toolbar button; content max-width 720px centered |
| **Landscape phone** | Edge case | Tab bar stays bottom; sheets respect safe area; don't assume portrait |

Always test: **no overflow at 320px**, headings don't orphan, tap targets ≥44pt even at 320px, safe-area insets honored (notch, Dynamic Island, home indicator, camera cutout, rounded corners). Use `env(safe-area-inset-*)` or `SafeAreaView` — never hard-code top padding.

## Empty States, Loading & Error — Mobile's Most Under-Designed Surface

Most apps polish populated screens and ship grey `No data yet` for the zero state. That zero state is the *first* thing a new user sees — it determines activation. Design 4 variants per critical screen. Full anatomy, code, and instrumentation: `patterns/component-recipes.md` (Empty State + Skeleton + Toast sections) and `tokens/token-examples.md` for tone per category:

| Variant | When | What It Needs |
|---|---|---|
| **First-use (zero → one)** | Never had data | Name the *outcome* not the feature ("Track which deals are stalling" > "No projects"), ghost preview of populated UI, **one** primary CTA + one low-commitment escape (sample data / import / demo) |
| **User-cleared** | They archived/deleted everything | Celebrate or confirm: "All caught up" / "Inbox zero", offer undo or filter reset — don't re-teach |
| **No results** | Search/filter returned 0 | Show exact query, suggest closest matches, one-tap `Clear filters` + preserve search input |
| **Error / Offline** | Fetch failed, no network | Name what broke in human words ("Couldn't load — check connection") + `Retry` + cached fallback if possible; never masquerade as "0 items" (that panics users who had data) |

**Anatomy (mobile, in order top→bottom):** lightweight icon 30-40% viewport height max → one headline (verb-first: `Create your first project` > `No projects`) → one body sentence → **one primary CTA** (full-width on phone, 48dp tall) → secondary link (ghost row preview preferred over illustration — it teaches structure).

**Loading hierarchy by duration:**
```
0-300ms → show nothing (avoid flash)
300ms-2s → skeleton of known structure (prevents layout shift)
2s+ → skeleton + progress/copy ("Syncing your transactions…")
∞ / background → non-blocking indicator (pull-to-refresh spinner, inline progress) — never lock whole screen
```
**Error copy rules:** be explicit what happened, whether data is safe, and what to do next. Provide retry as thumb-reachable bottom bar action. Cache last-known content when possible (offline-first).

**Accessibility for empties:** heading is real `<h2>`; decorative preview `aria-hidden`; primary CTA receives focus on mount; container `role="status"` `aria-live="polite"` so screen readers announce after filter/search; muted text still ≥4.5:1.

## Responsive Typography & Safe Areas

- Use `clamp()` or fluid tokens — one token change propagates H1→caption without per-screen media queries.
- Hierarchy via **size + color**, not weight alone — especially on iOS where Inter/SF weight is 400 for body.
- Respect safe areas: status bar, Dynamic Island, notch, home indicator, rounded corners. Critical text and tappable areas never in the unsafe zone.
- Large text scales: `heading`/`display` scale with viewport; `ui`/`numeric`/`code` stay **fixed** so controls don't shift.

## Anti-Patterns — What Kills "Dribbble-Worthy" Mobile Status

Summary below. **Full checklist with 13 universal + 24 mobile-specific tells + red-flags**: `references/anti-slop-checklist.md` — run it before every handoff. It includes detection criteria, category-specific fixes (fintech/commerce/social/health/AI), and the self-correction script.

| Pattern | Problem | Fix | Ref |
|---|---|---|---|
| Identical iOS & Android UI | Feels alien on one platform, loses trust on install | Share brand tokens, adapt nav/icons/gestures per platform | `anti-slop` M1 |
| Primary CTA at top | Requires grip shift, fails thumb test on every use | Move to sticky bottom bar / bottom sheet actions / tab-integrated FAB | M2 |
| Hamburger hiding primary nav | Discoverability drops 50%+; users never find features | Bottom tabs (3-5) for primary; drawer only for secondary | M4 |
| Touch targets <44pt / cramped spacing | Misclicks, accessibility failure, App Store rejection risk | Enforce 44pt/48dp min + 8pt gap; audit with overlay | M3 |
| Applying Liquid Glass to every card/row | Visual clutter, hierarchy collapse, legibility loss | Glass ONLY on functional layer (tab bar, toolbar, sheet, FAB) | M5 |
| Using password fields in 2026 | Friction + security theater; passkeys are expected | Passkey + biometric primary; email magic link fallback; no CAPTCHA | M7 |
| 5-screen feature tour before value | Users skip; activation never happens | Show value first; contextual hints inline, fade after competence shown | — |
| Generic `No data yet` everywhere | Dead pixel that kills activation | Variant empties: first-use ghost preview + one CTA + instrumentation | M20 |
| Inverted dark mode (just negated light) | Bad contrast, wrong elevation, OLED glow issues | Dark mode is a second semantic token set — different surface hierarchy, not inverted colors | M21 |
| Over-animation (everything staggers) | Cheap, battery drain, motion sickness | Max 1 orchestrated entrance (first 6 items); micro-interactions only where they confirm state | U6 |
| Auto-spinning / bouncing / excessive parallax | Distracts, hurts perf on mid-range Android | Keep animations <300ms, 1-2 depth layers max, respect reduced-motion | `motion-system.md` |
| No offline/empty/error design | System shows frozen spinner or blank white | Design all states; skeleton > spinner; cached fallback + retry | M17 |
| Centered illustration + tiny CTA on empty | CTA below fold on small phones; users never see action | Full-width primary CTA + ghost preview; CTA in thumb zone | M20 |
| Pure white (#fff) on pure black (#000) | Harsh on OLED, unrealistic contrast | Offset: near-black #08080c-#141414, off-white #f5f5f7 | U7 |

## Technology Recommendations by Task

| Task | Best Tool (Mobile) | Alternative | When to Avoid |
|---|---|---|---|
| **Design / Prototype** | Figma + Apple iOS 18/26 kit + Material 3 kit (official) | Baseline / Appetite UI / Sublima (production Figma kits with tokens) | Don't invent custom components when platform kits exist |
| **Cross-platform build** | React Native + Expo + NativeWind (tokens → Tailwind) | Flutter (Material + Cupertino) | Need Liquid Glass fidelity → native SwiftUI |
| **iOS-native build** | SwiftUI + UIKit bridges (Liquid Glass APIs via `.glassEffect()` in iOS 26 SDK) | UIKit | Cross-platform sharing >80% → RN |
| **Android-native** | Compose Material 3 Expressive (dynamic color, expressive motion) | Views + Material Components | iOS-first brand needs Liquid Glass — use Compose only for Android artifact |
| **Animation** | Reanimated 3 / Motion (Framer) + native spring | Lottie (complex vector) / Rive (interactive) | Heavy JS-driven animation on low-end Android — use native driver |
| **Design tokens pipeline** | Figma Variables → Tokens Studio → Style Dictionary → CSS vars / Swift / XML | Supernova / Specify | Single-platform + single theme → CSS vars are enough |
| **Haptics** | `expo-haptics` / `UIImpactFeedbackGenerator` / `HapticFeedbackConstants` | — | Always pair haptics with gesture — never ship silent gestures |
| **Icons** | SF Symbols 6 (iOS) + Material Symbols (Android) | Phosphor / Lucide (cross-platform) | Don't use emoji as icons |
| **Charts** | Victory Native / react-native-gifted-charts | D3 via Skia | Simple stat → HTML number + sparkline, not full chart lib |
| **State / Nav** | Expo Router / React Navigation (tab+stack+sheet) + Zustand | — | Deep linking must account for active tab + stack position |

## Workflow Integration

When using this skill — **do not free-hand tokens/components/motion**. Source them from the bundled libraries. If the app category is not among the 7 modes / 9 token examples, the *Custom Category Research Protocol* (Mode Selection) is mandatory before step 5.

1. Load this skill (it triggers automatically on mobile design requests or via slash command)
2. **Read the router**: `workflows/index.md` — pick mode/token/layout/signature via decision trees + build matrix (Expo / SwiftUI / Compose / Flutter). If unlisted, router directs you to the Custom Research Protocol.
3. Run Phase 1 (Direction Discovery) — ask up to 5 questions OR make informed assumptions and state them. **Immediately after Phase 1**, test: *Is this category in the 7 modes or `tokens/token-examples.md`?* If **no** → execute the 5-8 live web searches defined in Mode Selection, synthesize a bespoke mode/theme/signature (do not proceed with a mismatched catalog theme).
4. Choose 1 color theme, 1 typography pairing, 1 layout philosophy from the catalogs — **or** use the custom theme you just synthesized from research — plus platform soul (Liquid Glass vs M3 Expressive vs adaptive). Fork from `tokens/token-examples.md` (9 pre-composed mobile themes or the *Template: Custom Category* scaffold) rather than authoring hexes from scratch. For custom categories, name tokens by domain (`--sky-now`, `--aqi-moderate`, `--alert-severe`) after your research findings.
5. Design the token system and write it as a token block (CSS vars / Figma Variables style) — include motion + haptic tokens from the chosen example. Dark mode = semantic override, not invert; glass = functional-layer only. For custom: cite 1 line per research query that justifies the hue/type choice.
6. Identify the signature element (mobile-tactile) and justify it — what will be screen-recorded? Pair with haptic/spring from `references/motion-system.md`. For custom: the signature must be *domain-specific* from your research (e.g., weather time-scrub, not generic card lift).
7. Map the screen flow (not isolated screens): launch → onboarding → auth → home → detail → create → empty/loading/error for each critical screen — use `patterns/layout-patterns.md` (#1-#12) for each screen's layout and `patterns/component-recipes.md` for its components. For custom: flow comes from the competitor teardown you did (core jobs + edge cases you found).
8. Build with **thumb-zone-first** layout: primary actions bottom, bottom sheets for secondary, 44pt targets, no horizontal overflow at 320px — audit with `assets/README.md` thumb-zone overlay
9. Self-check against `references/anti-slop-checklist.md` (U1-U13 + M1-M24) + accessibility floor (contrast, Dynamic Type 135%, screen reader labels, gesture fallbacks, glass over content test)
10. If in existing codebase mode: match their tokens first (grep `tokens.json` / `theme.ts`), extend where needed, preserve Style Dictionary → CSS vars / Swift / XML pipeline. For custom categories, extend with new semantic tokens rather than repurposing fintech/commerce hues.
11. Present with device frames (iPhone 15 Pro + Pixel 8, `assets/README.md`) + interaction notes (gesture + haptic + motion spec per screen, sourced from `references/motion-system.md`). For custom: include a 1-line `Research → Synthesis` footnote per screen that traces the design back to a query.

## Output Format

For each mobile app request, respond with this structure. Every section must cite which bundled library it sourced (so output is traceable, not free-handed):

```
## Direction Chosen
- Mode: [Expressive Consumer / Crafted Utility / Trust-Finance / Health & Fitness / Commerce / Game / Enterprise] — via `workflows/index.md` decision tree
- Subject: [what app you're building and why + target user]
- Platform Soul: [iOS 26 Liquid Glass / Material 3 Expressive / Adaptive (primary: __)] — via `workflows/index.md` soul selector
- Color Theme: [name] ([hex values + glass token]) — forked from `tokens/token-examples.md` (e.g., Vault / Pace / Atelier / Field)
- Typography: [pairing name] ([display + body + mono])
- Layout: [approach + navigation pattern (tabs/stack/sheet)] — via `patterns/layout-patterns.md` # + `workflows/index.md` nav selector
- Signature Element: [description + why tactile + haptic pairing] — motion from `references/motion-system.md`

## Token System
[CSS variables block: color (bg/surface/text/muted/primary/accent/success/warning/error/border/glass) + spacing + radius + elevation + typography + motion/haptics — forked from `tokens/token-examples.md`, tuned ≤3 hues]
[Note dark mode semantic override (not invert) + safe-area `env()` + Reduced Transparency fallback]

## Screen Map
[Flow diagram or ordered list: Launch → Onboarding → Auth → Home → Detail → Create → Search → Profile — each with: role, key content, thumb-zone placement, distinctive move — layouts sourced from `patterns/layout-patterns.md` #1-#12]
[For each critical screen note: primary CTA location (bottom sticky), empty/loading/error variant (from `patterns/component-recipes.md`), gesture + haptic spec (from `references/motion-system.md`)]

## Component Specs
[Tab bar / Nav bar / Bottom sheet / Cards / Lists / FAB / Chips / Inputs / Charts — with sizes, states (default/pressed/disabled/loading/error/success), and platform adaptation notes — sourced from `patterns/component-recipes.md`]

## Motion & Haptics
[Transition plan (push/modal/sheet/shared-element), micro-interactions, skeleton strategy, reduced-motion fallback, haptic map per interaction — sourced from `references/motion-system.md` — Liquid Glass morph uses `GlassEffectContainer` + `glassEffectID`]

## Implementation Notes
[Build stack choice (Expo/SwiftUI/Compose/Flutter) — via `workflows/index.md` build matrix; navigation (tab+stack+sheet preservation), token pipeline (Figma Variables → Tokens Studio → Style Dictionary → CSS/Swift/XML), accessibility (contrast + 135% type + focus trap), responsive (320 → tablet), offline handling, assets (`assets/README.md` frames)]
```

## Quick Start — No Questions Needed

If the user says "build a [category] mobile app" with enough context, skip questions and proceed. State assumptions inline. Don't wait for approval if the brief has enough signal.

Examples (predefined):
- "Build a mobile banking app" → infer: Trust/Finance mode, Midnight Pro or Liquid Glass theme, SF Native / Swiss Classic type, stack+card + bottom-sheet architecture, card-fan + drag-to-pay signature, passkey auth, thumb-bottom actions
- "Create a fitness tracking app" → infer: Health & Fitness mode, Nordic Frost or Aurora, Humanist Warm type, bento + stack layout, breathing-ring + haptic-synced signature, large 48dp log buttons, Apple Health sync pattern
- "Design a social feed app with AI" → infer: Expressive Consumer mode, Liquid Glass or Midnight Pro, Geometric type, tab+stack + bottom sheet, pull-to-refresh-becomes-composer signature, glassmorphic AI answer cards
- "Need a shopping app" → infer: Commerce mode, Editorial Cream or Mono Noir, Editorial type, 2-col product grid + bottom sheet variants, scrub-to-rotate signature, sticky bottom checkout CTA + Apple/Google Pay
- "Make a habit tracker" → infer: Crafted Utility, Aurora / Pastel Pop, Playful Rounded type, list-first + bento strikes grid, streak flame signature, empty = ghost preview + single CTA

Example (unlisted — must research):
- "Design a weather app" → **Not in 7 modes** → run Custom Research Protocol: queries `Dribbble best weather app design 2025 2026` → finds atmospheric gradients + glass radar sheets; `Carrot Weather UI teardown` → map-first + hourly scrub + alerts as inline banners; `weather app UX best practices` → current + hourly + 10-day + radar + alerts IA. Synthesize → `Mode: Custom (Health & Fitness + Data-Dense) → Atmospheric — calm glanceable + chart density, hourly scrub signature`, tokens with `--sky-now: #3a8dde` / `--alert-severe: #dc2626` (from research, not generic Aurora), layout `Canvas (radar) + Stack&Card (hourly)`, signature `drag time-scrub with live gradient shift + haptic tick per hour`.
- "Build a real estate app" → Not in catalog → research `best real estate apps 2026 Zillow UI` → map pin → sheet morph + filter chips for price/beds, Commerce + Trust hybrid, warm neutral palette from teardown.

## Notes for Future Passes

Keep a `DESIGN_NOTES.md` next to the project tracking: what direction you tried, which `tokens/token-examples.md` theme you forked, what the user reacted positively/negatively to, thumb-zone heatmap findings (`assets/README.md` overlay saves), which empty state variant lifted activation, what you'd avoid repeating. This compounds — you develop a taste + ergonomics profile for each user.

---

## File Structure (this skill)

```
dribbble-world-class-mobile-design/
├── SKILL.md                      ← you are here — primary manual
├── tokens/token-examples.md      ← 9 mobile themes (fintech/health/social/commerce/productivity/AI/gaming) + dark/motion tokens
├── patterns/component-recipes.md ← thumb-sized buttons, tab bars, sheets, cards, lists, skeletons, empties — CSS + Swift/Compose notes
├── patterns/layout-patterns.md   ← 12 mobile layouts + safe-area + foldable helpers
├── references/motion-system.md   ← springs, haptic map, Liquid Glass rules, Reanimated/CSS recipes
├── references/anti-slop-checklist.md ← U1-U13 + M1-M24 tells + self-correction script
├── workflows/index.md            ← router: mode/token/layout/signature decision trees + build matrix
└── assets/README.md              ← device frames, adaptive icons, splash, thumb-zone overlay
```
Use the router (`workflows/index.md`) first if you're unsure where to start — it points you to the exact token/layout/component file for your app category.

---

## Extended Theme Deep Dives (Mobile)

### Fintech / Banking (mobile)
- **Trust signals:** Monospace for numbers (tabular), 4.5:1 contrast on amounts, biometric gate on launch, amount always in `text-body-lg` weight 600, not muted.
- **Dark default:** Midnight Pro with single vivid accent; depth via border + subtle glow, not heavy shadows. Card art with specular highlight on tilt (Liquid Glass) or tonal surface (M3).
- **Home = Bento mobile:** Balance hero (large 34px display figure, secondary caption) + 4 quick actions (Send, Request, Top up, More) in pill grid → recent transactions as grouped list, not cards.
- **Transfers:** Progressive 3-step with bottom-sticky CTA; number pad is native (`inputMode="numeric"`), amount entry has haptic tick per digit; confirm uses `haptic-success`.
- **Charts:** Sparklines, not cluttered dashboards; period chips (1D/1W/1M/1Y) are filter chips in a horizontal scroll, not tabs.

### Health / Fitness / Wellness
- **Calm palette:** Nordic Frost or Aurora with desaturated accent; never neon for health data. Muted used at ≥4.5:1 after token audit.
- **Logging:** One-thumb, 48dp targets, plus/minus steppers with `light` haptic per increment, long-press accelerates.
- **Rings / Progress:** Large tappable hero (ring, circle, bar) is the signature — breathing animation 2s ease-in-out, paired with haptic `soft` pulse.
- **Permissions:** Request health/camera/motion only after value shown; explain benefit in priming sheet ("Enable motion to count steps even when closed").

### Social / Community / Creator
- **Feed:** Single-column stack, card with 16px inset, avatar 32px + name 13px label + timestamp caption; swipe-left reply, swipe-right mark-read — each distinct haptic, fallback via long-press menu.
- **Stories/Chips:** Horizontal snap carousel 80px items, peek 16px, spring snap.
- **Composer:** Bottom sheet that becomes full-screen on focus; keyboard accessory bar with media + AI suggestions; send button in thumb zone (bottom trailing inside sheet).
- **Social proof:** "Join X million" is muted caption inside bento, not full section — 2026 users skim it.

### Commerce / Marketplace / Food
- **Product card:** 2-col grid, 12px gap, image 1:1, title 15px, price 17px semibold, 2-line clamp; on tap shared element into detail.
- **Detail:** Image carousel with scrub dots + pinch-zoom that expands from tap point; variants in bottom sheet (not inline) to keep thumb zone; `Add to cart` is sticky bottom bar (full-width, 56px) + `haptic-medium`.
- **Checkout:** Apple Pay / Google Pay first button, divider, then card form; 3-4 fields per step, inline validation, never 10-field wall.
- **Empty cart:** Ghost cart illustration (30% height) + "Your cart is empty" + browsing CTA (bottom) — not apologies.

### Productivity / Tools
- **Home:** List-first with sticky section headers ("Today", "Upcoming") + swipe actions (archive/delete with icon morph).
- **Detail as sheet:** On phone, detail is stack push; on tablet, side sheet. Keep primary action (Complete/Edit) bottom-anchored.
- **Search:** Bottom sheet or dedicated tab; recents + chips + live results with skeleton per row.

### Gaming / Playful / Education
- **HUD:** Liquid Glass floating controls (clear variant over game art) or M3 expressive chips with spring press.
- **Onboarding as play:** First-use empty is a playable sample — e.g., Duolingo streak card, quiz with instant feedback animation, progress ring.
- **Haptics as UI:** Every coin, swipe, level-up has distinct haptic + sound toggle; respect mute switch.

### AI-Native Apps
- **2026 expectation:** Adaptive AI layouts (cards reorder by behavior), but only if you have distinct use modes — don't shuffle for novelty.
- **Signature patterns:** Orb/waveform that reacts to voice input; context-aware chip row that reorders; answer cards with spatial depth (glass + blur + parallax) — inspired by visionOS but restrained to one surface.
- **Privacy-first UI:** Green/red permission dots per capability (like Signal 2026); prompt composer shows what data is used *inline*, not in settings.

### App Icon & Splash
- **Icon:** iOS 1024×1024 master, corners applied by system — never pre-round. Android adaptive 108×108dp with 72dp safe zone. Liquid Glass icons: layered glass with specular highlight; provide Clear variant (translucent) + tinted.
- **Splash:** Use system splash API (Android 12+ `SplashScreen` + iOS `LaunchScreen.storyboard`). Splash is brand moment (<300ms), not loading screen — never park a spinner behind your logo. Skeleton the *app* instead.

## Component Library Reference (Mobile)

Full CSS/JSX/Swift/Compose recipes: `patterns/component-recipes.md`. Summary below:

### Buttons (thumb-sized — all ≥44pt)
```
Primary: filled, radius-full (pill) or radius-md (12-16px M3), 48-56px tall, glass shimmer on press (iOS) or ripple (Android)
  States: default / pressed (scale 0.97 + haptic light) / disabled (0.38 opacity) / loading (spinner replaces label, keep width)
Secondary: tonal / outlined, same height, border 1px token
Ghost: no border, text only, min 44pt tap area expanded with padding
Icon: 44×44pt min, circular 48px or rounded 12px, centered icon 20-24px
CTA Bar: sticky bottom, safe-area padded, full-width primary + ghost secondary — the mobile CTA pattern (not hero CTA)
  → See `patterns/component-recipes.md` § Button System + `assets/README.md` for frame spec
```

### Cards & Lists (→ `patterns/component-recipes.md` § Card & List System)
```
Card: surface + radius-card 16px + shadow-sm, inset 16px, press: scale 0.98 + shadow-md + haptic light
  Bento variants: span full / half / 2:1; consistent internal padding; press elevates
Glass Card (iOS only, floating): glass-regular + backdrop-blur 20px + glass-border; ONLY for controls over maps/photos
List Row: 56-64px tall (one-line: 56, two-line: 64, three-line: 80), leading icon 40px, trailing chevron/disclosure
  Grouped: insetGrouped style (16px outer margin, 12px section radius, divider inset 16px from leading)
  Swipe action: reveal on swipe 80px, icon + label, threshold haptic medium at commit
Bottom Sheet: radius-sheet 20px top, grabber 36×5px muted, 2 detents (medium/large), scrim 40% on M3 / blur on iOS
  → Recipes: `patterns/component-recipes.md` § Bottom Sheet + List Row + Glass Card
```

### Navigation Components (→ `patterns/component-recipes.md` § Navigation Components + `patterns/layout-patterns.md` #7 Tab & Stack)
```
Tab Bar (iOS Liquid Glass): glass-regular, height 49pt + safe-area, icon 24px + label 10px caption, active = primary + indicator dot; shrinks on scroll; floating concentric with device corners
Navigation Bar (M3): surface + indicator pill for active, height 80dp container, label 12px, icon 24px; rail variant on tablet (left, 80dp wide)
Top Bar: 56dp tall, title 17-22px, leading back + title + trailing actions (max 2 icons, overflow → sheet); translucent glass on iOS
FAB: M3 large FAB (56dp) or small (40dp) — trailing bottom inside content, not overlapping tab bar; iOS alternative = bottom toolbar primary action
Search: sticky top glass bar (iOS) or top app bar search (M3) — input 48dp tall, radius-full, placeholder muted, live results with skeleton rows
Chips: filter chips horizontal scroll, 32dp tall, radius-full, selected = tonal primary, unselected = outlined
  → Recipes + layout specs: `patterns/component-recipes.md` + `patterns/layout-patterns.md` #7
```

### Inputs & Forms (progressive disclosure) (→ `patterns/component-recipes.md` § Input & Form System)
```
Field: 48-56dp tall, radius 12px, border 1px + focus ring 2px primary, label 13px floating or above, error below with icon
  States: default / focused (ring) / error (error border + shake + haptic-error) / success (check + haptic-success) / disabled
Keyboard: native inputMode, accessory bar (prev/next/done) stays above keyboard; never hide primary action behind keyboard
Form layout: 3-4 fields max per step, bottom CTA stays visible (keyboard-aware), inline validation on blur, not on keystroke
Stepper / Picker: iOS wheel or inline; Android dropdown sheet — both 48dp targets, haptic light per value change
  → Recipes: `patterns/component-recipes.md` § Input & Form System + `workflows/index.md` checkout rules
```

### Typography Scale (same as tokens — single source)
```
Display: 34 / 28 / 22  — hero, balance, large numbers (weight 600-700, tracking -0.03em)
Title: 22 / 20 / 17    — screen titles, card titles (weight 600)
Body: 17 / 15          — readable content (line-height 1.5-1.625)
Label: 15 / 13 / 11    — buttons, captions, overline (uppercase + tracking 0.05em + mono)
Never below 13px for interactive labels; body never below 15px
```

### State Layers (→ `patterns/component-recipes.md` § Feedback & State Layers)
```
Skeleton: rounded rect matching card/row shape, shimmer 1.2s, 40% muted — recipe + reduced-motion guard in `patterns/component-recipes.md`
Toast/Snackbar: bottom-anchored above tab bar, 48dp tall, radius-full, 3s auto-dismiss, swipe-to-dismiss + haptic light
Error banner: inline (not full-screen) if only one module failed; full-screen only if whole screen failed — with Retry as bottom CTA
Offline: persistent indicator (subtle top bar or bottom banner) + queue badge; keep last-known content visible
  → Recipes: `patterns/component-recipes.md` § Toast + Skeleton + Empty State
```

### Spacing Cheat Sheet (phone)
```
Screen vertical rhythm: section-gap 32px between major blocks, stack-md 16px inside cards
Horizontal: gutter 16px (small phone stays 16px), card inset 16px, list horizontal 16px
Never use 1-3px arbitrary gaps — snap to 4/8 grid
```

## Final Directives

- Every app has a thumb. Design for it first, aesthetics second, platform conventions third — in that order.
- Spend your boldness in ONE tactile place. Let the signature element own the haptic + motion surprise.
- Restraint makes the bold stuff pop. Quiet everything around the signature; empty space is not wasted — it's the thumb's breathing room.
- If something could be a website shrunk to a phone, change it. Use native patterns: bottom sheets, swipe actions, bottom sticky CTAs, passkeys, skeletons, haptics.
- Glass belongs on the functional layer. Content is opaque and legible. When in doubt, use regular glass over busy content, not clear.
- Empty states are product. Ghost preview > illustration. One CTA. Instrument `view → click → time-to-first-action`.
- Dark mode is a second semantic token set with its own elevation. Never just invert.
- Mockups in device frames should look like real iOS 26 / M3 Expressive apps — with Dynamic Island, safe areas, status bar, and correct nav patterns — not floating rectangles.
- When in doubt, open Apple HIG (ios 26 Liquid Glass) and Material 3 Expressive docs and extract patterns, not copies. Reference real award-winning apps in the same category.
- An app screen is done when removing any single element would hurt clarity, and adding any single element would hurt thumb reach. And the whole flow is done when a new user can go from splash to first value in <60s without reading a tutorial.

---

## Appendix: Quick Token Starter (Copy-Paste)

```css
:root {
  /* Paste your theme here (Aurora/Midnight/Liquid Glass/M3 Expressive/etc.) */
  --bg: #f8f7f4; --surface: #ffffff; --surface-raised: #ffffff;
  --text: #0f172a; --muted: #64748b; --border: rgba(15,23,42,0.08);
  --primary: #7c3aed; --primary-pressed: #6d28d9; --accent: #ec4899;
  --success: #16a34a; --warning: #f59e0b; --error: #ef4444;
  --glass: rgba(255,255,255,0.72); --glass-border: rgba(255,255,255,0.5);

  --space-1: 4px; --space-2: 8px; --space-3: 12px; --space-4: 16px;
  --space-6: 24px; --space-8: 32px; --space-12: 48px;
  --gutter: 16px; --section-gap: 32px;
  --radius-card: 16px; --radius-sheet: 20px; --radius-full: 999px;
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.06); --shadow-md: 0 4px 12px rgba(0,0,0,0.08);

  --text-display-lg: clamp(28px, 7vw, 34px);
  --text-body-lg: 17px; --text-label-md: 13px;
  --duration-base: 250ms; --ease-out: cubic-bezier(0,0,0.2,1);

  /* Safe area — never hard-code top padding */
  --safe-top: env(safe-area-inset-top);
  --safe-bottom: env(safe-area-inset-bottom);
}

/* Dark override — semantic, not inverted */
@media (prefers-color-scheme: dark) {
  :root {
    --bg: #08080c; --surface: #121214; --surface-raised: #1c1c1f;
    --text: #f1f1f3; --muted: rgba(255,255,255,0.55);
    --border: rgba(255,255,255,0.08);
    --glass: rgba(28,28,31,0.68); --glass-border: rgba(255,255,255,0.08);
    --shadow-sm: 0 1px 2px rgba(0,0,0,0.4);
  }
}

/* Reduced transparency — solid fallback for glass */
@media (prefers-reduced-transparency: reduce) {
  :root { --glass: var(--surface); --glass-border: var(--border); }
}
```
