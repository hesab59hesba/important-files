# Mobile Component Recipes

Every recipe is thumb-first (44×44pt iOS / 48×48dp Android min, 8pt gap), token-driven, and platform-adaptive. No magic hex, no `px` outside tokens. Test at 320px and 135% Dynamic Type before shipping.

> Assets/icons: SF Symbols 7 (iOS, 6,900+ symbols, 9 weights) or Material Symbols (Android, variable weight/fill). Never emoji as icons.

---

## Button System (sticky bottom is the mobile CTA pattern)

### Primary Button (full-width sticky — default mobile CTA)

```css
.btn-primary-mobile {
  display: inline-flex; align-items: center; justify-content: center; gap: 8px;
  width: 100%; height: 56px; /* 44pt min, 56 is comfortable thumb */
  padding: 0 24px;
  background: var(--primary); color: var(--on-primary, #fff);
  border: none; border-radius: var(--radius-full);
  font: 600 17px var(--font-body); letter-spacing: -0.01em;
  cursor: pointer;
  transition: transform 0.15s var(--ease-out), opacity 0.15s;
  /* haptic: light on press */
}
.btn-primary-mobile:active { transform: scale(0.97); }
.btn-primary-mobile:disabled { opacity: 0.38; pointer-events: none; }
.btn-primary-mobile:focus-visible { outline: 2px solid var(--primary); outline-offset: 2px; }

/* Sticky bottom bar container */
.cta-bar {
  position: sticky; bottom: 0; z-index: 20;
  padding: 12px var(--gutter) calc(12px + env(safe-area-inset-bottom));
  background: linear-gradient(to top, var(--surface) 70%, transparent);
  backdrop-filter: blur(0); /* iOS: switch to glass-regular if you need translucency */
  border-top: 1px solid var(--border);
}
.cta-bar--glass {
  background: var(--glass-regular);
  backdrop-filter: blur(20px) saturate(160%);
  border-top: 1px solid var(--glass-border);
}
```

Usage: Cart, checkout, transfer confirm, create. One primary per screen. Secondary is text link beside it, not second pill that competes.

### Secondary / Tonal Button

```css
.btn-tonal {
  height: 48px; padding: 0 20px;
  background: var(--surface-container, rgba(124,58,237,0.12));
  color: var(--primary); border: none; border-radius: var(--radius-full);
  font: 600 15px var(--font-body);
}
.btn-outlined {
  height: 48px; padding: 0 20px;
  background: transparent; color: var(--text);
  border: 1.5px solid var(--border); border-radius: var(--radius-full);
  font: 600 15px var(--font-body);
}
```

### Ghost / Text Button

```css
.btn-ghost {
  min-height: 44px; /* tap area even though text is 13px */
  padding: 10px 14px; border-radius: 8px;
  background: transparent; color: var(--primary); border: none;
  font: 600 15px var(--font-body);
}
.btn-ghost--muted { color: var(--muted); }
```

### Icon Button (44pt circle guaranteed)

```css
.btn-icon {
  width: 48px; height: 48px; min-width: 48px; min-height: 48px;
  border-radius: 50%; border: 1.5px solid var(--border);
  background: transparent; color: var(--text);
  display: inline-flex; align-items: center; justify-content: center;
  font-size: 20px;
}
.btn-icon--filled { background: var(--surface-raised); border-color: transparent; }
.btn-icon:active { transform: scale(0.96); }
.btn-icon:focus-visible { outline: 2px solid var(--primary); outline-offset: 2px; }
```

---

## Card & List System

### Card (opaque — content layer, NEVER glass)

```css
.card-mobile {
  background: var(--surface); border: 1px solid var(--border);
  border-radius: var(--radius-card); padding: 16px;
  transition: transform 0.2s var(--ease-out), box-shadow 0.2s;
}
.card-mobile:active { transform: scale(0.98); box-shadow: var(--shadow-md); }
.card-mobile--bento { /* used in 2-col bento grids */
  display: flex; flex-direction: column; gap: 12px;
}
.card-mobile--featured { border-color: transparent; box-shadow: var(--shadow-md); }
.card-mobile--featured:focus-within { outline: 2px solid var(--primary); outline-offset: 2px; }
```

### Glass Card (FLOATING ONLY — over maps/photos/video)

```css
.card-glass--floating {
  background: var(--glass-regular);
  backdrop-filter: blur(20px) saturate(160%);
  border: 1px solid var(--glass-border);
  border-radius: var(--radius-card);
  box-shadow: var(--shadow-glass);
}
/* NEVER use on list rows or feed cards — causes blur budget blowout on scroll */
```

### List Row (56-64px, insetGrouped style)

```css
.list-group {
  background: var(--surface); border-radius: 12px;
  margin: 0 var(--gutter); overflow: hidden;
  border: 1px solid var(--border);
}
.list-row {
  display: flex; align-items: center; gap: 12px;
  min-height: 56px; padding: 12px 16px;
  background: transparent; border: none; width: 100%; text-align: left;
}
.list-row--two-line { min-height: 64px; }
.list-row--three-line { min-height: 80px; }
.list-row + .list-row { border-top: 1px solid var(--border); }
.list-row__leading { width: 40px; height: 40px; border-radius: 10px; flex: none; display: grid; place-items: center; background: var(--surface-raised); }
.list-row__title { font: 600 17px var(--font-body); color: var(--text); }
.list-row__subtitle { font: 400 15px var(--font-body); color: var(--muted); }
.list-row__trailing { margin-left: auto; color: var(--muted); }
/* Swipe action reveal */
.list-row--swipeable { position: relative; overflow: hidden; }
.swipe-action {
  position: absolute; inset: 0 0 0 auto; width: 80px;
  display: grid; place-items: center; background: var(--error); color: #fff;
  transform: translateX(100%); transition: transform 0.25s var(--ease-out);
}
.list-row--swipeable[data-swipe="open"] .swipe-action { transform: translateX(0); }
```

---

## Navigation Components

### Bottom Tab Bar (iOS Liquid Glass — floating, shrinks on scroll)

```css
.tab-bar {
  position: fixed; left: 12px; right: 12px; bottom: calc(12px + env(safe-area-inset-bottom));
  display: flex; justify-content: space-around; align-items: center;
  height: 64px; padding: 6px 8px;
  background: var(--glass-regular);
  backdrop-filter: blur(20px) saturate(160%);
  border: 1px solid var(--glass-border);
  border-radius: 32px; /* concentric with device corners */
  box-shadow: var(--shadow-glass);
  z-index: 30;
  transition: transform 0.3s var(--ease-out), opacity 0.2s;
}
.tab-bar--shrunk { transform: translateY(8px) scale(0.96); opacity: 0.9; }
.tab-item {
  flex: 1; display: flex; flex-direction: column; align-items: center; gap: 2px;
  min-width: 44px; min-height: 44px; justify-content: center;
  background: none; border: none; color: var(--muted);
  font: 500 10px var(--font-body); letter-spacing: 0.02em;
}
.tab-item[aria-selected="true"] { color: var(--primary); }
.tab-item__icon { font-size: 24px; line-height: 1; }
.tab-item__label { font-size: 10px; }
.tab-item__badge {
  position: absolute; top: 6px; right: 18px;
  min-width: 18px; height: 18px; padding: 0 5px;
  background: var(--error); color: #fff; border-radius: 999px;
  font: 700 11px var(--font-body); display: grid; place-items: center;
}
@media (prefers-reduced-transparency: reduce) {
  .tab-bar { background: var(--surface); backdrop-filter: none; }
}
```

Android alternative (M3 NavigationBar):

```css
.nav-bar--m3 {
  position: fixed; left: 0; right: 0; bottom: 0;
  height: 80px; padding-bottom: env(safe-area-inset-bottom);
  display: flex; justify-content: space-around; align-items: center;
  background: var(--surface-container); border-top: 1px solid var(--border);
}
.nav-bar--m3 .tab-item { flex: 1; }
.nav-bar--m3 .tab-item[aria-selected="true"] .tab-item__icon {
  background: var(--primary); color: var(--on-primary);
  border-radius: 16px; padding: 4px 16px;
}
```

### Top Bar (translucent on iOS)

```css
.top-bar {
  position: sticky; top: 0; z-index: 20;
  height: 56px; padding: 0 var(--gutter);
  padding-top: env(safe-area-inset-top);
  display: flex; align-items: center; gap: 12px;
  background: var(--glass-regular);
  backdrop-filter: blur(20px) saturate(160%);
  border-bottom: 1px solid var(--border);
}
.top-bar__title { font: 600 17px var(--font-body); color: var(--text); }
.top-bar__action { min-width: 44px; min-height: 44px; }
```

### Bottom Sheet (detents: peek / medium / large)

Mobile web (CSS + JS) — native is `UISheetPresentationController` / `ModalBottomSheet`:

```html
<div class="sheet-scrim" data-open="true"></div>
<div class="sheet" role="dialog" aria-modal="true" data-detent="medium">
  <div class="sheet__grabber" aria-hidden="true"></div>
  <div class="sheet__header">
    <h2 class="sheet__title">Filters</h2>
    <button class="btn-icon" aria-label="Close">✕</button>
  </div>
  <div class="sheet__body"><!-- scrollable content --></div>
  <div class="sheet__footer cta-bar"><!-- bottom actions --></div>
</div>
```
```css
.sheet-scrim {
  position: fixed; inset: 0; background: rgba(0,0,0,0.4);
  opacity: 0; pointer-events: none; transition: opacity 0.25s;
}
.sheet-scrim[data-open="true"] { opacity: 1; pointer-events: auto; }
.sheet {
  position: fixed; left: 0; right: 0; bottom: 0;
  background: var(--surface); border-radius: var(--radius-sheet) var(--radius-sheet) 0 0;
  max-height: min(92dvh, 92vh); display: flex; flex-direction: column;
  transform: translateY(100%); transition: transform 0.35s var(--ease-spring);
  box-shadow: 0 -8px 32px rgba(0,0,0,0.2);
}
.sheet[data-detent="medium"] { transform: translateY(0); height: 50dvh; }
.sheet[data-detent="large"] { transform: translateY(0); height: 92dvh; }
.sheet[data-detent="peek"] { transform: translateY(0); height: 24dvh; }
.sheet__grabber {
  width: 36px; height: 5px; border-radius: 999px;
  background: var(--border); margin: 10px auto 0;
}
.sheet__header { display: flex; align-items: center; justify-content: space-between; padding: 12px var(--gutter); border-bottom: 1px solid var(--border); flex: none; }
.sheet__body { overflow-y: auto; padding: 16px var(--gutter); flex: 1; -webkit-overflow-scrolling: touch; }
.sheet__footer { flex: none; border-top: 1px solid var(--border); }
/* Inset on large phones so content peeks */
@media (min-width: 430px) {
  .sheet { left: 8px; right: 8px; border-radius: var(--radius-sheet); bottom: 8px; max-height: 88dvh; }
}
```

Native notes:
- iOS: `sheet.detents = [.medium(), .large()]`, `sheet.selectedDetentIdentifier = .medium`, `sheet.prefersGrabberVisible = true`, `sheet.largestUndimmedDetentIdentifier = .medium` for map-floating.
- Compose: `ModalBottomSheet(sheetState = rememberModalBottomSheetState(skipPartiallyExpanded = false))` with 28px top shape.
- **Rule**: ≤2 detents. 3+ is unpredictable. Nesting sheets is forbidden — push within sheet instead.

### FAB (M3) — integrated, not overlapping tab bar

```css
.fab {
  position: fixed; right: 16px; bottom: calc(96px + env(safe-area-inset-bottom)); /* above tab bar */
  width: 56px; height: 56px; border-radius: 16px;
  background: var(--primary); color: var(--on-primary);
  border: none; display: grid; place-items: center; font-size: 24px;
  box-shadow: var(--shadow-md);
}
.fab--small { width: 40px; height: 40px; border-radius: 12px; font-size: 20px; }
.fab:active { transform: scale(0.96); }
/* On large sheet/tablet, FAB moves into toolbar — not floating */
```

---

## Input & Form System

```css
.field {
  width: 100%; height: 56px; padding: 14px 16px;
  background: var(--surface); border: 1.5px solid var(--border);
  border-radius: 12px; font: 400 17px var(--font-body); color: var(--text);
  transition: border-color 0.2s, box-shadow 0.2s;
}
.field:focus { outline: none; border-color: var(--primary); box-shadow: 0 0 0 3px rgba(124,58,237,0.15); }
.field[aria-invalid="true"] { border-color: var(--error); }
.field::placeholder { color: var(--muted); }
.field__label { font: 500 13px var(--font-body); color: var(--muted); margin-bottom: 6px; display: block; letter-spacing: 0.01em; }
.field__error { font: 400 13px var(--font-body); color: var(--error); margin-top: 6px; display: flex; gap: 6px; align-items: center; }
.field__helper { font: 400 13px var(--font-body); color: var(--muted); margin-top: 6px; }

/* Segmented / Chips */
.chip-row { display: flex; gap: 8px; overflow-x: auto; scrollbar-width: none; padding-bottom: 2px; }
.chip-row::-webkit-scrollbar { display: none; }
.chip {
  height: 32px; padding: 0 14px; border-radius: var(--radius-full);
  border: 1.5px solid var(--border); background: transparent;
  font: 500 14px var(--font-body); color: var(--text); white-space: nowrap; flex: none;
}
.chip[aria-selected="true"] { background: var(--primary); color: var(--on-primary); border-color: transparent; }
.chip:focus-visible { outline: 2px solid var(--primary); outline-offset: 2px; }

/* Stepper / Quantity (44dp) */
.stepper { display: inline-flex; align-items: center; gap: 8px; }
.stepper button { width: 44px; height: 44px; border-radius: 50%; border: 1.5px solid var(--border); background: var(--surface); font-size: 18px; }
```

**Form rules (mobile checkout):** single-column, 3-4 fields max per step, correct `inputmode`/`type`/`autocomplete` (`email`→email keyboard, `tel`→dial pad, `postal-code`→numeric), inline validation on blur (error scrolls to field, not top), sticky CTA stays above keyboard (`visualViewport` resize), address autocomplete via Google Places / `autocomplete` attributes (`given-name`, `family-name`, `street-address`, etc.), guest checkout default.

---

## Feedback & State Layers

### Toast / Snackbar (above tab bar, thumb-dismissible)

```css
.toast {
  position: fixed; left: var(--gutter); right: var(--gutter);
  bottom: calc(88px + env(safe-area-inset-bottom)); /* above tab bar */
  display: flex; align-items: center; gap: 12px;
  min-height: 48px; padding: 12px 16px;
  background: #1a1a1e; color: #fff; border-radius: var(--radius-full);
  font: 500 15px var(--font-body);
  box-shadow: var(--shadow-md);
  transform: translateY(12px); opacity: 0; transition: all 0.25s var(--ease-out);
}
.toast[data-show="true"] { transform: translateY(0); opacity: 1; }
.toast button { color: var(--accent); font-weight: 600; background: none; border: none; }
```

### Empty State (ghost preview > illustration)

```html
<section class="empty" role="status" aria-live="polite">
  <div class="empty__preview" aria-hidden="true">
    <!-- ghost row(s): faded real UI, not generic illustration -->
    <div class="skeleton-row" style="opacity:0.45"></div>
  </div>
  <h2 class="empty__title">Create your first project</h2>
  <p class="empty__body">Projects hold your files, tasks, and teammates. Most teams start with a welcome tour.</p>
  <button class="btn-primary-mobile" style="margin-top:12px">Create project</button>
  <a class="btn-ghost" href="#">Use a template</a>
</section>
```
```css
.empty { display: flex; flex-direction: column; align-items: center; gap: 8px; padding: 32px var(--gutter); text-align: center; }
.empty__title { font: 700 20px var(--font-body); color: var(--text); margin: 8px 0 0; }
.empty__body { font: 400 15px var(--font-body); color: var(--muted); max-width: 32ch; line-height: 1.5; }
.empty__preview { width: 100%; max-width: 360px; opacity: 0.9; }
```

Variants: `first-run` (ghost preview + one primary), `cleared` ("All caught up" + undo), `no-results` (preserve query/chips + `Clear filters`), `error` (human words + `Retry`, cache last-known content).

### Skeleton (not spinner for known structure)

```css
.skeleton {
  background: linear-gradient(90deg, var(--surface-raised) 25%, var(--border) 50%, var(--surface-raised) 75%);
  background-size: 200% 100%;
  animation: shimmer 1.2s ease-in-out infinite;
  border-radius: 8px;
}
.skeleton--row { height: 64px; }
.skeleton--card { height: 120px; border-radius: var(--radius-card); }
@keyframes shimmer { 0% { background-position: 200% 0; } 100% { background-position: -200% 0; } }
@media (prefers-reduced-motion: reduce) { .skeleton { animation: none; background: var(--surface-raised); } }
```

Loading hierarchy: 0-300ms nothing → 300ms-2s skeleton → 2s+ skeleton + copy ("Syncing…") → background task = inline spinner, never whole-screen lock.

### Badge / Tag

```css
.badge {
  display: inline-flex; align-items: center; gap: 6px;
  height: 24px; padding: 0 10px; border-radius: var(--radius-full);
  font: 600 12px var(--font-mono, monospace); letter-spacing: 0.04em; text-transform: uppercase;
}
.badge--success { background: rgba(34,197,94,0.15); color: var(--success); }
.badge--warning { background: rgba(251,191,36,0.2); color: #b45309; }
.badge--neutral { background: var(--surface-raised); color: var(--muted); border: 1px solid var(--border); }
```

---

## Typography Components (mobile scale)

```html
<p style="font:500 11px var(--font-mono); letter-spacing:0.08em; text-transform:uppercase; color:var(--primary);">Section label</p>
<h2 style="font:800 clamp(24px,6vw,32px)/1.1 var(--font-display); letter-spacing:-0.02em; color:var(--text);">Hero headline</h2>
<div style="font:800 clamp(28px,7vw,40px)/1 var(--font-display); letter-spacing:-0.03em;">2,847</div>
<span style="font:600 13px var(--font-body); color:var(--muted);">projects shipped</span>
```

---

## Search & Filters

```css
.search-bar {
  display: flex; align-items: center; gap: 10px;
  height: 48px; padding: 0 16px;
  background: var(--surface); border: 1.5px solid var(--border);
  border-radius: var(--radius-full); /* pill */
}
.search-bar input { flex: 1; border: none; background: transparent; font: 400 17px var(--font-body); color: var(--text); outline: none; }
.search-bar input::placeholder { color: var(--muted); }
.search-bar--glass { background: var(--glass-regular); backdrop-filter: blur(12px); }
```

Treat search as bottom sheet on maps/commerce: pull handle + chips inside sheet, not top bar.

---

## Responsive Grid Helper (mobile-first)

```css
.grid-auto { display: grid; gap: 12px; grid-template-columns: 1fr; }
@media (min-width: 430px) { .grid-auto { grid-template-columns: repeat(2, 1fr); gap: 16px; } }
@media (min-width: 768px) { .grid-auto { grid-template-columns: repeat(3, 1fr); gap: 20px; } }
/* On tablet, bottom tab bar → sidebar rail (80dp) */
@media (min-width: 768px) {
  .tab-bar { left: 0; right: auto; top: 0; bottom: 0; width: 80px; height: 100dvh; border-radius: 0; flex-direction: column; padding: 16px 0; }
  .tab-item { flex-direction: column; }
}
```

---

## Focus & Accessibility (non-negotiable)

```css
*:focus-visible { outline: 2px solid var(--primary); outline-offset: 2px; border-radius: 4px; }
a:focus-visible { outline-offset: 4px; border-radius: 2px; }
input:focus-visible, textarea:focus-visible, select:focus-visible { outline: none; box-shadow: 0 0 0 3px rgba(124,58,237,0.15); }

/* Touch target audit */
@media (pointer: coarse) {
  button, a, [role="button"] { min-height: 44px; min-width: 44px; }
}
```

- Every icon-only button has `aria-label`.
- Decorative preview/ghost rows are `aria-hidden`.
- Sheet is `role="dialog"` `aria-modal="true"` with focus trap + Escape to close.
- Never rely on color alone — badge + label + icon.

---

## Noise / Texture (use sparingly, mobile perf cost)

```css
/* Light grain via SVG — keep opacity ≤0.03 */
.texture-grain {
  position: relative;
}
.texture-grain::after {
  content: ""; position: absolute; inset: 0; pointer-events: none; opacity: 0.025;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
}
```
