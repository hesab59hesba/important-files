# Dribbble World-Class Mobile Design — Skill Index

This skill produces Dribbble-worthy, award-level **mobile app UIs** — not websites in a phone frame. Every output is thumb-first, platform-aware (iOS 26 Liquid Glass vs Material 3 Expressive), and structured for real device ergonomics.

## How to Use This Skill

1. **Read this file** for routing and workflow selection
2. **Read `../SKILL.md`** — primary operating manual (Mode Selection + 3-phase process: Direction → Tokens → Signature Element)
3. **Pick tokens** from `../tokens/token-examples.md` by app category (fintech, health, social, commerce, etc.) or compose custom
4. **Apply layouts** from `../patterns/layout-patterns.md` + components from `../patterns/component-recipes.md`
5. **Check motion+haptics** in `../references/motion-system.md` and anti-slop in `../references/anti-slop-checklist.md` before shipping
6. **Produce app** following output format in `SKILL.md` (Direction → Token System → Screen Map → Component Specs → Motion/Haptics → Implementation Notes)

## File Structure

```
dribbble-world-class-mobile-design/
├── SKILL.md                         ← Primary manual (always read first)
├── tokens/
│   └── token-examples.md            ← 9 mobile token systems by category (fintech/health/social/commerce/productivity/AI/gaming + variants)
├── patterns/
│   ├── layout-patterns.md           ← 12 mobile layouts (Stack&Card, Bottom Sheet Arch, Bento Mobile, List-First, Carousel, Canvas, Tab+Stack, etc.) + safe-area helpers
│   └── component-recipes.md         ← Ready-to-use mobile components (tab bar, top bar, bottom sheet, cards, lists, buttons, inputs, toasts, skeletons, empty states...)
├── references/
│   ├── motion-system.md             ← Spring curves, Reanimated/Glass recipes, haptic map, reduced-motion
│   └── anti-slop-checklist.md       ← U1-U12 universal + mobile-specific tells + self-correction
├── workflows/
│   └── index.md                     ← This file (routing, decision trees, token/layout selectors)
└── assets/                          ← App icon / splash templates, device frames, thumb-zone overlays
```

## Quick Decision Trees

### Step 0: Is the requested category listed? (Do this first — before tokens/layouts)

```
Requested app → Is it in the 7 modes / tokens/token-examples.md (fintech, health, social, commerce, productivity, AI, gaming)?
│
├─ YES → Use that branch (see App Category trees below)
│
└─ NO  → Custom Category Research Protocol (MANDATORY — do not skip)
    │
    ├─ 1. Declare: Mode: Custom — [Candidate] → Research → [Final base mode remap]
    │     e.g., Weather → Custom (Health & Fitness + Data-Dense) — glanceable calm + radar/charts;
    │           Real Estate → Custom (Commerce + Trust) — imagery + high-stakes trust;
    │           Education → Custom (Game/Playful + Crafted Utility) — streaks + structured lessons
    │
    ├─ 2. Execute 5-8 LIVE web searches BEFORE any design (see SKILL.md for exact queries):
    │     • 2× Dribbble/awards: `Dribbble best [category] mobile app design 2025 2026` + `[category] mobile app award winning UI 2026`
    │     • 1× Platform: `[category] iOS HIG Material Design mobile patterns`
    │     • 1× Journeys: `[category] mobile app UX best practices user flow 2026`
    │     • 1× Teardown: `best [category] apps UI teardown 2026` (Carrot Weather / Zillow / Flighty / Duolingo — actual apps)
    │     • 1× Tokens (if needed): `[category] design tokens color typography trends`
    │     Document 1-line finding per query — you will cite these in Token System + Screen Map.
    │
    ├─ 3. Synthesize from findings (do NOT force a mismatched predefined theme):
    │     • Token: Use `tokens/token-examples.md` → Template: Custom Category scaffold — name hues by domain (--sky-now, --alert-severe, --aqi-moderate), not generic --primary copy
    │     • Layout: Pick from `patterns/layout-patterns.md` #1-#12 based on teardown (e.g., weather: Canvas radar #6 + Stack&Card hourly #1; real estate: Carousel #5 + Bottom Sheet #2)
    │     • Signature: Domain-native tactile idea FROM research (weather: time-scrub with gradient shift + haptic tick/hour; real estate: pin → sheet morph with image peek)
    │     • Screen Map: Derive IA from teardown jobs (weather: Current + Hourly + 10-day + Air Quality + Radar + Alerts + Locations — with 4 empty/loading/error variants each)
    │
    ├─ 4. Validate: Does custom mode hue stay outside 200-290° slop band? Contrast 4.5:1 outdoors? Glass only on functional layer?
    │
    └─ 5. Produce output with traceability: every Token choice + Screen cites `Research: [query → finding]` — see tokens/token-examples.md worked Weather "Cirrus" example.
         If you cannot cite a query, you hallucinated — go back to step 2.

Example — Weather "Cirrus" (unlisted → researched):
  • Queries → Findings: Dribbble weather → atmospheric blue-teal gradients + glass radar sheets; Carrot tear → map-first + personality + hourly scrub; UX best practices → current 72pt glanceable + alerts as inline banners
  • Remap: Health & Fitness + Data-Dense (calm glance + chart density)
  • Token: --sky-now #3a8dde / --alert-severe #dc2626 / --aqi-* — warm white surface for outdoor legibility (not Vault's #08080c)
  • Layout: Hero 40vh current + hourly snap + daily stack + radar Canvas + glass sheet
  • Signature: drag time-scrub with live sky-gradient shift + haptic selection/tick

Example — Real Estate "Found" (unlisted):
  • Teardown Zillow → map pin → sheet morph + price/bed filter chips + image-first 2-col grid; trust palette warm neutrals + reassurance copy
  • Token: --price-display / --bed-chip / --tour-primary — not Commerce's Mono Noir copy
```

### App Category → Mode → Token → Layout → Signature (for listed categories)

```
User: "Build a mobile banking app"
│
├─ Mode: Trust / Finance
├─ Token: Midnight Pro + Liquid Glass (Vault example, tokens/token-examples.md)
├─ Layout: Bento Mobile (home) + List-First (transactions) + Bottom Sheet (transfer) — patterns/layout-patterns.md #3,4,2
├─ Signature: Drag-to-pay slider + haptic ticks (references/motion-system.md)
└─ A11y: Tabular nums, 44pt targets, biometric gate (SKILL.md checklist)

User: "Fitness tracker"
│
├─ Mode: Health & Fitness
├─ Token: Nordic Frost / Aurora (Pace example)
├─ Layout: Bento Mobile (Today) + Stack&Card (History) + Centered Column (empty)
├─ Signature: Breathing ring + haptic pulse
└─ Perf: No parallax >2 layers, test outdoor contrast

User: "Social feed with AI"
│
├─ Mode: Expressive Consumer
├─ Token: Midnight Pro (Campfire) or Liquid Glass
├─ Layout: Stack&Card feed + Carousel snap (stories) + Bottom Sheet comments (#2, #5)
├─ Signature: Pull-to-refresh BECOMES composer (sheet peeks at 15%)
└─ Motion: Shared element card→detail, haptic selection on swipe-reply

User: "Shopping / marketplace"
│
├─ Mode: Commerce
├─ Token: Editorial Cream / Mono Noir (Atelier)
├─ Layout: 2-col product grid + Product Split Hero + Sticky Bottom Bar (#1, #8, #12)
├─ Signature: Scrub-to-rotate + bottom-sheet variants
└─ Checkout: Single-col, 3-4 fields/step, Apple/Google Pay first, sticky CTA

User: "Notes / productivity"
│
├─ Mode: Crafted Utility
├─ Token: Aurora (Field)
├─ Layout: List-First grouped + Stack&Card detail (#4, #1)
├─ Signature: Card expands via FLIP + swipe morph
└─ Empty: Ghost-row preview, not illustration

User: "Game / daily puzzle"
│
├─ Mode: Game / Playful
├─ Token: Pastel Pop (Grain)
├─ Layout: Centered Column canvas + floating glass toolbar (#9 + #6)
├─ Signature: Flame lick + success haptic burst
└─ Haptics: every coin/swipe has distinct pattern + mute toggle
```

### Navigation Pattern Selector

```
How many top-level destinations?
│
├─ 1 (single-task: calculator, camera) → No tabs. Single screen + toolbars.
│
├─ 2 → Toggle / segmented control (not tab bar — 2 tabs is wasteful)
│
├─ 3-5 peer destinations (most apps) → Bottom tabs, icon + label, each with its own stack
│   ├─ iOS 26: floating glass capsule, shrinks on scroll, concentric
│   └─ Android: M3 NavigationBar with indicator pill; tablet → Rail
│
├─ 5+ equally important → You have too many. Group into 3-5 or move to Profile tab.
│   └─ If truly complex (B2B) → Tabs for top 4 + Profile tab housing More (grouped list)
│
└─ Secondary/rare → Hamburger drawer ONLY for secondary (Help, Legal, Account). Never primary.
Filters / sort / share / confirm → Bottom sheet (#2), not nav.
One defining action (Compose, Scan, Add) → FAB (M3) or bottom toolbar primary, not a tab.
```

### Platform Soul Selector

```
Where does this app live?
│
├─ iOS only / premium US/EU → iOS 26 Liquid Glass primary
│   └─ Tab bar = glass capsule, sheets with detents + grabber, SF Symbols, .glassEffect() + GlassEffectContainer
│
├─ Android only / emerging markets → Material 3 Expressive primary
│   └─ Dynamic color (tonal palette), NavigationBar + FAB, Material Symbols, surface-container steps
│
├─ Cross-platform (Expo / RN / Flutter) → Shared semantic tokens, adaptive shell
│   └─ Primitive → Semantic → Component (W3C DTCG + Style Dictionary → CSS vars / Swift / XML)
│   └─ Icons: Phosphor/Lucide for shared spots, SF/Material for platform chrome
│
└─ Marketing site + app → Web = dribble-world-class-design; App = this skill. Share tokens, adapt mechanics.
```

## Token Selection Guide (Mobile)

| Domain | Recommended Theme (tokens/token-examples.md) | Why |
|---|---|---|
| Neobank / fintech light | Aurora + Midnight Pro dark pair | Trust + OLED economy |
| Trading / crypto dark | Midnight Pro / Neon Tokyo tinted | Data clarity + glow instead of shadow |
| Health / fitness | Nordic Frost or Aurora | Clinical but warm, outdoor legibility |
| Wellness / journal | Editorial Cream / Flora | Calm, serif warmth |
| Social / creator | Midnight Pro or Neon Tokyo | Dark listening/viewing UI |
| Community / neighborhood | Pastel Pop or Aurora | Approachable, light |
| Fashion / luxury commerce | Mono Noir or Editorial Cream | Editorial, image-first |
| Food / delivery | Lava (dark warm) | Appetite, speed |
| Productivity / tools | Aurora | System-respectful, long-session |
| AI assistant | Cyber Violet / Midnight Pro | Forward, answer-card depth |
| Gaming / kids | Pastel Pop or Neon Tokyo | Play + haptics |
| **Not listed? (weather, real estate, education, travel, events, pets, civic, etc.)** | **`Template: Custom Category` scaffold** (tokens/token-examples.md) — name hues by domain after research, don't reuse generic fintech palette | **Mandatory research synthesis — see Step 0 above** |

> For unlisted domains, do NOT pick the closest row from this table as your final theme. Use it only to pick a *candidate* base mode, then synthesize a bespoke token set via the 5-8 searches. Document the divergence: “Weather candidate Nordic Frost → bespoke atmospheric tokens `--sky-now / --alert-severe` per Dribbble + Carrot teardown.”

## Workflow Integration

This skill works with:

- **frontend-design** / **ios-animation-design**: execution + Liquid Glass/Compose specifics
- **design-blueprint**: structured `DESIGN.md` before coding
- **product-design**: flows beyond visuals (IA, task completion)

Recommended order:

1. Load this skill for direction, tokens, layouts, motion+haptics, anti-slop
2. Load `frontend-design` for HTML/CSS/React Native constraints (env, system fonts)
3. Load `ios-animation-design` if building native iOS 26 glass + motion
4. Load `design-blueprint` if you need a `DESIGN.md` spec before implementation

## Screen Flow Checklist (Mobile)

For every app, map **all states per critical screen**, not just happy path:

- [ ] Launch (system splash <300ms, not custom loader)
- [ ] Onboarding (1-3 screens max, value-first, Skip visible — not feature tour)
- [ ] Auth (passkey + biometric primary, magic link fallback, no password strength meter)
- [ ] Permission priming (contextual sheet *before* system dialog explains "why")
- [ ] Home / Feed (bento or list, skeletons, pull-to-refresh with haptic)
- [ ] Detail (stack push, shared element, swipe-back)
- [ ] Create / Composer (bottom sheet or sticky composer, thumb actions bottom)
- [ ] Search (bottom sheet on maps/commerce, chips + recent + live skeleton rows)
- [ ] Settings / Profile (grouped list, insetGrouped, 48dp rows)
- [ ] Empty (4 variants: first-use ghost preview + one CTA, cleared, no-results, error — see patterns/component-recipes.md)
- [ ] Loading states: 0-300ms none / 300ms-2s skeleton / 2s+ progress copy
- [ ] Error / Offline: plain language + retry above keyboard + cached fallback
- [ ] Success (confetti + success haptic, not just green toast)
- [ ] Deep link + tab preservation test
- [ ] 320px overflow, 135% Dynamic Type, Increase Contrast, Reduced Transparency, Reduced Motion

## Build Execution Matrix

| Stack | Tab Pattern | Glass Implementation | Haptics |
|---|---|---|---|
| **Expo / React Native** | Expo Router (tabs) + NativeWind + `react-native-gesture-handler` + `gorhom-bottom-sheet` | `expo-blur` + translucent hex tokens; native iOS build uses `.glassEffect` via native module | `expo-haptics` (`selection`, `impact/light|medium|heavy`, `notification/success|warning|error`) |
| **SwiftUI (iOS 26)** | `TabView` + `TabBarMinimizeBehavior` | `.glassEffect()` + `GlassEffectContainer` + `.interactive()` + `glassEffectID` for morph | `UIImpactFeedbackGenerator`, `UINotificationFeedbackGenerator`, `sensoryFeedback` in SwiftUI |
| **Compose (M3 Expressive)** | `NavigationBar` / `NavigationRail` + `ModalBottomSheet` | `surface-container` tonal steps (no blur — M3 uses color, not glass) | `HapticFeedbackConstants` |
| **Flutter** | `NavigationBar` / `NavigationRail` + `showModalBottomSheet` + `DraggableScrollableSheet` | Frosted via `BackdropFilter` (budget 1-2 layers) | `HapticFeedback.lightImpact() / mediumImpact() / heavyImpact()` |
| **Mobile web (PWA)** | CSS tab bar + sticky cta-bar (see recipes) | CSS `backdrop-filter: blur(20px) saturate(160%)` + `supports` fallback | `navigator.vibrate` (light: 20ms, medium: 40ms) + no-op fallback |

## Self-Check Before Every Output

Run mentally — any fail → revise before presenting. For unlisted categories, also run the *Custom* checks (13-15):

1. [ ] Did I pick a specific hue outside 200-290° violet band? (not generic purple-blue)
2. [ ] Is dark a semantic token set, not inverted light? Tested OLED + Increase Contrast?
3. [ ] Is glass ONLY on functional layer (tab bar, top bar, sheet, FAB)? No glass on cards/rows?
4. [ ] Are all CTAs in thumb zone (bottom sticky bar / sheet footer)? No primary at top?
5. [ ] Bottom tabs 3-5, labeled, each with own stack; hamburger only for secondary?
6. [ ] All touch targets ≥44pt/48dp + 8pt gap, verified 320px + 135% type?
7. [ ] One signature tactile element (with haptic + spring) and rest is quiet?
8. [ ] Empty states are 4 variants with ghost preview, not blank / "No data yet"?
9. [ ] Motion is spring (`ease-spring`) with reduced-motion fallback, never generic `ease-in-out` everywhere?
10. [ ] No slop copy ("Seamlessly unlock", "Next-Gen", em-dash anywhere, "—") and no lorem ipsum?
11. [ ] Passkey auth, single-col forms, correct inputmode/autocomplete, sticky CTA above keyboard?
12. [ ] A11y: `4.5:1` contrast, focus rings, `aria-label` on icon buttons, `aria-modal` + focus trap on sheets, keyboard nav?
13. [ ] **If custom category:** Did I run 5-8 live searches before any design? Can I cite 1-line finding per query?
14. [ ] **If custom:** Are tokens named by domain purpose (`--sky-now`, not generic `--primary` copy) and derived from research, not memory?
15. [ ] **If custom:** Is the signature domain-native from teardown (weather scrub, real-estate pin morph) — not generic card lift — and is the IA from teardown jobs with 4-state empties?
