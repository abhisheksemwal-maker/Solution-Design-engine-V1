# Solution Design Autolayout V1

Deterministic auto-layout rules. Every rule is binary: correct or incorrect.
No ranges. No "it depends." No judgment calls.

Two consumers, one truth:
- **Pratibimb** spawns this when creating layouts in Figma (Chrome DevTools MCP / figma-bridge)
- **Maverick** spawns this when building Kotlin/Compose APKs (wiom-frontend-dev)

If Figma and Compose diverge on the same screen, the build is wrong.

---

## 1. SIZING MODE DECISION TREE

Every node has exactly one width mode and one height mode.
Walk the tree top-to-bottom. First match wins.

### 1a. Width Mode

```
Is it an icon, avatar, or explicit graphic?
  Icon?                        → FIXED (24)
  Icon circle?                 → FIXED (48)
  Avatar circle?               → FIXED (32)
  Shutter button?              → FIXED (64)
  Image/photo container?       → FIXED (explicit w×h, e.g. 180×180)
  Graphic/illustration?        → FIXED (96×96 or spec'd size)

Is it an inline content element?
  Badge / tag / chip?          → HUG
  Price / number label?        → HUG
  Tooltip?                     → HUG (root), FIXED (inner text zone max-width)
  Coupon CTA?                  → HUG

Is it everything else?
  Screen root frame?           → FILL (device width, NOT a fixed pixel value)
  Status bar?                  → FILL
  App bar?                     → FILL
  Bottom sheet?                → FILL
  Tab bar?                     → FILL
  Dialog?                      → FILL (with 24dp margin each side on overlay)
  Step progress segment?       → FILL (equal distribution among siblings)
  OTP single box?              → FILL (equal distribution, renders ~76 at 360 base)
  Card?                        → FILL
  CTA button?                  → FILL
  Input field?                 → FILL
  Text (heading/body/label)?   → FILL
  Everything else?             → FILL
```

**Rule: FILL is the default.** You need a reason to use FIXED or HUG.
FIXED = icons, avatars, images, and explicit graphic elements with spec'd dp sizes.
HUG = inline content that sizes to its characters (badges, prices, chip labels, tooltips).
FILL = everything else. Width is a CONSEQUENCE of parent padding, not a specified value.

**Critical (L32-34): Figma DS base frame is 360px. ALL pixel widths in Figma (328, 312, 264) are derived from `parent_width - padding_chain`. In Compose: always use `fillMaxWidth()` + parent padding. Never hardcode container widths.**

### 1b. Height Mode

```
Status bar?                  → FIXED (24) — DS source component, NOT 44
App bar (header zone)?       → FIXED (56)
Input field (inner field)?   → FIXED (48)
Icon circle?                 → FIXED (48)
Icon?                        → FIXED (24)
Avatar circle?               → FIXED (32)
Step progress segment?       → FIXED (2)
Bottom sheet handle?         → FIXED (4)
Shutter button?              → FIXED (64)
Tab bar?                     → FIXED (48)
Dense menu item?             → FIXED (40)
Image/photo container?       → FIXED (explicit height, e.g. 180)
Scrollable content area?     → FILL (takes remaining space)
CTA button (any size)?       → HUG (height = padding + text line-height + padding)
                                Standard: 12 + 24 + 12 = 48
                                Small: 12 + 20 + 12 = 44
                                Coupon: 4 + 20 + 4 = 28
Screen root frame?           → HUG (in Figma, shows full content)
Card?                        → HUG
Dialog?                      → HUG
Bottom sheet?                → HUG (max 85% viewport, then scroll)
Section container?           → HUG
Text node (any)?             → HUG — NEVER FIXED, no exceptions
Everything else?             → HUG
```

**Rule: HUG is the default for height.** You need a reason to use FIXED or FILL.
FIXED = DS-specified constant for structural elements (status bar, app bar, icons, images).
FILL = element must absorb remaining space (scrollable body in Compose).
HUG = content-driven height (cards, CTAs, text, containers). CTA height is HUG — the 48dp is a consequence of padding+text, not a fixed value (L67).

### 1c. Platform Implementation

| Mode | Figma API | Compose |
|------|-----------|---------|
| FIXED width | `node.resize(value, node.height)` then `node.layoutSizingHorizontal = "FIXED"` | `Modifier.width(value.dp)` |
| FILL width | `node.layoutSizingHorizontal = "FILL"` | `Modifier.fillMaxWidth()` |
| HUG width | `node.layoutSizingHorizontal = "HUG"` | Default (no width modifier) |
| FIXED height | `node.resize(node.width, value)` then `node.layoutSizingVertical = "FIXED"` | `Modifier.height(value.dp)` |
| FILL height | `node.layoutSizingVertical = "FILL"` | `Modifier.weight(1f)` inside Column, or `Modifier.fillMaxHeight()` |
| HUG height | `node.primaryAxisSizingMode = "AUTO"` (if root) or `node.layoutSizingVertical = "HUG"` | Default (no height modifier) |

### 1d. Figma Ordering Law

This is not a guideline. Violating this order produces silent bugs.

```
Step 1: frame.layoutMode = "VERTICAL" (or "HORIZONTAL")
Step 2: frame.resize(W, tempH)              — sets FIXED width
Step 3: frame.counterAxisSizingMode = "FIXED" — locks width
Step 4: Set padding (paddingTop, paddingRight, paddingBottom, paddingLeft)
Step 5: Set itemSpacing (gap)
Step 6: Append ALL children via frame.appendChild(child)
Step 7: Set child.layoutSizingHorizontal = "FILL" — AFTER append, not before
Step 8: frame.primaryAxisSizingMode = "AUTO" — LAST, height hugs content
```

Why this order:
- `resize()` resets `primaryAxisSizingMode` to FIXED. Setting AUTO before resize is wasted.
- `layoutSizingHorizontal = "FILL"` before `appendChild()` throws: "node must be a child of auto-layout frame."
- `primaryAxisSizingMode = "AUTO"` before all children are appended → incorrect height calculation.
- (L100) For HORIZONTAL frames: `primaryAxisSizingMode = "AUTO"` overrides `layoutSizingHorizontal` to HUG. Must re-set `layoutSizingHorizontal = "FILL"` AFTER any primaryAxisSizingMode change on HORIZONTAL frames.
- For HORIZONTAL frames that need HUG height: set `counterAxisSizingMode = "AUTO"` (counterAxis = height for HORIZONTAL).

---

## 2. DIRECTION TABLE

Every auto-layout frame has exactly one direction. No exceptions.

| Component | Direction | Figma | Compose |
|-----------|-----------|-------|---------|
| Screen root | VERTICAL | `"VERTICAL"` | `Column` |
| App bar | HORIZONTAL | `"HORIZONTAL"` | `Row` |
| App bar — left section (icon+title) | HORIZONTAL | `"HORIZONTAL"` | `Row` |
| CTA area (1 or 2 buttons) | VERTICAL | `"VERTICAL"` | `Column` |
| Card | VERTICAL | `"VERTICAL"` | `Column` / `Card { Column }` |
| List row | HORIZONTAL | `"HORIZONTAL"` | `Row` |
| Form (stacked fields) | VERTICAL | `"VERTICAL"` | `Column` |
| Input field wrapper (label + field) | VERTICAL | `"VERTICAL"` | `Column` |
| Input field inner (icon + text + action) | HORIZONTAL | `"HORIZONTAL"` | `Row` inside `OutlinedTextField` |
| Tab bar | HORIZONTAL | `"HORIZONTAL"` | `TabRow` |
| Bottom sheet | VERTICAL | `"VERTICAL"` | `Column` inside `ModalBottomSheet` |
| Dialog | VERTICAL | `"VERTICAL"` | `Column` inside `AlertDialog` / `Dialog` |
| OTP row | HORIZONTAL | `"HORIZONTAL"` | `Row` |
| Step progress bar | HORIZONTAL | `"HORIZONTAL"` | `Row` |
| Radio/checkbox group | VERTICAL | `"VERTICAL"` | `Column` |
| Single radio/checkbox item | HORIZONTAL | `"HORIZONTAL"` | `Row` |
| Tag/chip group | HORIZONTAL | `"HORIZONTAL"` | `FlowRow` (wrapping) |
| Avatar overlap stack | HORIZONTAL | `"HORIZONTAL"` | `Row` (negative offset) |
| Stat tiles row | HORIZONTAL | `"HORIZONTAL"` | `Row` with equal `weight(1f)` |
| Side-by-side cards | HORIZONTAL | `"HORIZONTAL"` | `Row` with equal `weight(1f)` |
| Hero card | VERTICAL | `"VERTICAL"` | `Column` |
| Notification item | HORIZONTAL | `"HORIZONTAL"` | `Row` (icon + content) |
| Timeline item | HORIZONTAL | `"HORIZONTAL"` | `Row` (indicator + content) |
| Key-value summary row | HORIZONTAL | `"HORIZONTAL"` | `Row` (label + value, space-between) |
| Section (heading + content) | VERTICAL | `"VERTICAL"` | `Column` |

**Rule: NEVER use `layoutMode = "NONE"` for content layout.**
Absolute positioning (`Box` in Compose, `"NONE"` in Figma) is ONLY for:
- Overlays / scrims
- Floating action buttons
- Badge positioned on top of icon
- Camera viewfinder overlay

---

## 3. SPACING — EXACT VALUES

### 3a. Padding Table

Every context has one correct padding. Not a range.

| Context | Top | Right | Bottom | Left | Token |
|---------|-----|-------|--------|------|-------|
| Screen root (content area) | 0 | 16 | 0 | 16 | space-16 |
| App bar | 4 | 4 | 4 | 4 | space-4 |
| Card | 16 | 16 | 16 | 16 | card-padding |
| CTA area | 12 | 16 | 24 | 16 | mixed |
| Input field inner | 12 | 16 | 12 | 16 | mixed |
| Bottom sheet content | 24 | 16 | 48 | 16 | mixed (L95: extra bottom) |
| Dialog | 32 | 24 | 32 | 24 | mixed (L20) |
| List row | 14 | 14 | 14 | 14 | off-grid* |
| Chip / tag | 4 | 12 | 4 | 12 | mixed |
| Coupon CTA (inline) | 4 | 8 | 4 | 8 | L74: tighter micro-CTA |
| Bottom sheet action bar | 0 | 16 | 16 | 16 | L166: no top padding (content 48bp provides gap) |

*List row 14px is a known Wiom DS exception. Flag in audit but do not auto-correct.

### 3b. Gap Table (itemSpacing)

Every context has one correct gap.

| Context | Gap | Token |
|---------|-----|-------|
| Between screen sections | 24 | section-gap |
| Between cards in a list | 16 | space-16 |
| Between form fields | 12 | space-12 |
| Label to input field | 8 | space-8 |
| Icon to text (inline, horizontal) | 8 | space-8 |
| Between stacked CTAs | 12 | space-12 |
| App bar internal elements | 4 | space-4 |
| OTP boxes | 8 | space-8 |
| Step progress segments | 8 | space-8 |
| Inside card (between child elements) | 8 | space-8 |
| Inside card (between subsections) | 12 | space-12 |
| Between list rows | 0 | — (divider separates) |
| Between radio selection cards | 12 | space-12 (tighter than standard 16) |
| Content-to-CTA in overlay (dialog/sheet) | 48 | space-48 |
| Heading group (title→subtitle) | 8 | space-8 |
| Avatar overlap | -8 | — (negative) |
| Side-by-side cards | 16 | space-16 |
| Stat tiles | 8 | space-8 |
| Timeline/status zone internal | 6 | off-grid (compact info) |

### 3c. The Fundamental Rule

> **Internal padding of a component MUST BE ≤ external gap between sibling components.**

This is the single most important spacing rule. It creates unconscious visual grouping.

| Verdict | Internal padding | External gap |
|---------|-----------------|--------------|
| PASS | 16 | 16 | (equal) |
| PASS | 12 | 16 | (internal < external) |
| PASS | 8 | 12 | (internal < external) |
| **FAIL** | 24 | 16 | (internal > external) |
| **FAIL** | 16 | 12 | (internal > external) |

### 3d. Minimum Spacing Between Elements

| Axis | Minimum | Notes |
|------|---------|-------|
| Horizontal (between sibling elements in a Row) | 4px | Tighter than this = elements feel merged |
| Vertical (between sibling elements in a Column) | 8px | Tighter than this = text lines feel squished |

These are floor values. Most contexts require more (see §3b gap table).
A gap of 0 is only legal between list rows separated by a divider.

### 3e. Allowed Spacing Values (Grid)

The ONLY legal values for any padding, gap, or margin:

```
4  8  12  16  24  32  40  48  64  72  84
```

Any value not in this set is a FAIL. Common violations:

| Illegal value | Correct to |
|---------------|-----------|
| 10 | 8 or 12 |
| 11 | 12 |
| 14 | 12 or 16 (exception: Wiom list row) |
| 15 | 16 |
| 18 | 16 or 24 |
| 20 | 16 or 24 |
| 22 | 24 |

### 3f. Platform Implementation

| Property | Figma API | Compose |
|----------|-----------|---------|
| Padding (uniform) | `frame.paddingTop = frame.paddingBottom = frame.paddingLeft = frame.paddingRight = v` | `Modifier.padding(v.dp)` |
| Padding (asymmetric) | `frame.paddingTop = T; frame.paddingRight = R; frame.paddingBottom = B; frame.paddingLeft = L` | `Modifier.padding(start = L.dp, top = T.dp, end = R.dp, bottom = B.dp)` |
| Gap | `frame.itemSpacing = v` | `verticalArrangement = Arrangement.spacedBy(v.dp)` or `horizontalArrangement = Arrangement.spacedBy(v.dp)` |
| Gap vs margin rule | Gap applies uniformly to ALL children. If one child needs different spacing, use individual child padding. | Same — `spacedBy` is uniform. Use `Spacer(Modifier.height(v.dp))` for exceptions. |

---

## 4. ALIGNMENT

### 4a. Primary Axis Alignment (along the layout direction)

| Component | Align | Figma | Compose |
|-----------|-------|-------|---------|
| Screen root | TOP | `primaryAxisAlignItems = "MIN"` | `verticalArrangement = Arrangement.Top` (default) |
| Card | TOP | `"MIN"` | Default |
| Dialog | TOP | `"MIN"` | Default |
| Form | TOP | `"MIN"` | Default |
| App bar | CENTER | `"CENTER"` | `verticalAlignment = Alignment.CenterVertically` (on Row) |
| Empty state screen | CENTER | `"CENTER"` | `verticalArrangement = Arrangement.Center` |
| Bottom sheet | TOP | `"MIN"` | Default |
| Key-value row | SPACE_BETWEEN | `"SPACE_BETWEEN"` | `horizontalArrangement = Arrangement.SpaceBetween` |

### 4b. Cross Axis Alignment (perpendicular to layout direction)

| Component | Align | Figma | Compose |
|-----------|-------|-------|---------|
| Screen root (Column) | START | `counterAxisAlignItems = "MIN"` | `horizontalAlignment = Alignment.Start` (default) |
| Card (Column) | START | `"MIN"` | Default |
| Form (Column) | START | `"MIN"` | Default |
| App bar (Row) | CENTER | `"CENTER"` | `verticalAlignment = Alignment.CenterVertically` |
| List row (Row) | CENTER | `"CENTER"` | `verticalAlignment = Alignment.CenterVertically` |
| CTA area (Column) | STRETCH | `"MIN"` + children FILL | `horizontalAlignment = Alignment.CenterHorizontally` + `fillMaxWidth()` on buttons |
| Dialog (Column) | START | `"MIN"` | `horizontalAlignment = Alignment.Start` (L23: dialog content left-aligned) |
| OTP row (Row) | CENTER | `"CENTER"` | `verticalAlignment = Alignment.CenterVertically` |

### 4c. Text Alignment Rules

```
Is it body text, label, heading, or description?
  → LEFT (Start). No exceptions.

Is it an empty state message?
  → CENTER.

Is it a dialog title?
  → LEFT (Wiom DS uses left-aligned dialog titles, confirmed L23).

Is it a loading/processing message?
  → CENTER.

Is it text inside a CTA button?
  → CENTER (automatic — button centers its content).

Is it a price or number inside a badge?
  → CENTER.
```

**Rule: NEVER center body text or form labels.** Left-aligned text is faster to scan.

---

## 5. FIXED DIMENSIONS

Heights and icon sizes are constants. Widths are DERIVED (L32-34).
Only icons, avatars, images, and structural bars have absolute dp values.

| Element | Width | Height | Radius | Notes |
|---------|-------|--------|--------|-------|
| Screen frame | FILL (device) | HUG (Figma) | 0 | NOT a fixed pixel value |
| Status bar | FILL | 24 | 0 | L54: DS source = 24, not 44 |
| App bar (header zone) | FILL | 56 | 0 | |
| CTA primary | FILL | HUG (→48) | 16 | L67: 48 = p12 + text24 + p12 |
| CTA secondary | FILL | HUG (→48) | 16 | Same as primary |
| CTA small | FILL/HUG | HUG (→44) | 12 | L71: smaller radius |
| CTA coupon (inline) | HUG | HUG (→28) | 8 | L74: tightest CTA |
| Input field | FILL | 48 | 12 | Inner field: FIXED 48 |
| Card | FILL | HUG | 12 | |
| Icon circle | 48 | 48 | 24 | |
| Icon (inside circle) | 24 | 24 | 0 | |
| OTP box | FILL-equal | 48 | 12 | L4: FILL not FIXED 76 |
| Step progress segment | FILL-equal | 2 | 4 | |
| Avatar circle | 32 | 32 | 16 | |
| Bottom sheet handle bar | 120 (in padded zone) | 4 | 4 | |
| Dialog | FILL (24dp margin) | HUG | 24 | L21: width = viewport - 48 |
| Tab bar | FILL | 48 | 0 | |
| Shutter button | 64 | 64 | 32 | |
| Touch target minimum | 48 | 48 | — | Material |
| Bottom sheet | FILL | HUG (max 85%) | 24 top | |
| Dense menu item | 200 (dropdown) | 40 | — | L64: below 48 OK for dropdowns |
| Image container | FIXED (spec'd) | FIXED (spec'd) | 12 | L85: always NONE layout |
| Graphic/illustration | FIXED (96) | FIXED (96) | — | L99: LEFT-aligned |

---

## 6. SCROLL & OVERFLOW RULES

### 6a. Decision Tree

```
Total content height > available viewport?
  available = 844 - statusBar(24) - appBar(56) - ctaArea(~84) = 680
  │
  YES → Content area scrolls. CTA bar stays fixed at bottom.
  NO  → No scroll needed. Content and CTA in single Column.

Bottom sheet content > 85% of 844 (= 717)?
  │
  YES → Sheet content scrolls. Sheet height fixed at 717.
  NO  → Sheet hugs content. No scroll.

Text inside a card grows beyond expected?
  │
  → Card grows (HUG). Screen scrolls if needed. NEVER clip text.

List has more items than viewport?
  │
  → Use lazy list (LazyColumn). Header and CTA stay fixed.
```

### 6b. Screen Architecture Pattern

**Figma:** Build full-height frame (HUG). It will exceed 844px. This is correct — Figma shows the full scrollable content.

**Compose:**
```kotlin
Scaffold(
    topBar = { AppBar() },                    // FIXED, never scrolls
    bottomBar = { CTAArea() }                 // FIXED, never scrolls
) { innerPadding ->
    Column(
        modifier = Modifier
            .padding(innerPadding)
            .verticalScroll(rememberScrollState())  // SCROLLS
    ) {
        // All content here
    }
}
```

Or without Scaffold:
```kotlin
Column(modifier = Modifier.fillMaxSize()) {
    AppBar()                                        // FIXED
    Column(
        modifier = Modifier
            .weight(1f)                             // FILL remaining
            .verticalScroll(rememberScrollState())  // SCROLLS
    ) {
        // Content
    }
    CTAArea()                                       // FIXED at bottom
}
```

### 6c. Rules

| Rule | Verdict |
|------|---------|
| CTA inside scroll area | **FAIL** — CTA must be outside scroll, always visible |
| App bar inside scroll area | **FAIL** — app bar is fixed |
| Text node with `clip = true` or `overflow = hidden` | **FAIL** — text must never clip, container must HUG |
| Fixed-height card with overflow | **FAIL** — card height must be HUG |
| LazyColumn with < 10 static items | **FAIL** — use regular Column with scroll |

---

## 7. NESTING RULES

### 7a. Radius Nesting Formula

> **child_radius = parent_radius - parent_padding**
> If result < 0, child_radius = 0.

| Parent | Parent radius | Parent padding | Child radius |
|--------|--------------|----------------|-------------|
| Card (r:12) | 12 | 16 | 0 (12-16 < 0) |
| Dialog (r:24) | 24 | 24 | 0 (24-24 = 0) |
| Bottom sheet (r:24) | 24 | 16 | 8 (24-16) |
| Chip container (r:12) | 12 | 4 | 8 (12-4) |
| CTA (r:16) | 16 | — | N/A (leaf node, no children with radius) |

**Rule: A child's radius MUST NEVER exceed its parent's radius.**

**Practical override (L178):** The formula often yields 0 or negative for real components
(e.g., card r:16, p:24 → child = 16-24 = -8 → 0). In practice, designers use r:8 or r:12
for cards nested inside r:16 parents. Use "visually smaller" rather than strict formula.
Exception: if parent has NO fill/stroke (transparent wrapper), children keep standalone radius.

### 7b. Auto-Layout Nesting

- Auto-layout frames CAN contain auto-layout children. This is normal.
- Each nested level sets its OWN `layoutMode`, `padding`, `itemSpacing`.
- `FILL` width on a child means fill the PARENT's available width (after parent padding). NOT the screen width.
- A VERTICAL parent with a HORIZONTAL child is the standard card pattern (card Column → row Row).

### 7c. Depth Limit

**Maximum: 5 levels of nested auto-layout in a single component.**

```
Level 1: Screen root (VERTICAL)
  Level 2: Card (VERTICAL)
    Level 3: Row inside card (HORIZONTAL)
      Level 4: Text group (VERTICAL — title + subtitle)
        Level 5: Inline element (HORIZONTAL — icon + label)
```

If you reach level 6, the component is too complex. Decompose into sub-components.

### 7d. Figma-Specific Nesting

When building nested auto-layout in Figma:
1. Build innermost children FIRST
2. Wrap in parent frames working OUTWARD
3. Set `layoutSizingHorizontal = "FILL"` on children AFTER appending to parent
4. Set parent `primaryAxisSizingMode = "AUTO"` LAST

This inside-out order prevents Figma API from computing intermediate (wrong) sizes.

---

## 8. TEXT CONTAINER RULES

### 8a. Decision Tree

```
Text inside a CTA button?
  → Width: FILL, Height: HUG, maxLines: 1, textAlignHorizontal: CENTER (L69, L101)

Text inside a badge / tag / chip?
  → Width: HUG, Height: HUG, maxLines: 1

Text is a price or number?
  → Width: HUG, Height: HUG, maxLines: 1

Text is a single-line heading or title?
  → Width: FILL, Height: HUG, maxLines: 1, overflow: ellipsis

Text is a multi-line body or description?
  → Width: FILL, Height: HUG, maxLines: unlimited (or explicit limit)

Text is a form label?
  → Width: FILL, Height: HUG, maxLines: 1

Text is anything else?
  → Width: FILL, Height: HUG
```

### 8b. Absolute Rules

| Rule | Verdict if violated |
|------|-------------------|
| Text node has fixed width (except badge/chip/button) | **FAIL** |
| Text node has fixed height | **FAIL** — always |
| Text clips or truncates without `maxLines` set | **FAIL** |
| Two adjacent text containers (heading + body) both have width FILL and height HUG | **PASS** |
| Text inside HORIZONTAL row uses `weight(1f)` / `flex:1` for overflow control | **PASS** |
| Text inside HORIZONTAL row has fixed width causing clip | **FAIL** |

### 8c. Heading + Body Pattern

When a heading and body text stack vertically:

```
Column (VERTICAL, gap: 4 or 8) {
    Text(heading)    // width: FILL, height: HUG, maxLines: 1-2
    Text(body)       // width: FILL, height: HUG, maxLines: unlimited
}
```

**Figma:**
```js
const group = figma.createFrame();
group.layoutMode = "VERTICAL";
group.itemSpacing = 4; // or 8
group.primaryAxisSizingMode = "AUTO";
// heading and body nodes
heading.layoutSizingHorizontal = "FILL"; // after append
body.layoutSizingHorizontal = "FILL";    // after append
```

### 8d. Text in Horizontal Rows (name + value pattern)

```
Row (HORIZONTAL, SPACE_BETWEEN) {
    Text(label)   // flex: 1, overflow: ellipsis — takes remaining space
    Text(value)   // HUG width — never truncates
}
```

**Rule:** In a horizontal row with text + a fixed-width element, the text ALWAYS gets `flex:1` / `weight(1f)` / `layoutSizingHorizontal = "FILL"`. The fixed element stays HUG.

### 8e. Long Text Validation

**Every text container must survive 2x expected content length.**
- Hindi text is typically 1.3-1.5x longer than English for the same content.
- Test every screen with longest possible string.
- If it clips, overlaps, or breaks layout → the sizing mode is WRONG.

---

## 9. COMPOSE-SPECIFIC LAYOUT PATTERNS

### 9a. Screen Template

```kotlin
@Composable
fun ScreenName() {
    Column(modifier = Modifier.fillMaxSize().background(WiomColors.neutralWhite)) {
        // Status bar: handled by system, not a composable
        AppBar(title = "...", onBack = { })              // FIXED 56dp
        Column(
            modifier = Modifier
                .weight(1f)                               // FILL remaining
                .verticalScroll(rememberScrollState())
                .padding(horizontal = 16.dp),             // screen padding
            verticalArrangement = Arrangement.spacedBy(24.dp) // section gap
        ) {
            // Screen content sections here
        }
        CTAArea { /* buttons */ }                         // FIXED at bottom
    }
}
```

### 9b. Card Template

```kotlin
Card(
    modifier = Modifier.fillMaxWidth(),                   // FILL width
    shape = RoundedCornerShape(12.dp),                    // DS radius
    elevation = CardDefaults.cardElevation(1.dp)          // DS elevation
) {
    Column(
        modifier = Modifier.padding(16.dp),               // card-padding
        verticalArrangement = Arrangement.spacedBy(8.dp)  // internal gap
    ) {
        // Card content
    }
}
```

### 9c. List Row Template

```kotlin
Row(
    modifier = Modifier
        .fillMaxWidth()
        .padding(14.dp),                                  // Wiom list row padding
    horizontalArrangement = Arrangement.spacedBy(8.dp),   // icon-to-text gap
    verticalAlignment = Alignment.CenterVertically        // cross-axis center
) {
    IconCircle(icon = Icons.xxx, size = 48.dp)            // FIXED 48x48
    Column(
        modifier = Modifier.weight(1f),                   // text takes remaining
        verticalArrangement = Arrangement.spacedBy(4.dp)
    ) {
        Text(title)                                       // FILL, HUG height
        Text(subtitle)                                    // FILL, HUG height
    }
    Icon(Icons.chevronRight, tint = WiomColors.brand600)  // FIXED 24x24
}
```

### 9d. CTA Area Template

```kotlin
Surface(
    modifier = Modifier.fillMaxWidth(),
    shadowElevation = 8.dp                                // subtle top shadow
) {
    Column(
        modifier = Modifier.padding(start = 16.dp, top = 12.dp, end = 16.dp, bottom = 24.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp) // CTA gap
    ) {
        Button(                                            // Primary
            modifier = Modifier.fillMaxWidth(),  // HUG height (L67: 48 = p12+text24+p12)
            shape = RoundedCornerShape(16.dp)
        ) { Text("Primary CTA") }
        FilledTonalButton(                                 // Secondary (if exists)
            modifier = Modifier.fillMaxWidth(),  // HUG height (L67: 48 = p12+text24+p12)
            shape = RoundedCornerShape(16.dp),
            colors = ButtonDefaults.filledTonalButtonColors(
                containerColor = Color(0xFFFFE5F6)         // DS secondary tonal
            )
        ) { Text("Secondary CTA") }
    }
}
```

### 9e. Input Field Template

```kotlin
Column(
    verticalArrangement = Arrangement.spacedBy(8.dp)      // label-to-field gap
) {
    Text(label, style = labelStyle)                        // FILL, HUG
    OutlinedTextField(
        modifier = Modifier.fillMaxWidth(),  // HUG height (L67: 48 = p12+text24+p12)  // FILL width, FIXED 48 height
        shape = RoundedCornerShape(12.dp),                 // DS input radius
        // ...
    )
}
```

---

## 10. FIGMA-SPECIFIC LAYOUT PATTERNS

### 10a. Screen Frame

```js
const screen = figma.createFrame();
screen.name = "ScreenName";
screen.layoutMode = "VERTICAL";
screen.resize(390, 100);                    // width FIXED 390
screen.counterAxisSizingMode = "FIXED";     // lock width
screen.primaryAxisSizingMode = "AUTO";      // height HUG — set AFTER children
screen.paddingTop = 0;
screen.paddingRight = 0;
screen.paddingBottom = 0;
screen.paddingLeft = 0;
screen.itemSpacing = 0;                     // sections manage their own gaps
screen.fills = [{type: 'SOLID', color: {r: 0.98, g: 0.976, b: 0.988}}]; // neutral/white
```

### 10b. Card Frame

```js
const card = figma.createFrame();
card.layoutMode = "VERTICAL";
card.paddingTop = card.paddingBottom = card.paddingLeft = card.paddingRight = 16;
card.itemSpacing = 8;
card.cornerRadius = 12;
card.fills = [{type: 'SOLID', color: {r: 1, g: 1, b: 1}}];
card.effects = [{type: 'DROP_SHADOW', color: {r:0.086, g:0.063, b:0.129, a:0.06}, offset:{x:0,y:1}, radius:2, visible:true}];
// After appending to parent:
card.layoutSizingHorizontal = "FILL";
```

### 10c. Auto-Layout Child FILL Pattern

```js
// WRONG — throws error:
child.layoutSizingHorizontal = "FILL";
parent.appendChild(child);

// CORRECT — set AFTER append:
parent.appendChild(child);
child.layoutSizingHorizontal = "FILL";
```

### 10d. Text Node

```js
const text = figma.createText();
await figma.loadFontAsync({family: "Noto Sans", style: "Regular"});
text.characters = "Content here";
text.fontSize = 14;
text.textAutoResize = "HEIGHT";             // width FILL, height HUG
// After appending to auto-layout parent:
text.layoutSizingHorizontal = "FILL";
```

For HUG width (badges, prices):
```js
text.textAutoResize = "WIDTH_AND_HEIGHT";   // both HUG
// Do NOT set layoutSizingHorizontal = "FILL"
```

---

## 11. VALIDATION CHECKLIST

Binary pass/fail. Run on every screen.

### Spacing

| # | Rule | PASS condition |
|---|------|---------------|
| S1 | Grid compliance | Every padding, gap, margin ∈ {4, 8, 12, 16, 24, 32, 40, 48, 64, 72, 84} |
| S2 | Internal ≤ external | Every component's internal padding ≤ gap between it and siblings |
| S3 | Screen horizontal padding | = 16 (both sides) |
| S4 | No zero-padding content frames | Content frames (card, form, section) have padding > 0 |
| S5 | Minimum horizontal gap | ≥ 4px between horizontal siblings |
| S6 | Minimum vertical gap | ≥ 8px between vertical siblings (exception: list rows with dividers = 0) |

### Sizing

| # | Rule | PASS condition |
|---|------|---------------|
| Z1 | No fixed text height | Zero text nodes have explicit fixed height |
| Z2 | No fixed text width (content) | Zero body/heading/label text nodes have fixed width |
| Z3 | Screen height = HUG | Root frame primaryAxisSizingMode = AUTO |
| Z4 | Content uses FILL width | All cards, inputs, CTAs have width = FILL |
| Z5 | Fixed elements correct | Status=24, app bar=56, CTA=HUG(→48), icon circle=48, icon=24 |

### Direction & Alignment

| # | Rule | PASS condition |
|---|------|---------------|
| D1 | Direction matches table | Every component direction matches §2 table |
| D2 | CTA stacking | Multiple CTAs are VERTICAL, gap:12. Never side-by-side. |
| D3 | No centered body text | Body text, labels, descriptions all START-aligned |
| D4 | List rows cross-axis center | All list row items vertically centered |

### Touch & Interaction

| # | Rule | PASS condition |
|---|------|---------------|
| T1 | Touch targets | All interactive elements ≥ 48dp on both axes |
| T2 | Target spacing | ≥ 8dp between adjacent touch targets |
| T3 | CTA outside scroll | CTA bar is not inside the scrollable content area |

### Nesting

| # | Rule | PASS condition |
|---|------|---------------|
| N1 | Radius nesting | No child has radius > parent radius |
| N2 | Depth limit | ≤ 5 levels of nested auto-layout |
| N3 | FILL after append (Figma) | layoutSizingHorizontal = "FILL" set after appendChild |
| N4 | HUG-height last (Figma) | primaryAxisSizingMode = "AUTO" set after all children appended |

---

## 12. ANTI-PATTERNS

| Anti-pattern | Why wrong | Correct |
|---|---|---|
| Fixed height on card | Content changes → clips or wastes space | HUG height |
| Fixed width on body text | Language switch → clips | FILL width |
| `layoutMode = "NONE"` for content | No reflow, pixel-pushing | VERTICAL or HORIZONTAL |
| Side-by-side CTAs | DS violation, cramped on small screens | Stack vertically, gap:12 |
| `gap: 0` with manual Spacer everywhere | Fragile, doesn't scale | Use `itemSpacing` / `spacedBy` |
| Padding via invisible frames | Layout debt, confuses inspection | Use frame padding properties |
| Center-aligning form labels | Slower to scan | Left-align (START) |
| Nesting > 5 auto-layout levels | Unmaintainable | Decompose into sub-components |
| Internal padding > external gap | Elements feel disconnected | Internal ≤ external |
| `resize()` after `primaryAxisSizingMode = "AUTO"` | Resets to FIXED silently | resize() FIRST, AUTO LAST |
| `FILL` before `appendChild()` | Throws Figma API error | Set FILL after append |
| `primaryAxisSizingMode=AUTO` on HORIZONTAL frame without re-setting FILL | Width collapses to HUG, elements won't scale with parent | Re-set `layoutSizingHorizontal="FILL"` AFTER primaryAxisSizingMode change (L100) |
| Relying on parent CENTER to center CTA text | FILL-width text ignores parent alignment | Set `textAlignHorizontal="CENTER"` on the text node itself (L101) |
| `Modifier.fillMaxSize()` on every element | Layout fights, unexpected expansion | Only on screen root + scroll content |
| Hardcoded dp sprinkled in composables | Inconsistent, hard to audit | Centralize in Dimens object |
| `wrapContentHeight()` on scrollable area | Content overflows off-screen | Use `weight(1f)` for scroll region |

---

## 13. CONNECTED / OVERLAPPING ELEMENTS (L106, L115, L116)

Some elements visually overlap — a tip bar tucked behind an elevated card, a badge
on a corner. These break the normal auto-layout flow.

### 13a. When to Use

If the next sibling's y-position < current element's (y + height) in Figma, there's
an intentional overlap. This is NOT a layout error.

### 13b. Figma Implementation

Figma auto-layout doesn't support overlap. Use a wrapper with `layoutMode: "NONE"`:
```
Box (layoutMode: NONE)
├── Elevated card (position: relative, z higher)
└── Connected tip (position: relative, y overlaps card's bottom)
```

Or use negative `itemSpacing` in a VERTICAL frame (rare, only for avatar stacks).

### 13c. Compose Implementation

```kotlin
Box {
    Card(modifier = Modifier.zIndex(1f)) { /* elevated content */ }
    TipBar(modifier = Modifier.offset(y = (-12).dp).padding(top = 19.dp))
}
```

- Elevated card: `zIndex(1f)` + shadow
- Connected element: `offset(y = -overlapDp)` + extra top padding to account for hidden zone
- Connected element gets bottom-only radius: `RoundedCornerShape(bottomStart = 12.dp, bottomEnd = 12.dp)`

### 13d. Rules

| Rule | Value |
|---|---|
| Overlap amount | Typically 12dp |
| Connected element top padding | overlap + normal padding (e.g., 12 + 7 = 19) |
| Connected element radius | Bottom-only (0, 0, 12, 12) |
| Connected element height | HUG (auto-sizes to content, never fixed) |
| Border + shadow can coexist | Yes — border for definition, shadow for elevation |

---

## 14. SCREEN ARCHITECTURE (L77-L84, L103)

### 14a. Three-Zone Pattern

Every screen follows this structure:

```
Screen (VERTICAL, gap:0, p:0, FILL width, HUG height in Figma)
├── HEADER (FILL×HUG)
│   └── Status Bar (24) + App Bar (56) = 80
├── BODY (FILL×FILL in Compose, FILL×HUG in Figma)
│   └── Scrollable content with 16dp horizontal padding
└── FOOTER (FILL×HUG)
    └── CTA area: p:[12,16,24,16]
```

### 14b. Body Zone Internal Gap

| Screen type | Content gap | Example |
|---|---|---|
| Form screen | 16 or 24 | Input fields stacked |
| List screen | 0 | Rows separated by dividers |
| Detail screen | 24 | Sections with headings |
| Multi-step screen | 16 | Progress + content + actions |

### 14c. Footer Sub-Patterns

| Pattern | Height | Content |
|---|---|---|
| Single CTA | 84 | p:[12,16,24,16] + 1 button |
| Dual CTA | 144 | p:[12,16,24,16] + 2 buttons gap:12 |
| Bottom bar/tab | 84 (FIXED) | Navigation icons |
| No footer | 0 | List screens that scroll to bottom |
| Keyboard visible | CTA above keyboard | `imePadding()` in Compose |

### 14d. Contained Screens (agreements, confirmations)

Some screens use 24dp horizontal padding instead of 16dp for a more contained feel.
Content width = viewport - 48 (instead of viewport - 32).

---

## 15. FLOOR VALUES (minimum safety thresholds)

### Heights
| Element | Min height | Rationale |
|---|---|---|
| Tappable element | 48dp | Material touch target |
| Input field | 48dp | DS constant |
| Single-line text | 16dp | fs:10 + line-height |
| Drag handle bar | 4dp | Below = invisible |
| Divider | 1dp | Below = sub-pixel issues |

### Paddings
| Context | Min | Rationale |
|---|---|---|
| Screen horizontal | 16dp | Content never touches edge |
| Card internal | 12dp | Text feels trapped below this |
| CTA horizontal | 16dp | Label needs room |
| Dialog (any side) | 24dp | Below = cramped |

### Gaps
| Context | Min | Rationale |
|---|---|---|
| Horizontal siblings | 4dp | Below = merge visually |
| Vertical siblings | 8dp | Below = squished |
| Adjacent touch targets | 8dp | Prevents mis-taps |

### Device Safety
| Rule | Test |
|---|---|
| Min supported width | 320dp — no clipping, no overflow |
| Max common width | 412dp — no excessive stretching |
| CTA visible above keyboard | Always, via `imePadding()` |
| Bottom sheet max | 85% viewport height |

---

## 16. CAMERA SCREEN PATTERN (L121-L123)

Camera screens are the ONE exception to "always use auto-layout at root."

### Structure
```
Screen (FILL width, FIXED viewport height, layoutMode: NONE)
├── Viewfinder (FILL×FILL, dark bg #4D4D4D, clips overflow)
│   ├── Camera feed (wider than screen, IMAGE fill, scaleMode:CROP)
│   └── Shutter button (64×64, absolute: bottom-center, margin-bottom ~40dp)
├── App bar (FILL×HUG, overlaid on top of viewfinder, semi-transparent or solid)
├── Flash/torch icon (24×24, absolute position)
└── [Optional] Guide overlay (dashed rect for Aadhaar/document capture)
```

### Rules
| Rule | Value |
|---|---|
| Root layout | NONE (exception — controls overlay on viewfinder) |
| Viewfinder fills screen | FILL×FILL (or FIXED viewport size in Figma) |
| Camera feed aspect | Wider than viewport, CROP mode, no letterboxing |
| Shutter button | 64×64 FIXED, bottom-center absolute |
| App bar | Overlaid, z-index above viewfinder |
| Guide overlay (documents) | Dashed stroke rect, centered, box-shadow for dark surround |

### Compose
```kotlin
Box(modifier = Modifier.fillMaxSize()) {
    CameraPreview(modifier = Modifier.fillMaxSize())  // viewfinder
    TopAppBar(modifier = Modifier.align(Alignment.TopCenter))  // overlaid
    ShutterButton(modifier = Modifier.align(Alignment.BottomCenter).padding(bottom = 40.dp))
}
```

---

## 17. SIDE-BY-SIDE CARDS PATTERN (L124)

Two (or more) equal-width cards in a HORIZONTAL row.

### Structure
```
Row (FILL×HUG, HORIZONTAL, gap:16, p:[top,16,bottom,16])
├── Card A (FILL×HUG via weight(1f), r:16)
└── Card B (FILL×HUG via weight(1f), r:16)
```

Width per card = (parent_width - padding_left - padding_right - gap) / card_count.
At 360 base with 2 cards: (360 - 32 - 16) / 2 = 156 each.

### Figma
Both cards use `layoutSizingHorizontal = "FILL"` inside HORIZONTAL parent = equal distribution.

### Compose
```kotlin
Row(
    modifier = Modifier.fillMaxWidth().padding(horizontal = 16.dp),
    horizontalArrangement = Arrangement.spacedBy(16.dp)
) {
    Card(modifier = Modifier.weight(1f)) { /* content */ }
    Card(modifier = Modifier.weight(1f)) { /* content */ }
}
```

---

## 18. QUANTITY COUNTER PATTERN (L171-174)

### Structure
```
Card (FILL×HUG, VERTICAL, gap:12, p:16, r:16)
├── Counter row (FILL×HUG, HORIZONTAL, SPACE_BETWEEN, bottom stroke divider)
│   ├── Price/label (HUG×HUG, fs:20/Bold)
│   └── Counter button (HUG×HUG, HORIZONTAL, p:[8,16,8,16], r:12, fill:brand/600)
│       └── "–  1  +" (single text node, fs:16/Bold, centered)
└── Info section (FILL×HUG, VERTICAL, gap:8)
```

### Rules
- Counter is ONE tappable button, not 3 separate elements
- Counter radius: 12 (small CTA variant)
- Counter height: HUG → 40 (p:8 + text:24 + p:8)
- Price and counter separated by SPACE_BETWEEN
- Bottom stroke on counter row divides it from info below

---

## 19. NESTED CARDS PATTERN (L175-179)

### Type A: Cards inside section (no visual nesting)
```
Section (FILL×HUG, VERTICAL, gap:12, NO fill, NO stroke, NO radius)
├── Title text
└── Card list (FILL×HUG, VERTICAL, gap:12)
    ├── Card (FILL×HUG, r:16, fill:#FAF9FC, stroke:#E8E4F0)
    ├── Card (same)
    └── Card (same)
```
Parent is transparent wrapper. Cards appear standalone. NOT true visual nesting.

### Type B: Card inside colored card (true visual nesting)
```
Outer card (FILL×HUG, r:16, fill:#008043 or other color, p:24)
└── Inner card (FILL×HUG, r:8-12, fill:#FAF9FC, stroke:accent)
```
Inner card radius smaller than outer. Use r:8 or r:12 inside r:16 parent.

### Type C: Stroke-only accent zone inside card
```
Colored card (r:16, fill:colored)
└── Info zone (FILL×HUG, stroke:accent, NO fill, NO radius)
```
Lighter nesting — just a border callout, not a full card.

---

## 20. ERROR BANNER PATTERN (L162-163)

```
Banner (FILL width, HUG height, fill:#D92130, p:[12,16,12,16], r:0)
└── Row (FILL×HUG, HORIZONTAL, gap:8, crossAlign:CENTER)
    ├── Icon (20×20 or 24×24)
    └── Text (FILL×HUG, fs:14/Regular, #FAF9FC white)
```

- Full width (no side padding, no radius) — edge to edge
- Sits between header and content
- Uses negative/600 red background
- r:0 = urgency signal (not dismissible)

---

## 21. VISIBILITY-DRIVEN HEIGHT PATTERN (L2, L43, L117)

**The rule:** States change via visibility toggle on pre-built children. NEVER add/remove
children or change layout properties (direction, gap, padding, sizing) between states.

### How it works
1. Build ALL children for the maximal state (every possible section present)
2. Set optional children to `visible = false`
3. Parent uses HUG height — hidden children cost 0 space
4. When state changes: toggle visibility. Parent height adjusts automatically.
5. Gap between children is pre-set but has no visual effect when adjacent child is hidden

### Applies to
- Input-Field: subtext hidden in non-error states (L2)
- Accordion: answer zone hidden when collapsed (L43)
- App bar: subheading hidden in non-Big variant (L61)
- Bottom sheet: hidden Card instances for different states (L117)

### In Compose
```kotlin
AnimatedVisibility(visible = showSubtext) {
    Text(errorMessage, ...) // space appears/disappears with animation
}
```

---

## 22. ZERO-PADDING COMPONENT ROOT (L1, L53)

**The rule:** Component roots have padding [0,0,0,0]. The PARENT screen/form handles
positioning via its own padding and gap.

Components never assume their own margins. This allows:
- Same component in different contexts (screen body vs dialog vs bottom sheet)
- Parent controls spacing between siblings
- Component only controls its INTERNAL layout

Exception: full-width zones (status bar, app bar, error banner, bottom sheet header)
have their OWN internal padding because they span edge-to-edge.

---

## 23. SPAWNING PROTOCOL

### When Pratibimb spawns this skill (Figma context):
1. Use §1 sizing tree, §2 direction table, §3 spacing values
2. Follow §1d ordering law religiously
3. Use §10 Figma-specific patterns as templates
4. Run §11 validation after every frame creation
5. §7d inside-out nesting order for complex components

### When Maverick spawns this skill (Compose context):
1. Use §1 sizing tree, §2 direction table, §3 spacing values
2. Use §9 Compose templates as starting points
3. All dp values from a centralized `Dimens` object (never inline literals)
4. Run §11 validation during code review / audit
5. §6 scroll architecture — CTA outside scroll, content inside

### Cross-platform parity check:
After building in both Figma and Compose, verify:
- Same direction on every component
- Same padding values
- Same gap values
- Same sizing modes (FILL/FIXED/HUG)
- Same fixed dimensions
- Same radius values
- Same alignment

If any value differs between Figma and Compose for the same screen → one of them is wrong. This skill is the tiebreaker.

---

## 24. SETTINGS MENU ROW PATTERN (Section 3 extraction)

Full-scroll settings screen: 12 menu items in VERTICAL list. Source: Partner Settings 1341h.

### Menu Row Atom
```
Menu Row (FILL×HUG→80, HORIZONTAL, gap:16, p:[16,16,16,16], r:16)
  stroke: #E8E4F0, sw:1, fill: none
  ├── Content (FILL×HUG, HORIZONTAL, gap:16, crossAlign:CENTER)
  │   ├── Icon circle (48×48, HORIZONTAL, gap:10, p:12, r:50, fill:#F1EDF7)
  │   │   └── Icon (24×24, FIXED)
  │   └── Text group (FILL×HUG, VERTICAL, gap:4)
  │       ├── Title (FILL, fs:16/Bold "Display Bold", #161021, lh:24)
  │       └── Subtitle (FILL, fs:14/Regular "Display Regular", #161021, lh:20)
  └── Chevron arrow (INSTANCE, 24×24, FIXED) — inner vector fill:#D9008D
```

### Menu list container
```
Menu list (FILL×HUG, VERTICAL, gap:16, p:0)
└── [12× Menu Row]
```

### Profile section (above menu list)
- Partner name: fs:20/Bold, #161021 — HUG width
- Username: fs:16/Regular, #161021 — HUG width
- Both absolute-positioned above menu list in source (but should be VERTICAL stack)

### Rules
- Row gap:16 matches item internal padding — equal grouping
- Some rows use gap:10 (bill payment, team manage) — off-grid exception
- Icon circle r:50 (some) vs r:40 (others) — should be r:24 per §5
- Logout row: title only, no subtitle — text group h:24
- Badge inline: Pathshala row has Frame(HORIZONTAL, gap:8) containing title + badge chip

---

## 25. STAT TILE ROW PATTERN (Section 3 extraction)

3 equal-width tiles showing label+value. Source: Strike Rate bottom sheet.

### Structure
```
Stat row (FILL×HUG, HORIZONTAL, gap:16, crossAlign:CENTER)
├── Tile (FILL×HUG→80, VERTICAL, gap:16, p:12, r:12, fill:#FAF9FC, stroke:#E8E4F0)
│   ├── Label (FILL, fs:12/Regular, #161021, lh:16)
│   └── Value (FILL, fs:16/Bold, #161021, lh:24)
├── Tile (same)
└── Accent Tile (FILL×HUG→80, VERTICAL, gap:16, p:12, r:12, fill:#FFE6CC, stroke:#FFBDB3)
    ├── Label (FILL, fs:12/Regular, #161021, lh:16)
    └── Value (FILL, fs:16/Bold, #161021, lh:24)
```

### Tile specs
| Type | Fill | Stroke |
|---|---|---|
| Normal | #FAF9FC | #E8E4F0 |
| Accent (result metric) | #FFE6CC | #FFBDB3 |

### Rules
- All tiles: `layoutSizingHorizontal: FILL` → equal width distribution
- Width per tile = (parent - gap×2) / 3
- Internal gap:16 (label to value) — generous for readability
- Tile radius:12 (card radius)
- Tile padding:12 uniform
- Bottom sheet uses standard header: slider(120×4, r:4, #D7D3E0) + title(fs:24/Bold)
- Content padding: [24,16,48,16] — matches §3a bottom sheet

---

## 26. FAQ ACCORDION PATTERN (Section 3 extraction)

Bottom sheet with expandable FAQ items. 4 variants: 272, 416, 656, 768h.

### Collapsed question
```
faq-card (FILL×HUG, VERTICAL, gap:0, r:16, fill:none, stroke:none)
└── question (FILL×56, HORIZONTAL, gap:16, p:16, r:MIXED, fill:#F1E5FF)
    ├── title (FILL, fs:16/Bold, #6D17CE, lh:24)
    └── arrow circle (24×24, r:24, fill:#D9008D)
        └── white chevron (11×7, fill:#FAF9FC)
```

### Expanded state
```
faq-card (FILL×HUG, VERTICAL, gap:0, r:16, fill:#FAF9FC, stroke:#D6B2FF sw:1)
├── question (FILL×56, HORIZONTAL, gap:16, p:16, r:MIXED, fill:#FAF9FC) — bg changes to white
│   ├── title (FILL, fs:16/Bold, #6D17CE)
│   └── arrow circle (rotated)
├── answer (FILL×HUG, VERTICAL, gap:16, p:[0,16,16,16])
│   └── description (FILL, fs:16/Regular, #161021, lh:24)
├── [next question — purple text link, SPACE_BETWEEN row]
│   ├── question text (HUG, fs:16/Bold, #6D17CE)
│   └── arrow (INSTANCE, 24×24, fill:#6D17CE)
├── [next answer if expanded]
└── ...
```

### State table
| State | Question fill | Card fill | Card stroke |
|---|---|---|---|
| Collapsed | #F1E5FF (purple bg) | none | none |
| Expanded | #FAF9FC (white) | #FAF9FC | #D6B2FF |

### Key colors (info/purple palette)
- Question bg: #F1E5FF
- Question text: #6D17CE (info/600)
- Card stroke when open: #D6B2FF
- Arrow circle: #D9008D (brand)
- Nested question chevrons: #6D17CE (matches question text)

### Title spec differs from standard
- FAQ sheet title: fs:20/Bold (NOT fs:24 like standard bottom sheets)
- Subtitle present: fs:16/Regular — "काम शुरू हुआ: 11:35 AM"
- Heading frame height: 104 (title 56 + subtitle 24 + padding)

---

## 27. COMMENT INPUT SHEET PATTERN (Section 3 extraction)

Bottom sheet with text input. Two states: empty and filled+keyboard.

### Empty state
```
Sheet (360×720, r:MIXED)
├── header (360×88) — Title "इंटरनेट में क्या समस्या थी?" (fs:24/Bold)
└── Content (360×144, VERTICAL, gap:48, p:[24,16,48,16], fill:#FAF9FC)
    └── Input-Field (328×72, VERTICAL, gap:8)
        └── field (328×72, HORIZONTAL, gap:12, p:[12,16,12,16], r:12, fill:#FAF9FC, stroke:#D7D3E0)
```

### Filled state with keyboard
```
Sheet (360×720, r:MIXED)
├── header (360×88)
└── Content (360×512, VERTICAL, gap:48, p:[24,16,0,16], fill:#FAF9FC)
    └── Form (360×488, VERTICAL, gap:32)
        ├── Input-Field (328×120, VERTICAL, gap:8)
        │   └── field (328×120, r:12, stroke:#352D42 sw:1) — ACTIVE stroke
        └── Keyboard zone (360×336, VERTICAL)
            ├── CTA area (360×80, VERTICAL, p:16, fill:#FAF9FC)
            │   └── CTA button (328×48, r:16, fill:#D9008D)
            └── Keyboard (360×256, INSTANCE, fill:#161021)
```

### Input field state changes
| Property | Empty/Inactive | Active/Filled |
|---|---|---|
| Stroke color | #D7D3E0 | #352D42 |
| Height | 72 | 120 (multi-line) |
| Content padding bottom | 48 | 0 (keyboard fills) |

### Rules
- CTA appears ABOVE keyboard in filled state (keyboard toolbar pattern)
- Form gap: 32 between input and keyboard+CTA zone
- Input grows vertically as user types (72→120)
- Active stroke is darker (#352D42 = neutral/800)

---

## 28. CELEBRATION CARD PATTERN (Section 3 extraction)

Two variants: positive (won lottery) and negative (missed). Both 360×800, gradient bg.

### Gradient background
```
GRADIENT_LINEAR: #FFFFFF at 20% → #FFCCED at 100%
```
Confetti vectors: #FFCCD0, 3 groups layered at 360×275

### Positive (won lottery)
```
Screen (360×800, layoutMode:NONE, gradient)
├── Confetti frame (360×824)
├── Top app bar (360×80, close icon)
├── Physical card visual (328×576, r:8, fill:#FFFFFF, stroke:#000000 0.53px)
├── GiftBox icon (60×60, r:150, fill:#FFFFFF) — overlapping card edge
└── Purple hero (328×640, HORIZONTAL, gap:10, p:[32,23,32,23], r:8, fill:#6D17CE)
    ├── Header: "बम्पर गारंटी" (fs:32/Bold, #FAF9FC) + "₹200 या अधिक"
    ├── Divider (stroke:#D6B2FF, p:[16,0,0,0])
    │   └── "आज ही, इस काम को पूरा करने पर" (fs:20/Bold, #FAF9FC)
    └── Embedded ticket (280×352, r:16, fill:#FAF9FC, stroke:#E8E4F0)
        └── Full service ticket with timer + CTA
```

### Negative (missed lottery)
```
Screen (360×800, layoutMode:NONE, gradient)
├── Confetti frame
├── Top app bar (close icon)
├── Empty card placeholder (296×256, r:8, fill:#FFFFFF)
├── Summary card (328×144, VERTICAL, p:[24,16,24,16], r:16, fill:#FAF9FC, stroke:#E8E4F0)
│   └── Customer info row (r:12, fill:#F1EDF7)
├── Message "भाई, रेड ज़ोन में..." (fs:20/Bold, #161021, CENTER)
└── Secondary CTA bar (360×80, fill:#FFCFE6)
    └── CTA (328×48, r:16, fill:#FFE5F6)
        └── "पैसे कमाने के लिए अन्य काम देखें" (fs:16/Bold, #D9008D, CENTER)
```

### Ghost/Secondary CTA in colored container
```
Container (360×80, fill:#FFCFE6)
└── CTA (FILL×48, HORIZONTAL, gap:12, p:[12,16,12,16], r:16, fill:#FFE5F6)
    └── Text (FILL, fs:16/Bold, #D9008D, textAlign:CENTER)
```
- The CTA sits in a colored band (#FFCFE6 light pink)
- CTA itself uses tonal fill (#FFE5F6) not solid brand
- This is the DS secondary CTA variant

---

## 29. SUMMARY KEY-VALUE CARD PATTERN (Section 3 extraction)

Payment breakdown with line items, subtotals, and grand total. Source: Bill recharge_total payment.

### Screen structure
```
Screen (360×720, fill:#FAF9FC)
├── Header (360×80, fill:#FAF9FC — light variant, close icon)
│   └── Title "रेटिंग बोनस" (fs:24/Bold, #161021)
├── Amount hero (328×82, VERTICAL, gap:0, p:[0,0,16,0], bottom stroke:#D7D3E0)
│   └── Row (HORIZONTAL, SPACE_BETWEEN)
│       ├── Amount (fs:24/Bold, #161021) + date range (fs:14/Regular, #665E75)
│       └── Pie chart icon (66×66)
└── Detail section (328×632, VERTICAL, gap:24)
    ├── Rating block (FILL, VERTICAL, gap:12, p:[0,0,24,0], bottom stroke:#E8E4F0)
    │   ├── Section header card (FILL×56, p:16, r:16, fill:#F1EDF7)
    │   │   └── Row (SPACE_BETWEEN): label + value
    │   └── Detail card (FILL, VERTICAL, gap:12, p:16, r:16, fill:#F1EDF7)
    │       ├── Line item rows (each HORIZONTAL, SPACE_BETWEEN)
    │       ├── Subtotal row (bottom stroke:#D7D3E0)
    │       │   └── Value in green (#008043)
    │       ├── Formula text (fs:14/Regular, #008043)
    │       ├── Grand total row (top stroke:#352D42 — DARK divider)
    │       └── Total row (HORIZONTAL, SPACE_BETWEEN)
    │           ├── "कुल पेमेंट" (fs:20/Bold, #161021)
    │           └── "₹1,026" (fs:20/Bold, #161021)
    └── Previous card (FILL×132, VERTICAL, gap:12, p:16, r:16, fill:#FAF9FC, stroke:#E8E4F0)
```

### Key-value row atom
```
Row (FILL×HUG, HORIZONTAL, SPACE_BETWEEN, crossAlign:CENTER)
├── Label (HUG or FILL, fs:16/Regular, #161021)
└── Value (HUG, fs:16/Bold, #161021 or #008043 for positive)
```

### Divider hierarchy
| Stroke color | Meaning |
|---|---|
| #D7D3E0 | Standard separator |
| #E8E4F0 | Subtle/card border |
| #352D42 | Grand total emphasis (darkest) |

### Nested color zones
- Screen: #FAF9FC
- Detail cards: #F1EDF7 (lavender) — info zone
- Previous card: #FAF9FC with stroke — card-on-page
- Previous inner row: #F1EDF7 — card-inside-card

---

## 30. PILL FILTER BAR PATTERN (Section 3 extraction)

Horizontal row of filter pills. Source: Earn Money wallet transactions.

### Structure
```
Filter bar (361×60, fill:#F1EDF7, stroke:#D7D3E0)
└── Pill row (328×36, HORIZONTAL, gap:16)
    ├── Active pill (HUG×36, VERTICAL, p:[8,12,8,12], r:8, fill:#FFCCED, stroke:#D9008D sw:1)
    │   └── Text (fs:14/SemiBold, #1A1A1A)
    ├── Dropdown pill (HUG×36, HORIZONTAL, gap:4, p:[7,12,7,12], r:8, stroke:#D7D3E0 sw:1)
    │   ├── Text (fs:16/Regular, #1A1A1A)
    │   └── Arrow (20×20 INSTANCE, fill:#352D42)
    └── [more pills...]
```

### Pill specs
| Type | Fill | Stroke | Text |
|---|---|---|---|
| Active | #FFCCED | #D9008D | fs:14/SemiBold |
| Inactive/dropdown | none | #D7D3E0 | fs:16/Regular |

### Rules
- Pill radius: 8 (smaller than card r:12, CTA r:16)
- Active pill padding: [8,12,8,12]
- Inactive pill padding: [7,12,7,12] — NOTE: 7 is off-grid, should be 8
- Dropdown arrow: 20×20 (compact, not 24)
- Bar bg: #F1EDF7 (lavender), full width
- Pill gap: 16

---

## 31. TRANSACTION CARD PATTERN (Section 3 extraction)

Two-zone card used in wallet/transaction lists. Source: Earn Money Landing (wallet view).

### Structure
```
Transaction Card (328×HUG, VERTICAL, gap:0)
└── Card body (FILL×HUG, VERTICAL, gap:0)
    ├── Top bar (FILL×60-64, HORIZONTAL, gap:12, p:[12,16,12,16], r:MIXED, fill:#E8E4F0, stroke:#D7D3E0)
    │   ├── Avatar/icon (40×40, GROUP)
    │   ├── Type label (HUG×20, HORIZONTAL, gap:6)
    │   └── Amount (FILL×24, fs:16/Bold)
    │       Color: credit=#161021, debit=#E01E00
    └── Detail body (FILL×120, VERTICAL, gap:16, p:[12,16,12,16], r:MIXED, fill:#FAF9FC, stroke:#D7D3E0)
        ├── Info row (FILL×20, HORIZONTAL, SPACE_BETWEEN)
        ├── Date row (FILL×16, HORIZONTAL, SPACE_BETWEEN)
        └── Comment bubble (FILL×28, VERTICAL, gap:4, counterAlign:MAX)
```

### Amount color convention
| Type | Color |
|---|---|
| Credit/positive | #161021 (text-primary) |
| Debit/negative | #E01E00 (custom red) |

### Date section header (between card groups)
```
Date header (360×36, HORIZONTAL, gap:10, p:[8,16,8,16], fill:#F1EDF7, stroke:#E8E4F0)
└── Date text (HUG, fs:14/SemiBold, #161021)
```

---

## 32. TAB BAR PATTERN (Section 3 extraction)

Two-tab navigation inside PA header. Source: Earn Money Landing with tabs.

### Structure
```
Tabs (INSTANCE, 360×48, HORIZONTAL, gap:0)
├── Selected tab (180×48, HORIZONTAL, p:[12,0,12,0], r:12, stroke:#E8E4F0 sw:0)
│   └── Label (fs:14/Regular, #FAF9FC, CENTER)
└── Deselected tab (180×48, HORIZONTAL, p:[12,0,12,0], r:0, stroke:#E8E4F0)
    └── Label (fs:14/SemiBold, #FAF9FC, CENTER)
```

### Tab state spec
| Property | Selected | Deselected |
|---|---|---|
| Width | 180 (50%) | 180 (50%) |
| Height | 48 | 48 |
| Radius | 12 (top corners) | 0 |
| Stroke sw | 0 (hidden) | visible |
| Text weight | Regular | SemiBold |
| Text color | #FAF9FC | #FAF9FC |

### Rules
- Tabs sit INSIDE header (PA_Headers h:128 = 80 + 48)
- Both tabs: white text on #443152 bg
- Selected tab: r:12 for subtle shape cue
- Font weight INVERTED: selected=Regular, deselected=SemiBold (ground truth)
- Tab is a DS INSTANCE (reusable component)

---

## 33. ISP RECHARGE LIST PATTERN (Section 3 extraction)

Data-heavy customer list with tooltips. Source: ISP Recharge List.

### Screen structure
```
Screen (360×720, fill:#F1EDF7 — LAVENDER bg, not white)
├── PA_Headers (360×80, fill:#443152)
├── Urgency banner (360×52, p:[12,16,12,16], fill:#D92130)
│   └── Row (FILL×28, HORIZONTAL, gap:8)
│       ├── "😕" emoji (fs:20) — NOT Material icon
│       └── Warning text (fs:14/SemiBold, #FAF9FC)
├── Section heading "प्लान अपडेट और रिचार्ज" (fs:20/Bold)
└── Customer cards (328×228 each, VERTICAL, gap:16, p:16, r:12, fill:#FAF9FC, stroke:#E8E4F0)
```

### Customer card internal
```
Card (328×228, VERTICAL, gap:16, p:16, r:12, fill:#FAF9FC, stroke:#E8E4F0)
├── Name row (FILL, HORIZONTAL, SPACE_BETWEEN)
│   ├── Name (HUG, fs:16/Regular)
│   └── Checkbox (24×24, INSTANCE)
├── Details grid (GROUP, 223×80 — 2×2 icon+label pairs)
│   └── Each: (HORIZONTAL, gap:8): icon(20×20) + labels(VERTICAL, gap:0)
└── Tooltip (296×72, VERTICAL, gap:0)
    ├── Bubble arrow (40×8) — pointing up
    └── Message (296×64, HORIZONTAL, gap:8, p:12, r:12, fill:#FFDA40)
        ├── Warning icon (20×20)
        └── Text (FILL, fs:14/SemiBold, #806000)
```

### Tooltip/warning spec
```
fill: #FFDA40 (warning yellow)
text: fs:14/SemiBold, #806000 (dark amber)
icon: 20×20
padding: 12 uniform
radius: 12
```

### Key observations
- Screen bg #F1EDF7 (lavender) distinguishes list screens from form screens
- Detail icons: 20×20 (compact, not 24)
- Checkbox on each card — batch operation affordance
- Sections separated by heading + gap:24

---

## 34. RADIO SELECTION SHEET PATTERN (Section 3 extraction)

Bottom sheet with radio card options. Source: Language picker (2 variants: 280h, 344h).

### Radio card specs
| State | Fill | Stroke | Radio color |
|---|---|---|---|
| Selected | #FFCCED | #D9008D, 1px | #D9008D |
| Unselected | #FAF9FC | #D7D3E0, 1px | #A7A1B2 |

### Structure
```
Sheet (360×HUG, VERTICAL, gap:0)
├── header (INSTANCE, 360×88) — "ऐप की भाषा चुनें" (fs:24/Bold)
├── content (360×192, VERTICAL, gap:48, p:[16,16,48,16], fill:#FAF9FC)
│   └── options (328×128, VERTICAL, gap:16)
│       ├── Card (328×56, HORIZONTAL, gap:16, p:16, r:16)
│       │   ├── Radio (24×24, INSTANCE)
│       │   └── Details (FILL×24)
│       └── Card (same)
└── [Optional] Action bar (360×64, VERTICAL, gap:12, p:[0,16,16,16], fill:#FAF9FC)
    └── CTA (FILL×48, r:16, fill:#D9008D) — "भाषा बदलें"
```

### Rules
- Radio card radius: 16
- Radio card padding: 16 uniform
- Gap between cards: 16
- Action bar is SEPARATE from content — own padding
- 280h variant: no action bar (change immediate on selection)
- 344h variant: has confirm CTA

---

## 35. INSTALLATION REWARD HERO PATTERN (Section 3 extraction)

Full-screen success card with dark header and scrollable ticket list. Source: Installation reward.

### Structure
```
Screen (360×720, fill:#FAF9FC)
├── Top app bar (360×80, close icon)
├── Header Container (328×676, VERTICAL, gap:24, p:[32,24,0,24], r:16, fill:#008043)
│   ├── Amount "₹3000" (large, #FAF9FC) + "Monthly Device Bonus Earned" (fs:32/Bold)
│   ├── Divider (stroke:#D6B2FF, p:[16,0,0,0])
│   │   └── Credit date info (fs:20/Bold, #FAF9FC)
│   └── Ticket list (280×360, VERTICAL, gap:24)
│       └── [12× Service Ticket mini-cards]
└── Action bar (360×80, fill:#FAF9FC)
    └── CTA "Earn more money" (328×48, r:16, fill:#D9008D)
```

### Service ticket mini-card (dark variant)
```
Ticket (280×68, VERTICAL, gap:10, p:[22,16,22,16], r:12, fill:#443152, stroke:#D9008D)
└── Row (FILL×24, HORIZONTAL, SPACE_BETWEEN)
    ├── Label group (HORIZONTAL, gap:8)
    └── Expand arrow (24×24)
```

### Rules
- Green success bg #008043 for entire header container
- Dark tickets (#443152) inside green container
- Ticket stroke: brand pink #D9008D
- Large typography: fs:32/Bold for message
- Generous padding: [32,24,0,24]
- Ticket padding [22,16,22,16] — 22 is off-grid, should be 24

---

## 36. WALLET BALANCE HEADER PATTERN (Section 3 extraction)

Compact balance display + ghost CTA. Source: Earn Money partner success fee.

### Structure
```
Balance header (360×92, VERTICAL, gap:16, p:16, fill:#F1EDF7)
└── Row (FILL×60, HORIZONTAL, SPACE_BETWEEN, crossAlign:CENTER)
    ├── Balance group (HUG×60, HORIZONTAL, gap:12, crossAlign:CENTER)
    │   ├── Wallet icon (50×50, GROUP)
    │   └── Info (VERTICAL, gap:4)
    │       ├── Label (fs:14/Regular)
    │       └── Amount (fs:24/Bold, #161021)
    └── Ghost CTA (HUG×32, HORIZONTAL, p:[6,8,6,8], r:8, stroke:#D9008D sw:1, fill:none)
        └── Text (fs:14/SemiBold, #D9008D)
```

### Ghost CTA (outline-only variant)
```
Ghost CTA (HUG×32, p:[6,8,6,8], r:8)
  fill: none (transparent)
  stroke: #D9008D, 1px
  text: fs:14/SemiBold, #D9008D
```

### Size comparison
| CTA variant | Height | Radius | Padding |
|---|---|---|---|
| Primary | HUG→48 | 16 | [12,16,12,16] |
| Secondary tonal | HUG→48 | 16 | [12,16,12,16] |
| Ghost (outline) | HUG→32 | 8 | [6,8,6,8] |
| Coupon/inline | HUG→28 | 8 | [4,12,4,12] |

---

## 37. PROMOTIONAL BANNER PATTERN (Section 3 extraction)

Horizontal banner with illustration. Source: Earn Money "Stronger Together".

### Structure
```
Banner (360×87, HORIZONTAL, gap:4, p:[8,16,8,16], fill:#FFCCED)
└── Row (FILL×71, HORIZONTAL, gap:8)
    ├── Illustration (61×71, GROUP/IMAGE, FIXED)
    └── Text group (FILL×52, VERTICAL, gap:8)
        ├── Title (FILL, fs:16/Bold, #161021)
        └── Subtitle (FILL, fs:14/SemiBold, #161021)
```

### Rules
- Banner bg: #FFCCED (brand tonal pink)
- No CTA — informational only
- No radius — edge-to-edge
- Padding: [8,16,8,16] — compact vertical
- Illustration: fixed size, left-aligned
- Text: takes remaining space via FILL

---

## 38. REQUEST DEVICE SHEET PATTERN (Section 3 extraction)

Bottom sheet for ordering devices. Source: Request Device (420h).

### Structure
```
Sheet (360×420, VERTICAL, gap:0, r:MIXED)
├── header (360×88) — "अपनी जरूरत अनुसार टॉप-अप करें" (fs:24/Bold)
└── Content (360×332, VERTICAL, gap:48, p:[24,16,24,16], fill:#FAF9FC, counterAlign:MAX)
    ├── Order section (328×188, VERTICAL, gap:12)
    │   ├── Order card (328×156, VERTICAL, gap:12, p:16, r:16, fill:#FAF9FC, stroke:#E8E4F0)
    │   │   ├── Top row (FILL×56, HORIZONTAL, SPACE_BETWEEN, bottom stroke:#E8E4F0)
    │   │   └── Info (FILL×56, VERTICAL, gap:8)
    │   └── Warning "वॉलेट बैलेंस : ₹500" (fs:14/SemiBold, #E01E00)
    └── CTA "टॉप-अप करें" (328×48, r:16, fill:#D9008D, CENTER text)
```

### Rules
- Warning text: #E01E00 (same custom red as debit amounts)
- Content-to-CTA gap: 48 (standard for sheets)
- Card inside sheet: standard r:16, stroke:#E8E4F0
