# Mobile Anti-Slop Checklist

Definitive list of patterns that make an *app* look AI-generated. Flag before presenting. If any hit on **U1-U5**, revise immediately — these are high-probability tells.

---

## Universal Tells (every app — mobile + web)

| # | Pattern | What It Looks Like | Why It Fails | Fix |
|---|---|---|---|---|
| **U1** | Generic purple→blue gradient on hero/CTA | `from-blue-600 to-indigo-700` wash behind everything | Appears on every AI app; 200-290° hue band = 94% of AI output | Pick brand-tied hue *outside* 200-290° or derive from category (fintech indigo-cyan, commerce near-black, wellness amber) |
| **U2** | Identical rounded cards everywhere | 6 cards, `rounded-2xl shadow-lg p-6` clone | Template, zero hierarchy | Vary: bento spans (full/half), insetGrouped lists instead of cards for lists, one featured card with elevation |
| **U3** | Emoji as icons | `🚀✨💡` in place of icon set | Unprofessional, no weight/scale control | SF Symbols 7 (iOS) or Material Symbols (Android); Lucide/Phosphor only as fallback with documented justification |
| **U4** | Uniform spacing everywhere | `gap-6` between every element, dead-center layout | No hierarchy, safe-average | 4/8 grid with relation-based rhythm: inside control 8px < inside card 16px < between sections 32px |
| **U5** | `lorem ipsum` or vaporware copy | "Seamlessly unlock your potential" | Instant AI tell | Real placeholders tied to domain (e.g., "House on 5th — 3 bed, 2 bath, $847k" not "Product 1") |
| **U6** | Everything animated | Every element fades + slides on entry | Distracting, burns battery on mid-tier Android | One orchestrated entrance (first 6 items, 60ms stagger) + micro-interactions only where they confirm state |
| **U7** | Pure #fff on pure #000 | `#ffffff` text on `#000000` bg | Harsh on OLED, unrealistic | Offset: `#08080c-#1a1a1a` dark, `#f5f5f5-#fafafa` light |
| **U8** | One sans everywhere, same weight | Inter everywhere, no display/body/mono split | Flat personality | Pair display (Satoshi/Outfit/Charter) + body (Inter/SF) + mono for data; max 2 families |
| **U9** | Em-dash everywhere (`—`) | Dramatic ` — ` between clauses | #1 LLM stylistic tell | Use comma, colon, or two sentences. Ban `—` in visible copy entirely. |
| **U10** | En-dashes as separators | `Home – Search – Profile` | Model misuse | Use plain hyphen for ranges only; nav uses spacing, not dashes |
| **U11** | Identical CTA everywhere | "Get Started" on every section | Conversion spam | One primary per screen, verb-specific: "Add book", "Connect account", "Place order" |
| **U12** | Cardocalypse | Cards inside cards inside cards | Noise, flatten hierarchy | Flatten: space → bg shift (3-5% lightness) → elevation; border only as last resort |
| **U13** | Unmeasured contrast | Thin gray on white passes "by eye" | Fails WCAG, fails outdoor sunlight | Measure APCA (Lc ≥75 body, ≥45 large) or WCAG AA 4.5:1; script before ship |

---

## Mobile-Specific Slop

| # | Pattern | Symptom | Fix |
|---|---|---|---|
| **M1** | Identical iOS & Android UI | Same tab visuals, same nav, same fonts on both | Share semantic tokens, adapt shell: glass tab bar vs M3 nav bar, SF vs Material symbols, sheet detents vs scrim |
| **M2** | Primary CTA at top | Checkout button top-right | Move to **sticky bottom bar** (thumb zone). 75% use one thumb; top CTA loses 18-22% add-to-cart. |
| **M3** | Tiny / crowded tap targets | 32px close icon, 4px gap | Enforce 44×44pt / 48×48dp min + 8pt gap. Audit with pointer-coarse overlay. |
| **M4** | Hamburger hiding primary nav | Top-left ☰ holds 6 sections | Bottom tabs 3-5 for primary. Hamburger only for secondary (Help, Legal) — hiding primary drops engagement 30-50%. |
| **M5** | Glass on everything | Every card is `backdrop-filter: blur` | Glass ONLY on functional layer (tab bar, top bar, sheet, FAB over maps). Cards/rows stay opaque — blur budget 2 layers max. |
| **M6** | Stacking glass on glass | Sheet over glass tab bar with glass cards inside | Never stack glass. Use `GlassEffectContainer` for multiple glass elements so they share sampling. |
| **M7** | Password field in 2026 | Password + strength meter + "Forgot?" | Passkey + biometric primary (Face ID / fingerprint) + magic link fallback. No CAPTCHA. |
| **M8** | Desktop collapse as mobile | 4-col grid → 1-col blind stack, CTAs still top | Mobile is its own hierarchy: re-flow columns, reposition CTAs bottom, simplify density. |
| **M9** | Horizontal overflow | Fixed width cards cause sideways scroll at 320px | Fluid widths, no `overflow:hidden` on scroll parents, test 320/375/430/768. |
| **M10** | CTA lost off-screen | Long form pushes "Place order" below fold, sticky absent | Sticky CTA lifts 5-12% checkout completion. Pin with `position: sticky` + `safe-area-inset`. |
| **M11** | Wrong keyboard / no autofill | Card field opens QWERTY, address requires manual typing | Correct `inputmode`/`type` + `autocomplete` vocab (`given-name`, `postal-code`, `cc-number`, `email`). Autofill cuts time 40-60%. |
| **M12** | Bottom tabs 5+ or 2 | 6 cramped tabs (each <78pt on 390pt bar) or 2 tabs as bar | 3-5 is sweet spot; 2 → toggle, 6+ → group into Profile→More. |
| **M13** | Nesting bottom sheets | Sheet opens sheet opens sheet | Forbidden. Push within same sheet or use modal. Max 2 detents per sheet. |
| **M14** | FAB covering content | Large FAB overlaps last list row + tab bar | M3: FAB trailing-bottom but integrated; on sheet/tablet move into toolbar. Test with keyboard open. |
| **M15** | Sticky chrome hiding focused input | Field focused but hidden behind tab bar / keyboard | Ensure focus never obscured (WCAG 2.4.11); use `visualViewport` resize; action stays reachable with keyboard. |
| **M16** | No focus / no keyboard path | Div-buttons, no `role`, no Enter/Space | Native `<button>`, `aria-label` on icon-only, focus ring, focus trap on sheets, logical focus order. |
| **M17** | No states designed | Only happy path; loading = frozen, empty = white void | Design every screen's matrix: default/loading/skeleton/empty/first-run/cleared/no-results/error/offline/permission/success. |
| **M18** | System back / edge-swipe broken | Left-edge drawer blocks iOS edge swipe, custom back ignores gesture | Reserve left edge: drawer only via button/handle, respect system back gesture; test both swipe + button. |
| **M19** | Overused Inter / Poppins defaults | Inter paragraph + Inter heading + Inter data | Leave 200-290° hue + Inter default — drop slop score 15-20pts. Pick brand-tied pairing (e.g., Charter+Inter for wellness). |
| **M20** | Generic empty with sad illustration | Large cloud icon + "No data yet" + tiny CTA below fold | Ghost preview (faded real UI) at 30-40% height + verb-first headline + full-width bottom CTA; CTA in thumb zone. |
| **M21** | Inverted dark mode | `filter: invert()` or naive negated colors | Dark is semantic override (elevation via border/glow, not shadow). Test #08080c OLED + Increase Contrast. |
| **M22** | Fake perfect data | "99.99% uptime", "$199" strike "$99" | Organic values: "47.2% faster", "(128 reviews)", honest pricing; label mocks explicitly. |
| **M23** | Neon on dark everywhere | Cyan+v Ord glow on every surface | Restrict neon to one accent surface; rest matte/solid. |
| **M24** | Multi-col form on mobile | Two fields side-by-side at 375px | Single-column below 768px; every field full-width. |

---

## Category-Specific Mobile Anti-Patterns

### Fintech / Trading
| Pattern | Fix |
|---|---|
| Rainbow chart (every bar different color) | One accent (gain) + muted context bars; loss = semantic error only |
| Amount in muted/small text | Amount 34px display, tabular mono, 4.5:1 contrast |
| No biometric gate | Gate on launch/resume with Face ID + fallback; badge sensitive amounts |

### Commerce / Marketplace
| Pattern | Fix |
|---|---|
| Price `$99` with fake strikethrough `$199` | Real pricing only |
| No filtering/sort on grid | Chips + bottom sheet filters; `Clear filters` preserves query |
| Tiny product thumbnails | 1:1 at 2 cols, 12px gap, peek carousel |
| Trust badges in footer | Inline between product list and sticky CTA, near payment field |

### Social / Feed
| Pattern | Fix |
|---|---|
| Centered hero badge above feed title | Integrate badge into header or remove |
| Inconsistent gesture (swipe left sometimes deletes, sometimes replies) | One gesture = one meaning; threshold at 30-50% + haptic commit; always offer long-press fallback |
| Banners as primary nav | Bottom tabs for primary; sheets for context |

### Health / Fitness
| Pattern | Fix |
|---|---|
| Neon intensity on health data | Desaturated semantic colors; test outdoor contrast |
| 48dp log targets missing | 48dp min + haptic tick; long-press accelerate |
| Permission before value | Show value, then contextual priming sheet before system dialog |

### AI Apps
| Pattern | Fix |
|---|---|
| AI claim with stock photo person | Real citations, sample Q&A, ghost result preview |
| Shuffling layout for "personalization" | Reorder only if distinct use modes; don't shuffle for novelty |
| Privacy buried in settings | Inline green/red dots per capability, like Signal 2026 |

---

## Red Flags Checklist (ship blocker)

Answer honestly before handoff:

- [ ] Could this app swap logos with a competitor and nobody notices? → Not specific enough — tighten palette + signature element.
- [ ] Would a designer at Linear/Revolut/Strava say "AI made this"? → Fix U1-U5 first.
- [ ] Do cards/rows/sheets all share identical radius/shadow? → Vary or flatten.
- [ ] Does the layout die at 320px or 135% type? → Reflow, don't just stack.
- [ ] Are all caps labels + wide tracking + em-dashes present? → Remove — LLM tells.
- [ ] Is dark just inverted light? → Rewrite dark as semantic token set.
- [ ] Does every screen have a glass layer? → Strip to functional layer only.
- [ ] Are empty states blank or "No data yet"? → Ghost preview + one CTA + instrument `view→click→time-to-first-action`.
- [ ] Do gestures lack visible fallbacks? → Add long-press / button alternative.
- [ ] Is motion generic `ease-in-out` with linear 0.1s stagger everywhere? → Replace with spring (see motion-system.md) + 40-60ms stagger for first 6 items only.

## Self-Correction Process (run every output)

1. Scan U1-U5 — if any hit, stop and revise (highest AI detection).
2. Scan M1-M10 — thumb / glass / tabs failures are mobile credibility killers.
3. Scan category block for your app type.
4. Ask: *"Would a studio that ships one app per quarter produce this?"* If no, iterate.
5. Measure: run `DESIGN.md` token contrast (APCA/WCAG), tab-width math (390×49pt bar ÷ tabs), touch target audit, 320/375/430/768 overflow check.
6. Re-read every visible string — ban list: *Seamlessly, Effortlessly, Unlock the power of, Built for the modern web, Streamline your workflow, Next-Gen, Game-changer, Elevate, Unleash, Acme, Nexus, SmartFlow, "—"*. Rewrite with domain-specific verbs.

> Goal is not to avoid every pattern — bottom tabs, cards, gradients can be right when driven by brief. The tell is the **undifferentiated default** (no committed hue, no thumb decision, no variant empty, no spring). A gradient hero is fine when it is the brand color. A card grid is fine when the content is comparable. Ship a decision, not an average.
