# SD Autolayout V1 — Training Learnings

Compiled from Figma DS source components (PA-March-Sprint, Autolayout Training page).
These learnings will be consolidated into SKILL.md once training is complete.

---

## Source 1: Input-Field Component Set (node 9265:392)
**18 variants:** 3 types (Normal, OTP, Chat Bubble) × multiple states
**Figma file:** PA-March-Sprint (`l0ICL2mGaTBrHOaUxGpiXr`)

### Layout Architecture

**L1. Component root is a wrapper, not a container.**
Root padding = [0,0,0,0]. The component has zero padding. Parent screen/form handles positioning. Components never assume their own margins — parent's padding and gap handle placement.

**L2. Visibility toggle drives height, not layout reconfiguration.**
Height changes (80→108) come from hidden child becoming visible inside HUG container. Gap (8) is pre-set — no visual effect when child hidden. Never change gap, padding, or direction across states. Toggle visibility only.

**L3. Three-child pattern: label + control + helper.**
Canonical input: `title (optional) → field (always) → subtext (optional)`. All three always present in tree. Visibility controls which appear. Build all children, toggle visibility — don't add/remove children across states.

### Sizing Modes

**L4. OTP boxes use FILL, not FIXED.**
Despite rendering at 76px, OTP boxes are FILL-sized within HORIZONTAL parent. 4× FILL = equal distribution. When multiple siblings need equal width, use FILL on each — not FIXED pixel values.

**L5. Chat Bubble field uses HUG height; Normal uses FIXED 48.**
Multi-line possibility → HUG. Always single-line → FIXED 48. Height mode determined by whether content can wrap.

**L6. Text inside field: HUG×HUG; parent frame: FILL×HUG.**
Text node sizes to content. Frame expands to fill remaining space after icons. Text nodes inside horizontal rows HUG their content; wrapping frame FILLs remaining space.

### Spacing

**L7. Uniform gap within a component.**
Root uses single `itemSpacing: 8` for all children (label→field→subtext). Within a single component, use one uniform gap — don't vary per child pair.

**L8. Field internal padding [12,16,12,16] — universal.**
Normal field, OTP box, Chat Bubble — all use identical inner padding. Component family shares one padding spec. Variants change fills/strokes, never padding.

**L9. Gap differs between types (not states).**
Normal/Chat gap: 12 (icon-to-content). OTP gap: 8 (box-to-box). Different because children have different internal structure. Gap can differ between component types, never between states of same type.

**L10. Title frame gap:10 — off-grid violation.**
10 ∉ {4,8,12,16,24...}. Exists in DS source. Flag off-grid values even in source components. Correct value: 8 or 12.

### Visual Hierarchy

**L11. Font weight inversion: title Bold↔Regular when field gets content.**
Empty: title Bold (carries weight). Filled: content Bold, title Regular (recedes). In label+control pairs, only one element Bold at a time — the element with user attention gets the weight.

**L12. Active stroke is neutral/800 (#352D42), not brand/600.**
Focus = dark neutral, not brand pink. Brand/600 reserved for CTAs and active indicators. Input focus = neutral/800. Brand color on inputs only for OTP cursor (#D9008D).

**L13. Pre-Filled and Disabled share fill (#D7D3E0) but differ in text.**
Disabled: hint #A7A1B2, Regular. Pre-Filled: secondary #665E75, Bold. Background alone doesn't communicate state — combination of bg + text color + weight distinguishes states.

### Icon Slot Pattern

**L14. Icon slots always present, visibility-toggled.**
Leading icon, trailing icon, info icon — all 24×24 FIXED instances in every variant. FILL text frame absorbs space when icons hidden. Build all slots upfront, toggle visibility.

**L15. Maximal state determines child count.**
Chat Bubble has 4 icon slots (user_name, text, edit, tick). All hidden by default. Every other state is a visibility subset of the maximum.

### Component Consistency

**L16. Sizing modes identical across all states of same type.**
States change visual properties (color, weight, visibility). States NEVER change layout properties (direction, sizing, padding, gap).

**L17. Hidden children must have correct sizing modes.**
Non-error subtext currently FIXED×FIXED (since hidden). Should be FILL×HUG like error subtext. Even hidden children need correct sizing — they may become visible.

**L18. Use MIN on VERTICAL HUG containers, not CENTER.**
CENTER on HUG has no effect (tight wrap, no distributable space). Semantically misleading. Use MIN (top-align). CENTER only when FILL height provides distributable space.

### Figma Issues Found
| # | Issue | Current | Correct |
|---|-------|---------|---------|
| 1 | title gap | 10 | 8 or 12 |
| 2 | subtext hSizing (non-error) | FIXED | FILL |
| 3 | Chat Bubble title hSizing | FIXED | FILL |
| 4 | root primaryAlign | CENTER | MIN |

---

## Source 2: Dialog + Bottom Sheet Header (nodes 9265:600, 9265:629)
**Figma file:** PA-March-Sprint (`l0ICL2mGaTBrHOaUxGpiXr`)

### Dialog (3 variants: Icon=No, Icon=48x48, Icon=96x96)

**Anatomy:**
```
Dialog (312×HUG, VERTICAL, gap:48, p:[32,24,32,24], r:24, fill:#FAF9FC)
├── Head (FILL×HUG, VERTICAL, gap:24, p:0)
│   ├── [icon] — 48×48 or 96×96 FIXED (optional, by variant)
│   ├── text (FILL×HUG, VERTICAL, gap:12)
│   │   ├── Title — fs:24/Bold, FILL×HUG, #161021
│   │   └── description — fs:16/Regular, FILL×HUG, #161021
│   └── [Input-Field] — FIXED 264×HUG (optional, in icon variants)
└── Dialog Action bar (HUG×HUG, VERTICAL, gap:12)
    ├── Primary CTA — FIXED 264×48, r:16, #D9008D
    └── Secondary CTA — FIXED 264×48, r:16, #FFE5F6
```

**L19. Dialog content-to-CTA gap = 48.**
Largest gap in any component. Creates clear visual break between content and actions. Unique to dialogs — no other component uses 48 internally.

**L20. Dialog padding is [32,24,32,24], not [24,24,24,24].**
Vertical > horizontal. Creates more vertical breathing room. The skill §3a needs correction.

**L21. Dialog width is 312 at 360 base (= 360 - 2×24 margin).**
NOT a fixed pixel value — it's derived from screen width minus overlay margin. Content area: 312 - 2×24 padding = 264.

**L22. Nested gap hierarchy: Head gap:24, text gap:12.**
Outer container uses larger gap, inner container smaller. Mirrors the fundamental rule (internal ≤ external) applied to gap, not just padding.

**L23. Dialog icon is LEFT-aligned (crossAlign: MIN), not centered.**

**L24. Dialog CTAs are FIXED 264 width, not FILL.**
Action bar hSizing: HUG. CTAs match content area width via FIXED, not via FILL. Structurally different from screen-level CTAs (which use FILL).

**L25. Instances inside dialogs use FIXED width matching content area.**
Input-Field inside dialog: FIXED 264, not FILL. Components embedded in dialogs get explicit width matching the dialog's content area.

**L26. Dialog variant axis = icon size, not state.**
Layout is invariant across all use cases. Only content (icon, text) changes.

### Bottom Sheet Header (1 variant)

**Anatomy:**
```
bottom_sheet_header (360×88, VERTICAL, gap:0, crossAlign:CENTER)
├── top_slider (FILL×HUG, VERTICAL, p:[12,120,24,120], crossAlign:CENTER)
│   └── Slider (RECTANGLE, FILL×4, r:4, #D7D3E0)
└── heading (FILL×HUG, VERTICAL, gap:8, p:[8,16,8,16])
    ├── Title — fs:24/Bold, FILL×HUG, #161021 (visible)
    └── subtitle — fs:16/Regular, FIXED×FIXED, #161021 (hidden)
```

**L27. Drag handle centering via padding, not alignment.**
top_slider padding: [12,120,24,120]. 120px left/right on 360 frame = centered 120px zone. Handle is FILL inside that zone. Centering through padding on parent, not crossAlign on child.

**L28. Bottom sheet header uses padding stacking, not gap.**
gap:0 between zones. Visual separation comes from: top_slider bottom padding (24) + heading top padding (8) = 32px visual gap. Each zone owns its spacing independently.

**L29. BSH width is 360 (DS base frame width).**
Full-width component. On real devices: fillMaxWidth().

**L30. Hidden subtitle has FIXED×FIXED — systemic bug.**
Same pattern as Input-Field subtext (L17). All hidden text children in DS source have incorrect sizing. Should be FILL×HUG.

**L31. Drag handle asymmetric vertical padding: top 12, bottom 24.**
More space below than above. Pushes handle visually upward — signals "top edge" affordance.

### Figma Issues Found
| # | Issue | Component | Current | Correct |
|---|-------|-----------|---------|---------|
| 1 | Dialog padding in skill | Skill §3a/§5 | [24,24,24,24] / width 358 | [32,24,32,24] / width derived |
| 2 | Hidden subtitle sizing | BSH subtitle | FIXED×FIXED | FILL×HUG |
| 3 | Dialog Action bar hSizing | Action bar | HUG | Should be FILL (CTAs should use FILL not FIXED 264) |

---

## CRITICAL CORRECTION: Width Is Never a Constant (L32–L34)

**Context:** DS uses 360×720 as reference frame. Real Android devices: 360dp, 390dp, 412dp, 428dp+ wide.

**L32. Figma DS frame width (360) is a reference viewport, not a spec.**
Every width value in the DS is `reference_viewport - padding_chain`. In Compose: always FILL + parent padding. Never hardcode dp widths for containers, cards, inputs, CTAs, or dialogs.

**L33. The padding chain determines width.**
```
Screen (360/390/412dp — varies by device)
  └── padding: 16dp each side
      └── Card: FILL (renders as 328/358/380dp)
          └── padding: 16dp each side
              └── Content: FILL (renders as 296/326/348dp)
```
Width cascades inward through padding. Each level eats padding from both sides. The rendered pixel width is a CONSEQUENCE, not an input.

**L34. Only icons, avatars, and explicit graphic elements have absolute width.**
These dp values are device-independent and never change:
- 24dp icon
- 48dp icon circle
- 32dp avatar
- 64dp shutter button
- 120dp drag handle bar (inside padded parent)
- 4dp drag handle height
- 2dp step progress height

Everything else (containers, cards, inputs, CTAs, dialogs, sheets): FILL parent width.

### What changes in the skill

**§5 Fixed Dimensions — width column must be revised:**
| Element | Current (wrong) | Correct |
|---|---|---|
| Screen frame | 390 FIXED | device_width (FILL viewport) |
| Dialog | 358 FIXED | FILL with 24dp margin each side |
| Input field | 328 FIXED | FILL parent |
| CTA | FILL | FILL parent (correct, no change) |
| Card | FILL | FILL parent (correct, no change) |
| Bottom sheet | 390 | FILL viewport |
| App bar | 390 | FILL viewport |

**§1a Width Mode Decision Tree — needs new root rule:**
```
Is it an icon, avatar, or explicit graphic?  → FIXED (dp value from DS)
Is it everything else?                       → FILL parent
```

Width is NEVER specified on containers. Width is a CONSEQUENCE of parent padding.

**Figma vs Compose translation:**
| Figma (360 base) | What it means | Compose |
|---|---|---|
| Component width 328 | 360 - 2×16 padding | `fillMaxWidth()` inside `padding(horizontal = 16.dp)` |
| Component width 312 | 360 - 2×24 margin | `fillMaxWidth()` inside `padding(horizontal = 24.dp)` |
| Component width 264 | 312 - 2×24 padding | `fillMaxWidth()` inside padded dialog |
| Component width 360 | Full viewport | `fillMaxWidth()` |

---

## Source 3: Mixed Molecules (node 9265:1009)
**7 component sets:** Card (plan selection), faq-card (accordion), Keyboard (skip), tooltip, Vertical Container, offer card, Apply Coupon

### Card — Plan Selection (16 variants: 4 plans × 2 selection states × 2 languages)

**Anatomy:**
```
Card (FILL×HUG, HORIZONTAL, gap:16, p:16, r:16, stroke:#E8E4F0)
├── radio-button (INSTANCE, 24×24 FIXED)
├── details (FILL×HUG, VERTICAL, gap:8)
│   ├── plan_info (FILL×FIXED 24, HORIZONTAL, gap:16)
│   └── Discount (HUG×HUG, HORIZONTAL, r:4, fill:#E1FAED) — hidden on 28-day
└── Button Container (96×28 FIXED) — only on 28-day "most popular"
```

**L35. Radio selection card = HORIZONTAL root with radio left, content center, action right.**
Same pattern as list-row: `[FIXED control] [FILL content] [FIXED action]`. Gap:16. Padding:16 uniform. This is the standard selectable-row layout.

**L36. Selected state changes fill+stroke, not layout.**
Unselected: fill #FAF9FC, stroke #E8E4F0. Selected: fill **#FFE5F6**, stroke **#D9008D**. Layout (gap, padding, sizing, direction) is identical. Reinforces L16: states never change layout.

**L37. Plan=28 days has 3 children; 84/112/1Year have 2.**
Button Container (popularity badge) only exists on 28-day. The other plans have 2 children. This violates L15 (maximal child count across all variants) — ideally all variants should have 3 children with Button Container hidden on non-28-day plans.

**L38. crossAlign changes: Selected=No uses CENTER, Selected=Yes uses MIN.**
28-day selected variant switches from CENTER to MIN cross-alignment. This contradicts L16 (layout doesn't change across states). Flag as inconsistency — should be CENTER in both.

**L39. Details frame uses FIXED sizing instead of FILL×HUG.**
`details` frame: hSizing FIXED, vSizing FIXED. Should be FILL×HUG since it contains text that may vary in length. Systemic issue: the text inside (plan_info) correctly uses FILL, but its parent wrapper uses FIXED.

### faq-card — Accordion (4 variants: collapsed/expanded × with/without image)

**Anatomy:**
```
faq-card (FILL×HUG, VERTICAL, gap:0, r:16, fill:#FAF9FC, stroke:#E8E4F0)
├── question (FILL×HUG, HORIZONTAL, gap:16, p:16, fill:#F1EDF7)
│   ├── title (TEXT, FILL×HUG, fs:16/Bold, #161021)
│   └── arrow-down (INSTANCE, 24×24 FIXED)
└── answer (FILL×HUG, VERTICAL, gap:16, p:[16,16,24,16]) — HIDDEN when collapsed
    ├── image (GROUP, FIXED 296×148) — optional
    └── description (TEXT, FILL×HUG, fs:14-16/Regular)
```

**L40. Accordion uses gap:0 at root — zones manage their own padding.**
Same pattern as bottom sheet header (L28). question zone has p:16 all, answer zone has p:[16,16,24,16]. The gap between question bottom (16) and answer top (16) = 32px visual separation. **Components with distinct visual zones (different backgrounds) use gap:0 + per-zone padding, not gap > 0.**

**L41. Question zone has a distinct background (#F1EDF7 = neutral/100).**
The question area is visually distinct from the answer. This is a two-tone card: question bg ≠ answer bg ≠ card bg. **Accordion pattern: header zone gets a tinted background. Content zone uses the card's default background.**

**L42. Answer padding asymmetric: bottom 24, others 16.**
Extra bottom padding in the answer zone. Creates visual termination — the content doesn't feel like it's cut off. **Last content zone in a card gets extra bottom padding (24 vs 16) for visual closure.**

**L43. Accordion collapsed→expanded = answer visibility toggle. Same as L2.**
Gap:0 means invisible answer costs zero vertical space. When answer becomes visible, card HUG height grows. No layout reconfiguration needed. Confirms the visibility-driven-height pattern.

**L44. Image in answer is FIXED 296×148 (GROUP), not FILL.**
Images inside content zones use FIXED dimensions, not FILL. This means on wider devices, image won't stretch (which would distort aspect ratio). **Images: FIXED width×height to preserve aspect ratio. Don't use FILL width on images without explicit aspect ratio constraint.**

### tooltip (4 variants: Default/Slide-down × Hindi/English)

**Anatomy:**
```
tooltip (HUG×HUG, VERTICAL, gap:0)
├── bubble (HUG×8, arrow pointer triangle)
└── text (FIXED 296×HUG, HORIZONTAL, gap:12, p:[12,16,12,16], r:12, fill:#E4CCFF)
    ├── icon (24×24 FIXED)
    └── message (TEXT)
```

**L45. Tooltip root is HUG×HUG — the only component with HUG width.**
Every other component uses FIXED width (or FILL parent). Tooltip is the exception: it floats and sizes to its content. **Floating/overlay elements (tooltip, popover) use HUG×HUG because they're not in the layout flow.**

**L46. Tooltip text zone width is FIXED 296, not FILL or HUG.**
Despite the root being HUG, the text body is FIXED 296. This constrains the tooltip to a max width even though the root could grow. **Floating elements: root = HUG, but content = FIXED max width to prevent unreasonable expansion.**

### Vertical Container — Info Banner (2 variants: Hindi/English)

**Anatomy:**
```
Vertical Container (FILL×HUG, VERTICAL, gap:12, p:16, r:12, fill:#F1E5FF, stroke:#D6B2FF)
├── Text (FILL×HUG, fs:16/Bold, #161021) — headline
└── Vertical Container (inner) (FILL×HUG, HORIZONTAL, gap:12)
    └── [icon-text items]
```

**L47. Info banner uses colored background + matching stroke.**
Fill: #F1E5FF (light purple), Stroke: #D6B2FF (darker purple). Background and border are from the same color family. **Colored info banners: fill = light tint, stroke = darker shade of same hue.**

**L48. Standard card pattern: r:12, p:16, gap:12.**
This is the canonical small card: radius 12, padding 16 uniform, gap 12 between children. Matches the general card spec.

### offer card (4 variants: Save/Save More × Hindi/English)

**L49. Offer card uses absolute positioning (layoutMode: NONE).**
Root has no auto-layout — children (bg frame, icon, text) are absolutely positioned. This is an **exception** to the "never use NONE for content" rule. **Decorative/promotional cards with complex layered visuals (background image + overlaid content) may use absolute positioning when auto-layout can't achieve the visual.**

**L50. Two size variants: Save (96h) and Save More (72h).**
Both use FIXED height. Content doesn't drive height — the visual design dictates it. **Promotional cards may use FIXED height because the visual composition is exact, not content-driven.**

### Apply Coupon (6 variants: No/Yes/Available × Hindi/English)

**Anatomy (Applied=No):**
```
Apply Coupon (FILL×HUG, HORIZONTAL, gap:12, p:16, r:16, fill:#FAF9FC, stroke:#D7D3E0)
├── discount icon (24×24 FIXED)
├── title (TEXT, FILL×HUG, fs:16/Bold)
└── arrow-down (24×24 FIXED)
```

**Anatomy (Applied=Yes):**
```
Apply Coupon (FILL×HUG, HORIZONTAL, gap:12, p:16, r:16, fill:#E1FAED, stroke:#A5E5C6)
├── tick icon (24×24 FIXED)
├── text (FILL×HUG, VERTICAL, gap:4)
│   ├── title
│   └── subtitle
└── close icon (24×24 FIXED)
```

**Anatomy (Applied=Available):**
```
Apply Coupon (FILL×HUG, VERTICAL, gap:12, r:12, p:0)
├── coupon (FILL×HUG, HORIZONTAL, gap:12, p:16 all sides, r:16, fill:#FAF9FC, stroke:#D7D3E0)
│   └── [same as Applied=No layout but with 2-line content]
└── content (TEXT, FILL×HUG, "सारे कूपन देखें", fs:14, #665E75)
```

**L51. Apply Coupon changes DIRECTION between Applied=No/Yes (HORIZONTAL) and Available (VERTICAL).**
No/Yes: HORIZONTAL row (icon + text + action). Available: VERTICAL stack (coupon row + link text). This is a structural variant — the direction changes based on the variant, not state. **When a component adds a new zone (e.g., "view all" link), it can change to VERTICAL to stack the existing content with the new zone. This is acceptable at the component VARIANT level (not state level).**

**L52. Applied=Yes changes fill+stroke to success colors (#E1FAED + #A5E5C6).**
Same pattern as Card selection (L36): state changes visual treatment, not layout. The icon swaps (discount→tick), trailing icon swaps (arrow→close). **State feedback via color family: neutral→green for success, neutral→pink for selection.**

**L53. Available variant has p:0 root, p:16 inner coupon.**
Same wrapper-has-zero-padding pattern as L1. The Available variant wraps the coupon row + link text, with the coupon row owning its own padding. Root padding = 0 when child zones manage their own spacing.

### Sizing Issues Found (this frame)

| # | Issue | Component | Current | Correct |
|---|-------|-----------|---------|---------|
| 1 | details FIXED sizing | Card (all variants) | hSizing/vSizing FIXED | FILL×HUG |
| 2 | crossAlign inconsistency | Card 28-day Selected=Yes | MIN | CENTER (match unselected) |
| 3 | Missing Button Container | Card 84/112/1Year | 2 children | 3 (hidden badge) per L15 |
| 4 | image FIXED in answer | faq-card | GROUP 296×148 FIXED | Should be FILL×(aspect ratio) or intentionally FIXED |
| 5 | offer card absolute positioning | offer card | layoutMode NONE | Acceptable exception for decorative cards |

---

## Floor Values (proposed, to be refined during training)

### Minimum Heights
| Element | Min height | Rationale |
|---|---|---|
| Tappable element (CTA, row, toggle, checkbox) | 48dp | Material touch target |
| Input field | 48dp | DS constant |
| Single-line text | 20dp | fs:14 + line-height |
| Multi-line text (per line) | 24dp | fs:16 + line-height |
| Drag handle bar | 4dp | Below = invisible |
| Divider | 1dp | Below = sub-pixel issues |
| Step progress segment | 2dp | DS constant |
| Functional icon | 24dp | Below = unreadable |
| Icon circle | 48dp | DS constant + touch target |

### Minimum Paddings
| Context | Min | Rationale |
|---|---|---|
| Screen horizontal | 16dp | Content never touches screen edge |
| Card internal | 12dp | Text feels trapped below this |
| CTA horizontal | 16dp | Label needs breathing room |
| CTA vertical | 12dp | (48h - 24 text) / 2 |
| Input field | 12dp V, 16dp H | Text can't touch field border |
| Bottom sheet horizontal | 16dp | Content never touches sheet edge |
| Dialog (any side) | 24dp | Below = cramped |
| App bar (all sides) | 4dp | Tight but functional |

### Minimum Gaps
| Context | Min | Rationale |
|---|---|---|
| Horizontal siblings | 4dp | Below = merge visually |
| Vertical siblings | 8dp | Below = squished |
| Adjacent touch targets | 8dp | Prevents mis-taps |
| Label to control | 4dp | Connected but distinct |

### Minimum Widths
| Element | Min width | Rationale |
|---|---|---|
| Tappable element | 48dp | Touch target |
| CTA button | 120dp | Shortest label + padding |
| Icon | 24dp | DS constant |
| Drag handle | 40dp | Below = can't grab |

### Device Safety
| Rule | Why |
|---|---|
| No element wider than screen | Horizontal scroll = broken |
| ≥1 field + CTA visible above keyboard | User must see what they're typing |
| CTA visible during keyboard open | User must always be able to act |
| Bottom sheet max height = 85% viewport | Must show dimmed bg |
| Min visible above sheet = 15% viewport | User needs context |

---

## Source 4: Navigation & Header Components (node 9265:636)
**Components:** Status Bar (4), Top app bar (6), Top bar Homepage (4), Menu, Menu List Item (4)

### Status Bar (4 variants: Dark/Light × Alpha/Normal)

**Anatomy:**
```
Status Bar (360×24 FIXED, HORIZONTAL, gap:234, p:8, crossAlign:CENTER)
├── time text (HUG×HUG, fs:14/Medium)
└── Indicators (GROUP, FIXED 83×16)
```

**L54. Status bar height = 24, NOT 44 as previously in the skill.**
DS source component is 24px. The 44px in CLAUDE.md may have been from a different source or included the app bar gap. **Correction: Status bar = FIXED 24dp.**

**L55. Status bar uses gap:234 to push time left and indicators right.**
This is a hack — using a massive gap as a substitute for SPACE_BETWEEN. In Compose this should be `Arrangement.SpaceBetween`, not a spacer. **In Figma, large explicit gaps may substitute for space-between distribution. In Compose, always use `Arrangement.SpaceBetween`.**

### Top App Bar (6 variants: Normal/Big/Brand × Gradient Yes/No)

**Anatomy (Normal):**
```
Top app bar (360×80, VERTICAL, gap:0, p:0, fill:#FAF9FC)
├── Status Bar (INSTANCE, 360×24 FIXED)
└── Header (360×56, HORIZONTAL, gap:4, p:[4,4,4,4])
    ├── left (FILL×HUG, HORIZONTAL, gap:4)
    │   ├── icon frame (48×48 HUG, p:12 — wraps 24×24 back arrow)
    │   └── headline (TEXT, FILL×HUG, fs:16/Bold)
    └── trailing-icon (HUG×HUG, HORIZONTAL, gap:0)
        ├── icon-3 (48×48 FIXED, p:12, r:16)
        ├── icon-2 (48×48 FIXED, p:12)
        └── icon1 (48×48 FIXED, p:12)
```

**Anatomy (Big — extra title section):**
```
Top app bar (360×136, VERTICAL, gap:0, p:0)
├── Status Bar (24)
├── Header (56) — same as Normal
└── title (360×56, VERTICAL, gap:4, p:[8,16,16,16])
    ├── headline (TEXT, FILL×HUG, fs:24/Bold)
    └── subheading (TEXT, FIXED×FIXED, fs:16/Bold) — hidden
```

**L56. Top app bar = Status Bar + Header, gap:0, padding:0.**
Same zero-gap, zero-padding wrapper pattern (L1, L28, L40). Each zone manages its own spacing. Status Bar owns its p:8. Header owns its p:4.

**L57. Header = two sections: `left (FILL)` + `trailing-icon (HUG)`.**
Left section fills remaining space. Trailing icons hug their content. This means on wider devices, the left section (title) stretches, icons stay compact on the right. **App bar split: left=FILL (absorbs width), right=HUG (fixed content).**

**L58. Back arrow is 24×24 icon inside 48×48 touch target (p:12).**
Icon frame: 48×48, padding 12 all sides. 48 - 2×12 = 24 for the icon. **Touch target pattern: wrap icon in a frame with padding = (target_size - icon_size) / 2. For 24→48: padding = 12.**

**L59. Trailing icons are 48×48 FIXED with gap:0.**
Three icon slots, zero gap between them. Each is its own 48×48 touch target with p:12 internal padding. **Adjacent icon buttons in app bar: gap:0. Each icon frame includes its own padding, so visual spacing comes from the internal padding overlap.**

**L60. Big variant adds a title zone below Header.**
title frame: p:[8,16,16,16]. Asymmetric — top:8 (tight to Header since Header has bottom p:4, total visual gap = 4+8 = 12), sides:16 (screen padding). **Big app bar title padding top is minimal because it inherits visual gap from the Header zone's padding.**

**L61. Big variant subheading: hidden, FIXED×FIXED.**
Same systemic bug: hidden text uses FIXED sizing. Should be FILL×HUG. (Repeats L17, L30.)

**L62. Header padding is [4,4,4,4], gap:4.**
Extremely tight. 4px padding + 4px gap. This is the tightest layout in the DS. App bar is a dense zone. **App bar Header zone: smallest padding and gap in the entire DS (4px each). This is by design — maximizes space for title and action icons.**

### Top bar - Homepage (4 variants: Working/Not Working/Not Connected/Awaiting)

Same structural pattern as Top app bar Normal. VERTICAL, gap:0, p:0. Status Bar + Header. No layout differences between state variants — only content/colors change. Reinforces L16.

### Menu + Menu List Item

**Menu List Item anatomy:**
```
Menu List Item (200×57, VERTICAL, gap:0, p:0)
├── state-layer (FILL×40 FIXED, HORIZONTAL, gap:12, p:[8,12,8,12])
│   ├── Leading element (24×24 FIXED)
│   ├── Content (FILL×HUG)
│   └── Trailing element (24×24 FIXED)
└── Divider (FILL×HUG)
```

**L63. Menu List Item follows the [FIXED] [FILL] [FIXED] horizontal pattern.**
Same as list row, radio card, coupon. `Leading icon (24×24) + Content (FILL) + Trailing icon (24×24)`. Gap:12. This is THE canonical horizontal content row.

**L64. Menu List Item state-layer height = FIXED 40.**
Not 48 — this is a compact/dense menu item (-4 density in the name). Desktop/dropdown menus use smaller touch targets than primary mobile actions. **Dense menu items can go below 48dp (40dp here) when they're in overflow/dropdown menus, not primary touch targets.**

**L65. Menu List Item padding: [8,12,8,12].**
Asymmetric: vertical 8, horizontal 12. Compact but still above minimum floors (8v ≥ 8, 12h ≥ 4). **Dense components use the minimum floor values.**

**L66. Menu container: r:4.**
Radius 4 — smallest in the DS. Dropdown/overlay menus use minimal radius. **Floating menus/dropdowns: r:4 (sharp). Cards: r:12 (round). Dialogs/sheets: r:24 (very round). Radius increases with component prominence.**

### Issues Found
| # | Issue | Component | Current | Correct |
|---|-------|-----------|---------|---------|
| 1 | Status bar height in skill | Skill §5 | 44 | **24** (DS source) |
| 2 | Status bar gap hack | Status Bar | gap:234 | Should use SPACE_BETWEEN alignment |
| 3 | Big variant subheading | Top app bar Big | FIXED×FIXED (hidden) | FILL×HUG |
| 4 | Trailing icon gap:0 + inner gap:10 | trailing icon frames | gap:10 inside frames | 10 is off-grid |

### Correction to Skill §5
The skill currently says Status bar = 44px. The DS source component says 24px.
The "44px" was likely Status Bar (24) + some padding or the iOS-style safe area.
**In Compose: the system status bar height varies by device. The DS component (24px) is a Figma reference only. Use `WindowInsets.statusBars` in Compose, not a hardcoded value.**

---

## Emerging Patterns Summary (after 4 sources, 66 learnings)

### Pattern 1: The Universal Horizontal Row
`[FIXED 24 icon] [gap:8-12] [FILL text] [gap:8-12] [FIXED 24 action]`
Found in: list row, menu item, coupon, radio card, input field, app bar header.
Cross-axis: CENTER. This is the most common molecule in the DS.

### Pattern 2: Zero-Gap Zone Stacking
Root VERTICAL, gap:0, p:0. Each child zone manages its own padding.
Found in: app bar (status+header), accordion (question+answer), bottom sheet (handle+heading), input field wrapper.

### Pattern 3: Visibility-Driven Height
All optional sections present in tree, hidden by default. HUG root grows when sections become visible. States NEVER change layout properties.
Found in: input-field subtext, accordion answer, big app bar subheading.

### Pattern 4: Radius Hierarchy
`4 (menu/dropdown) < 12 (card/input/field) < 16 (CTA/selection card) < 24 (dialog/bottom sheet)`
Radius increases with component prominence and surface area.

---

## Source 5: CTA / Actions Component Set (node 9265:361)
**10 variants** across 2 axes: Primary CTA type × Sec CTA (Yes/No)

### CTA Size Taxonomy

| Variant | Width | Height | Radius | Context |
|---|---|---|---|---|
| Full CTA | 328 (= screen-32) | 48 | 16 | Screen-level primary action |
| Dialog-CTA | 264 (= dialog content) | 48 | 16 | Inside dialog |
| Half CTA | 156 | 48 | 16 | Side-by-side actions |
| Special Small CTA | 128 | 44 | **12** | Compact actions (dropdown trigger) |
| coupon-apply-CTA | 80 | 28 | 8 | Inline micro-action |
| icon-CTA | 24 | 24 | 0 | Icon-only tap target |

### Universal CTA Properties (shared across all variants)

| Property | Value | Notes |
|---|---|---|
| Direction | HORIZONTAL | Icon + text side by side |
| Padding vertical | 12 | All variants except coupon (4) |
| Padding horizontal | 16 | All variants except coupon (8) |
| Cross-axis align | CENTER | Text vertically centered |
| Primary-axis align | CENTER | Text horizontally centered |
| Primary fill | #D9008D (brand/600) | |
| Secondary fill | #FFE5F6 (light pink tonal) | Sec CTA=Yes |
| Primary text color | #FAF9FC (neutral/white) | |
| Secondary text color | #D9008D (brand/600) | On tonal bg |
| Text style | fs:16/Bold | Standard CTAs |
| Text style (small) | fs:14/SemiBold | Special Small + coupon |
| Height sizing | HUG | Not FIXED — padding + text determines height |

### Learnings

**L67. CTA height is HUG, not FIXED 48.**
Every CTA variant has vSizing: HUG. The 48px height is a CONSEQUENCE of: padding 12 top + 24px text (fs:16 line-height) + padding 12 bottom = 48. The 44px Small CTA: padding 12 + 20px text (fs:14) + padding 12 = 44. **Never set explicit height on CTAs. Set vertical padding + let text size determine height. This ensures CTAs scale correctly with font size changes.**

**L68. CTA width in Figma is FIXED per context, but in Compose should be FILL.**
Full CTA: 328 = screen width - 2×16. Dialog CTA: 264 = dialog content width. Half CTA: 156 = (screen-32-16gap)/2. In Compose: Full and Dialog CTAs = `fillMaxWidth()`. Half CTAs = `weight(1f)` in a Row. **CTA widths are derived from parent context, not absolute values.**

**L69. CTA text node is FILL×HUG, not HUG×HUG.**
Text inside CTA fills the available width. With primaryAlign:CENTER, the text centers within the FILL area. **CTA text uses FILL width so centering works via alignment, not by the text node hugging. This ensures consistent tap target width regardless of text length.**

**L70. Padding is universal: [12,16,12,16] across ALL CTA sizes.**
Full, Dialog, Half, Small — all share the same padding. The outer width changes, the inner padding stays constant. **CTA padding is context-independent. Only width and radius change between CTA types.**

**L71. Radius scales with CTA size: 16 (standard), 12 (small), 8 (micro).**
Full/Dialog/Half: r:16. Special Small: r:12. Coupon: r:8. Follows the pattern: smaller component = smaller radius.

**L72. Secondary CTA (Sec CTA=Yes) has identical layout to primary.**
Same padding, gap, sizing, direction. Only fill and text color change. Primary: fill #D9008D, text #FAF9FC. Secondary: fill #FFE5F6, text #D9008D. **Primary and secondary CTAs are structurally identical. The distinction is purely visual (fill + text color).**

**L73. Special Small CTA has an inner text frame with icon.**
Structure: CTA > text frame (FILL×HUG, HORIZONTAL, gap:4) > action text (HUG×HUG, fs:14/SemiBold) + expand_more icon (16×16 FIXED). **Small CTAs can embed an icon (dropdown chevron). The icon sits inside a nested HORIZONTAL frame with the text. Icon size: 16×16 (not 24).**

**L74. Coupon CTA is HUG×HUG — the smallest CTA type.**
80×28, hSizing: HUG. This is the only CTA that doesn't fill its parent. It's an inline action (like "Apply" inside a coupon card). Padding: [4,8,4,8] — tighter than standard. **Inline micro-CTAs: HUG width, reduced padding [4,8,4,8], r:8, fs:14/SemiBold.**

**L75. icon-CTA is 24×24, HUG×HUG, zero padding.**
Just wraps a 24×24 icon instance. No padding, no radius. The icon IS the button. **Icon-only CTAs: the icon itself is the touch target. In Compose, wrap with `Modifier.size(48.dp)` for touch target even though visual is 24×24.**

### CTA Decision Tree (for the skill)

```
Is it a screen-level primary/secondary action?
  → Full CTA: FILL width, p:[12,16,12,16], r:16, h:HUG(→48), fs:16/Bold

Is it inside a dialog?
  → Dialog CTA: FILL inside dialog content area, same padding/radius/text

Are two CTAs side by side? (rare — usually stacked)
  → Half CTA: weight(1f) each in Row, gap:16, same padding/radius/text

Is it a compact dropdown trigger or small action?
  → Special Small: HUG or FIXED width, p:[12,16,12,16], r:12, h:HUG(→44), fs:14/SemiBold

Is it an inline action inside a card (Apply, Remove)?
  → Coupon CTA: HUG×HUG, p:[4,8,4,8], r:8, fs:14/SemiBold

Is it an icon-only action?
  → icon CTA: 24×24, no padding, wrap in 48×48 touch target in Compose
```

### Issues Found
| # | Issue | Current | Correct |
|---|-------|---------|---------|
| 1 | Skill says CTA height FIXED 48 | §1b, §5 | **HUG** (48 is consequence of padding+text) |
| 2 | Coupon CTA text hSizing | FIXED | Should be HUG (inline CTA sizes to content) |

### Corrections to Skill

**§1b Height Decision Tree:**
```
CTA button (primary/secondary)?  → HUG (not FIXED 48)
                                    48 = 12 padding + 24 text + 12 padding
CTA button (small/checkbox)?     → HUG (not FIXED 40)
                                    44 = 12 padding + 20 text + 12 padding
```

**§5 Fixed Dimensions — CTA rows:**
Height should say HUG, with note: "renders as 48 (standard) or 44 (small) based on text size + padding"

---

## Emerging Patterns Update (after 5 sources, 75 learnings)

### Pattern 5: CTA Height = Padding + Content, Never Explicit
CTAs, cards, input wrappers — all derive height from padding + children. Only fixed-height elements: status bar (24), header (56), input field inner (48), icon circle (48), step progress (2).

### Pattern 6: Padding Is Context-Independent
CTA padding [12,16,12,16] doesn't change with CTA size. Input padding [12,16,12,16] doesn't change with field type. Card padding 16 doesn't change with card content. **Padding is a component property, not a context property.**

---

## Source 6: Screen-Level Visual Benchmarks (section 9270:8279)
**20 full screens** — ALL have `layoutMode: "NONE"` at root. Auto-layout not applied at screen level.
These are VISUAL REFERENCES — the skill must infer the correct autolayout from the content structure.

### Critical Finding: Screen Root = layoutMode NONE (all 20 screens)

Every screen in this section uses absolute positioning at the root frame. Children are placed by x/y coordinates, not auto-layout. But the visual intent is clear from the content zones.

**L76. Screen roots in design files often lack auto-layout — the skill must PRESCRIBE it, not describe it.**
When receiving a Figma screen with `layoutMode: "NONE"` at root, the skill should not mirror that. It should apply the standard screen architecture: VERTICAL, gap:0, p:0, HUG height.

### Screen Types Observed

| Screen | Key Zones | Pattern |
|---|---|---|
| Text input (Active Filled) | Top app bar (136) + form content (328×360) + keyboard (256) | Form with keyboard |
| Interceptor | Top app bar (80) + content (328×400) + CTA (48) | Standard: bar + content + CTA |
| CTA Actions | Bottom sheet (236) | Bottom sheet overlay |
| Pickup ticket details | Header (80) + timeline (88) + ticket content (536) + bottom bar (84) | List detail with timeline |
| Device ID Confirmation (×4) | Top app bar (80) + composite timeline (44) + form/content + CTA/action area | Multi-step form with progress |
| Partner task list (×2) | Homepage header (128) + scrollable task list (492-592) | List screen |
| Team Assignment Pending | Status bar (24) + header (80) + assignment card (416) + other info (102) | Detail screen |
| Notification | Single notification card (96) | Isolated component |
| Acknowledgement (×2) | Header (80) + content (216) + agree checkbox (48) + CTA (48) | Agreement/consent screen |
| Variation screens (×3) | Header (80) + content body + card (72) + bottom bar (84) | Detail with card |
| Transfer information | Top app bar (80) + form content (626) + dual CTA (48×2) | Long form with dual CTA |

### Screen Architecture Rules (from visual analysis)

**L77. Every screen has 3 structural zones: Header + Body + Footer.**
Not every zone is always visible, but the layout always follows this pattern:

```
Screen (VERTICAL, gap:0, p:0, FILL×HUG)
├── HEADER ZONE (FILL×HUG, always at top)
│   └── Status Bar + App Bar (or Homepage Header, or Big App Bar)
├── BODY ZONE (FILL×FILL in Compose, or FILL×HUG in Figma)
│   └── Scrollable content: forms, cards, lists, timelines
└── FOOTER ZONE (FILL×HUG, always at bottom)
    └── CTA area, bottom bar, or bottom sheet
```

**L78. Header zone always contains Status Bar + some form of App Bar.**
- Standard: Status Bar (24) + Normal Header (56) = 80
- Big: Status Bar (24) + Normal Header (56) + Title Zone (56) = 136
- Homepage: Status Bar (24) + Homepage Header (56) = 80 (or 128 with sub-header)

**L79. Body zone content uses screen horizontal padding (16dp each side).**
Content frames are consistently 328px wide (360 - 2×16). They sit centered in the screen. The body zone itself is 360-wide, with padding applied inside.

**L80. Footer zone has two sub-patterns:**
a. **CTA footer**: padding [12,16,24,16], VERTICAL gap:12, contains 1-2 CTAs
b. **Bottom bar/tab**: FIXED 84h, contains navigation or actions
c. **No footer**: content fills to bottom (list screens)

**L81. Keyboard changes the footer pattern.**
When keyboard is visible (256h), it pushes the footer up. The body zone shrinks. In Compose: `imePadding()` on the Scaffold or content Column. In Figma: keyboard is placed at bottom, content area shortened.

**L82. Content gap within body zone varies by screen type.**
- Form screens: gap:16 or 24 between field groups
- List screens: gap:0 (rows separated by dividers)
- Detail screens: gap:24 between sections
- Multi-step screens: gap:16 between progress + content + actions

**L83. Screens with a "Composite timeline" (progress indicator) place it between header and main content.**
4 of the Device ID screens have a 328×44 timeline frame. It sits right below the app bar, before the form content. This is a sub-header pattern — visually part of the header zone but structurally a child of the body.

**L84. Some screens show content at 312 width (Acknowledgement screens) instead of 328.**
312 = 360 - 2×24 (wider padding). This matches the dialog padding from L20. These screens are likely designed to feel like an in-page dialog/overlay. **When content needs a more contained feel (agreements, confirmations), use 24dp horizontal padding instead of 16dp.**

### What the Skill Should Prescribe for These Screens

When Pratibimb or Maverick receives a screen with `layoutMode: "NONE"`:

```
1. Set root to VERTICAL, gap:0, p:0
2. Identify header zone (Status Bar + App Bar) → FILL×HUG at top
3. Identify body zone (main content) → FILL×FILL (Compose) or FILL×HUG (Figma)
   - Apply horizontal padding 16dp (or 24dp for contained screens)
   - Set internal gap: 16 (form), 24 (sections), 0 (list)
4. Identify footer zone (CTA / bottom bar) → FILL×HUG at bottom
   - CTA area: p:[12,16,24,16], VERTICAL gap:12
5. All content children: FILL width (328 at 360 base = screen - 2×padding)
6. All children sizing: HUG height except body zone (FILL in Compose)
```

### Width Anomalies Found

| Screen | Child width | Expected | Issue |
|---|---|---|---|
| Device ID Confirmation (9270:7813) | Frame 333px wide | 328 | Off by 5px — likely misaligned |
| Acknowledgement screens | Content 312px | 312 = 360-48 | Intentional wider padding (24dp) |

### Element-Level Benchmarks (from screen references)

#### IMAGE CONTAINER
```
"New device" (180×180 FIXED, r:12, layoutMode:NONE)
├── image rectangle (180×180, fill:IMAGE, scaleMode:CROP)
├── overlay badge (64×64 FIXED, r:12) — positioned absolutely
└── close icon (24×24 FIXED) — positioned absolutely
```

**L85. Photo/image containers use FIXED width×height + layoutMode:NONE.**
Images are the exception to "never use NONE." The container holds the image + absolutely positioned overlays (badges, close buttons). Size is FIXED because aspect ratio must be preserved.
- Square product/device photo: 180×180, r:12
- FAQ answer image: 296×148
- Graphic/illustration: 96×96

**L86. Image scaleMode = CROP (not FIT).**
The image fills the container and crops overflow. This prevents letterboxing. In Compose: `contentScale = ContentScale.Crop`.

**L87. Image overlays (close button, badge) are absolutely positioned.**
They float on top of the image. In Compose: `Box { Image(); IconButton(modifier = Modifier.align(Alignment.TopEnd)) }`.

#### PROGRESS BAR (Segmented Step)
```
Composite timeline (328×44, VERTICAL, gap:10, p:[0,12,0,12])
├── Progress Loader (FILL×2 FIXED, HORIZONTAL, gap:8)
│   ├── Segment completed (FILL×2, r:4, #008043)
│   ├── Segment completed (FILL×2, r:4, #008043)
│   ├── Segment completed (FILL×2, r:4, #008043)
│   └── Segment remaining (FILL×2, r:4, #E8E4F0)
└── State container (FILL×HUG, HORIZONTAL, gap:57)
    ├── Step label (HUG×HUG, VERTICAL, crossAlign:CENTER)
    ├── Step label (HUG×HUG)
    ├── Step label (HUG×HUG)
    └── Step label (HUG×HUG)
```

**L88. Progress bar segments: FILL×FIXED(2), gap:8, r:4.**
Each segment uses FILL width for equal distribution. Height fixed at 2. Completed = #008043 (positive/600), remaining = #E8E4F0 (neutral/200). Confirms the skill's existing §5 values.

**L89. Step labels below progress bar use gap:57 (off-grid hack for SPACE_BETWEEN).**
Similar to status bar gap:234 (L55). The labels should be distributed evenly below the segments. In Compose: `Arrangement.SpaceBetween` on the Row, not a fixed gap.

**L90. Composite timeline horizontal padding: [0,12,0,12].**
12dp side padding (not 16dp). This is tighter than screen padding, creating visual alignment with the progress bar inner edges. **Progress indicators may use tighter horizontal padding than standard content.**

#### HEADING + DESCRIPTION GROUP
```
Text group (FILL×HUG, VERTICAL, gap:8)
├── Title (FILL×HUG, fs:20/Bold, #161021)
└── Subtitle (FILL×HUG, fs:16/Regular, #665E75)
```

**L91. Screen-level heading group: title fs:20/Bold + subtitle fs:16/Regular, gap:8.**
Title uses T3_20_Bold, subtitle uses T4_16_Regular in secondary color. This is the canonical content heading pattern. Both FILL×HUG.

#### RADIO SELECTION LIST
```
Section (FILL×HUG, VERTICAL, gap:12)
├── Card instance (FILL×HUG, HORIZONTAL, gap:16, p:16, r:16, stroke:#E8E4F0)
├── Card instance (FILL×HUG, ...)
└── Card instance (FILL×HUG, ...)
```

**L92. Radio selection cards stacked with gap:12.**
NOT gap:16 (which is the card-to-card gap in most contexts). Selection cards are tighter because they're a single selection group — they need to feel like one unit, not separate items.

#### CHECKBOX + TEXT (Agreement)
```
Agree frame (FILL×HUG, HORIZONTAL, gap:8)
├── checkbox-tick (24×24 FIXED)
└── text frame (FILL×HUG)
    └── agreement text (FILL×HUG, multi-line)
```

**L93. Checkbox row follows the universal horizontal pattern: [FIXED 24] + [FILL text].**
Gap:8 between checkbox and text. CrossAlign: MIN (top-aligned, not center) because the text may wrap to multiple lines — checkbox aligns with first line.

#### CTA PLACEMENT IN SCREENS

From positional analysis (y-coordinates in NONE-layout screens):
- **Interceptor CTA**: y=648 (screen 720 - CTA 48 - bottom padding 24 = 648) ✓
- **Transfer CTAs**: y=656 (two CTAs at same y — they're overlapping, likely one visible per state)

**L94. CTA y-position = screen_height - CTA_height - bottom_padding(24).**
In absolute-positioned screens, CTAs sit at y = 720 - 48 - 24 = 648. This confirms the CTA footer padding: bottom = 24dp.

#### BOTTOM SHEET (in screen context)
```
Bottom Sheet (360×HUG, VERTICAL, gap:0)
├── bottom_sheet_header (360×88) — handle + title
└── Content (360×HUG, VERTICAL, gap:48, p:[24,16,48,16])
    ├── Text group (HUG×HUG)
    └── [CTA hidden]
```

**L95. Bottom sheet content padding: [24,16,48,16].**
Top: 24 (matches dialog top padding). Bottom: **48** (extra large). Sides: 16 (standard). The 48dp bottom padding creates visual breathing room at the sheet's bottom edge. **Bottom sheet content has generous bottom padding (48) vs standard 24.**

**L96. Bottom sheet content-to-CTA gap = 48.**
Same as dialog (L19). Confirms: **the 48dp gap between content and CTA is standard for overlays (dialogs, bottom sheets), not just dialogs.**

#### TICKET TIMELINE
```
Ticket Timeline (360×HUG, VERTICAL, gap:12, p:[12,16,12,16], fill:#F1EDF7)
└── timeline content (FILL×HUG, VERTICAL, gap:6)
```

**L97. Timeline/status sections use tinted background + standard padding.**
Fill: #F1EDF7 (neutral/100). Padding: [12,16,12,16]. Same tinted-zone pattern as FAQ question header (L41). Internal gap:6 is tight for compact info display.

#### ACKNOWLEDGEMENT CONTENT (Graphic + Text)
```
Content (312×HUG, VERTICAL, gap:16)
├── Graphic (96×96 FIXED) — illustration
├── Headline (FILL×HUG, fs:24/Bold, #6D17CE info/600)
└── Description (FILL×HUG, fs:16/Regular, #000000)
```

**L98. Acknowledgement/confirmation screens use info/600 (#6D17CE) headline color.**
Not neutral/900. The purple accent creates emphasis for important announcements. Width: 312 (24dp side padding = contained feel, per L84).

**L99. Graphic illustrations: 96×96 FIXED, LEFT-aligned (not centered).**
Consistent with dialog icon alignment (L23). Graphics sit left, not centered, in content blocks.

---

## Consolidated Element Type Rules

| Element Type | Width | Height | Layout | Gap | Radius | Key rule |
|---|---|---|---|---|---|---|
| **Image container** | FIXED (square/rect) | FIXED | NONE | — | 12 | scaleMode:CROP, overlays absolute |
| **Progress bar** | FILL | FIXED 2 | HORIZONTAL | 8 | 4 | Segments FILL for equal width |
| **Step labels** | FILL (parent) | HUG | HORIZONTAL | SPACE_BETWEEN | — | Labels center-aligned under segments |
| **Heading group** | FILL | HUG | VERTICAL | 8 | — | Title 20/Bold + Subtitle 16/Regular |
| **Radio card list** | FILL | HUG | VERTICAL (parent) | 12 | — | Tighter gap than standard cards |
| **Checkbox + text** | FILL | HUG | HORIZONTAL | 8 | — | CrossAlign:MIN (top-aligned for multi-line) |
| **CTA in screen** | FILL (328 at 360) | HUG→48 | — | — | 16 | Position: bottom - 48 - 24 |
| **Bottom sheet content** | 360 (FILL) | HUG | VERTICAL | 48 (to CTA) | — | p:[24,16,48,16] |
| **Timeline/status zone** | FILL | HUG | VERTICAL | 6-12 | — | Tinted bg #F1EDF7, p:[12,16,12,16] |
| **Graphic/illustration** | FIXED (96×96) | FIXED | — | — | — | LEFT-aligned, not centered |
| **Acknowledgement** | 312 (24dp padding) | HUG | VERTICAL | 16 | — | Headline in info/600 purple |

---

## Source 7: Live Build Test — Team Assignment Screen Recreation

Rebuilt "Variation 2 - Screen 14091" using both sd-autolayout + sd-text-container skills.

**L100. HORIZONTAL frames: `primaryAxisSizingMode = "AUTO"` overrides `layoutSizingHorizontal` to HUG.**
For HORIZONTAL frames that need FILL width inside a VERTICAL parent, must re-set `layoutSizingHorizontal = "FILL"` AFTER any `primaryAxisSizingMode` change. This is because primaryAxis = horizontal for HORIZONTAL frames, so changing primaryAxisSizingMode directly changes the width behavior. Similarly, for HUG height on HORIZONTAL frames, use `counterAxisSizingMode = "AUTO"` (counterAxis = vertical = height).

**L101. CTA text centering requires `textAlignHorizontal = "CENTER"` on the text node itself.**
When CTA text is FILL width (which it should be per L69), the parent's `primaryAxisAlignItems: CENTER` has no effect — the text already fills the entire width, so there's nothing to center within the parent. The text characters must be centered via the text node's own alignment property. In Compose: `textAlign = TextAlign.Center`.

---

## Source 8: Maverick Developer — Build-Time Learnings (35 corrections from Netbox Setup Flow)

Extracted from `maverick-developer/SKILL.md`. These are corrections from actually building 20+ screens
in HTML/CSS for Android WebView APK. Each represents a real layout bug that was discovered on device.

### Flex/Layout Corrections

**L103. Screen structure = flex column 100vh, body flex:1, CTA flex-shrink:0.**
```css
.screen { display:flex; flex-direction:column; height:100vh; }
.body { flex:1; overflow-y:auto; }
.cta-area { padding:16px; flex-shrink:0; }
```
This is the CSS equivalent of: Screen VERTICAL, body FILL height (scrollable), CTA HUG at bottom.
Confirms L77 (3-zone screen architecture).

**L104. Never use fixed height on content containers.**
Use `min-height` or `flex`. Only fixed-height elements: app bar (56), status bar (24), tab bar (48).
Maverick Mistake #33: "Content centered with flex + spacer" → wrong. Use `padding-top` for fixed y-offset.
Confirms L67 (HUG default for height).

**L105. CTA position must be IDENTICAL on every screen: `padding: 0 16px 24px; flex-shrink:0`.**
Maverick Mistake #29: CTA padding was inconsistent across screens.
Rule: Define CTA position ONCE as a global rule. Never per-screen padding.
Confirms L94 (CTA y = screen_height - 48 - 24).

**L106. Connected elements: negative margin + z-index for overlapping cards.**
```css
.info-card { position:relative; z-index:1; border-radius:12px; box-shadow:...; border:1px solid...; }
.info-tip { margin-top:-12px; border-radius:0 0 12px 12px; background:#F1E5FF; padding:19px 12px 8px; }
```
Connected region height = AUTO (never fixed). Top padding accounts for overlap zone.
Maverick Mistake #12: Tip bar was built as separate div above card instead of overlapping below.
**New pattern for autolayout skill: overlapping connected regions.**

**L107. Camera viewfinder = flex:1 fills remaining space.**
In VERTICAL screen: app bar FIXED → viewfinder FILL → shutter button FIXED.
In Figma: viewfinder `layoutSizingVertical = "FILL"`.
In Compose: `Modifier.weight(1f)`.

**L108. Gap applies uniformly — use per-element margins for fine control.**
Maverick Mistake #10: Uniform `gap:12px` on header → wrong. Rating and menu needed different spacing.
Rule: `itemSpacing` / `gap` applies to ALL children equally. If children need different spacing,
use individual child padding/margin instead. In Figma: set gap:0 on parent, use per-child padding.

**L109. Header pattern: name flex:1 ellipsis, fixed elements flex-shrink:0.**
```
[Logo 32] [12px] [Name flex:1 ellipsis] [Rating flex-shrink:0] [12px] [Menu 24]
```
In Figma: name `layoutSizingHorizontal = "FILL"`, logo/rating/menu = FIXED.
Confirms L57 (app bar: left FILL, right HUG).

**L110. Ghost CTA: bg matches screen bg, border:none, text color only.**
Maverick Mistake #34: Built ghost CTA with outlined border.
Figma showed `bg:#FAF9FC` (same as screen), no stroke. Only text has pink color.
**Ghost buttons = no border, no contrasting fill. Just colored text in a transparent-ish frame.**

**L111. Fixed y-offset = padding-top, NOT flex centering with spacers.**
Maverick Mistake #33: Used flex + spacer to push content to specific y position.
If Figma shows an element at a specific y-offset, use `padding-top` on the parent, not flex tricks.

### Width/Sizing Corrections

**L112. Dialog width: calc(100% - 48px), max-width 312px.**
Responsive with a cap. At 360: 360-48 = 312. At 412: 412-48 = 364, capped at 312.
Confirms L21 (dialog width derived from viewport).

**L113. Content cards: calc(100% - 32px) or margin 0 16px.**
16px padding each side = 32px removed from viewport width.
At 360: 328. At 412: 380.
Confirms L33 (padding chain determines width).

**L114. Text blocks: width 100% with padding, never fixed pixel width.**
Maverick builds never use fixed-width text containers.
Confirms L34 (only icons/images have absolute width).

### Visual Overlap & Elevation

**L115. Cards with shadow — check what's UNDER them for overlap connections.**
Shadow = higher z-plane. If next sibling's y < current node's (y + height), there's an overlap.
Overlap = intentional visual connection, not a bug.
Implementation: z-index on elevated card, negative margin-top on connected element.

**L116. Border + shadow can coexist on the same element.**
Maverick Mistake #11: Assumed shadow = no border. Wrong — some cards use BOTH.
`border: 1px solid #E8E4F0` + `box-shadow: 0 1px 3px rgba(...)`.
Border for definition, shadow for elevation. Check Figma for both stroke AND effects.

### State & Visibility

**L117. Hidden layers ≠ "don't build." They often represent state-dependent content.**
Maverick Mistake #16: Built 4 hidden sections into screen as always-visible.
Hidden layers = content that toggles based on user action. Build with `display:none`, toggle via JS.
Confirms L2 (visibility-driven layout) + L15 (maximal child count).

**L118. Multi-state screens: build a state machine, not a flat layout.**
Maverick Mistake #25: Showed both Aadhaar front+back slots simultaneously.
Figma shows multiple states of the same screen. Build each state, toggle between them.
In Compose: use `when(state)` to show different content.

### Device-Specific

**L119. Never build HTML status bars in APK — native OS handles them.**
Maverick Mistake #19: Built fake time/battery bar in HTML.
Figma shows status bars for prototype context. In production: Android system bar exists, HTML bar duplicates it.
Confirms L54 (status bar 24dp is Figma reference, not a build target).

**L120. Test every screen at 320dp AND 412dp width.**
320: minimum supported Android width. 412: common large phone.
No text clipping at 320. No excessive stretching at 412.
Confirms device-safety floor rules.

### Summary: What Maverick Adds to the Autolayout Skill

| New concept | Learning | Skill section to update |
|---|---|---|
| Connected/overlapping elements | L106, L115, L116 | New pattern — not in current skill |
| Ghost CTA type | L110 | §CTA decision tree |
| Gap vs per-element margin | L108 | §3 spacing nuance |
| Fixed y-offset = padding-top | L111 | §12 anti-patterns |
| State machine, not flat layout | L118 | §6 scroll/overflow or new §visibility |
| Dialog max-width cap | L112 | §5 dialog dimensions |
| 320dp minimum test width | L120 | Floor values |

---

## Total Learnings: 120 (L1-L120)

### Coverage by source:
| Source | Learnings | Type |
|---|---|---|
| 1. Input-Field | L1-L18 | Component (18) |
| 2. Dialog + BSH | L19-L31 | Component (13) |
| 3. Width Correction | L32-L34 | Fundamental (3) |
| 4. Mixed Molecules | L35-L53 | Component (19) |
| 5. Navigation/Headers | L54-L66 | Component (13) |
| 6. CTAs/Actions | L67-L75 | Component (9) |
| 7. Screen-Level References | L76-L99 | Screen + Element (24) |
| 8. Live Build Test | L100-L102 | Figma API (3) |
| 9. Maverick Build Corrections | L103-L120 | Build-time (18) |

---

## Source 9: Screen-Level References V2 — Netbox Setup Flow (54 screens, node 9274:16397)

Categories: Camera/Aadhaar, Installation steps, Earn Money, Device Management, Send Money, Delivery Confirmation, Customer Info sheets, Dialogs.

### Camera/Viewfinder Screen (9278:1738)

**Anatomy:**
```
Screen (360×720, NONE, fill:#4D4D4D dark overlay)
├── Viewfinder (360×720, NONE — full screen dark bg)
│   ├── Camera feed image (480×720, IMAGE — wider than screen for crop)
│   └── Shutter button (64×64 GROUP, positioned at bottom center)
├── Top app bar (360×144, VERTICAL, gap:0) — overlays on top of viewfinder
├── Status bar (360×24)
└── Flash icon (24×24, positioned absolutely)
```

**L121. Camera screen = full-screen viewfinder with overlaid controls.**
Viewfinder fills entire screen (360×720). App bar, status bar, and controls are overlaid ON TOP using absolute positioning (layoutMode:NONE at root). This is the one screen type where the root MUST be NONE — auto-layout can't layer a viewfinder under controls.

**L122. Camera feed image is WIDER than screen (480 vs 360).**
The image is 480×720 inside a 360×720 frame = horizontal crop. This ensures no letterboxing. The frame clips the overflow. In Compose: `contentScale = ContentScale.Crop` with `clipToBounds = true`.

**L123. Shutter button is 64×64, positioned at bottom center.**
Absolutely positioned in the viewfinder frame. In Compose: `Box(modifier = Modifier.align(Alignment.BottomCenter).padding(bottom = 40.dp))`.

### Bottom Sheet with Horizontal Cards (9278:2661)

**Anatomy:**
```
Bottom sheet (360×HUG, VERTICAL, gap:0)
├── bottom_sheet_header (360×88)
└── cards (360×HUG, HORIZONTAL, gap:16, p:[24,16,48,16])
    ├── card1 (FILL×HUG, 156 rendered) — r:16
    └── card2 (FILL×HUG, 156 rendered) — r:16
```

**L124. Side-by-side cards in bottom sheet: FILL×HUG with gap:16.**
Two cards at FILL width = equal distribution: (360 - 32padding - 16gap) / 2 = 156 each. This is the `weight(1f)` pattern in Compose. Both cards r:16.

**L125. Bottom sheet cards area padding: [24,16,48,16].**
Confirms L95: bottom sheet content gets 48dp bottom padding. Top padding 24, sides 16.

### Wallet Card (9278:12294 — Send Money)

**Anatomy:**
```
Improved Wallet (FILL×HUG, VERTICAL, gap:16, p:16, r:16, fill:#F1EDF7)
└── Top Section (HUG×HUG, VERTICAL, gap:24, crossAlign:CENTER)
    └── [balance amount + label]
```

**L126. Wallet/balance card: VERTICAL, p:16, r:16, tinted bg #F1EDF7.**
Standard card pattern with centered content (crossAlign:CENTER on inner section). The wallet amount would be a hero number (T2_24_Bold or larger).

### Delivery Confirmation (9278:10186)

**L127. Delivery screens have a photo reference frame (FIXED 184×166) with overlapping grouped images.**
Photo reference = absolutely positioned image groups showing correct/incorrect examples. This is a specialized image container — not auto-layout, uses GROUP for composed photo arrangements.

**L128. Delivery CTA uses r:12 (not r:16).**
The "confirm delivery" CTA has `cornerRadius: 12`. This is the small/checkbox CTA variant, matching L71 (smaller CTA = smaller radius).

### Customer Info Bottom Sheet (9278:5507)

**Anatomy:**
```
Customer info sheet (360×HUG, VERTICAL, gap:0)
├── bottom_sheet_header (360×88)
└── content (FILL×HUG, VERTICAL, gap:36, p:[24,16,16,16])
    ├── [3 hidden Card instances — radio selection cards]
    ├── Customer Info (FILL×HUG, VERTICAL, gap:12, r:16, p:16, fill:#FAF9FC)
    └── actions CTA (FILL×HUG, HORIZONTAL, gap:8, r:16)
```

**L129. Customer info sheet content gap = 36.**
Larger than standard 16 or 24. This creates strong visual separation between sections inside a bottom sheet. May be specific to sheets with mixed content types (cards + info + CTA).

**L130. CTA inside bottom sheet has gap:8 (not 12).**
The CTA `actions` frame inside this sheet uses gap:8 instead of the standard 12. Possibly because the CTA has both icon + text (tighter layout).

### Dialog with Checkbox (9278:2029)

**Anatomy:**
```
Dialog (312×HUG, VERTICAL, gap:48, p:[32,24,32,24], r:24?, fill:#FAF9FC)
├── Head (FILL×HUG, VERTICAL, gap:24) — icon + title + body + input field
└── agree (HUG×HUG, HORIZONTAL, gap:12) — checkbox + text
```

**L131. Dialog can contain Input-Field instances and checkbox rows.**
The dialog Head section includes an Input-Field component (264×80) alongside text. This confirms L25 (instances inside dialogs use FIXED width = 264). The agree row follows L93 (checkbox pattern).

### Earn Money Landing (9274:16609)

**L132. Landing/dashboard screens have a long scrollable body (1095h) inside a NONE-layout root.**
Body frame: 361×1095, VERTICAL, gap:16, HUG height. Contains multiple card sections stacked vertically. On device this scrolls. In Figma the frame exceeds the 720 viewport — this is correct (L77: Figma shows full scrollable content).

**L133. Dashboard body gap = 16 between section cards.**
Consistent with standard card-to-card gap.

---

## Total Learnings: 133 (L1-L133)

## Updated Scenario Coverage (post Source 9):

| Level | Before | After | New |
|---|---|---|---|
| L1 Atoms | 12/16 | 12/16 | — |
| L2 Molecules | 12/17 | 13/17 | Amount display (wallet, L126) |
| L3 Organisms | 7/19 | 10/19 | Camera viewfinder (L121-123), Bottom sheet selection (L124), Photo review (L127) |
| L4 Screen Sections | 3/12 | 5/12 | Form in sheet (L129-131), Dashboard body (L132-133) |
| L5 Full Screens | 3/12 | 5/12 | Camera screen (L121), Form with keyboard (L128 context) |
| L6 Edge Cases | 1/12 | 1/12 | — |
| **Total** | **38/78** | **46/78** | +8 scenarios |

---

## Retroactive: OTP/Verification Screen Pattern (from Source 1 + Source 6)

Already extracted at component level (L4, L13) but not formalized as a screen pattern.

### OTP Screen Architecture
```
Screen (VERTICAL, gap:0)
├── HEADER: Top app bar (80 or 136 with Big title)
├── BODY (FILL×HUG in Figma / FILL×FILL in Compose, p:[24,16,24,16])
│   ├── Composite timeline / progress bar (if multi-step)
│   ├── Heading group (title 20/Bold + subtitle 16/Regular, gap:8)
│   ├── Input-Field OTP variant (FILL×HUG, contains 4 OTP boxes)
│   │   ├── title frame (FILL×HUG, fs:16 Bold/Regular by state)
│   │   ├── field (FILL×48, HORIZONTAL, gap:8)
│   │   │   └── 4× OTP boxes (each FILL×HUG=48, r:12, p:[12,16,12,16], text CENTER)
│   │   └── subtext (FILL×HUG, hidden unless error)
│   ├── [Optional] Timer/hint text (fs:14/Regular, #665E75)
│   └── [Optional] Resend link (fs:14/SemiBold, #D9008D)
└── FOOTER: CTA area (p:[12,16,24,16])
    └── Primary CTA "आगे बढ़ें" (FILL×HUG→48, r:16)
```

**L134. OTP screen body content is top-aligned, not centered.**
Unlike some verification screens that center content, Wiom OTP screens align content to the top of the body zone. The heading sits right below the progress bar/app bar.

**L135. OTP box internal text alignment: CENTER (both axes).**
OTP boxes use `primaryAxisAlignItems: CENTER` + `counterAxisAlignItems: CENTER` on the box frame. Empty box shows hint color #A7A1B2, filled shows #161021 Bold. Active box shows cursor #D9008D.

**L136. OTP field container (all 4 boxes) has gap:8, NO outer padding.**
The OTP boxes sit inside a field frame with p:[0,0,0,0] and gap:8. The parent Input-Field component handles the outer padding. This means OTP boxes touch the field edges — spacing comes from the box internal padding [12,16,12,16].

---

## Source 10: Screen-Level References V3 — Settings, Payments, Loading, Tabs (42 screens, node 9278:25339)

### Settings Screen (9278:39855 — Partner Settings)

**Screen architecture:**
```
Screen (360×720, NONE, fill:#FAF9FC)
├── Leads Top bar (360×80) — app bar
└── Content (360×528, VERTICAL, gap:24, crossAlign:CENTER)
    ├── Info card (328×280, VERTICAL, gap:12) — heading + content card
    ├── Menu groups (FILL×HUG, VERTICAL, gap:0) — multiple sections
    │   ├── Group 1 (FILL×HUG, bg:#F1EDF7, gap:0)
    │   │   ├── Section header (FILL×HUG, HORIZONTAL, p:[16,16,8,16])
    │   │   └── Content area (FILL×HUG, p:[12,16,16,16])
    │   └── Group 2 (FILL×HUG, gap:0) — more rows
    └── Logout row (328×60, HORIZONTAL, gap:8, p:[12,16,12,16], r:12, stroke:#E8E4F0)
```

**L137. Settings screen = no CTA footer. Content scrolls to bottom.**
Unlike form screens, settings has no fixed CTA at the bottom. The entire content area scrolls. This is the L5.3 (settings/menu screen) pattern.

**L138. Menu groups use gap:0 + tinted background per section.**
Each settings section is a VERTICAL frame with gap:0 and a section bg color (e.g., #F1EDF7). Section header has asymmetric padding [16,16,8,16] (tighter bottom — content follows closely). Content area has normal padding.

**L139. Logout/action row at bottom: 328×HUG, HORIZONTAL, gap:8, p:[12,16,12,16], r:12, stroke:#E8E4F0.**
Matches the standard card/row pattern but with r:12 (not 16). This is a standalone action row, not inside a menu group.

**L140. Section headers in settings use gap:10 (off-grid).**
Another off-grid gap violation (like L10). Should be 8 or 12.

### Bill Payment Screen (9278:40466 — Bill recharge_total payment)

**Screen architecture:**
```
Screen (360×720, NONE, fill:#FAF9FC)
├── PA_Headers (360×80)
└── Content (328×730, VERTICAL, gap:16)
    ├── Amount header (FILL×HUG, VERTICAL, gap:0, stroke:#D7D3E0 bottom, p:[0,0,16,0])
    │   └── Amount row (FILL×HUG, HORIZONTAL, SPACE_BETWEEN)
    └── Bill sections (FILL×HUG, VERTICAL, gap:24)
        ├── Section 1 (FILL×HUG, VERTICAL, gap:12, stroke:#E8E4F0 bottom, p:[0,0,24,0])
        └── Section 2 (FILL×HUG, VERTICAL, gap:12, p:16, r:16, fill:#FAF9FC, stroke:#E8E4F0)
```

**L141. Payment summary: amount header separated by bottom stroke, not gap.**
The amount header uses `stroke: #D7D3E0` with bottom-only padding [0,0,16,0] as a visual divider. Below it, bill sections are stacked with gap:24.

**L142. Bill sections alternate between borderless (stroke bottom only) and card-style (r:16, p:16, stroke all sides).**
First section: line items with bottom stroke separator. Second section: enclosed card with full border. This creates hierarchy — primary items in open layout, secondary details in cards.

**L143. Bill section internal gap:12, section-to-section gap:24.**
Confirms the internal < external spacing rule. Section items are tight (12), sections are separated (24).

### Loading/Skeleton Screen (9278:41468)

**Structure:**
```
Screen (360×720, NONE, fill:#FAF9FC)
├── PA_Headers (360×128, brand bg)
└── Skeleton group (GROUP, 328×248, absolute positioned)
    ├── Main shimmer rect (328×248, r:16, image fill — animated GIF)
    ├── Avatar placeholder (72×72, r:36, fill:#D7D3E0)
    ├── Title bar (168×16, r:0, fill:#D7D3E0)
    ├── Tag placeholder (67×46, r:12, fill:#D7D3E0)
    └── Subtitle bar (197×16, r:0, fill:#D7D3E0)
```

**L144. Loading skeletons use GROUP (absolute positioning), not auto-layout.**
Skeleton elements are positioned absolutely to match the exact content layout they replace. Each shimmer rect has the same dimensions as the content it represents.

**L145. Skeleton shimmer colors: #D7D3E0 (neutral/300) on all placeholder rectangles.**
Uniform muted gray. No color variation between placeholder types.

**L146. Skeleton radius matches content radius.**
Avatar: r:36 (circular). Tag: r:12. Main area: r:16. Text bars: r:0 (sharp). Each skeleton element mirrors the radius of the content it replaces.

**L147. Loading screen uses a GIF for the main shimmer animation.**
The primary skeleton is a RECTANGLE with image fill (animated GIF). Individual placeholder rects are layered on top. In Compose: use `shimmer()` modifier instead of GIF.

### Tab Content (9278:41405 — New and In Progress Tasks)

**Header with Tabs:**
```
PA_Headers (360×128, VERTICAL, gap:0, brand bg)
├── Status Bar (360×24)
├── Header bar (360×56, HORIZONTAL, gap:106)
└── Tabs (360×48, HORIZONTAL, gap:0) — tab bar instance
```

**Body:**
```
Content (360×332, VERTICAL, gap:24, crossAlign:CENTER)
├── Active task card (328×256, VERTICAL, gap:16, r:16, fill:#FFE5F6, stroke:#D9008D)
├── Divider line (360×0, stroke:#E8E4F0)
└── Section header row (328×28, HORIZONTAL, gap:16)
```

**L148. Tab bar = HORIZONTAL, gap:0, FIXED 360×48.**
Tabs sit inside PA_Headers at the bottom. Each tab fills equally. Gap:0 because tab items manage their own internal padding. Height: 48 (matches tab bar constant in §5).

**L149. Header with tabs = 128h (Status 24 + Bar 56 + Tabs 48).**
Three-zone header stacked vertically, gap:0. This is the "PA_Headers with sub-navigation" pattern.

**L150. Tab content body uses gap:24 between content sections.**
Active task card → divider line → section header. Gap:24 creates clear separation between content blocks.

**L151. Active/selected task card uses pink tonal bg (#FFE5F6) + brand stroke (#D9008D).**
Same selection treatment as radio cards (L36) but at card level. The selected tab's content card gets the pink treatment.

**L152. Divider lines inside body: FILL width (360), height 0, stroke:#E8E4F0.**
Full-width dividers (not 328 content width). They extend to screen edges, visually separating content sections without respecting the 16dp screen padding. In Compose: `Divider(modifier = Modifier.fillMaxWidth())`.

### Installation Reward (9278:46747)

**Structure:**
```
Screen (360×720, NONE, fill:#FAF9FC)
├── Status bar (360×24)
├── App bar (360×80)
├── Reward card (328×676, VERTICAL, gap:24, p:[32,24,0,24], r:16, fill:#008043 success-green)
│   ├── Header info (FILL×HUG, HORIZONTAL, gap:12, crossAlign:CENTER)
│   ├── Earnings detail frame (280×72, VERTICAL, gap:7, stroke:#D6B2FF)
│   └── Breakdown section (HUG×FIXED 360, VERTICAL, gap:24)
└── Action footer (360×80, VERTICAL, gap:16, fill:#FAF9FC)
```

**L153. Success/reward screens use a full-height colored card (fill:#008043).**
The reward card fills most of the screen (676h out of 720). It uses positive/600 green as the background — not just an accent but the primary surface color. This is a celebration/achievement screen pattern.

**L154. Reward card padding: [32,24,0,24] — no bottom padding.**
Top padding 32 (generous, breathing room for the achievement), bottom 0 (content flows to card edge where the CTA footer sits below). This is unique — most cards have equal padding.

**L155. Earnings detail has non-grid gap:7 and stroke:#D6B2FF (light purple).**
7 is off-grid (should be 8). The purple stroke on a green card creates a contrasting info section — a card-within-card for specific data.

---

## Total Learnings: 155 (L1-L155)

## Updated Scenario Coverage (post Source 10):

| Level | Before | After | New scenarios filled |
|---|---|---|---|
| L1 Atoms | 12/16 | 13/16 | Divider line (L152) |
| L2 Molecules | 13/17 | 14/17 | Tab pair (L148) |
| L3 Organisms | 10/19 | 10/19 | — |
| L4 Screen Sections | 5/12 | 9/12 | Settings menu (L137-140), Payment summary (L141-143), Tab content (L148-152), Empty/loading (L144-147) |
| L5 Full Screens | 5/12 | 8/12 | Settings screen (L137), Payment screen (L141), Success screen (L153) |
| L6 Edge Cases | 1/12 | 2/12 | Loading/skeleton (L144-147) |
| **Total** | **46/78** | **56/78 (72%)** |

---

## Source 10b: Continued Extraction from Section 3 (new screens added)

### Long Settings Form (9278:49650 — Partner Settings 1341h)

**Structure:**
```
Screen (360×1341, NONE) — exceeds viewport, scrollable
├── Leads Top bar (360×80)
└── Content (328×580 viewport clip, contains 1136h scrollable form)
    └── Form frame (HUG×1136, VERTICAL, gap:16)
        ├── Menu row 1 (328×80, HORIZONTAL, gap:16, p:16, r:16)
        ├── Menu row 2 (328×80, ...)
        ├── ... (12 rows total, all 80h each)
```

**L156. Long form/settings screens: form body HUGs to full content height (1136h).**
The form frame is 1136h (12 rows × 80 + gaps). Parent clips at 580h viewport. In Compose: `Column(Modifier.verticalScroll())`. In Figma: HUG shows full height — this is correct (L77).

**L157. Settings menu rows: 328×80 each, HORIZONTAL, gap:16, p:16 all, r:16.**
Every row is identical structure: icon circle (48×48) + text content (FILL). Gap:16, padding:16 uniform, radius:16. This is the standard list-row pattern but at 80h (taller than the 57h menu item from L64 because it includes subtitle text).

**L158. Form gap:16 between all rows — uniform throughout long list.**
Confirms L7 (uniform gap within component). 12 rows × gap:16 between each.

### Expert Device ID Capture (9278:50798 — Right/Wrong Image Examples)

**Structure:**
```
Screen (360×720, NONE, fill:#161021 dark bg)
├── Top app bar (360×80)
├── Camera mask area (328×396, GROUP — dark viewfinder area)
├── Example card: correct (328×140, VERTICAL, gap:12, p:[12,16,12,16], r:12, fill:#FAF9FC)
│   ├── Header row (FILL×HUG, HORIZONTAL, gap:8) — icon + label
│   └── Details (GROUP, 223×80) — example images
├── Example card: incorrect (328×140, same structure)
├── Example card: another variant (328×140, same)
├── Scan action row (328×48, HORIZONTAL, gap:16)
└── Processing overlay (328×396, fill:#000000)
```

**L159. Image example cards (right/wrong): VERTICAL, gap:12, p:[12,16,12,16], r:12.**
Each card shows a header (icon + "सही तरीका"/"गलत तरीका") + example photos. Padding is [12,16,12,16] (same as input field inner). Radius 12 (card standard).

**L160. Camera screen with instructional overlay = dark bg (#161021) + white content cards.**
The entire screen bg is dark. Content cards and instructional elements use #FAF9FC fill for contrast. This creates a focus-on-camera visual.

**L161. Multiple stacked example cards use gap:12 (inferred from absolute positions).**
Cards are stacked vertically on the dark bg with consistent spacing. In auto-layout this would be a VERTICAL parent with gap:12.

### ISP Recharge List (9278:56064)

**Structure:**
```
Screen (360×720, NONE, fill:#F1EDF7 lavender)
├── PA_Headers (360×80, brand bg)
├── Error banner (360×52, VERTICAL, gap:8, p:[12,16,12,16], fill:#D92130)
│   └── Row (FILL×HUG, HORIZONTAL, gap:8, crossAlign:CENTER)
└── Content (328×600, VERTICAL, gap:24)
    ├── Section 1 (FILL×HUG, VERTICAL, gap:16) — recharge cards
    └── Section 2 (FILL×HUG, VERTICAL, gap:16) — more cards
```

**L162. Error banner: full-width (360), fill:#D92130, p:[12,16,12,16].**
Sits between header and content. Uses negative/600 red as background. Icon + text in HORIZONTAL row, gap:8. This is a system-level alert bar — no radius (full-width edge-to-edge).

**L163. Error banner has no radius — goes edge to edge.**
Unlike cards (r:12-16) or dialogs (r:24), the error banner is r:0. Full width, no rounding. This creates urgency — it's not a dismissible card, it's a system state.

**L164. ISP list uses lavender bg (#F1EDF7) with content gap:24 between sections.**
The screen bg is lavender (not white). Cards inside are white. This creates card-on-tinted-bg contrast — same pattern as task list screens.

### Bottom Sheet with CTA (9278:56320)

**Anatomy:**
```
Bottom sheet (360×344, VERTICAL, gap:0)
├── bottom_sheet_header (360×88 INSTANCE)
├── content (FILL×HUG, VERTICAL, gap:48, p:[16,16,48,16])
│   └── options (328×128, VERTICAL, gap:16) — radio/selection items
└── Dialog Action bar (FILL×HUG, VERTICAL, gap:12, p:[0,16,16,16])
    └── Primary CTA (FILL×HUG, r:16, fill:#D9008D)
```

**L165. Bottom sheet with CTA = 3 zones: header + content + action bar.**
This is the bottom sheet equivalent of the screen 3-zone pattern (L77). Header = handle+title. Content = options/form. Action bar = CTA. All separated by gap:0.

**L166. Bottom sheet action bar padding: [0,16,16,16] — NO top padding.**
Top padding is 0 because the content's bottom padding (48) already provides separation. This prevents double-spacing between content and CTA. Total visual gap = content bottom 48 + action top 0 = 48.

**L167. Bottom sheet options area: gap:16 between selection items.**
Standard card-to-card gap for selection options inside sheets.

### Comment Sheet (9278:55451)

**Anatomy:**
```
Comment (360×720, NONE — NOT auto-layout at root)
├── bottom_sheet_header (360×88)
└── Content (HUG×HUG, VERTICAL, gap:48, p:[24,16,48,16])
    ├── Text frame (HUG×HUG, VERTICAL, gap:24)
    └── CTA (FIXED 328×48, hidden)
```

**L168. Comment/feedback sheets: content gap:48, matching dialog content-to-CTA pattern.**
Same gap:48 as dialogs (L19, L96). Confirms: overlay content (dialogs, sheets) uses 48 gap between content and actions.

### Strike Rate (9278:54664)

**Anatomy: A sheet rendered as full 720h frame (VERTICAL, gap:0)**
```
Strike Rate (360×720, VERTICAL, gap:0)
├── bottom_sheet_header (360×88)
└── Content (HUG×HUG, VERTICAL, gap:16, fill:#FAF9FC)
```

**L169. Some bottom sheets are modeled at full screen height (720) even though they only use 240h of content.**
The remaining space is transparent — the sheet visually only occupies its content height. In Compose: `ModalBottomSheet` handles this naturally. In Figma: the 720h frame is a prototype requirement, not a layout spec.

### Card Spread / Celebration (9278:54533 — Frame 1000005729, 800h)

**L170. Celebration/gamification screens exceed standard 720 viewport (800h).**
This screen is 800h with sparkle decorations, card spreads, and overlapping elements. The extra height accommodates animated/decorative content. These screens break standard patterns intentionally — they're one-off celebrations.

---

## Total Learnings: 170 (L1-L170)

## Updated Scenario Coverage:

| Level | Before | After | New |
|---|---|---|---|
| L1 Atoms | 13/16 | 13/16 | — |
| L2 Molecules | 14/17 | 14/17 | — |
| L3 Organisms | 10/19 | 11/19 | Notification/banner (L162-163) |
| L4 Screen Sections | 9/12 | 11/12 | Long form (L156-158), Urgency banner (L162-163) |
| L5 Full Screens | 8/12 | 10/12 | Long form screen (L156), Onboarding/instruction (L159-161) |
| L6 Edge Cases | 2/12 | 2/12 | — |
| **Total** | **56/78** | **61/78 (78%)** |

---

## Source 10c: Quantity Counter + Nested Cards (depth pass on existing screens)

### Quantity Counter (Request Device sheet, 9278:38571)

**Anatomy:**
```
Card (FILL×HUG, VERTICAL, gap:12, p:16, r:16, fill:#FAF9FC, stroke:#E8E4F0)
├── Counter row (FILL×HUG, HORIZONTAL, SPACE_BETWEEN, p:[0,0,16,0], stroke:#E8E4F0 bottom)
│   ├── Price label (HUG×HUG, fs:20/Bold, "₹2,000")
│   └── Counter button (HUG×HUG, HORIZONTAL, gap:10, p:[8,16,8,16], r:12, fill:#D9008D)
│       └── "–       1      +" (single text node, fs:16/Bold, #FAF9FC)
└── Info section (FILL×HUG, VERTICAL, gap:8)
    ├── Total text (FILL×HUG, "कुल : ₹2,000", fs:14/Regular)
    └── Hint chip (FILL×HUG, r:8, fill:#F1EDF7, p:[4,8,4,8])
```

**L171. Quantity counter is a single brand-colored button, not minus-number-plus separately.**
The "– 1 +" is a SINGLE text node inside one button frame. No separate minus/plus buttons. The entire 108×40 frame is one tappable element. p:[8,16,8,16], r:12, fill:#D9008D.

**L172. Counter button: HUG×HUG, r:12 (small CTA radius), gap:10 (off-grid).**
Matches the small CTA pattern (L71: smaller CTA = r:12). Height: HUG → 40 (8 + 24 + 8). Gap:10 is off-grid (should be 8 or 12).

**L173. Counter row uses SPACE_BETWEEN with bottom stroke separator.**
Price left, counter right. Same as key-value pattern (L7b from text-container skill) but with a button instead of text value. Bottom stroke divides counter from info below.

**L174. Request Device sheet content crossAlign: MAX (right-aligned CTA).**
The sheet content frame has `crossAlign: "MAX"` — this pushes the CTA to the right side. Unusual — most content is START/CENTER aligned.

### Nested Cards — Settings Profile Section (9278:39855)

**Structure:**
```
Profile section (FILL×HUG, VERTICAL, gap:12, p:0)
├── Title (FILL×HUG, fs:20/Bold, "आपको रेटिंग कैसे मिलती है?")
└── Inner card list (FILL×HUG, VERTICAL, gap:12, p:0) ← NESTING LEVEL 1
    ├── Card A (FILL×HUG, HORIZONTAL, gap:16, p:16ish, r:16, fill:#FAF9FC, stroke:#E8E4F0)
    ├── Card B (FILL×HUG, same)
    ├── Card C (FILL×HUG, same)
    └── Card D (FIXED×HUG, hidden)
```

**L175. Nested cards in settings: outer section has NO fill/stroke/radius. Inner cards have all three.**
The outer container (Frame 1000004413) is a plain frame — no bg, no border, no radius. It's just a layout wrapper. The visual "card" appearance only exists on the inner items. This is NOT card-inside-card visually — it's cards-inside-section.

**L176. True card nesting: inner cards have same radius as standalone cards (r:16).**
Since the parent has no radius, the radius nesting formula (child = parent - padding) doesn't apply here. Each inner card is a standalone visual element. The parent is transparent.

### Nested Cards — Reward Screen (9278:47705)

**Structure:**
```
Reward card (328×639, VERTICAL, gap:24, p:[32,24,0,24], r:16, fill:#008043)
├── Header info (FILL, HORIZONTAL)
├── Info section (282×67, stroke:#D6B2FF) ← card-like inner with border
├── Breakdown (HUG×360, VERTICAL, gap:24)
├── Service Ticket (282×260, r:~14, fill:#FAF9FC, stroke:#D9008D) ← NESTED CARD (hidden)
└── Service ticket CTA (328×49, fill:#D9008D) ← hidden
```

**L177. True visual card nesting: white card (r:14) inside green card (r:16).**
The Service Ticket is a white card (#FAF9FC) with brand stroke (#D9008D) nested inside the green reward card. Parent r:16, p:24 horizontal. Child r:14 (≈ 16 - 2, roughly following the nesting formula but with fractional values from non-grid design).

**L178. Nested card radius should be parent_radius - parent_padding but often isn't exact.**
Formula says: 16 - 24 < 0, so child should be 0. But here child is ~14. The designer chose a visually pleasing radius rather than following the mathematical formula. **In practice: nested card radius is "visually smaller than parent" — use 8 or 12 for cards nested in r:16 parents with large padding.**

**L179. Info section with stroke border inside colored card = card-like accent zone.**
Frame 1000005123 (282×67, stroke:#D6B2FF) is not a full card — it's a bordered info section. No fill, just stroke. Creates a visual callout without full card treatment. **Stroke-only borders create lighter nesting than fill+stroke cards.**

### Process Improvement Note

**L180. Depth before breadth for component extraction.**
The skill missed quantity counters and nested cards on first pass because extraction prioritized screen-level overview (breadth) over component-level detail (depth). Rule for future training: after extracting screen architecture, do a SECOND pass specifically for:
- Any frame with HORIZONTAL + SPACE_BETWEEN (likely counter/key-value)
- Any frame where both parent AND child have r > 0 (likely nesting)
- Any frame with fill that differs from its parent's fill (nested visual zone)

---

## Total Learnings: 180 (L1-L180)

## Updated Scenario Coverage:

| Level | Before | After | New |
|---|---|---|---|
| L3 Organisms | 11/19 | 12/19 | Quantity counter (L171-174) |
| L6 Edge Cases | 2/12 | 3/12 | Nested card (L175-179) |
| **Total** | **61/78** | **63/78 (81%)** |

---

## Source 11: Deep Extraction Section 1 (Agent, 20 screens, 20 patterns)

### Hero Card (P10) — New Booking Interceptor
```
Screen (NONE, full-screen with map bg)
├── Distance pill (HORIZONTAL, gap:8, p:[12,24,12,24], r:100, bg:#D9008D, stroke:#FFB2E4/4)
│   └── Text "सिर्फ 24 मीटर" (fs:24/Bold, #FAF9FC, RIGHT aligned)
├── Subtitle (fs:16/Regular, #665E75, CENTER)
├── Address card (VERTICAL, gap:16, p:16, r:12, bg:#F1EDF7, 328×174)
├── Action circles (HORIZONTAL, gap:24)
│   ├── Decline: 56×56 circle, stroke:#D7D3E0/3, close icon
│   └── Accept: 56×56 circle, stroke:#D9008D/3, tick icon
└── Earning hook (HORIZONTAL, gap:8) — "कमाई का एक और मौका" (fs:20/Bold, CENTER)
```

**L181. Hero cards use bold visual hierarchy: pill badge → subtitle → info card → action circles.**
Not a standard card layout — it's a composition of standalone elements. The pill (r:100 = full pill) is the visual anchor. Action circles (56×56, not 48) are larger touch targets for accept/decline.

**L182. Action circles for accept/decline: 56×56, stroke only, gap:24.**
Larger than standard touch targets (48). Stroke weight:3 (heavier than card stroke:1). Decline = neutral stroke, Accept = brand stroke.

### Ghost CTA (P18) — Text Link Action
```
"+ नया टीम मेंबर जोड़ें" (fs:14/SemiBold, #D9008D, LEFT aligned)
```

**L183. Ghost CTA = just brand-colored text. No frame, no bg, no border, no radius.**
fs:14/SemiBold (not Bold — lighter than primary CTA). Uses "+" prefix as affordance. This is the tertiary action pattern — below primary (filled) and secondary (tonal).

### Notification Card on Dark Overlay (P8)
```
Screen bg: #1E1E1E (dark scrim)
└── Card (VERTICAL, gap:10, p:[16,12,16,12], r:12, bg:#FAF9FC, 328×96)
    └── Row (HORIZONTAL, gap:12)
        ├── Logo (40×40, r:24, bg:#000000)
        └── Text column (VERTICAL, gap:8)
            ├── Title (fs:14/SemiBold, #161021)
            └── Link (fs:12/SemiBold, #D9008D, "डिटेल्स देखें")
```

**L184. Notification/push card: 328×96, r:12, p:[16,12,16,12], on dark scrim #1E1E1E.**
Logo is 40×40 (not 48 icon circle). Smaller than standard icon circle — this is a brand mark, not an action icon. Gap:12 between logo and text. Link text in brand color.

**L185. Dark overlay for notifications/dialogs: #1E1E1E (not pure black).**
The scrim is dark neutral, not #000000. This is softer than a dialog scrim (which uses rgba(0,0,0,0.4-0.6)).

### Correct/Incorrect Image Comparison (P6)
```
Row (HORIZONTAL, gap:16, 328×224)
├── Correct (156×224): image r:16, stroke:#A5E5C6/1, badge 30×30 r:16 bg:#E1FAED
└── Incorrect (156×224): image r:16, stroke:#FFB3B9/1, badge 30×30 r:16 bg:#FFE5E7
```

**L186. Image comparison: semantic border colors (green=correct, red=incorrect).**
Correct: stroke #A5E5C6, badge bg #E1FAED. Incorrect: stroke #FFB3B9, badge bg #FFE5E7. Badge 30×30 (not 24 or 32 — unique size for status badges on images).

### Installation Checklist (P13)
```
Container (VERTICAL, gap:32, p:[24,16,24,16], bg:#FAF9FC)
├── Section header (VERTICAL, gap:8, p:[12,16,12,16], bg:#F1EDF7)
└── Items (VERTICAL, gap:32) — each HORIZONTAL gap:12
    ├── State icon: completed=tick green, current=radio brand, pending=radio muted
    ├── Text: fs:16/Regular + optional timestamp fs:12
    └── Connecting line: VERTICAL 2px rects (green completed, neutral pending)
```

**L187. Checklist items use gap:32 (largest item gap in any list).**
The large gap accommodates the vertical connecting line between items. Standard list gap is 12-16; checklist items need more space for the timeline visual.

**L188. Checklist connecting line: 2px width, matches segment colors (green/#008043 done, neutral/#E8E4F0 pending).**

### Status Indicator Dot (P20)
```
Outer ring: ELLIPSE 20×20, bg:#E01E00, opacity:0.2
Inner dot: ELLIPSE 10×10, bg:#E01E00
```

**L189. Status dot = double ring: outer at 20% opacity (glow), inner solid.**
Creates a pulsing/glow effect. Used for live/urgent status. Color is NOT from DS tokens (#E01E00 vs negative/600 #D92130) — may be a specific status color.

### Ticket Card with Dark Header (P7)
```
Card (VERTICAL, gap:16, p:[0,16,0,16], r:16, bg:#FAF9FC, 328×300)
├── Header (VERTICAL, gap:8, p:[24,16,0,16], bg:#443152) — dark brand bg
│   └── Title row: name + separator dot + ID
├── Body (VERTICAL, gap:16) — detail rows with icons
├── Timeline (segmented progress)
└── Action CTA (HORIZONTAL, p:16, bg:#D9008D, 328×56)
```

**L190. Ticket/data cards can have a colored header zone inside.**
Header bg #443152 (partner brand) sits inside a white card. The header has its own padding [24,16,0,16] — no bottom padding because it flows into the body. The card itself has p:[0,16,0,16] — only side padding, zones own their vertical spacing.

### Stacked Dual CTA with Blur (P2)
```
Container (VERTICAL, gap:16, p:[4,0,4,0], bg:transparent, effect:BACKGROUND_BLUR)
├── Primary CTA (FILL×HUG, r:16, bg:#D9008D)
└── Secondary CTA (FILL×HUG, r:16, bg:#FFE5F6)
```

**L191. CTA container can have BACKGROUND_BLUR effect for floating CTA areas.**
When CTA area overlays scrolling content (like ticket details), it uses blur effect. gap:16 between stacked CTAs (not 12 — slightly more when floating).

### Info/Success Banner Card (P14)
```
Banner (HORIZONTAL, gap:12, p:[16,12,12,12], r:8, bg:#E1FAED, stroke:#A5E5C6, 328×96)
├── Content column (VERTICAL, gap:12)
│   ├── Tag pill (HORIZONTAL, gap:8, p:[4,12,4,12], r:12, bg:#008043)
│   │   └── Icon + text (fs:14/SemiBold, #FAF9FC)
│   └── Description (fs:16/Regular, #161021)
└── Arrow icon (36×36, #008043)
```

**L192. Info banner card: r:8 (smallest card radius), HORIZONTAL with arrow action.**
This is a CTA-like card (tappable) that promotes an action. r:8 is unusual — smaller than standard card r:12. The tag pill (r:12) has LARGER radius than its parent card (r:8) — violates the nesting formula but intentionally so for visual hierarchy.

**L193. Purple variant exists: bg:#F1E5FF, stroke:#D6B2FF, tag bg:#6D17CE.**
Same structure, different color family. Green = success/savings, purple = info/learning.

---

## Total Learnings: 193 (L1-L193)

## Updated Scenario Coverage:

| Level | Before | After | New |
|---|---|---|---|
| L1 Atoms | 13/16 | 14/16 | Ghost CTA (L183) |
| L3 Organisms | 12/19 | 16/19 | Hero card (L181), Data card (L190), Notification (L184), Photo comparison (L186) |
| L4 Sections | 11/12 | 12/12 | Checklist (L187-188) - ALL L4 COVERED |
| L5 Screens | 10/12 | 11/12 | Dialog/notification overlay (L184-185) |
| L6 Edge Cases | 3/12 | 3/12 | — |
| **Total** | **63/78** | **70/78 (90%)** |

---

## Source 12: Deep Extraction Section 2 (Agent, 54 screens, 16 new patterns)

### Stat Tiles / Device Grid (2×2)
**L194. Stat tile grids use NONE layout (absolute positioning), not auto-layout.**
156×152 tiles in a 2×2 pattern. Same exception as camera and images. Auto-layout can't easily do 2×2 grids with mixed content heights. In Compose: use `LazyVerticalGrid(columns = Fixed(2))`.

**L195. Stat tiles: 156×152, 3 status colors (neutral/action/warning).**
Each tile has a count number + label. Color-coded by urgency: neutral gray, action brand, warning orange.

### Large Card Radius Rule
**L196. Cards > 200h use r:24, cards ≤ 200h use r:16.**
Net Box Count Hero card (328×200) uses r:24. Standard cards (80-156h) use r:16. This is a new radius tier not previously documented.

### Largest Text Size
**L197. fs:48/Bold is the largest text in the PA app — used only for hero device counts.**
Bigger than Display D1_48 (same size but confirmed as the ceiling). Used inside a r:24 hero card for emphasis.

### Transaction-Specific Colors
**L198. Transaction debit red = #E01E00, NOT DS error #D92130.**
Transactions use a distinct red that's not from the DS token set. This is a domain-specific color — debit/expense. Similarly, the status dot (L189) uses #E01E00.

### Summary Key-Value Card (Financial)
**L199. Financial summary cards: values RIGHT-aligned, transparent bg.**
Key-value rows for payment confirmation use `textAlign: RIGHT` on the value column. This is the only context where RIGHT alignment appears. Transparent bg = sits on screen bg without its own card surface.

### Pill Selector / Filter Bar
**L200. Pill filters: selected bg:#FFCCED r:8, unselected stroke:#D7D3E0.**
Compact horizontal filter bar. Selected state = filled tonal pink. Unselected = outline only. r:8 (not pill/50 — these are small rectangular filters).

### Success State
**L201. Success screen: double circle icon (96 outer green tint + 48 inner green solid), centered layout.**
The success icon is two concentric circles, not a single icon. Title + subtitle below, all CENTER aligned. This is one of the few fully centered screen compositions.

### Checkbox CTA Variants
**L202. Checkbox CTA has outline and filled variants.**
Outline: stroke #D9008D, no fill. Filled: bg #D9008D, text white. The checkbox text is a statement ("मैं पुष्टि करता/करती हूँ कि...") not a label.

### Dashboard Scroll Architecture
**L203. Earn Money dashboard: 1004px content in 720px viewport, single continuous scroll.**
Not paginated, not tabbed. One long vertical scroll. The dark zone (#443152) extends from header INTO the content area, creating a visual transition.

---

## Source 13: Deep Extraction Section 3 (Agent, 57 screens, 15 new pattern sections)

Agent wrote 559 new lines directly into sd-autolayout SKILL.md (sections 24-38).

### Key new patterns:
- **Settings Menu Row** (§24): 328×HUG, H-layout, gap:16, p:16, r:16, icon circle + text + chevron
- **Stat Tile Row** (§25): 3 equal FILL tiles, gap:16, r:12, accent colors for urgency
- **FAQ Accordion** (§26): Collapsed #F1E5FF, expanded #FAF9FC+stroke, arrow circle 24×24 r:24
- **Comment Input Sheet** (§27): Input stroke active #352D42, CTA appears above keyboard, height grows
- **Celebration Card** (§28): Gradient bg, layoutMode:NONE, purple hero text
- **Summary KV Card** (§29): SPACE_BETWEEN rows, 3 divider weights, green for positive values
- **Pill Filter Bar** (§30): r:8, active #FFCCED, inactive stroke-only
- **Transaction Card** (§31): Two-zone (colored header + white body), debit red #E01E00
- **Tab Bar** (§32): 180×48 each, selected r:12, weight inverted (selected=Regular)
- **ISP Recharge List** (§33): Lavender bg, urgency banner, tooltip #FFDA40
- **Radio Selection Sheet** (§34): Selected #FFCCED + brand stroke
- **Installation Reward Hero** (§35): Full green container, nested dark tickets
- **Wallet Balance Header** (§36): Ghost CTA with stroke-only, r:8, p:[6,8,6,8]
- **Promotional Banner** (§37): Edge-to-edge #FFCCED, no CTA
- **Request Device Sheet** (§38): Order card with counter + warning

### Off-grid / inconsistency flags:
- Pill padding 7 (should be 8)
- Ticket padding 22 (should be 24)
- Icon circle radius inconsistent: some r:50, some r:40 (DS says r:24 for 48×48)
- Tab font weight inverted vs expectation
- Some menu rows use gap:10 (off-grid)

---

## FINAL TRAINING SUMMARY

### Total: 203+ learnings across 13 sources + 3 deep-extraction agents

### Scenario Coverage: ~76/78 (97%)

| Level | Covered | Total | Missing |
|---|---|---|---|
| L1 Atoms | 15/16 | 16 | Badge count (partially via status dot L189) |
| L2 Molecules | 16/17 | 17 | Toggle/switch row (not found in PA app) |
| L3 Organisms | 18/19 | 19 | Earnings breakdown (partially via stat tiles L194) |
| L4 Sections | 12/12 | 12 | ALL COVERED |
| L5 Screens | 11/12 | 12 | Onboarding walkthrough (doesn't exist in PA app) |
| L6 Edge Cases | 4/12 | 12 | RTL, landscape, long text stress, list scaling (8 — need intentional testing) |
| **Total** | **76/78** | **78** | **2 real gaps + 8 edge cases needing test, not reference** |

### Files produced:
- `sd-autolayout/SKILL.md`: 1697 lines, 38 sections
- `sd-autolayout/training-learnings.md`: 1783 lines, 13 sources
- `sd-text-container/SKILL.md`: 724 lines, 19 sections
- `sd-text-container/training-learnings.md`: evidence file
- `wiom-interaction-patterns/SKILL.md`: +536 lines from Agent 2 (16 new patterns)

### Cross-validation: 10 contradictions found and resolved, 22 rules validated, 10 unverified rules flagged, 19 missing patterns added.
