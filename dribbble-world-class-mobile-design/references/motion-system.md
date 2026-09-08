# Motion & Haptics System (Mobile)

Mobile motion is functional — it communicates *state change, spatial relationship, and feedback*. Decorative motion is slop. On mobile, every ms costs battery on mid-range Android; every miss loses trust.

> Principle: `haptics + motion + visual` in harmony. A gesture without haptics is a guess. Animation without reduced-motion fallback is inaccessible.

---

## Timing Scale (shared — never improvise)

| Token | Duration | Easing | Use Case |
|---|---|---|---|
| **Instant** | 0ms | — | Visibility toggles |
| **Micro** | 150ms | `ease-out` (spring-light) | Button press 0.97 scale, chip select, icon morph |
| **Quick** | 250ms | `cubic-bezier(0.4,0,0.2,1)` or `.spring(response:0.35, damp:0.85)` | Push/pop, sheet settle, toast |
| **Standard** | 350ms | `.spring(response:0.4, damp:0.8)` | Modal/sheet spring from bottom, expand |
| **Entrance** | 300-500ms | `.spring(response:0.5, bounce:0.3)` | Staggered list entrance (first 6 only) |
| **Expressive** | 500ms | `.bouncy (response:0.5, damp:0.6)` | Playful wins (confetti, flame lick) |
| **Ambient** | 2-4s `ease-in-out` infinite | — | Breathing ring, pulse — only one ambient at a time |

**Mobile budgets:** interactions that should feel instant stay **<300ms**. Sheets/expands use spring, not linear. Every duration token has a `@media (prefers-reduced-motion)` counterpart (fade or none).

---

## Spring Catalog (SwiftUI → Reanimated → CSS mapping)

iOS 26 Liquid Glass uses springs exclusively for interactive motion:

```swift
// Default (recommended for most transitions)
.spring(response: 0.35, dampingFraction: 0.85)
// Bouncy (playful: puzzle win, add-to-cart bounce)
.spring(response: 0.4, dampingFraction: 0.6)
// Snappy (quick precise: tab select, chip)
.spring(response: 0.25, dampingFraction: 0.9)
// Interactive (gesture-driven: sheet drag, scrub)
.interactiveSpring(response: 0.15, dampingFraction: 0.86)
```

Mapping:

| Platform | Spring Equivalent |
|---|---|
| SwiftUI | `.spring(response:, dampingFraction:)` / `.interactiveSpring` |
| Reanimated (RN) | `withSpring({ damping: 18, stiffness: 220 })` snappy / `{ damping: 14, stiffness: 160 }` bouncy |
| CSS | `cubic-bezier(0.34,1.56,0.64,1)` (expressive) + `cubic-bezier(0,0,0.2,1)` (out) fallback |
| Compose | `spring(dampingRatio = 0.8f, stiffness = 320f)` snappy / `0.6f / 200f` bouncy |

**Rule**: Same spring params for same interaction type across the app. Define 2 presets (snappy + expressive) and reuse. Consistency compounds.

---

## Haptic Map (pair EVERY key interaction)

| Interaction | Haptic | iOS API | Android | RN (expo-haptics) | Web fallback |
|---|---|---|---|---|---|
| Tap / chip select | Light impact | `UIImpactFeedbackStyleLight` | `CLOCK_TICK` | `Haptics.selectionAsync()` | `navigator.vibrate(20)` |
| Confirm / add-to-cart | Medium impact | `Medium` | `KEYBOARD_TAP` | `Haptics.impactAsync(Medium)` | `vibrate(40)` |
| Destructive / delete commit | Heavy impact | `Heavy` | `LONG_PRESS` / `HEAVY_CLICK` | `Haptics.impactAsync(Heavy)` | `vibrate([40,30,40])` |
| Success (paid, completed) | Success notification | `UINotificationSuccess` | `CONFIRM` | `Haptics.notificationAsync(Success)` | — |
| Error / validation fail | Error notification | `UINotificationError` | `REJECT` / `VIRTUAL_KEY` | `notificationAsync(Error)` | — |
| Warning | Warning | `UINotificationWarning` | `VIRTUAL_KEY` | `notificationAsync(Warning)` | — |
| Swipe threshold reached | Medium tick | `Medium` at commit | `CONTEXT_CLICK` | `impactAsync(Medium)` | `vibrate(20)` |
| Pull-to-refresh trigger | Medium tick | `Medium` | `GESTURE_THRESHOLD_ACTIVATE` | `impactAsync(Medium)` | — |
| Stepper +/- tick | Light per increment | `Light` | `CLOCK_TICK` | `selectionAsync()` | — |
| Breathing pulse | Soft continuous | `soft` (watchOS) / `Light` repeated | — | `selectionAsync()` on inhale | — |
| Alignment (drag to grid) | Alignment | `UISelectionFeedback` | `CLOCK_TICK` | `selectionAsync()` | — |

**Semantics matter**: Impact = physical metaphor (snap, collision, detent). Notification = outcome (success/error). Selection = value change. Never repurpose — e.g., don't use `Success` for a plain toggle.

---

## Motion Patterns by Element

### Screen Transitions

| Transition | Motion | Haptic | Reduced-Motion |
|---|---|---|---|
| Stack push | Slide next from trailing 16px + fade 280-320ms `ease-out` | — | Fade only |
| Stack pop (edge swipe) | Interactive spring follows finger; velocity decides commit | `light` on commit | Instant |
| Tab switch | Crossfade 200ms + indicator pill spring | `selection` | Crossfade 150ms |
| Sheet present | Spring from bottom 350ms bouncy | `light` | Fade + slide 200ms |
| Sheet drag dismiss | Follows finger, snap threshold 30-50% of height | `medium` at commit | Fade out 150ms |

### Shared Element (card → detail)

```javascript
// FLIP technique: animate from tapped frame bounds to detail
card.addEventListener('click', (e) => {
  const rect = e.currentTarget.getBoundingClientRect();
  // Reanimated / Motion / SwiftUI matchedGeometryEffect — same intent
});
```
Timing: 260ms spring, backdrop blur fades to 60% scrim. Interruptible — back swipe cancels.

### List Entrance (first load)

```javascript
gsap.from('.list-row', {
  y: 12, opacity: 0,
  duration: 0.35, stagger: 0.06, ease: 'power2.out',
  // MAX 6 items staggered; rest without stagger to avoid endless cascade
});
```
Reduced-motion: no `y`, fade only 150ms.

### Micro-Interactions

| Element | Pressed | Success | Error |
|---|---|---|---|
| Primary button | `scale:0.97` 80ms + light haptic | Check draw + success haptic | Shake 4px + error haptic |
| Toggle | Knob spring 200ms | — | — |
| Chip | Scale 0.98 + tonal fill | — | — |
| Swipe action | Icon morphs as threshold → commit | Medium haptic at threshold | — |

### Scroll Effects

| Effect | Implementation | When |
|---|---|---|
| Reveal fade-up | `IntersectionObserver` adds `.in-view`; `y:12→0`, `opacity 0→1`, 300ms | Cards, section headers |
| Parallax (≤2 layers) | Content under glass refracts 4-6% slower than chrome | Hero image behind glass bar only |
| Pin (avoid on mobile) | Horizontal scroll pinned section is desktop pattern — don't use on phones | Desktop preview of app only |

---

## Liquid Glass Motion (iOS 26)

Rules distilled from Apple WWDC 2025:

1. **Navigation layer only** — tab bar, top bar, toolbar, sheet, FAB. Never on cards/rows/list backgrounds.
2. **Never glass on glass** — stacking recalculates blur per frame and kills perf (especially in scrolling lists).
3. **Group via Container** — multiple glass elements must share a `GlassEffectContainer` for single-pass sampling and morphing.
4. **Materialize, don't fade** — glass appears by increasing light-bending, not opacity fade. In SwiftUI: `transition(.scale(0.9).combined(with:.opacity))` + spring.
5. **Morph with IDs** — buttons expanding into menus morph via `glassEffectID` inside shared container + `withAnimation(.bouncy)`.

SwiftUI quick ref:

```swift
// Basic glass
.glassEffect() 
.glassEffect(.regular) 
.glassEffect(.clear) 
.glassEffect(.regular.tint(.blue).interactive()) // iOS interactive = touch shimmer

// Grouped (required for multiple)
GlassEffectContainer(spacing: 20) {
  HStack {
    Button("A") {}.glassEffect()
    Button("B") {}.glassEffect()
  }
}

// Morphing
@Namespace var ns
.glassEffectID("camera", in: ns)

// Performance: switch to opaque during fast position animation
```

**Blur budget**: >2 simultaneous glass surfaces on-screen degrades mid-range Android / older iPhones. In scrolling lists, row backgrounds must be opaque — glass in a `List`/`CollectionView` cell forces blur redraw every frame.

---

## Reanimated / Gesture Recipes (React Native)

### Bottom Sheet with Detents

```javascript
import BottomSheet, { BottomSheetView } from '@gorhom/bottom-sheet';
import { GestureHandlerRootView } from 'react-native-gesture-handler';

<BottomSheet
  snapPoints={['50%', '92%']}
  handleIndicatorStyle={{ backgroundColor: tokens.border, width: 36, height: 5 }}
  backgroundStyle={{ backgroundColor: tokens.surface, borderRadius: tokens.radiusSheet }}
  enablePanDownToClose
>
  <BottomSheetView>{/* sheet body */}</BottomSheetView>
</BottomSheet>
```

### Swipe Action (threshold + haptic)

```javascript
import { Swipeable } from 'react-native-gesture-handler';
import * as Haptics from 'expo-haptics';

const renderRightActions = () => <View style={{ width: 80, backgroundColor: tokens.error }}><Text>Delete</Text></View>;

<Swipeable
  renderRightActions={renderRightActions}
  onSwipeableWillOpen={() => Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium)}
  friction={2}
  rightThreshold={40}
>
  <ListRow />
</Swipeable>
```

### Press Spring (pull-to-refresh style)

```javascript
import Animated, { useSharedValue, withSpring, useAnimatedStyle } from 'react-native-reanimated';

const scale = useSharedValue(1);
const style = useAnimatedStyle(() => ({ transform: [{ scale: scale.value }] }));
// onPressIn: scale.value = withSpring(0.97, { damping: 18, stiffness: 320 })
// onPressOut: scale.value = withSpring(1, { damping: 18, stiffness: 320 })
```

---

## CSS-Only Recipes (Mobile Web / PWA)

### Sticky Bottom CTA + Tab Bar

```css
.cta-bar { position: sticky; bottom: 0; padding-bottom: calc(12px + env(safe-area-inset-bottom)); }
.tab-bar { position: fixed; bottom: calc(12px + env(safe-area-inset-bottom)); /* floating */ }
@media (min-width: 768px) {
  .tab-bar { position: fixed; left: 0; top: 0; width: 80px; height: 100dvh; bottom: auto; border-radius: 0; }
}
```

### Skeleton Shimmer (respects reduced-motion)

```css
.skeleton {
  background: linear-gradient(90deg, var(--surface-raised) 25%, var(--border) 50%, var(--surface-raised) 75%);
  background-size: 200% 100%;
  animation: shimmer 1.2s ease-in-out infinite;
}
@keyframes shimmer { to { background-position: -200% 0; } }
@media (prefers-reduced-motion: reduce) {
  .skeleton { animation: none; background: var(--surface-raised); }
}
```

### Empty Transition (exit faster than entry)

```css
.empty { animation: emptyIn 220ms var(--ease-out) both; }
.empty--exit { animation: emptyOut 130ms var(--ease-out) both; } /* 0.6× entry */
@keyframes emptyIn { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: none; } }
@keyframes emptyOut { from { opacity: 1; } to { opacity: 0; transform: translateY(-4px); } }
```

---

## Accessibility Notes for Motion

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
  /* Keep essential state transitions as instant crossfade — don't remove information */
  .skeleton { animation: none !important; }
}
```

Additional:
- Detect `accessibilityReduceMotion` (SwiftUI `@Environment`) / `UIAccessibility.isReduceMotionEnabled` and swap morph → crossfade.
- Detect `Reduce Transparency` → replace glass blur with opaque `var(--surface)` + border.
- Detect `Increase Contrast` → stronger borders, higher opacity.
- Never use motion as sole communicator — pair with haptic + copy ("Payment succeeded" toast, not just check animation).
- Auto-playing motion must have visible pause/stop.
- Preserve `Dynamic Type` — test morph/sheet at largest size; glass containers must not clip enlarged text.

---

## GSAP ScrollTrigger (if used on marketing preview of app)

```javascript
// Reveal section (mobile: simpler, no pin)
gsap.from('.reveal-section', {
  scrollTrigger: { trigger: '.section', start: 'top 88%' },
  y: 12, opacity: 0, duration: 0.35, ease: 'power2.out'
});
// Stagger first 6 cards only
gsap.from('.card', {
  scrollTrigger: { trigger: '.grid', start: 'top 88%' },
  y: 12, opacity: 0, duration: 0.35, stagger: 0.06, ease: 'power2.out',
  // optionally: batch via ScrollTrigger.batch for perf
});
```

Avoid on mobile: pinned horizontal scroll, parallax >2 layers, `pin:true` (desktop pattern — jank on mobile scroll).

