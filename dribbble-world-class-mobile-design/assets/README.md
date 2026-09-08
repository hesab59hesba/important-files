# Assets — Dribbble World-Class Mobile Design

This folder is for binary and generated assets that trials and demos produce. It is empty by design (binary assets are ignored by the skill loader), but this README documents the recommended mobile asset conventions so generations stay consistent.

## Device Frames (use for presentation)

- **iPhone reference**: 390×844 pt (iPhone 15/16 Pro, 3×). Frame the HTML/CSS output in a 390pt wide container with `border-radius: 48px` outer, `Dynamic Island` notch overlay, and `home indicator` pill.
- **Android reference**: 412×915 dp (Pixel 8). Rounded 20px, no notch.
- **Thumb-zone overlay**: Figma plugins "Thumb Zone" or "Touch Heatmap" — export as 40% opacity PNG at 390×844 and place over screenshot to audit reach.
- Preferred export: embed device frame via CSS (`device-frame--iphone`), not raster image, so text remains selectable.

## App Icon Templates

- **iOS**: 1024×1024 master, no pre-rounding (system masks). Provide layered variant (foreground + background) for parallax. Test at 20×20 (notification) — must remain legible.
- **Android adaptive**: 108×108dp canvas, 72dp safe zone. Provide `ic_launcher.xml` structure (`foreground` + `background` + `monochrome` for themed icons on Android 13+).
- **Liquid Glass icon**: layered glass with specular highlight. Provide `clear` variant (translucent, for light wallpapers) and respect `Increase Contrast` path (flatter).

## Splash / Launch

- **iOS**: `LaunchScreen.storyboard` pattern — centered wordmark + background color token, no spinner. Transition to skeleton, not blank.
- **Android 12+**: `SplashScreen` API with `windowSplashScreenBackground` = `--surface`, `windowSplashScreenAnimatedIcon`, exit via `SplashScreenViewProvider`.
- Keep splash <300ms perception — never park a loader behind logo.

## Thumb-Zone & Safe-Area Overlays

- Figma: create 390×844 frame, paint three zones: bottom third green (easy), middle yellow (stretch), top red (hard). Use as audit layer before handoff.
- Code: `env(safe-area-inset-*)` overlays for iPhone notch/Dynamic Island + Android cutout. Never hard-code 44px status padding.

## Icon Sources

- SF Symbols 7 (Apple, 6,900+, 9 weights) — iOS primary
- Material Symbols (Google, variable weight/fill/opsz) — Android primary
- For cross-platform shared visuals: Phosphor (fill + regular) or Lucide — pick one family and stay there.

## Empty-State Preview Assets

- Ghost row previews: generate from real list-row/card markup at `opacity:0.42` + `aria-hidden`, not from image. Keeps preview in sync with populated UI.
- If illustration needed (onboarding only): keep under 30-40% viewport height, single-tone line art matching palette, not full-color hero.

## Figma/Generate Hints

- Build tokens as Figma Variables collections (`Primitives`, `Semantic Light`, `Semantic Dark`) with modes for Light/Dark → export via Tokens Studio → `tokens.json` (W3C DTCG 2025.10: `$value`/`$type`) → Style Dictionary → CSS vars / Swift / XML.
- Mark all focus rings, empty variants, and sheet detents as Figma component variants so they are not forgotten.

> Tip: When you generate a mobile app, save its screenshot + thumb-overlay composite here as `vault-thumb-audit.png` for the `DESIGN_NOTES.md` trail. That note compounds — you build a taste + ergonomics profile per user across sessions.
