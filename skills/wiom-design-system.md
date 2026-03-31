# Wiom Design System — Pratibimb Reference
**Source:** CA_Design-System (`T0klEs1aPBk7BOVZonc8JC`) · Final Documentation node `7154:25059`
**Authority:** This file overrides generic UI Designer defaults when working on any Wiom product.

---

## 1. COLORS

### Brand Colors
| Token | Hex | Usage |
|-------|-----|-------|
| `brand/600` | `#D9008D` | Primary CTAs, active states, links, chevrons |
| `brand/secondary` | `#443152` | Partner app header background |

### Neutral Scale
| Token | Hex | Usage |
|-------|-----|-------|
| `neutral/900` | `#161021` | Primary text (`pri_text`) |
| `neutral/800` | `#352D42` | Dark icon circle backgrounds |
| `neutral/700` | `#665E75` | Secondary text (`sec_text`) |
| `neutral/500` | `#A7A1B2` | Hint text, disabled states, version strings |
| `neutral/300` | `#D7D3E0` | Borders, dividers |
| `neutral/200` | `#E8E4F0` | Subtle section backgrounds |
| `neutral/100` | `#F1EDF7` | Icon circle backgrounds (lavender tint) |
| `neutral/white` | `#FAF9FC` | Page background, card background |

### Semantic Colors
| Token | Hex | Usage |
|-------|-----|-------|
| `positive/600` | `#008043` | Success states, confirmations |
| `negative/600` | `#D92130` | Error states, destructive actions |
| `warning/600` | `#FF8000` | Warning states |
| `info/600` | `#6D17CE` | Info / purple accent |

### Special
| Token | Value | Usage |
|-------|-------|-------|
| `gold_gradient` | `#D5AF4A → #F2E983 → #D5B14B` | Lottery, premium, achievements |
| `Gold Gradient` | Figma fill style | Use only for reward/incentive moments |

### Color Rules for Wiom
- NEVER use pure `#000000` or `#FFFFFF` — always use the neutral scale
- Brand pink (`#D9008D`) is for ONE primary action per screen — do not use for decorative elements
- Partner app header is always `brand/secondary` (`#443152`)
- Background hierarchy: `neutral/white` → `neutral/100` → `neutral/200` (never random greys)
- Success = green (`positive/600`), Error = red (`negative/600`) — non-negotiable semantic meaning

---

## 2. TYPOGRAPHY

**System:** Atomic design system — 22 defined text styles
**Primary fonts:** Noto Sans (English/Latin) + Noto Sans Devanagari (Hindi/Devanagari script)
**Script detection:** Use `/[\u0900-\u097F]/` regex — auto-switch font family
**Font style naming:** Inter uses `"Semi Bold"` (space); Noto Sans uses `"SemiBold"` (no space)

### Type Scale (22 styles)

| Category | Style | Size | Weight | Token Name |
|----------|-------|------|--------|------------|
| Display | D1 | 48px | Bold | `Display/D1_48_Bold` |
| Display | D2 | 40px | Regular | `Display/D2_40_Regular` |
| Title | T1 | 32px | Bold | `Title/T1_32_Bold` |
| Title | T2 | 24px | Bold | `Title/T2_24_Bold` |
| Title | T2 | 24px | Regular | `Title/T2_24_Regular` |
| Title | T3 | 20px | Bold | `Title/T3_20_Bold` |
| Title | T3 | 20px | Medium | `Title/T3_20_Medium` |
| Title | T3 | 20px | Regular | `Title/T3_20_Regular` |
| Title | T4 | 16px | Bold | `Title/T4_16_Bold` |
| Title | T4 | 16px | Regular | `Title/T4_16_Regular` |
| Label | L1 | 16px | Bold | `Label/L1_16_Bold` |
| Label | L1 | 16px | Regular | `Label/L1_16_Regular` |
| Label | L2 | 14px | Semibold | `Label/L2_14_Semibold` |
| Label | L2 | 14px | Regular | `Label/L2_14_Regular` |
| Label | L3 | 12px | Semibold | `Label/L3_12_Semibold` |
| Body | B1 | 16px | Bold | `Body/B1_16_Bold` |
| Body | B1 | 16px | Semibold | `Body/B1_16_Semibold` |
| Body | B1 | 16px | Medium | `Body/B1_16_Medium` |
| Body | B1 | 16px | Regular | `Body/B1_16_Regular` |
| Body | B2 | 14px | Semibold | `Body/B2_14_Semibold` |
| Body | B2 | 14px | Medium | `Body/B2_14_Medium` |
| Body | B2 | 14px | Regular | `Body/B2_14_Regular` |
| Body | B3 | 12px | Regular | `Body/B3_12_Regular` |
| Body | B4 | 10px | Regular | `Body/B4_10_Regular` |

### Typography Rules for Wiom
- Use ONLY the 22 defined styles — no custom font sizes
- App bar / screen titles: `Title/T3_20_Bold` or `T4_16_Bold`
- Body copy: `Body/B2_14_Regular` or `B1_16_Regular`
- CTA labels: `Label/L2_14_Semibold` or `Body/B2_14_Semibold`
- Hint / secondary: `Body/B3_12_Regular` or `B4_10_Regular`
- Hindi text always uses Noto Sans Devanagari — never Inter for Devanagari script
- Maximum 3 type styles per screen section

---

## 3. SPACING

**System:** Multiples of 4px — aligned with Google Material Design guidelines
**Scale:** `4, 8, 12, 16, 24, 32, 40, 48, 64, 72, 84px`

### Common Patterns
- Screen horizontal padding: `16px` (occasionally `20px` in prototypes)
- Between cards: `16px`
- Inside card padding: `12–16px`
- App bar height: `56px`
- Status bar height: `44px`
- Content top (below status + app bar): `100–116px`
- CTA button height: `48px`
- Bottom safe area / CTA margin from bottom: `24–32px`
- Icon circle: `48×48px` with `cornerRadius: 24`
- Icon inside circle: `24×24px`

### Spacing Rules
- NEVER use values not on the 4px grid (no 10px, 11px, 15px, etc.)
- Component internal spacing must be ≤ external spacing between components

---

## 4. BORDER RADIUS

**System:** Multiples of 4px
| Element | Radius |
|---------|--------|
| Large cards, sheets | `16px` |
| Medium cards, inputs | `12px` |
| Small chips, tags | `8px` |
| Pills / fully rounded | `100px` (or 50% of height) |
| Icon circles (48px container) | `24px` |
| CTAs / buttons | `12px` |
| Nested elements | Always smaller than parent |

---

## 5. ELEVATION / SHADOWS

**Defined Figma styles:**
| Style | Use |
|-------|-----|
| `Elevation-DOWN/1dp-shadow` | Cards, subtle lift |
| `Elevation-DOWN/2dp-shadow` | Modals, popovers |
| `Elevation-DOWN/4dp-shadow` | Dialogs, bottom sheets |
| `Elevation-DOWN/4dp-shadow-stronger` | High-emphasis overlays |
| `Elevation-UP/-2dp-shadow` | Inset / pressed states |
| `Elevation-UP/-4dp shadow` | Deep inset states |
| `Glow/pink-glow-4dp` | Brand CTA glow (use sparingly) |
| `Glow/green-glow-4dp` | Success / positive glow |

---

## 6. ICONS

**Source:** Google Material Icons library + custom Wiom icons from CA_Design-System
**Size standard:** 24×24px icon inside 48×48px circle frame
**Icon circle backgrounds:** `neutral/100` (#F1EDF7) for light, `neutral/800` (#352D42) for dark
**CRITICAL:** Never set `fills` on icon component instances — it overpaints the SVG. Use `inst.fills = []`

### Key Icon Component Keys (for Figma API import)
| Icon | Key | Usage |
|------|-----|-------|
| `arrow_back` | `4c6e3e9f0c66fbe464b1561e9143f1e74f3b0b35` | App bar back |
| `home_wifi` | `8a4e64653cdfdd777949f4327962c1676b23fd8d` | Net Box / WiFi home |
| `wifi` | `44d8397990c5c2694f8bee21b0da459da29b0aeb` | WiFi icon |
| `Router` | `2ac0f9783700fb159e62b2be5f7036762c7d7d2f` | Net Box / router |
| `rupee_reciept` | `a8c24c3860db8e68850113172a0aefbdcfdb2ed9` | Wallet / payment |
| `online_payment` | `fdf395dfe1deee28a397adcefdcb5240bbfaf72e` | Online payment |
| `ticket` | `aed4f38072d7ccee872d62637fdcee0003d02c8e` | Partner success fee |
| `receipt` | `9f5a532f18be27cd6857138436dd5f9db3cfbb72` | Bill / receipt |
| `info_filled` | `e0fd5f6f9eeecdefd351c3000f61f2cccde6e39e` | Info / trust line |
| `info_outline` | `05873f986d173cb4cb7bfbbfe589d34dc830dd89` | Info outline |
| `settings` | `7c1b7728da1b05cf6b84f94b09203bdb669e8569` | Settings |
| `help_center` | `7fb2628d9f2fa69b4eb520fbf2ec67d11bf4e9ad` | FAQ / help |
| `language` | `06bba4da5148d7a114aa967d4c34d8810aff344c` | Language selector |
| `logout` | `5930a8b6cb4e4e53eee6f508e017089dfb8a9035` | Logout |
| `stars` | `e15d1425109c29941756a7aed01d226133401a95` | Achievements |
| `discount` | `672bfb414f8d88c2237c4c206ba2882a9095000e` | Offers / discount |
| `annoucement` | `230b75521f097976f2a921d46df87d72567797f9` | Announcements |

---

## 7. COMPONENT PATTERNS

### App Bar
- Height: `56px` + `44px` status bar = `100px` total top chrome
- Background: `brand/secondary` (#443152) for Partner app; `neutral/white` or `brand/600` tints for Customer app
- Back button: `arrow_back` icon, 24px, left-aligned at 16px
- Title: `Title/T3_20_Bold`, vertically centered, `neutral/white` on dark bar

### CTA Buttons
- Height: `48px`, `cornerRadius: 12px`
- Primary fill: `brand/600` (#D9008D) for customer app; `neutral/900` or `brand/600` for partner app
- Label style: `Label/L2_14_Semibold` or `Body/B2_14_Semibold`, centered
- Full-width on mobile (390 - 32px padding = 358px wide)
- ONE primary CTA per screen section

### Cards / List Rows
- Background: `neutral/white` (#FAF9FC)
- Border: `1px solid neutral/300` (#D7D3E0)
- Corner radius: `12px`
- Padding: `16px`
- Shadow: `Elevation-DOWN/1dp-shadow`

### Status / Chips
- Height: `28–32px`, `cornerRadius: 100px` (pill)
- Background varies by state: success=`positive/600` tint, error=`negative/600` tint, neutral=`neutral/100`
- Label: `Label/L3_12_Semibold`

### OTP / Code Display
- Large display: `Display/D1_48_Bold` or `Title/T1_32_Bold`
- Background: `neutral/100` or `neutral/200`, `cornerRadius: 16px`
- Each digit box: `neutral/white` bg, `neutral/300` border, `12px` radius

---

## 8. BILINGUAL COPY RULES

- **Primary language:** Hindi (Devanagari script) for all customer-facing strings in Tier 2/3 context
- **Secondary language:** English for partner/expert app where literacy is higher
- **Romanized Hindi** (transliteration) is acceptable in wireframes and lo-fi stages
- **Font auto-detect:** `/[\u0900-\u097F]/` test → Noto Sans Devanagari; else Noto Sans
- Tone: conversational, warm, direct — never corporate or formal
- Error messages: always tell what happened + what to do next
- CTA labels: outcome-first ("₹300 wapas paayein" not "Submit return")

---

## 10. FIGMA TOKEN BINDING (Chrome DevTools MCP)

This section documents how to bind Wiom DS tokens to nodes programmatically via `evaluate_script`.

### Step 1 — Detect if DS is loaded
`figma.getLocalPaintStylesAsync()` returning 0 does NOT mean DS is absent — it means DS is an external library (correct). To confirm variables are accessible, call `figma.variables.getVariableByIdAsync(knownId)` — if it returns a variable object, the DS is loaded.

### Step 2 — Collect available variable IDs
Scan all pages to collect full variable IDs already used in the file:
```js
const allVarIds = new Set();
for (const page of figma.root.children) {
  function collect(n) {
    if (n.boundVariables) {
      for (const [, val] of Object.entries(n.boundVariables)) {
        const items = Array.isArray(val) ? val : [val];
        items.forEach(v => { if (v?.id) allVarIds.add(v.id); });
      }
    }
    if (n.children) n.children.forEach(collect);
  }
  collect(page);
}
// Then resolve: name → { id, key }
const byName = {};
for (const id of allVarIds) {
  const v = await figma.variables.getVariableByIdAsync(id);
  if (v) byName[v.name] = { id: v.id, key: v.key };
}
```

### Step 3 — Variable ID Reference (claude-playground file)
All IDs below are for the Wiom CA_Design-System library as loaded in `PwrhO9KnvdZ4NNYwz3B85d`.

**Color Variables:**
| Token name in DS | Variable ID | Semantic use |
|-----------------|-------------|--------------|
| `Colors/background/bg_neutral_pri` | `VariableID:bc80aa323958a1a8815b4c3122424e11ca0e294b/6280:34` | Page bg (neutral/white) |
| `Colors/background/bg_partner_header` | `VariableID:6db51d512e1f949e29285a1effc31fdf5784f600/2110:486` | Status bar + app bar (brand/secondary) |
| `Colors/background/bg_surface` | `VariableID:99796eefcd9d2b6b436d4153a949cd3daad085ee/7097:106` | Card / surface white |
| `Colors/background/bg_positive_pri` | `VariableID:bf215bf51f0ea45a6007c6b2cd1ced5f1903d493/7097:111` | Positive/success fill |
| `Colors/background/bg_negative_pri` | `VariableID:3692910b5a1f0f6936896189c64659c276c9ad08/...` | Error fill |
| `Colors/background/bg_neutral_sec` | `VariableID:2fa45cea8e7bdb80ccb341a8d0cc0d3ed15e7784/...` | Secondary neutral bg |
| `Colors/background/bg_brand_pri` | `VariableID:d570137d51ecb0978aa9c3c4d4108dd411656ae3/...` | Brand primary bg |
| `Colors/text/pri_text` | `VariableID:60ba8cadc843b04f74c4d19b7fcb71642c84b55b/7097:134` | Primary text (neutral/900) |
| `Colors/text/sec_text` | `VariableID:c26eeab61a181b478cdda7deb59a6fe031a8708a/7097:135` | Secondary text (neutral/700) |
| `Colors/text/hint_text` | `VariableID:822a55e1118fc173f218b928db2715de5f8758c1/...` | Hint / disabled text |
| `Colors/text/dark_fg` | `VariableID:da6ae9bdd965e05b12b7996358cad1e44014246e/2110:434` | Text on dark backgrounds |
| `Colors/text/positive_text` | `VariableID:edec13fff1b8fdbc7b443324b95767b16ec9bc6f/...` | Positive text |
| `Colors/text/negative_text` | `VariableID:572d0a21cbd5c1a987971c481166b50a7ecff8a2/...` | Negative/error text |
| `Colors/actions/pri_cta_bg` | `VariableID:d98c4edec099fb356272c9f7d78e1a845bd0d309/7097:142` | Primary CTA fill (brand/600) |
| `Colors/actions/pri_cta_fg` | `VariableID:7a42a41cf17a864b6841405abb1e67518b8dfab1/7097:143` | Text on primary CTA |
| `Colors/actions/sec_cta_bg` | `VariableID:e8b96aeb45b44de9cd0c8d96f7f3d87ea85640e9/...` | Secondary CTA fill |
| `Colors/actions/sec_cta_fg` | `VariableID:a33ccc460f884e4e11358a459026aa79275a1342/...` | Text on secondary CTA |
| `Colors/actions/link_label_fg` | `VariableID:9cc19c334de453b9a186280ae93e901bfe15e9c4/7097:137` | Link / brand-colored text |
| `Colors/stroke/stroke_neutral_pri` | `VariableID:99ce7c3d1445f1b6fac9c4fe6503ebb17dbf0cad/7097:123` | Dividers, borders |
| `Colors/stroke/stroke_neutral_sec` | `VariableID:faa0e224065be259904ca098f8f24dc15ea290f1/...` | Secondary borders |
| `Colors/stroke/stroke_brand` | `VariableID:896c691d10d92bc9669325f2c76710aa138f4b07/...` | Brand stroke |
| `Colors/icon/icon_positive` | `VariableID:78488e1847ad5a915d14c827ebc585c77ef8008d/2110:433` | Positive/success icon |
| `Colors/icon/icon_white` | `VariableID:7a986e81de1275806fa5c96d7398031c05f638bf/...` | White icon |
| `Colors/icon/icon_brand` | `VariableID:676fa11b3c4a85b25ff3a6009fb13e80dd035e1c/...` | Brand-colored icon |
| `Colors/icon/icon_neutral_action` | `VariableID:1c83ea90f1612670023c7aa80fb65593a521d14e/...` | Neutral actionable icon |

**Typography Variables:**
| Token name | Variable ID | Value |
|-----------|-------------|-------|
| `Font/Font Family` | `VariableID:fd87b9b2423ebebc2e54e4679ac848577c785a1a/6867:100` | "Noto Sans" — English only |
| `Weight/Regular` | `VariableID:5a9d63d27db3adbb8d1aa832652980ced10ef265/6867:101` | "Regular" |
| `Weight/SemiBold` | `VariableID:c309f3400e9756bc648443b103bfb36eeacec160/6867:102` | "SemiBold" |
| `Weight/Bold` | `VariableID:107acce85785a577f1296bf67defceda5436d0a4/6867:103` | "Bold" |
| `Size/12-dp` | `VariableID:58a9358d828027b4ea126076d8966e38cba6297f/6867:106` | 12 |
| `Size/14-dp` | `VariableID:1f460bfd6b7b236c31f77c601e84f6458268083f/6867:105` | 14 |
| `Size/16-dp` | `VariableID:48c9d53e18b6fde613f9abdd0cf856e6d913c81a/6867:104` | 16 |
| `Size/20-dp` | `VariableID:1e7a5e106269aaa4318de1e6dc0f9178346785f4/...` | 20 |
| `Size/24-dp` | `VariableID:235321b93c7421a8d83fd77158c6277620f19f8b/6867:109` | 24 |
| `Size/32-dp` | `VariableID:f795c8b32069a9d10201d02c08bd04991d4dadde/...` | 32 |
| `Size/40-dp` | `VariableID:fc9be64fbb68f1756f2ab2f8a8145d11b1dc544f/...` | 40 |
| `Size/48-dp` | `VariableID:64af65e2a7f5fbba069d575a9cfc73d851d2da09/...` | 48 |
| `Line Height/16\12-dp` | `VariableID:79d442bb4b97eab4c447b35fece99633400b9bcd/6867:114` | 16 |
| `Line Height/20\14-dp` | `VariableID:7a055203563331a2adbd94a334a1a4cde44ce5f1/6867:113` | 20 |
| `Line Height/24\16-dp` | `VariableID:af767cadd35fea8b5c15ed3912229fe91efcea73/6867:112` | 24 |
| `Line Height/28\20-dp` | `VariableID:1f921bf83781fb16faebd13fb51feb03c7f8874d/...` | 28 |
| `Line Height/32\24-dp` | `VariableID:c330a010390feb5ff257160d56cbe05a83f4309f/6867:116` | 32 |
| `Line Height/48\32-dp` | `VariableID:c8a166b4b154328d134911f19419405d50a82ba3/...` | 48 |

**Spacing Variables:**
| Token | Variable ID | Value |
|-------|-------------|-------|
| `System/spacing/space-4` | `VariableID:d97ebe3e608d5747e389ec59f633f2e7cd128a35/7097:95` | 4px |
| `System/spacing/space-8` | `VariableID:58622d7dfaa33abeb0f917b16f9e6d4054ef2431/7097:96` | 8px |
| `System/spacing/space-12` | `VariableID:ed3642e34cc338ece7f6d4c6946c2d4d0a7047d7/7097:97` | 12px |
| `System/spacing/space-16` | `VariableID:8e012bf07bb73cca435cd38ddbbd18544a7bc941/7097:98` | 16px |
| `System/spacing/space-24` | `VariableID:87c10ff3a9d7b74e2d6af7856772ff7ccc5529fc/7097:99` | 24px |
| `System/spacing/space-32` | `VariableID:3bf396b3c0a63b86bc9399f15b1b5de5aa765713/...` | 32px |
| `System/spacing/space-48` | `VariableID:b8e04bc9b01e94b29d0b7fb9a5bf0ac5991c75f5/...` | 48px |

**Radius Variables:**
| Token | Variable ID | Value |
|-------|-------------|-------|
| `System/radius/radius-main-card` | `VariableID:665d8b8ff535faa3371f2897cfb16c919fdbbd04/7097:88` | 12px (cards, CTAs) |
| `System/radius/radius-inner-card` | `VariableID:873eb1a88899e6a07b60b39ad139bc3438b74da7/...` | 8px (nested) |
| `System/radius/radius-large` | `VariableID:c309008d1ba7b7826e4114bd432bd3e9555db4a8/...` | 16px (dialogs) |
| `System/radius/radius-below-24dp` | `VariableID:7631f249dd10580c9a16660a7d956389bdfbd866/...` | small elements |
| `System/radius/radius-below-40dp` | `VariableID:6e331461e33c57b31d37f31ed4c9a3d2620d970f/...` | chips/tags |

**Published Text Style Keys (for `importStyleByKeyAsync`):**
| Style name | Key |
|-----------|-----|
| `Title/T2_24_Bold` | `6ee77bd636eca0d702d5b4e9a37dc89b0e708a97` |
| `Title/T3_20_Bold` | `f1874a9d0f346469854905dd4d6d6d8719ac8604` |
| `Title/T4_16_Bold` | `e13e73afa5bb1214357d51d4d1acc975895e8045` |
| `Title/T4_16_Regular` | `45f29a35be7e0e88b868f75a687b17c7dbd42b3d` |
| `Label/L1_16_Bold` | `85f6a6483ddf10d4ccd43b6a4a2bb7b4631a9042` |
| `Label/L2_14_Semibold` | `43ffc3d0e4180afd80b823088e6e00d466d0b59c` |
| `Label/L2_14_Regular` | `719121963b952d16d04080c0cec88bb838f149e9` |
| `Label/L3_12_Semibold` | `4f54ade8a716b00fea9e6eec2e0e3eba44dcf0c0` |
| `Body/B1_16_Bold` | `f42aba6f1936e1f624a34c423019f61d7b31bd07` |
| `Body/B1_16_Regular` | `368b1a0bf8aa710f63abee52f0ad31a5f8bdbd04` |
| `Body/B2_14_Semibold` | `f78eb6f46d2603906c615bacd0d149c17fe8b909` |
| `Body/B2_14_Regular` | `ba3bbf1e499da04a3fbd4f0e7c74af08e82d2348` |
| `Body/B3_12_Regular` | `0893e99d5250b9acd96a663ad36d57de9bc69e19` |
| `Elevation-DOWN/2dp-shadow` | `ada007f1e6031bce86fe3f1e328f1711451a0515` |

### Step 4 — Binding Patterns

**Fill binding (works in Chrome DevTools MCP):**
```js
const v = await figma.variables.getVariableByIdAsync(variableId);
const fills = [...node.fills];
fills[0] = figma.variables.setBoundVariableForPaint(fills[0], 'color', v);
node.fills = fills;
```

**Property binding (spacing, radius, font props):**
```js
const v = await figma.variables.getVariableByIdAsync(variableId);
node.setBoundVariable('fontSize', v);      // or padding, cornerRadius, etc.
```

**English text — full DS binding (textStyleId + color variable):**
```js
const textStyle = await figma.importStyleByKeyAsync(textStyleKey);
node.textStyleId = textStyle.id;           // sets size + weight + line height
const colorVar = await figma.variables.getVariableByIdAsync(colorVarId);
const fills = [...node.fills];
fills[0] = figma.variables.setBoundVariableForPaint(fills[0], 'color', colorVar);
node.fills = fills;
```

**Hindi text — individual variable bindings (no textStyleId):**
```js
// Load font first
await figma.loadFontAsync({ family: 'Noto Sans Devanagari', style: 'Bold' });
node.fontName = { family: 'Noto Sans Devanagari', style: 'Bold' }; // explicit, not variable
// Then bind size, weight, lineHeight as variables:
node.setBoundVariable('fontSize', sizeVar);
node.setBoundVariable('fontStyle', weightVar);
node.setBoundVariable('lineHeight', lhVar);
// Do NOT bind fontFamily — it resolves to "Noto Sans" which would break Devanagari rendering
const fills = [...node.fills];
fills[0] = figma.variables.setBoundVariableForPaint(fills[0], 'color', colorVar);
node.fills = fills;
```

### Step 5 — Critical Limitations
- **`figma.importVariableByKeyAsync` does NOT work in Chrome DevTools MCP** (browser context) — only in Figma plugin sandbox. Use `getVariableByIdAsync` with full IDs instead.
- **`figma.getLocalVariableCollectionsAsync()` returns [] for external libraries** — this is expected, not an error. Scan page boundVariables to find accessible variable IDs.
- **Radius:** Use `cornerRadius` for uniform radius; use `topLeftRadius/topRightRadius/bottomLeftRadius/bottomRightRadius` individually for asymmetric corners (both work with `setBoundVariable`).
- **DS does NOT publish Devanagari text styles** — Hindi text nodes can't use textStyleId cleanly. Bind individual font property variables instead.

---

## 9. PLATFORM CONTEXT

- **Customer App:** Android, Tier 2/3 India, Hindi-first, chat-based (Vyom AI bot)
- **Partner/Expert App:** Android, slightly more tech-literate, Hindi + English mixed
- **Screen size:** 390px wide (standard design canvas)
- **Touch targets:** minimum 44×44px, ideal 48×48px
- **Viewport:** 390 × 844px (standard phone frame for wireframes and hi-fi)
