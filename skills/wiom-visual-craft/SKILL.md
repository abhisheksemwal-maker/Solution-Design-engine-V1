# Wiom Visual Craft — Solution Design Engine V1

The visual design bible for Wiom products. Decides HOW screens look — which tokens, what hierarchy, what elevation, what polish — so the output feels Uber/Blinkit-level, not "functional but generic."

**Output format:** Compose-ready. Every rule maps to `Dimens.*`, `Color.kt`, `MaterialTheme.typography.*`, or specific Compose modifiers.

---

## Who Uses This

- **Tech** styling screens: "Which type style for this label? What shadow? Is this hierarchy right?"
- **Product** reviewing visual output: "Does this feel premium? Is the density right for the persona?"

This skill bridges the gap between "tokens applied correctly" and "this feels world-class." The Wiom Design System (`wiom-design-system.md`) tells you WHAT tokens exist. This skill tells you WHEN and HOW to use each one.

---

## The Three Wiom Users — Visual Implications

### Annu Bhaiyya (CSP/Partner App)
- **Density:** SPACIOUS. More whitespace, fewer elements per screen, bigger touch targets
- **Hierarchy:** ONE focal point per screen — the thing he should do next. Everything else recedes
- **Numbers:** amounts, counts, earnings are always the LARGEST text when they're the point
- **Trust signals:** visual proof matters — show checkmarks, green states, actual numbers

### Technician Rohit (Expert/Partner App)
- **Density:** EFFICIENT. More items per screen is acceptable — he's in a sequential flow
- **Hierarchy:** progress indicator + current step is always prominent. "Where am I in the flow?"
- **Status:** clear step completion states — done (green), current (brand), pending (neutral)
- **Speed:** minimal decoration. Function over form. Every pixel serves the task

### Verma Parivar (Customer App)
- **Density:** MINIMAL. Conversational, chat-based, one thing at a time
- **Hierarchy:** Vyom bot message is the anchor. User options are secondary
- **Warmth:** rounded, soft, approachable. No sharp edges, no dark heavy elements
- **Recharge metaphor:** always frame in terms of "₹X = Y days of net"

---

## 1. Typography Application

### The 22 Wiom Type Styles — When to Use Each

**Rule: MAX 3 type styles per visual section.** If you need a 4th, you have a hierarchy problem — simplify.

| Context | Style | Compose constant | When |
|---------|-------|-------------------|------|
| Screen title (app bar) | `T3_20_Bold` | `fontSize = 20.sp, fontWeight = FontWeight.Bold` | Every screen's top bar title |
| Section header | `T4_16_Bold` | `fontSize = 16.sp, fontWeight = FontWeight.Bold` | Card titles, section dividers, form group labels |
| Hero number | `T2_24_Bold` or `T1_32_Bold` | `fontSize = 24.sp` or `32.sp, Bold` | Earnings amount, OTP display, key metric — the ONE number that matters |
| Body text | `B2_14_Regular` | `fontSize = 14.sp, fontWeight = FontWeight.Normal` | Descriptions, explanations, dialog body, card content |
| Featured body | `B1_16_Regular` | `fontSize = 16.sp, fontWeight = FontWeight.Normal` | Primary content that needs more emphasis than standard body |
| Emphasized body | `B2_14_Semibold` | `fontSize = 14.sp, fontWeight = FontWeight.SemiBold` | Key information within body text — amounts inline, important labels |
| CTA label (standard) | `T4_16_Bold` | `fontSize = 16.sp, fontWeight = FontWeight.Bold` | Full, Half, Dialog CTA buttons |
| CTA label (small) | `L2_14_Semibold` | `fontSize = 14.sp, fontWeight = FontWeight.SemiBold` | Special Small CTA, Coupon CTA |
| Chip/tag label | `L3_12_Semibold` | `fontSize = 12.sp, fontWeight = FontWeight.SemiBold` | Status chips, badges, category tags |
| Secondary info | `B3_12_Regular` | `fontSize = 12.sp, fontWeight = FontWeight.Normal` | Timestamps, secondary labels, helper text, captions |
| Tertiary/hint | `B4_10_Regular` | `fontSize = 10.sp, fontWeight = FontWeight.Normal` | Version numbers, legal footnotes, subtle metadata |
| Navigation label | `L2_14_Regular` | `fontSize = 14.sp, fontWeight = FontWeight.Normal` | Menu items, nav drawer items, settings row labels |

### Font Family Rules
```kotlin
// Detect Hindi text and apply correct font family
val fontFamily = if (text.contains(Regex("[\\u0900-\\u097F]"))) {
    NotoSansDevanagari // Hindi/Devanagari
} else {
    NotoSans // English/Latin
}
```
- **Hindi text:** ALWAYS `Noto Sans Devanagari`. Never Inter, never default system font
- **English text:** ALWAYS `Noto Sans`. Inter is for Figma prototyping only, not production
- **Mixed text (Hindi + English in same string):** Use `Noto Sans Devanagari` — it has Latin fallback glyphs
- **Numbers within Hindi text:** same font family as the surrounding text

### Typography Anti-patterns
- Text size not in the scale (no 13.sp, 15.sp, 18.sp, 22.sp — use ONLY the defined sizes)
- Using `Bold` when the style calls for `SemiBold` or vice versa
- Center-aligning body text (center ONLY for isolated display text, empty states, or single-line hero numbers)
- More than 3 type styles in one visual section
- Using `T1_32_Bold` or `D1_48_Bold` for anything other than a hero number or display moment

---

## 2. Color Token Application

### When to Use Each Token

| Token | Hex | USE for | NEVER use for |
|-------|-----|---------|---------------|
| `brand/600` | `#D9008D` | ONE primary CTA per screen, active tab indicator, links, brand-colored icons, toggle ON state | Decorative backgrounds, section fills, body text, multiple CTAs on same screen |
| `brand/secondary` | `#443152` | Partner app header (status bar + app bar). That's it | Body content, cards, text, buttons |
| `neutral/900` | `#161021` | Primary text, headings, key labels, hero numbers | Backgrounds, large areas of fill |
| `neutral/800` | `#352D42` | Dark icon circle backgrounds, dark-mode elements | Body text (too dark, use 900 for text) |
| `neutral/700` | `#665E75` | Secondary text, supporting info, non-primary labels | Primary text, headings, CTAs |
| `neutral/500` | `#A7A1B2` | Hint text, placeholder text, disabled text, version strings, icons in muted state | Active text, any text the user must read to proceed |
| `neutral/300` | `#D7D3E0` | Borders, dividers — ONLY when whitespace alone isn't enough | Backgrounds, text, icons |
| `neutral/200` | `#E8E4F0` | Subtle section backgrounds, stroke for inputs/cards | Heavy borders, text |
| `neutral/100` | `#F1EDF7` | Icon circle backgrounds (lavender), light section differentiation, tag backgrounds | Main screen background, card backgrounds |
| `neutral/white` | `#FAF9FC` | Page background, card background, input field background | It's the base — don't "use" it, everything sits on it |
| `positive/600` | `#008043` | Success states ONLY: completion checkmarks, "चालू है" status, confirmed states | Decorative green, anything not representing actual success |
| `negative/600` | `#D92130` | Error states ONLY: form errors, failed states, destructive button text | Decorative red, warnings (use warning/600 for those) |
| `warning/600` | `#FF8000` | Warning states: low stock, approaching deadline, attention needed | Errors (use negative), decorative orange |
| `info/600` | `#6D17CE` | Instructional content, PayG system info, educational callouts, feature tips | CTAs (use brand/600), body text, decoration |

### Color Rules
- **Never** `#000000` or `#FFFFFF` — always use the neutral scale
- **Background hierarchy:** `neutral/white` (base) → `neutral/100` (section) → `neutral/200` (nested). Never random grays
- **Status indicators** use color + icon + text — never color alone (accessibility)
- **Dark backgrounds** (`brand/secondary`, `neutral/800`): text must be `neutral/white` or `#FAF9FC`
- **Light backgrounds** (`neutral/white`, `neutral/100`): text must be `neutral/900` or `neutral/700`

### Compose Color.kt Pattern
```kotlin
object WiomColors {
    // Brand
    val BrandPrimary = Color(0xFFD9008D)
    val BrandSecondary = Color(0xFF443152)
    val BrandTonal = Color(0xFFFFE5F6)     // Secondary CTA fill, light pink tint

    // Neutral scale
    val Neutral900 = Color(0xFF161021)
    val Neutral800 = Color(0xFF352D42)
    val Neutral700 = Color(0xFF665E75)
    val Neutral500 = Color(0xFFA7A1B2)
    val Neutral300 = Color(0xFFD7D3E0)
    val Neutral200 = Color(0xFFE8E4F0)
    val Neutral100 = Color(0xFFF1EDF7)
    val NeutralWhite = Color(0xFFFAF9FC)

    // Semantic
    val Positive600 = Color(0xFF008043)
    val Negative600 = Color(0xFFD92130)
    val Warning600 = Color(0xFFFF8000)
    val Info600 = Color(0xFF6D17CE)

    // Semantic card/icon backgrounds (from PA flows)
    val SuccessLight = Color(0xFFC9F0DD)   // Success dialog icon bg, success card tint
    val SuccessLightAlt = Color(0xFFE1FAED) // Alternative success icon bg
    val WarningLight = Color(0xFFFFE6CC)   // Warning dialog icon bg
    val WarningCardBg = Color(0xFFFFE9E5)  // Warning stat card, stk:#FF8000
    val InfoLight = Color(0xFFE4CCFF)      // Education section headers
    val InfoBorder = Color(0xFFD6B2FF)     // Education video preview border
    val PinkCardStroke = Color(0xFFFFB2E4) // Pink stat card stroke

    val DialogBg = Color(0xFFFAF9FC)       // Dialog background

    // Selection states
    val SelectedCardBg = Color(0xFFFFCCED)  // Radio-selected card background
    val SelectedCardStroke = BrandPrimary    // #D9008D stroke on selected card

    // Notification CTA text
    val BrandPinkAlt = Color(0xFFD92B90)    // Slightly different brand pink used in some notification CTAs

    // Info tag
    val InfoTagBg = Color(0xFFF1E5FF)       // Info tag background in bottom sheets
    val InfoTagText = Color(0xFF6D17CE)      // Info tag text (info/600)

    // Disabled CTA
    val DisabledCta = Color(0xFFA7A1B2)     // Disabled CTA bg — neutral/500, NOT dimmed brand

    // Date/deadline (from Maverick corrections)
    val DeadlineOrange = Color(0xFFB85C00)  // Deadline dates — DISTINCT from Warning600 (#FF8000)

    // Camera/action circle buttons
    val ActionCircleBg = Color(0xFFE8E4F0)  // Dual action circle background (call, direction)

    // Hint card backgrounds
    val HintPink = Color(0xFFF9DFEE)        // OTP/input hint cards
    val HintLavender = Color(0xFFF1EDF7)    // Neutral hint/info cards (same as Neutral100)
}
```

### Semantic Card Backgrounds (from PA flows)
Cards use semantic background tints — not just neutral. The tint signals the card's category:

| Card type | Background | Stroke | When to use |
|-----------|-----------|--------|-------------|
| Default/info | `#F1EDF7` (Neutral100) | `#E8E4F0` (Neutral200) | Standard info cards, settings rows |
| Brand/action | `#FFE5F6` (BrandTonal) | `#D9008D` (BrandPrimary) | Active task card, highlighted action |
| Warning/attention | `#FFE9E5` | `#FF8000` (Warning600) | Stat cards needing attention |
| Success | `#C9F0DD` or `#E1FAED` | `#008043` (Positive600) | Completion states, confirmed items |
| Education/info | `#E4CCFF` | `#D6B2FF` | Education headers, learning content |

### Dialog Icon Backgrounds
Dialog icon circles use semantic tint backgrounds (not the generic Neutral100):

| Dialog type | Icon bg | Icon color | Icon size |
|------------|---------|------------|-----------|
| Success | `#C9F0DD` or `#E1FAED` | `#008043` (positive) | 48×48 or 96×96 |
| Warning/pending | `#FFE6CC` | `#352D42` (neutral/800) | 48×48 |
| Error | light red tint | `#D92130` (negative) | 48×48 |
| Info | `#F1EDF7` (neutral/100) | `#352D42` (neutral/800) | 48×48 |
```

---

## 3. Spacing System

### The 4px Grid — Compose Dimens

```kotlin
object WiomSpacing {
    val xs = 4.dp
    val sm = 8.dp
    val md = 12.dp
    val lg = 16.dp
    val xl = 24.dp
    val xxl = 32.dp
    val xxxl = 40.dp
    val section = 48.dp
    val page = 64.dp
}
```

### When to Use Each Value

| Value | Use for |
|-------|---------|
| `4.dp` | Label-to-input gap, icon-to-text within a tight row, fine adjustments |
| `8.dp` | Between related inline elements (icon + text in a chip), tight list item padding |
| `12.dp` | Card internal padding (small cards), between sub-items within a section |
| `16.dp` | Screen horizontal padding, card internal padding (standard), between form fields, between cards |
| `24.dp` | Between major sections on a screen, CTA bottom margin, section header to content |
| `32.dp` | Large section gaps, empty state top padding, between content and CTA area |
| `40.dp` | Hero section padding, large visual breaks |
| `48.dp` | Between major page divisions, status bar + app bar combined with content start |
| `64.dp` | Page-level whitespace, top margin for centered empty states |

### The Internal ≤ External Rule
**Spacing INSIDE a component must be LESS THAN OR EQUAL TO spacing between components.**

```
✅ Card padding: 16.dp, gap between cards: 16.dp (equal = OK)
✅ Card padding: 12.dp, gap between cards: 16.dp (internal < external = better)
❌ Card padding: 24.dp, gap between cards: 16.dp (VIOLATION — card feels disconnected)
```

This is the most important spacing rule. It creates unconscious visual grouping.

### Fixed Component Heights
| Component | Height | Why |
|-----------|--------|-----|
| Status bar | 44.dp | Android system |
| App bar | 56.dp | Wiom standard |
| CTA button | 48.dp | Touch target + visual weight |
| Icon circle | 48.dp | Touch target for icon actions |
| Tab bar | 48.dp | Bottom navigation |
| Input field | 48-56.dp | Comfortable touch target |
| Chip/tag | 28-32.dp | Inline with text |

---

## 4. Border Radius

### Consistent Across All Screens (from DS ground truth)

| Element | Radius | Compose |
|---------|--------|---------|
| Standard CTAs (Full, Half, Dialog) | 16.dp | `RoundedCornerShape(16.dp)` |
| Cards, inputs | 12.dp | `RoundedCornerShape(12.dp)` |
| Small CTAs (Special Small) | 12.dp | `RoundedCornerShape(12.dp)` |
| Nested elements inside cards | 8.dp | `RoundedCornerShape(8.dp)` |
| Coupon/inline CTAs | 8.dp | `RoundedCornerShape(8.dp)` |
| Dialogs | 24.dp | `RoundedCornerShape(24.dp)` |
| Bottom sheets, large containers | 16.dp top | `RoundedCornerShape(topStart = 16.dp, topEnd = 16.dp)` |
| Chips, tags, pills | 100.dp (full round) | `RoundedCornerShape(100.dp)` |
| Icon circles | 24.dp (half of 48.dp) | `CircleShape` or `RoundedCornerShape(24.dp)` |

### The Nesting Rule
**Children always have SMALLER radius than parent.**
```
Parent card: 12.dp radius
  → Child image area: 8.dp radius
    → Nested badge: 4.dp radius or CircleShape
```

### Radius Exceptions (from PA flows)
- **Hero cards** (large, prominent): use **24.dp** (not 12.dp) — e.g., My Devices top card 328×200
- **Triple dot dropdown menu**: uses **4.dp** (intentionally sharp)
- **Checkbox CTA** (outlined confirmation): uses **12.dp** (not 16.dp like standard CTAs)
- **Progress bar segments**: use **4.dp** on 2.dp height bars

---

## 5. Elevation and Shadows

### Shadow Hierarchy

| Level | Use | Compose |
|-------|-----|---------|
| **Level 0** | Flat on surface — section backgrounds, inline content | No shadow, no elevation |
| **Level 1** | Cards, list items — one step above the page | `shadowElevation = 1.dp` or `tonalElevation = 1.dp` |
| **Level 2** | Sticky headers, CTA surface — floating above content | `shadowElevation = 4.dp` |
| **Level 3** | Bottom sheets, modals — clearly floating | `shadowElevation = 8.dp` |
| **Level 4** | Dialogs — highest prominence | `shadowElevation = 16.dp` |

### Rules
- **Cards:** Level 1 (subtle) — `Card(elevation = CardDefaults.cardElevation(defaultElevation = 1.dp))`
- **CTA surface:** Level 2 — the CTA area at the bottom floats above scrolling content
- **Bottom sheet:** Level 3 — clearly separated from the page
- **Dialogs:** Level 4 — demands attention
- **Brand glow** (pink): `brand/600` at 15% opacity as shadow — ONLY on hero CTAs, one per screen max
- **Success glow** (green): `positive/600` at 15% opacity — completion moments ONLY

### Connected Elements
When a card has a "tip" or "footer" attached below it:
```kotlin
// Elevated card
Card(
    elevation = CardDefaults.cardElevation(defaultElevation = 2.dp),
    shape = RoundedCornerShape(12.dp),
    modifier = Modifier.zIndex(1f)
) { /* card content */ }

// Connected tip bar (overlaps behind the card)
Box(
    modifier = Modifier
        .offset(y = (-12).dp) // overlap into card's shadow
        .background(
            color = WiomColors.Neutral100,
            shape = RoundedCornerShape(bottomStart = 12.dp, bottomEnd = 12.dp)
        )
        .padding(top = 20.dp, start = 12.dp, end = 12.dp, bottom = 8.dp) // extra top for overlap zone
) { /* tip content */ }
```

---

## 6. Stroke and Border Rules

| Element | Border | When |
|---------|--------|------|
| Cards | `1.dp solid neutral/300` + shadow | Always BOTH — border for definition, shadow for elevation |
| Inputs (default) | `1.dp solid neutral/300` (#D7D3E0) | Standard resting state |
| Inputs (focused) | `1.dp solid neutral/800` (#352D42) | User is actively typing — NOT brand/600 |
| Inputs (success) | `1.dp solid positive/600` (#008043) | Validation passed |
| Inputs (error) | `1.dp solid negative/600` (#D92130) | Validation failed |
| Inputs (pre-filled) | `1.dp solid neutral/300`, field bg `neutral/300` | System-provided, non-editable |
| Inputs (disabled) | `1.dp solid neutral/300`, field bg `neutral/300` | Not yet available |
| Dividers within cards | `1.dp solid neutral/200` | Lighter than card border — subordinate separation |
| Section dividers | None — use whitespace | Prefer 24.dp gap over visible lines |
| Buttons | No border | Primary = fill, ghost = text only. Outlined only for secondary/destructive |

### Compose Border Pattern
```kotlin
// Card with border + elevation
Card(
    border = BorderStroke(1.dp, WiomColors.Neutral300),
    elevation = CardDefaults.cardElevation(defaultElevation = 1.dp),
    shape = RoundedCornerShape(12.dp)
) { /* content */ }

// Input field — DS-correct colors
OutlinedTextField(
    colors = OutlinedTextFieldDefaults.colors(
        focusedBorderColor = WiomColors.Neutral800,   // #352D42 — DS active state
        unfocusedBorderColor = WiomColors.Neutral300,  // #D7D3E0 — DS default
        errorBorderColor = WiomColors.Negative600,     // #D92130
        cursorColor = WiomColors.BrandPrimary          // #D9008D — brand cursor
    ),
    shape = RoundedCornerShape(12.dp) // DS: r:12 for input fields
)
```

### When NOT to Use Borders
- Buttons (use fill contrast or text-only treatment)
- Between sections on a page (use whitespace: 24.dp gap)
- Around images within cards (let the image bleed to card edges or use radius)
- Around icon circles (the background fill IS the boundary)

---

## 7. Visual Hierarchy — The Scan Test

Every screen must pass the **3-second scan test:** can the user identify what to do next within 3 seconds?

### Building Hierarchy (in order of impact)
1. **Size** — larger = more important. Hero numbers, screen titles
2. **Weight** — bolder = more emphasis. SemiBold for labels, Bold for headings
3. **Color** — `neutral/900` (primary) vs `neutral/700` (secondary) vs `neutral/500` (tertiary)
4. **Position** — top-left gets read first. CTA at bottom gets acted on
5. **Whitespace** — more space around = more importance. Hero elements get breathing room
6. **Elevation** — higher = more prominent. CTA surface floats above content

### Common Hierarchy Patterns

**Dashboard/Home screen (Annu):**
```
[App bar: brand/secondary, T3_20_Bold title]
[Hero card: earnings amount in T2_24_Bold, full width, Level 1 elevation]
[Section header: T4_16_Bold + "सब देखें" link]
[Task cards: B2_14 content, L3_12 status chips, 16.dp gaps]
[Bottom nav or CTA]
```

**Sequential flow screen (Rohit):**
```
[App bar: back arrow + T3_20_Bold step title]
[Progress indicator: step X of Y]
[Instruction: B1_16_Regular, info/600 background tint for PayG tips]
[Content: form fields or action area]
[Primary CTA: bottom-anchored, brand/600]
```

**Chat/conversational screen (Verma):**
```
[App bar: minimal]
[Chat bubbles: Vyom = neutral/100 bg, User = brand/600 bg + white text]
[Status chip: centered, L3_12_Semibold, semantic color]
[Quick actions: horizontal scroll row of OutlinedButton chips]
[Input area: bottom-anchored, text field + send button]
```

---

## 8. Icon Consistency

### Standard Icon Treatment
```kotlin
// Icon inside a circle (menu items, settings rows, dashboard tiles)
Box(
    modifier = Modifier
        .size(48.dp)
        .background(WiomColors.Neutral100, CircleShape),
    contentAlignment = Alignment.Center
) {
    Icon(
        imageVector = Icons.Default.Star,
        contentDescription = null,
        modifier = Modifier.size(24.dp),
        tint = WiomColors.Neutral800 // or specific semantic color
    )
}
```

### Icon Color Rules
| Context | Icon tint |
|---------|----------|
| Standard menu/navigation | `neutral/800` (#352D42) |
| Active/selected state | `brand/600` (#D9008D) |
| Interactive (phone, link) | `brand/600` (#D9008D) — Maverick correction #5 |
| Inside info cards | `neutral/500` (#A7A1B2) — NOT neutral/700. Maverick correction #13 |
| Muted/disabled | `neutral/500` (#A7A1B2) |
| On dark background | `neutral/white` (#FAF9FC) |
| Success context | `positive/600` (#008043) |
| Error context | `negative/600` (#D92130) |
| Instructional/info | `info/600` (#6D17CE) |
| Dual action circles | `neutral/800` (#352D42) on #E8E4F0 bg |

### Filled vs Outlined Icon Meaning (Maverick correction #4)
- **Filled icon** (`person`, `check_circle`): confirmed, active, selected state
- **Outlined icon** (`person_outline`, `info_outline`): informational, passive, unselected
- NEVER mix filled and outlined in the same list — pick one style for the set
- **Always check Figma** — filled vs outlined changes the meaning at small sizes

### Camera Shutter Button Spec
```
Outer ring: 64×64, stroke #E8E4F0 (neutral/200), 2dp stroke width
Inner fill: 54×54, bg #D9D9D9 (grey)
```
- NOT white, NOT pink — grey fill is intentional (neutral camera action)
- Stroke color matches card borders (#E8E4F0), not brand

### Rules
- **All icons in a set must use the same visual weight** — don't mix filled and outlined in the same list
- **24.dp inside 48.dp circle** — standard for all icon-in-circle patterns
- **App bar icons:** 24.dp, no circle, direct tint
- **CTA icons (leading):** 18-20.dp, same color as button text
- **Deadline dates:** use `#B85C00` (deadline orange), NOT brand pink or warning/600. Maverick correction #8

---

## 9. Micro-Polish — The Premium Delta

These details separate "functional" from "Uber/Blinkit-level."

### 1. Whitespace Rhythm
Repeating spacing patterns create unconscious order:
```
Section A: 24.dp gap below
  Card 1: 16.dp internal padding, 16.dp gap below
  Card 2: 16.dp internal padding, 16.dp gap below
Section B: 24.dp gap below
  Card 3: 16.dp internal padding
```
The rhythm is `24 → 16 → 16 → 24 → 16`. Consistent, predictable, calming.

### 2. Consistent Alignment
- ALL text within a screen left-aligned to the same 16.dp left margin
- Card content aligns to card padding, not to screen padding
- Numbers in a column right-aligned for easy comparison
- Never mix center and left alignment within the same section

### 3. Touch Target Audit
```kotlin
// Visually small element with proper touch target
IconButton(
    onClick = { /* action */ },
    modifier = Modifier.size(48.dp) // 48.dp touch target
) {
    Icon(
        imageVector = Icons.Default.MoreVert,
        modifier = Modifier.size(24.dp) // visually 24.dp
    )
}
```
- Every tappable element: minimum 48.dp touch area
- Spacing between tap targets: minimum 8.dp
- Bottom of screen = easiest thumb reach. Put primary actions there

### 4. Loading State Craft
```kotlin
// Skeleton that matches the actual card layout
@Composable
fun CardSkeleton() {
    Card(shape = RoundedCornerShape(12.dp)) {
        Column(modifier = Modifier.padding(16.dp)) {
            ShimmerBox(Modifier.fillMaxWidth(0.6f).height(20.dp)) // title width
            Spacer(Modifier.height(8.dp))
            ShimmerBox(Modifier.fillMaxWidth().height(14.dp)) // body line 1
            Spacer(Modifier.height(4.dp))
            ShimmerBox(Modifier.fillMaxWidth(0.8f).height(14.dp)) // body line 2
        }
    }
}
```
- Skeleton MUST match the actual content layout. Not a generic shimmer block
- Shimmer animation: 1500ms loop, linear easing (only exception to the no-linear-easing rule)

### 5. Transition Craft
- Elements entering: fade in + slide up 12.dp, 300ms, `FastOutSlowIn`
- Elements leaving: fade out, 200ms, `FastOutLinearIn`
- Staggered lists: 60ms delay between items
- **Never** animate layout changes (width, height changes). Animate `alpha` and `translationY` only

### 6. Status Indicator Pattern
```kotlin
Row(
    verticalAlignment = Alignment.CenterVertically,
    horizontalArrangement = Arrangement.spacedBy(6.dp)
) {
    Box(
        modifier = Modifier
            .size(8.dp)
            .background(statusColor, CircleShape)
    )
    Text(
        text = statusLabel,
        style = Label_L3_12_Semibold,
        color = statusColor
    )
}
```
- Colored dot (8.dp circle) + text label + icon if space allows
- NEVER color alone — always color + text (accessibility)

### 7. Card Action Positioning
- Actions at BOTTOM of card, right-aligned
- Or: entire card is tappable with `clickable` modifier (single-action cards)
- Never scatter actions throughout the card body

### 8. Progress Bar Craft
```kotlin
// Segmented progress: 2.dp height, gap:8, r:4
Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
    repeat(4) { i ->
        Box(Modifier.weight(1f).height(2.dp)
            .background(
                if (i < step) activeColor else WiomColors.Neutral200,
                RoundedCornerShape(4.dp)
            ))
    }
}
```
- Active color: `positive/600` for task flows, `brand/600` for education
- Use below header OR within content area — never inside the header itself

### 9. Tab Visual Craft
- Selected tab: `fs:14/SemiBold`, text color matches header (white on dark)
- Deselected tab: `fs:14/Regular`, same color, weight difference is the only indicator
- Bottom divider: `neutral/200` (#E8E4F0), 1.dp
- Tabs live INSIDE the header block (below status bar + header row, above content)

### 10. Illustration Circles
For action landing screens and education flows:
- Size: 120×120 dp, `CircleShape`
- Background: `#F1E5FF` (light purple/info tint) — for neutral/informational
- Background: `#C9F0DD` (light green) — for success/completion
- Background: `#FFE6CC` (light orange) — for warning/attention
- Icon/illustration centered inside, 52-60dp
- Used as visual anchor above title + subtitle in centered content layouts

### 11. Notification Card
```
328×108, bg:#FAF9FC, r:12, p:[16,12,16,12]
  Logo: 40×40, r:24 (circular)
  Title: fs:16/Bold #161021, wraps to 2 lines max
  Timestamp: below title, smaller text
```
- Compact padding (12.dp vertical vs 16.dp horizontal)
- Logo is always circular (r:24 on 40×40)
- On dark background (#1E1E1E) when shown as system notification overlay

---

## 10. Micro-Interaction & Motion Design

Motion is what makes apps feel alive. Every animation must answer: "what changed and why?"

### Easing Curves (Compose)

| Curve | Compose spec | Use |
|-------|-------------|-----|
| **Standard (decelerate)** | `FastOutSlowInEasing` | Elements entering view — fast start, gentle landing |
| **Exit (accelerate)** | `FastOutLinearInEasing` | Elements leaving view — starts visible, accelerates out |
| **Standard (symmetric)** | `LinearOutSlowInEasing` | Elements repositioning within view |
| **Spring** | `spring(dampingRatio = 0.7f, stiffness = 300f)` | Playful elements — toggle switches, success checkmarks, pull-to-refresh |
| **Linear** | `LinearEasing` | ONLY for: progress bars, shimmer loops, loading spinners |

### Duration Scale

| Element | Enter | Exit | Why |
|---------|-------|------|-----|
| Button press feedback | 100ms | 80ms | Must feel instant |
| Toggle/switch | 150ms | 150ms | Spring physics, symmetric |
| Snackbar/toast | 200ms | 150ms | Enter slower to be noticed, exit fast |
| Bottom sheet | 300ms | 250ms | Substantial surface, needs weight |
| Dialog | 250ms | 200ms | Scale + fade, moderate weight |
| Screen transition | 300ms | 250ms | Slide, must feel responsive |
| Skeleton shimmer | 1500ms loop | — | Continuous, linear |
| Stagger between list items | 50-60ms delay | — | Creates cascade effect |

**Rule: Exit is ALWAYS faster than enter.** The user initiated the exit — honor that with speed.

### Choreography Patterns

**Screen enter (forward navigation):**
```kotlin
fadeIn(tween(300, easing = FastOutSlowInEasing)) +
slideInHorizontally(
    initialOffsetX = { it / 4 }, // 25% offset, not 100% — subtle
    animationSpec = tween(300, easing = FastOutSlowInEasing)
)
```

**Screen exit (back navigation):**
```kotlin
fadeOut(tween(200, easing = FastOutLinearInEasing)) +
slideOutHorizontally(
    targetOffsetX = { it / 4 },
    animationSpec = tween(200, easing = FastOutLinearInEasing)
)
```

**List item stagger:**
```kotlin
items.forEachIndexed { index, item ->
    AnimatedVisibility(
        visible = true,
        enter = fadeIn(tween(300, delayMillis = index * 50)) +
                slideInVertically(
                    initialOffsetY = { 24 }, // 24dp upward slide
                    animationSpec = tween(300, delayMillis = index * 50)
                )
    ) { ItemCard(item) }
}
```

**Dialog enter:**
```kotlin
scaleIn(
    initialScale = 0.92f, // barely smaller, not dramatic
    animationSpec = tween(250, easing = FastOutSlowInEasing)
) + fadeIn(tween(250))
```

**Bottom sheet enter:**
```kotlin
slideInVertically(
    initialOffsetY = { it }, // full height from bottom
    animationSpec = tween(300, easing = FastOutSlowInEasing)
)
```

### State Transitions

**CTA disabled → enabled:**
```kotlin
animateColorAsState(
    targetValue = if (isValid) WiomColors.BrandPrimary else WiomColors.DisabledCta,
    animationSpec = tween(200, easing = FastOutSlowInEasing)
)
```
This is the moment the CTA "lights up" — it should feel like a reward for completing input.

**Success checkmark:**
```kotlin
// Scale spring — bouncy entrance for celebration
val scale by animateFloatAsState(
    targetValue = if (success) 1f else 0f,
    animationSpec = spring(dampingRatio = 0.6f, stiffness = 400f)
)
```

**Progress bar fill:**
```kotlin
animateFloatAsState(
    targetValue = progress,
    animationSpec = tween(600, easing = FastOutSlowInEasing) // slow enough to be visible
)
```

**Skeleton shimmer:**
```kotlin
val shimmerTranslate = rememberInfiniteTransition()
val translateX by shimmerTranslate.animateFloat(
    initialValue = -1f,
    targetValue = 2f,
    animationSpec = infiniteRepeatable(
        tween(1500, easing = LinearEasing),
        RepeatMode.Restart
    )
)
// Apply as gradient offset on shimmer Box
```

### Motion Anti-Patterns
- **Never linear easing** for UI elements (only progress/shimmer)
- **Never animate layout** (width, height) — animate `alpha`, `translationY`, `scale` only
- **Never delay beyond 60ms per stagger step** — cascade becomes sluggish
- **Never animate on back navigation** beyond a fast fade — user wants to go back NOW
- **Respect `prefers-reduced-motion`** — skip non-essential animations, keep functional transitions (screen changes)

### Reference-Quality Motion (what to aim for)
- **Linear app:** enter animations on lists, spring physics on interactions, no gratuitous motion
- **Stripe:** subtle parallax on scroll, delicate card hover, functional loading states
- **iOS HIG:** consistent 300ms screen transitions, spring-based sheet drag, physics-respecting gestures

---

## 11. Responsive Width Handling

Design canvas is 390px but must work 320dp–412dp+.

### Width Strategy
```kotlin
// Cards: full width minus padding
Card(modifier = Modifier
    .fillMaxWidth()
    .padding(horizontal = 16.dp)
)

// CTAs: full width inside padded container
Button(modifier = Modifier
    .fillMaxWidth()
    .height(48.dp)
)

// Text: never fixed width — always fillMaxWidth or wrapContentWidth
Text(
    text = longText,
    modifier = Modifier.fillMaxWidth(),
    overflow = TextOverflow.Ellipsis,
    maxLines = 2
)

// Dialog: constrained max width
AlertDialog(
    modifier = Modifier.widthIn(max = 312.dp)
)
```

### Rules
- **Never** hardcode pixel widths for text or content containers
- **Always** use `fillMaxWidth()` + horizontal padding for cards and CTAs
- `Text` wrapping: use `maxLines` + `TextOverflow.Ellipsis` for names and dynamic content
- Test at 320.dp: nothing should overflow or clip
- Test at 412.dp: nothing should look stretched or have excessive whitespace

---

## Design Critique & Quality Audit (Maker-Checker Gate)

Run this BEFORE declaring any screen shippable. Three audit layers — pixel, UX, and polish. If any Critical item fails, the screen is NOT done.

### Layer 1: Pixel-Level Audit

**Alignment Grid:**
- [ ] All text left edges align to the same 16.dp screen padding
- [ ] Card content aligns to card padding, not screen padding
- [ ] Numbers in columns right-aligned for comparison
- [ ] Icon centers align vertically within rows (check optical center, not mathematical)
- [ ] CTA horizontally centered with equal padding left and right
- [ ] No orphan elements floating outside the grid (check at 320.dp AND 412.dp)

**Spacing Consistency:**
- [ ] Every spacing value is from the 4.dp grid (no 10, 11, 15, 18, 22)
- [ ] Same component type has IDENTICAL spacing across all screens in the flow
- [ ] Internal padding ≤ external gap (the fundamental rule)
- [ ] Whitespace rhythm is repeating (24→16→16→24, not random)
- [ ] Gap between last content item and CTA is consistent across screens (24-32.dp)

**Typography Scale:**
- [ ] Every text node uses one of the 22 DS type styles (no off-scale sizes)
- [ ] MAX 3 type styles per visual section
- [ ] Hindi text uses Noto Sans Devanagari, English uses Noto Sans (never Inter in production)
- [ ] Weight hierarchy is correct: Bold for headings, SemiBold for labels, Regular for body
- [ ] No text clipping or overflow at 320.dp width

### Layer 2: UX Audit

**Cognitive Load Score** (count these — lower is better):
- [ ] Decisions required on this screen: ≤2 for Annu/Verma, ≤3 for Rohit
- [ ] Tap count to complete the primary action: ≤2 taps from screen entry
- [ ] Text to read before acting: ≤3 lines for primary path
- [ ] Visual elements competing for attention: ≤1 hero element per screen

**Tap-Count Analysis:**
- [ ] Primary action reachable in 1-2 taps from screen load
- [ ] Secondary actions: 2-3 taps max
- [ ] Back/escape: always 1 tap (back button or gesture)
- [ ] No action requires scrolling to discover (CTA always visible)

**Error Recovery:**
- [ ] Every possible error has an inline message (not just a toast)
- [ ] Error message says what happened + what to do (see ux-copy skill)
- [ ] Retry is available for all retriable errors (network, timeout)
- [ ] Form errors preserve all valid input (never clear the whole form)
- [ ] Recovery path is ≤2 taps from error state

**State Completeness:**
- [ ] Loading state exists (skeleton matching content layout)
- [ ] Empty state exists (icon + title + explanation + optional CTA)
- [ ] Error state exists (inline, persistent, actionable)
- [ ] Success state exists (if applicable — dialog or inline)
- [ ] Pre-filled / disabled states correct (grey bg, muted text)

### Layer 3: Visual Polish Audit

**Shadow Consistency:**
- [ ] Cards: Level 1 (1.dp) — all cards at same elevation within a screen
- [ ] CTA surface: Level 2 (4.dp) — floats above content
- [ ] Dialogs: Level 4 (16.dp) — highest prominence
- [ ] No orphan shadows (shadow on an element that shouldn't have elevation)
- [ ] Brand glow: ONLY on hero CTA, max 1 per screen

**Icon Optical Alignment:**
- [ ] Icons in a list are all same size (24.dp) and same visual weight
- [ ] Icon circles are true circles (48×48, r:24)
- [ ] Icons centered within circles (not mathematically, but OPTICALLY — some shapes need nudging)
- [ ] Filled vs outlined consistent within same context
- [ ] Icon color matches context (see icon color rules in Section 8)

**Color Harmony:**
- [ ] Only DS tokens used — zero hardcoded hex values
- [ ] Semantic colors correct: success=green, error=red, warning=orange, info=purple
- [ ] Background hierarchy maintained: white → neutral/100 → neutral/200 (never reversed)
- [ ] Brand pink (#D9008D) used for ONE primary element only (not scattered)
- [ ] Text contrast: primary (#161021) > secondary (#665E75) > hint (#A7A1B2) — clear hierarchy

**Micro-Polish (the Uber/Blinkit delta):**
- [ ] Transitions defined: enter (300ms decelerate), exit (200ms accelerate)
- [ ] CTA disabled→enabled transition animated (color change, 200ms)
- [ ] Success moment has spring animation (checkmark, scale bounce)
- [ ] Skeleton shimmer matches actual content layout
- [ ] Touch targets ≥ 48.dp, spacing between targets ≥ 8.dp
- [ ] Border-radius nesting rule followed (child < parent)
- [ ] Connected elements overlap correctly (negative margin + z-index)

### Audit Scoring

| Rating | Criteria |
|--------|---------|
| **Ship it** | All Critical + Important items pass. Polish items ≥80% |
| **Fix first** | Any Critical item fails. Must fix before deployment |
| **Needs design review** | 3+ Important items fail. Escalate to design team |

### Audit Report Format
```
DESIGN AUDIT — [Screen/Flow Name] — [Date]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PIXEL AUDIT:     [X/6] alignment + [X/5] spacing + [X/5] typography = [X/16]
UX AUDIT:        [X/4] cognitive + [X/4] tap-count + [X/5] error + [X/5] states = [X/18]
POLISH AUDIT:    [X/5] shadow + [X/5] icons + [X/5] color + [X/7] micro-polish = [X/22]

TOTAL: [X/56]
VERDICT: [Ship it / Fix first / Needs design review]

CRITICAL FIXES:
1. [Finding] — [Location] — [Fix]

IMPORTANT FIXES:
1. [Finding] — [Fix]

POLISH ITEMS:
1. [Finding] — [Fix]
```

---

## Checklist Before Declaring Visual Work "Done" (Quick Version)

- [ ] MAX 3 type styles per visual section?
- [ ] All font sizes from the defined scale (10, 12, 14, 16, 20, 24, 32, 40, 48)?
- [ ] Font family correct (Devanagari for Hindi, Noto Sans for English)?
- [ ] All colors from DS tokens (no hardcoded arbitrary hex)?
- [ ] `brand/600` used for ONE primary CTA only?
- [ ] Status colors correct (positive=green, negative=red, warning=orange)?
- [ ] Spacing on 4.dp grid? No 10, 11, 15, 18 values?
- [ ] Internal spacing ≤ external spacing on all components?
- [ ] Border radius consistent (16 for CTAs, 12 for cards/inputs, 8 for nested, 24 for dialogs)?
- [ ] Card has BOTH border + shadow?
- [ ] Elevation hierarchy logical (cards < CTA surface < sheets < dialogs)?
- [ ] All tappable elements ≥ 48.dp touch target?
- [ ] Text left-aligned (center only for display text, empty states)?
- [ ] Icons consistent weight and size within same context?
- [ ] Loading state = skeleton matching content layout?
- [ ] Whitespace rhythm consistent (repeating patterns, not random gaps)?
- [ ] Tested at 320.dp and 412.dp widths mentally?
- [ ] 3-second scan test: can [persona] identify what to do next?
- [ ] OTP boxes: individual 76×48 boxes (not single field), gap:8, r:12
- [ ] Summary cards: fs:13 for key-value pairs, new values in #008043
- [ ] Photo review: r:24 for large, r:12 for compact, retake overlay on photo
- [ ] Wallet balance: amount is fs:24/700, largest element in the card
- [ ] Step progress: current=brand filled+SemiBold, completed=tick+Regular, pending=muted
