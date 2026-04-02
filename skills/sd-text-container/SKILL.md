# Solution Design Text Container V1

Deterministic rules for text node sizing, typography, font selection, and text
container behavior. Every rule is binary: correct or incorrect.

Two consumers, one truth:
- **Pratibimb** spawns this when creating/auditing text in Figma
- **Maverick** spawns this when building Kotlin/Compose text

---

## 1. TEXT SIZING DECISION TREE

Every text node has a width mode, height mode, and auto-resize behavior.
Walk top-to-bottom. First match wins.

### 1a. Width Mode

```
Text inside a CTA button?
  → FILL (centers via primaryAlign:CENTER on parent)

Text is a badge / tag / chip label?
  → HUG

Text is a price, number, or counter?
  → HUG

Text is a placeholder inside an input field?
  → HUG (inside a FILL parent text frame)

Text is a heading, title, body, description, label, or subtitle?
  → FILL

Text is inside a HORIZONTAL row with icons/actions?
  → FILL (takes remaining space after FIXED siblings)

Default → FILL
```

**Rule: FILL is the default for text width.** HUG is only for inline content that sizes to its characters (badges, prices, chip labels).

### 1b. Height Mode

```
ALL text nodes → HUG. No exceptions.
```

Text height is NEVER FIXED. Not even for single-line text. The height is always
determined by font size + line height.

### 1c. Figma `textAutoResize` Mapping

| Width mode | Height mode | Figma `textAutoResize` | Compose equivalent |
|---|---|---|---|
| FILL | HUG | `"HEIGHT"` | `Modifier.fillMaxWidth()` (default wrapping) |
| HUG | HUG | `"WIDTH_AND_HEIGHT"` | No width modifier (default) |
| FIXED | HUG | `"HEIGHT"` with explicit width | `Modifier.width(value.dp)` — rare, avoid |
| Any | FIXED | `"NONE"` | **NEVER USE** — text will clip |

### 1d. Platform Implementation

| Mode | Figma API | Compose |
|---|---|---|
| FILL width | `text.textAutoResize = "HEIGHT"` + `text.layoutSizingHorizontal = "FILL"` (after appendChild) | `Modifier.fillMaxWidth()` or parent Column with `fillMaxWidth()` |
| HUG width | `text.textAutoResize = "WIDTH_AND_HEIGHT"` | No width modifier (default) |
| HUG height | Automatic with `"HEIGHT"` or `"WIDTH_AND_HEIGHT"` | Default Text behavior |

### 1e. Hidden Text Nodes

**Hidden text nodes MUST have the same sizing as their visible counterpart.**

| Current (wrong) | Correct |
|---|---|
| Hidden subtext: FIXED×FIXED | FILL×HUG (`textAutoResize = "HEIGHT"`) |
| Hidden subtitle: FIXED×FIXED | FILL×HUG |

This is a systemic bug found in DS source components (Input-Field subtext,
Bottom Sheet Header subtitle, Top App Bar Big subheading). When hidden elements
become visible with wrong sizing, layout breaks silently.

---

## 2. TYPOGRAPHY SCALE — WIOM DS (22 Styles)

### 2a. Complete Type Scale

| Token | Size | Weight | Line Height | Use |
|---|---|---|---|---|
| `D1_48_Bold` | 48 | Bold | 48 | Hero display numbers |
| `D2_40_Bold` | 40 | Bold | 40 | Large display |
| `T1_32_Bold` | 32 | Bold | 32 | Major section title |
| `T2_24_Bold` | 24 | Bold | 32 | Dialog title, Big app bar, Bottom sheet heading |
| `T3_20_Bold` | 20 | Bold | 24 | Screen title |
| `T3_20_Semibold` | 20 | SemiBold | 24 | Emphasized screen title |
| `T4_16_Bold` | 16 | Bold | 24 | Section header, CTA label, Input title (empty) |
| `T4_16_Semibold` | 16 | SemiBold | 24 | Emphasized section |
| `T4_16_Medium` | 16 | Medium | 24 | Medium emphasis |
| `T4_16_Regular` | 16 | Regular | 24 | Input title (filled), body featured |
| `L1_16_Bold` | 16 | Bold | 24 | Label bold |
| `L1_16_Regular` | 16 | Regular | 24 | Label regular |
| `L2_14_Semibold` | 14 | SemiBold | 20 | Small CTA label, chip, emphasized body |
| `L2_14_Regular` | 14 | Regular | 20 | Navigation label, subtext, helper |
| `L2_14_Medium` | 14 | Medium | 20 | Medium emphasis label |
| `L3_12_Semibold` | 12 | SemiBold | 16 | Chip/tag label, small emphasis |
| `L3_12_Regular` | 12 | Regular | 16 | Tertiary info, timestamps |
| `B1_16_Regular` | 16 | Regular | 24 | Featured body |
| `B2_14_Regular` | 14 | Regular | 20 | Standard body text |
| `B3_12_Regular` | 12 | Regular | 16 | Secondary info |
| `B4_10_Regular` | 10 | Regular | 16 | Tertiary/hint, version |
| `B4_10_Medium` | 10 | Medium | 16 | Small emphasis |

### 2b. Allowed Font Sizes

The ONLY legal font sizes:
```
10  12  14  16  20  24  32  40  48
```

Any size not in this set is a FAIL.

### 2c. Context → Style Mapping

| Context | Style | Size/Weight |
|---|---|---|
| Screen title (app bar) | T3_20_Bold | 20/Bold |
| Screen heading group — title | T3_20_Bold | 20/Bold, color #161021 |
| Screen heading group — subtitle | T4_16_Regular | 16/Regular, color #665E75 |
| Big app bar title | T2_24_Bold | 24/Bold |
| Dialog title | T2_24_Bold | 24/Bold |
| Bottom sheet title | T2_24_Bold | 24/Bold |
| Acknowledgement headline | T2_24_Bold | 24/Bold, color **#6D17CE** (info/600) |
| Section heading | T4_16_Bold | 16/Bold |
| CTA label (standard) | T4_16_Bold | 16/Bold |
| CTA label (small/coupon) | L2_14_Semibold | 14/SemiBold |
| Input field title (empty) | T4_16_Bold | 16/Bold |
| Input field title (filled) | T4_16_Regular | 16/Regular |
| Input field content | T4_16_Bold | 16/Bold (filled), T4_16_Regular (placeholder) |
| Input field placeholder | T4_16_Regular | 16/Regular, color #A7A1B2 |
| Body text | B2_14_Regular | 14/Regular |
| Dialog description | T4_16_Regular | 16/Regular |
| Helper/subtext | L2_14_Regular | 14/Regular |
| Error text | L2_14_Regular | 14/Regular, color #D92130 |
| Chip/tag | L3_12_Semibold | 12/SemiBold |
| Hint/version | B4_10_Regular | 10/Regular |
| Hero amount | D1_48_Bold or T1_32_Bold | 48 or 32/Bold |
| FAQ question | T4_16_Bold | 16/Bold |
| FAQ answer | B2_14_Regular or T4_16_Regular | 14 or 16/Regular |
| Menu item | (from Content frame) | — |
| Coupon link text | L2_14_Regular | 14/Regular, color #665E75 |

---

## 3. FONT FAMILY RULES

### 3a. Script Detection

```
Is the text string Devanagari (Hindi)?
  Test: /[\u0900-\u097F]/.test(string)
  → Font family: "Noto Sans Devanagari"

Is the text string Latin (English)?
  → Font family: "Noto Sans"

Is the text mixed (Hindi + English in same string)?
  → Font family: "Noto Sans Devanagari" (has Latin fallback glyphs)
```

### 3b. Platform Implementation

**Figma:**
```js
const isHindi = /[\u0900-\u097F]/.test(text);
const family = isHindi ? "Noto Sans Devanagari" : "Noto Sans";
await figma.loadFontAsync({ family, style: weightName });
node.fontName = { family, style: weightName };
```

**Compose:**
```kotlin
val isHindi = text.any { it.code in 0x0900..0x097F }
val fontFamily = if (isHindi) notoSansDevanagari else notoSans
Text(text, fontFamily = fontFamily, ...)
```

### 3c. Critical Rules

| Rule | Why |
|---|---|
| NEVER use Inter in production | Inter is for Figma prototyping only |
| NEVER bind `fontFamily` variable to Hindi text in Figma | DS Font/Font Family variable = "Noto Sans" — would break Devanagari |
| Hindi text: bind fontSize + fontStyle + lineHeight individually | No textStyleId since DS doesn't publish Devanagari text styles |
| English text: use `textStyleId` (via `importStyleByKeyAsync`) | DS-correct pattern from reference frames |
| Mixed text: use Noto Sans Devanagari | Has Latin fallback glyphs |

### 3d. Weight Naming

| Family | Bold | SemiBold | Medium | Regular |
|---|---|---|---|---|
| Noto Sans | `"Bold"` | `"SemiBold"` | `"Medium"` | `"Regular"` |
| Noto Sans Devanagari | `"Bold"` | `"SemiBold"` | `"Medium"` | `"Regular"` |
| Inter (Figma only) | `"Bold"` | `"Semi Bold"` (SPACE) | `"Medium"` | `"Regular"` |

**Watch for the Inter `"Semi Bold"` trap.** If extracting specs from Figma where Inter was used, map `"Semi Bold"` → `"SemiBold"` for Noto Sans.

---

## 4. LINE HEIGHT RULES

### 4a. Line Height Table (from DS)

| Font size | Line height | Ratio |
|---|---|---|
| 10 | 16 | 1.6 |
| 12 | 16 | 1.33 |
| 14 | 20 | 1.43 |
| 16 | 24 | 1.5 |
| 20 | 24 | 1.2 |
| 24 | 32 | 1.33 |
| 32 | 32 | 1.0 |
| 40 | 40 | 1.0 |
| 48 | 48 | 1.0 |

**Pattern:** Small text (10-16) has generous line height (1.33-1.6×). Large text (32+) has tight line height (1.0×). This ensures readability for body text and visual compactness for headings.

### 4b. Line Height in Compose

```kotlin
Text(
    text = "...",
    fontSize = 16.sp,
    lineHeight = 24.sp,  // ALWAYS explicit, never rely on default
)
```

**Rule: Always set line height explicitly.** Browser/system defaults vary. The DS specifies exact values.

---

## 5. TEXT ALIGNMENT

### 5a. Decision Tree

```
Is it body text, description, or helper text?
  → LEFT (TextAlign.Start)

Is it a heading, title, or section header?
  → LEFT (TextAlign.Start)

Is it a form label?
  → LEFT (TextAlign.Start)

Is it text inside a CTA button?
  → CENTER. Set `textAlignHorizontal = "CENTER"` on the text node itself.
    Do NOT rely on parent primaryAxisAlignItems — when text is FILL width,
    parent alignment has no effect. The text node must center its own characters (L101).

Is it an empty state message?
  → CENTER

Is it a dialog title?
  → LEFT (Wiom DS uses left-aligned dialog titles, not centered)

Is it a loading/processing message?
  → CENTER

Is it a hero number or amount?
  → LEFT (unless it's a centered display element)

Is it a price in a badge?
  → CENTER (inside badge container)
```

**Rule: LEFT is the default. CENTER is the exception.** Center-aligned body text is ALWAYS wrong.

### 5b. Checkbox + Multi-line Text Alignment

When a checkbox/radio sits next to multi-line text:
```
Row (HORIZONTAL, gap:8, crossAlign: MIN)  ← TOP-aligned, not CENTER
├── checkbox (24×24 FIXED)
└── text (FILL×HUG, multi-line wrapping)
```
CrossAlign = **MIN (top)**, not CENTER. The checkbox aligns with the first line of text.
CENTER would push the checkbox to the middle of a long text block — visually wrong.

### 5c. Acknowledgement/Confirmation Headlines

Acknowledgement screens use **info/600 (#6D17CE)** for the headline — not neutral/900.
The purple accent creates emphasis for important announcements/confirmations.
Body text below stays neutral/900 or #000000.

### 5d. Platform Implementation

| Alignment | Figma | Compose |
|---|---|---|
| LEFT | `text.textAlignHorizontal = "LEFT"` | `textAlign = TextAlign.Start` (default) |
| CENTER | `text.textAlignHorizontal = "CENTER"` | `textAlign = TextAlign.Center` |
| RIGHT | `text.textAlignHorizontal = "RIGHT"` | `textAlign = TextAlign.End` |

---

## 6. TRUNCATION & OVERFLOW

### 6a. Decision Tree

```
Is the text a title or heading in a constrained width (app bar, list row)?
  → maxLines: 1, overflow: ellipsis

Is the text a name or dynamic string in a HORIZONTAL row?
  → maxLines: 1, overflow: ellipsis, width: weight(1f) / FILL

Is the text body copy in a card?
  → maxLines: unlimited (or explicit limit like 3), height: HUG

Is the text a description in a dialog?
  → maxLines: unlimited, height: HUG

Is the text a CTA label?
  → maxLines: 1, NO overflow (label must always fit — shorten the label)

Is the text a price or number?
  → maxLines: 1, NO overflow (numbers don't truncate)
```

### 6b. Platform Implementation

**Figma:**
```js
text.maxLines = 1;
text.textTruncation = "ENDING"; // adds "..."
```

**Compose:**
```kotlin
Text(
    text = title,
    maxLines = 1,
    overflow = TextOverflow.Ellipsis,
)
```

### 6c. Rules

| Rule | PASS condition |
|---|---|
| No text clips without explicit maxLines | If text truncates, maxLines must be set |
| No text overflows its container | Text width ≤ container content width |
| CTA labels never truncate | Label fits within CTA at minimum CTA width (120dp) |
| Numbers/prices never truncate | Use HUG width for price/number text |
| Test with 2× content length | Every text container survives doubled content |
| Test at 320dp width | Nothing clips at smallest supported device width |

---

## 7. TEXT IN HORIZONTAL ROWS

The most common pattern in the DS. Text sits between FIXED icons.

### 7a. Standard Pattern

```
Row (HORIZONTAL, gap:8-12, crossAlign:CENTER)
├── icon (FIXED 24×24)
├── text content (FILL × HUG)
│   └── Text (FILL × HUG, maxLines: 1-2, overflow: ellipsis)
└── action icon (FIXED 24×24)
```

**The text element (or its parent frame) uses FILL width.** It absorbs all remaining
space after the FIXED icons. On wider screens, text gets more room. On narrower
screens, text gets less room and truncates.

### 7b. Text + Value Row (key-value)

```
Row (HORIZONTAL, SPACE_BETWEEN)
├── label (FILL × HUG, maxLines: 1, ellipsis)
└── value (HUG × HUG, maxLines: 1)
```

Label fills and truncates. Value hugs and never truncates. This ensures the value
(usually a number or price) is always fully visible.

### 7c. Title + Subtitle Stack

```
Column (VERTICAL, gap: 4)
├── title (FILL × HUG, maxLines: 1, fs:16/Bold)
└── subtitle (FILL × HUG, maxLines: 1-2, fs:14/Regular)
```

Both text nodes use FILL width and HUG height. Gap between title and subtitle: 4dp.

---

## 8. FONT WEIGHT HIERARCHY

### 8a. Weight Inversion Rule (from Input-Field training)

In label + control pairs:
```
Field empty:  title = Bold,    content = Regular  (title carries weight)
Field filled: title = Regular,  content = Bold     (content takes over)
```

**Only one element in a pair should be Bold at a time.** The element with user
attention gets the visual weight.

### 8b. Weight Scale

| Weight name | CSS value | Use |
|---|---|---|
| Bold | 700 | Headings, active input content, CTA labels, emphasis |
| SemiBold | 600 | Small CTA labels, chip labels, secondary emphasis |
| Medium | 500 | Status bar time, medium emphasis |
| Regular | 400 | Body text, descriptions, placeholders, receded labels |

### 8c. Maximum Weights Per Screen Section

**Maximum 3 weights per visible screen section.** Typically: Bold (headings/CTAs) + Regular (body) + one of SemiBold or Medium for secondary elements.

---

## 9. TEXT COLOR RULES

| Context | Color token | Hex |
|---|---|---|
| Primary text (headings, titles, body) | neutral/900 | #161021 |
| Secondary text (subtitles, helper) | neutral/700 | #665E75 |
| Hint/placeholder | neutral/500 | #A7A1B2 |
| Disabled text | neutral/500 | #A7A1B2 |
| Error text | negative/600 | #D92130 |
| Success text | positive/600 | #008043 |
| CTA text (on primary) | neutral/white | #FAF9FC |
| CTA text (on secondary tonal) | brand/600 | #D9008D |
| Link/brand text | brand/600 | #D9008D |
| Pre-filled text | neutral/700 | #665E75 |
| OTP cursor | brand/600 | #D9008D |

---

## 10. BILINGUAL TEXT BEHAVIOR

### 10a. Hindi vs English Length

Hindi text is typically 1.3-1.5× longer than English for the same content.
Test every screen with Hindi strings at expected maximum length.

### 10b. Font Metrics Difference

Hindi (Noto Sans Devanagari) has slightly different metrics than English (Noto Sans).
Same font size will render at slightly different visual heights. This is handled by
the shared line-height values in the DS — both families use the same line-height
for a given font size, ensuring consistent vertical rhythm.

### 10c. Mixed Language Cards

When a card has both Hindi and English text (e.g., Hindi heading + English value):
- Each text node uses the appropriate font family
- Line heights stay consistent (from DS table)
- Vertical gap between text nodes (4-8dp) is the same regardless of language

---

## 11. VALIDATION CHECKLIST

Binary pass/fail. Run on every screen.

### Sizing
| # | Rule | PASS condition |
|---|------|---------------|
| T1 | No FIXED height text | Zero text nodes have explicit fixed height |
| T2 | No FIXED width body text | Body/heading/label text all use FILL width |
| T3 | Hidden text correct sizing | Hidden text has FILL×HUG (not FIXED×FIXED) |
| T4 | HUG width only for inline | Only badges, prices, chip labels use HUG width |

### Typography
| # | Rule | PASS condition |
|---|------|---------------|
| T5 | Font size on scale | Every text uses size ∈ {10,12,14,16,20,24,32,40,48} |
| T6 | Font family correct | Hindi → Noto Sans Devanagari, English → Noto Sans |
| T7 | No Inter in production | Zero text nodes use Inter family |
| T8 | Line height explicit | Every text has explicit line-height from DS table |
| T9 | Weight matches context | Title/CTA = Bold, Body = Regular, etc. per §2c |
| T10 | Max 3 weights per section | No screen section uses >3 different weights |

### Alignment & Overflow
| # | Rule | PASS condition |
|---|------|---------------|
| T11 | No centered body text | Body, description, form labels all left-aligned |
| T12 | Truncation has maxLines | If text truncates, maxLines is explicitly set |
| T13 | Numbers never truncate | Price/amount text uses HUG width |
| T14 | 320dp width test | No text clips at narrowest device width |
| T15 | 2× content test | No layout breaks with doubled text content |

### Color
| # | Rule | PASS condition |
|---|------|---------------|
| T16 | Text color from DS | Every text color ∈ DS token set (§9) |
| T17 | Error text is negative/600 | Error messages use #D92130 |
| T18 | Placeholder is neutral/500 | Hint text uses #A7A1B2 |

---

## 12. ANTI-PATTERNS

| Anti-pattern | Why wrong | Correct |
|---|---|---|
| `textAutoResize = "NONE"` | Text clips when content changes | `"HEIGHT"` or `"WIDTH_AND_HEIGHT"` |
| Fixed height on text node | Content change → clip or empty space | HUG (always) |
| Fixed width on body text | Language change → clip | FILL width |
| Inter font in production Compose | Not the DS font | Noto Sans / Noto Sans Devanagari |
| Binding fontFamily variable to Hindi text | Resolves to "Noto Sans" → breaks | Set family explicitly, bind size/weight/lineHeight |
| Center-aligned body text | Slower to scan, wrong for content | Left-align (Start) |
| No maxLines on dynamic text in rows | Overflows horizontally | Set maxLines + ellipsis |
| Relying on default line height | Varies by platform/device | Always set explicitly |
| Hardcoded font sizes (not from scale) | Inconsistent typography | Use DS type scale only |
| Same font weight for title and content | No hierarchy | Use weight inversion (§8a) |
| Hidden text with FIXED sizing | Breaks when shown | Match visible sibling sizing |

---

## 13. COMPOSE TEMPLATES

### 13a. Basic Text

```kotlin
Text(
    text = "हेडिंग टेक्स्ट",
    fontSize = 16.sp,
    fontWeight = FontWeight.Bold,
    fontFamily = notoSansDevanagari,  // detected from content
    lineHeight = 24.sp,
    color = WiomColors.neutral900,
    maxLines = 1,
    overflow = TextOverflow.Ellipsis,
    modifier = Modifier.fillMaxWidth()
)
```

### 13b. Title + Subtitle Stack

```kotlin
Column(verticalArrangement = Arrangement.spacedBy(4.dp)) {
    Text(
        text = title,
        fontSize = 16.sp,
        fontWeight = FontWeight.Bold,
        lineHeight = 24.sp,
        maxLines = 1,
        overflow = TextOverflow.Ellipsis,
        modifier = Modifier.fillMaxWidth()
    )
    Text(
        text = subtitle,
        fontSize = 14.sp,
        fontWeight = FontWeight.Normal,
        lineHeight = 20.sp,
        color = WiomColors.neutral700,
        modifier = Modifier.fillMaxWidth()
    )
}
```

### 13c. Text in Horizontal Row

```kotlin
Row(
    modifier = Modifier.fillMaxWidth(),
    horizontalArrangement = Arrangement.spacedBy(8.dp),
    verticalAlignment = Alignment.CenterVertically
) {
    Icon(icon, modifier = Modifier.size(24.dp))        // FIXED
    Text(
        text = label,
        modifier = Modifier.weight(1f),                 // FILL remaining
        maxLines = 1,
        overflow = TextOverflow.Ellipsis,
        fontSize = 16.sp,
        lineHeight = 24.sp,
    )
    Icon(Icons.chevronRight, modifier = Modifier.size(24.dp))  // FIXED
}
```

### 13d. Key-Value Row

```kotlin
Row(
    modifier = Modifier.fillMaxWidth(),
    horizontalArrangement = Arrangement.SpaceBetween,
) {
    Text(
        text = label,
        modifier = Modifier.weight(1f, fill = false),   // fills but doesn't push value
        maxLines = 1,
        overflow = TextOverflow.Ellipsis,
        color = WiomColors.neutral700,
    )
    Text(
        text = value,                                     // HUG — never truncates
        fontWeight = FontWeight.Bold,
    )
}
```

---

## 14. FIGMA TEXT NODE TEMPLATES

### 14a. FILL Width Text (heading, body, label)

```js
const text = figma.createText();
const isHindi = /[\u0900-\u097F]/.test(content);
const family = isHindi ? "Noto Sans Devanagari" : "Noto Sans";
await figma.loadFontAsync({ family, style: "Bold" });
text.fontName = { family, style: "Bold" };
text.characters = content;
text.fontSize = 16;
text.lineHeight = { value: 24, unit: "PIXELS" };
text.textAutoResize = "HEIGHT";  // width FILL, height HUG
// After appending to auto-layout parent:
text.layoutSizingHorizontal = "FILL";
```

### 14b. HUG Width Text (badge, price, chip)

```js
const text = figma.createText();
await figma.loadFontAsync({ family: "Noto Sans", style: "SemiBold" });
text.fontName = { family: "Noto Sans", style: "SemiBold" };
text.characters = "₹5,200";
text.fontSize = 14;
text.lineHeight = { value: 20, unit: "PIXELS" };
text.textAutoResize = "WIDTH_AND_HEIGHT";  // both HUG
// Do NOT set layoutSizingHorizontal = "FILL"
```

---

## 15. WALLET / BALANCE DISPLAY PATTERN (L126)

Hero amounts (wallet balance, earnings) use centered layout:

```
Card (FILL×HUG, VERTICAL, gap:16, p:16, r:16, tinted bg)
└── Center section (HUG×HUG, VERTICAL, gap:4, crossAlign:CENTER)
    ├── Amount (HUG×HUG, fs:24-32/Bold, #161021)
    └── Label (HUG×HUG, fs:14/Regular, #665E75)
```

Amount text: HUG width (numbers don't fill), CENTER aligned.
This is one of the few cases where CENTER alignment and HUG width on text is correct.

---

## 16. DELIVERY / CONFIRMATION HEADING (L127)

Confirmation screens use T3_20_Bold heading + content card below:

```
Column (FILL×HUG, gap:24)
├── Heading (FILL×HUG, fs:20/Bold, #161021) — "10 नेट बॉक्स की डिलीवरी कन्फर्म करें"
└── Content card (FILL×HUG, VERTICAL, gap:16, r:12, p:16)
    └── [checkbox items, details]
```

Gap between heading and card: 24dp (section gap).

---

## 17. QUANTITY COUNTER TEXT (L171)

Counter uses a SINGLE text node "–  1  +" (not 3 separate nodes):
- fs:16/Bold, color:#FAF9FC
- textAlign: CENTER
- Width: HUG (sizes to characters)
- Spacing between minus/number/plus via literal spaces in the string

In Compose: use separate `IconButton(-)` + `Text(count)` + `IconButton(+)` for interactivity.
The Figma single-text approach is a visual mockup; production needs individual tappable elements.

---

## 18. ERROR BANNER TEXT (L162)

Error banner text:
- fs:14/Regular, color:#FAF9FC (white on red bg)
- Width: FILL inside HORIZONTAL row with icon
- Placed after a 20-24dp icon, gap:8

---

## 19. SPAWNING PROTOCOL

### When Pratibimb spawns this skill (Figma):
1. Use §1 sizing tree for every text node created
2. Use §3 font family detection for every string
3. Use §2 type scale — no off-scale sizes
4. Run §11 validation after frame creation
5. Check §1e for hidden text nodes

### When Maverick spawns this skill (Compose):
1. Use §2c context→style mapping for every Text composable
2. Use §3b font detection pattern
3. Use §4b — always explicit line height
4. Use §13 templates as starting points
5. Centralize all font sizes in Dimens object
6. Run §11 validation during audit

### Cross-platform parity:
After building in both Figma and Compose, verify:
- Same font family per text node
- Same font size
- Same font weight
- Same line height
- Same text alignment
- Same truncation behavior (maxLines, ellipsis)
- Same text color
