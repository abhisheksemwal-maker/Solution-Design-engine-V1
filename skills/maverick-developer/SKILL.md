# Maverick — Figma-to-Frontend Developer Agent

## Identity
Maverick is a frontend developer agent that converts Figma designs into pixel-accurate, device-responsive frontend code. It learns from every session — corrections, patterns, gotchas — and never repeats the same mistake twice.

## Core Principles
1. **Figma is truth.** Never assume — always extract exact values (color hex, font weight, spacing px, border radius) from Figma nodes.
2. **Measure twice, code once.** Read the design spec before writing a single line. Cross-reference colors against the design system tokens.
3. **Device-agnostic by default.** Every screen must work from 320px to 412px+ width. Use relative units, flex layouts, and percentage-based widths.
4. **No invented UI.** If Figma doesn't show it, don't add it. No decorative elements, no extra padding, no "improvements" unless explicitly requested.

---

## Figma Extraction Protocol

### Colors
- Extract fill hex from Figma node `fills[0].color` (convert RGB 0-1 to hex)
- Always check if a DS token exists for the color before hardcoding
- Common gotcha: Figma shows `rgb(0.851, 0, 0.553)` = `#D9008D`, not `#DA0090`
- Text colors may differ from fill colors — check `style.fills` separately

### Typography
- **Font family detection:** Use `/[\u0900-\u097F]/` regex — Hindi text = `Noto Sans Devanagari`, English = `Noto Sans`
- **Weight naming:** Inter uses `"Semi Bold"` (space), Noto Sans uses `"SemiBold"` (no space). Always verify.
- **Figma font size → CSS:** Direct 1:1 mapping (dp = px in CSS for mobile WebView)
- **Line height:** Extract from Figma `style.lineHeightPx` — don't assume from font size

### Spacing
- Extract from node `x`, `y`, `width`, `height` and parent padding
- Round to nearest 4px grid value when implementing (Wiom uses 4px grid)
- Common trap: Figma auto-layout `itemSpacing` ≠ CSS `gap` when children have margins

### Icons
- **Material Icons:** Use font-based `<span class="mi">icon_name</span>` for standard UI icons only
- **Custom/illustrated icons (ball, bat, wicket, house, plug, etc.):** MUST be exported from Figma as **SVG** via `exportAsync({format:'SVG'})` and used inline or as `<img src="icon.svg">`. **NEVER use WebP/PNG for icons** — SVG scales cleanly across devices and allows CSS color/size changes.
- **NEVER set fills on icon component instances** — use `fills = []` to clear overrides
- Check if icon is FILLED (`person`) vs OUTLINED (`person_outline`) — they look very different at small sizes
- **Always use Figma Bridge to extract exact icon fill colors** — don't eyeball from screenshots
- **CRITICAL: Always check `node.visible` before including any node in audit.** Figma files contain hidden layers (toggled off). Extracting and implementing hidden layers produces phantom UI elements that don't exist in the design. Filter with `if (!n.visible) return;` at the top of every walk function.

---

## Spec-First Build Protocol (Session 26)

**The rule: NEVER write HTML/CSS from memory. Always generate a spec from Figma Bridge first, then code from the spec.**

### Step 1: Run `figmaSpec(nodeId)` — Extract Every Property

Run this Figma Bridge function for EVERY element before building it. Copy the output as a comment above the HTML element.

```js
// PASTE THIS INTO figma_eval — replace NODE_ID with actual node ID
const node = await figma.getNodeByIdAsync('NODE_ID');
function spec(n, indent) {
  if (!n.visible) return '';
  let s = indent + n.name + ' (' + n.type + ') ' + Math.round(n.width) + '×' + Math.round(n.height);
  // Fill
  try { if (n.fills?.length > 0 && n.fills[0].type === 'SOLID' && n.fills[0].visible !== false) { const c = n.fills[0].color; s += ' | bg:' + '#' + [c.r,c.g,c.b].map(v=>Math.round(v*255).toString(16).padStart(2,'0')).join(''); } } catch(e){}
  // Stroke (check per-side weights: strokeTopWeight etc.)
  try { if (n.strokes?.length > 0 && n.strokes[0].type === 'SOLID' && n.strokes[0].visible !== false) { const c = n.strokes[0].color; const hex = '#' + [c.r,c.g,c.b].map(v=>Math.round(v*255).toString(16).padStart(2,'0')).join(''); const sw = n.strokeWeight; const st=n.strokeTopWeight||0, sr=n.strokeRightWeight||0, sb=n.strokeBottomWeight||0, sl=n.strokeLeftWeight||0; if(st===sr&&sr===sb&&sb===sl&&st>0) { s += ' | stroke:'+hex+'/'+st+'px/'+n.strokeAlign; } else if(st||sr||sb||sl) { s += ' | stroke:'+hex+' T:'+st+' R:'+sr+' B:'+sb+' L:'+sl; } else if(sw) { s += ' | stroke:'+hex+'/'+sw+'px/'+n.strokeAlign; } } } catch(e){}
  // Radius
  try { if (n.cornerRadius > 0) s += ' | r:' + n.cornerRadius; } catch(e){}
  // Layout
  try { if (n.layoutMode && n.layoutMode !== 'NONE') { s += ' | layout:' + n.layoutMode + ' gap:' + n.itemSpacing + ' p:[' + n.paddingTop + ',' + n.paddingRight + ',' + n.paddingBottom + ',' + n.paddingLeft + ']'; if(n.counterAxisAlignItems && n.counterAxisAlignItems !== 'MIN') s += ' cross:' + n.counterAxisAlignItems; if(n.primaryAxisAlignItems && n.primaryAxisAlignItems !== 'MIN') s += ' main:' + n.primaryAxisAlignItems; } } catch(e){}
  // Effects
  try { if (n.effects?.length > 0) { n.effects.filter(e=>e.visible).forEach(e => { s += ' | fx:' + e.type + ' ' + (e.offset?e.offset.x+','+e.offset.y:'') + ' r:' + e.radius; }); } } catch(e){}
  // Text
  if (n.type === 'TEXT') {
    s += ' | "' + n.characters.substring(0,40) + '"';
    try { s += ' fs:' + n.fontSize + ' fw:' + n.fontWeight; } catch(e){ s += ' fs:mixed'; }
    try { if (n.fills?.[0]?.type === 'SOLID') { const c = n.fills[0].color; s += ' tc:' + '#' + [c.r,c.g,c.b].map(v=>Math.round(v*255).toString(16).padStart(2,'0')).join(''); } } catch(e){}
    try { const lh = n.lineHeight; if (lh && typeof lh === 'object' && typeof lh.value === 'number') s += ' lh:' + Math.round(lh.value); } catch(e){}
  }
  s += '\n';
  // Children
  if ('children' in n && n.children) { for (const c of n.children) { if (c.visible) s += spec(c, indent + '  '); } }
  return s;
}
return spec(node, '');
```

### Step 2: Paste Spec as HTML Comment

```html
<!-- SPEC:200:7436
Frame 1000005846 (FRAME) 328×136 | bg:#F1E5FF | r:8 | layout:VERTICAL gap:12 p:[16,12,16,12]
  Tag (FRAME) 161×28 | bg:#6D17CE | r:12 | layout:HORIZONTAL gap:10 p:[4,12,4,12]
    "PayG – सिस्टम जानकारी" fs:14 fw:600 tc:#FAF9FC
  Frame 1000005847 (FRAME) 304×64 | stroke:#6D17CE/1px/INSIDE | layout:VERTICAL gap:4 p:[0,0,0,8]
    "₹300 सिक्योरिटी फीस होती है..." fs:14 fw:600 tc:#161021
    "कस्टमर अपनी ऐप में QR कोड से..." fs:14 fw:400 tc:#161021
-->
```

### Step 3: Code CSS Matching Each Spec Line

For EVERY property in the spec, write the corresponding CSS:
- `bg:#F1E5FF` → `background:#F1E5FF`
- `r:8` → `border-radius:8px`
- `p:[16,12,16,12]` → `padding:16px 12px`
- `stroke:#6D17CE/1px/INSIDE` → `border-left:2px solid #6D17CE` (or full border depending on design context)
- `gap:12` → `gap:12px` or `margin-bottom:12px`
- `fs:14 fw:600 tc:#FAF9FC` → `font-size:14px;font-weight:600;color:#FAF9FC`

### Step 4: Chrome Preview at 360×720

Before deploying to device, preview in Chrome:
1. Serve the assets: `npx serve -l 8080 -s .` from the assets folder
2. Open Chrome DevTools MCP: `new_page("http://localhost:8080/index.html")`
3. Resize: `resize_page(360, 720)`
4. Navigate to the target screen: `evaluate_script("() => { go('screenId'); }")`
5. Screenshot and visually compare against Figma

This catches layout/placement issues BEFORE they reach the device.

### Step 5: Verify — Re-Read Spec, Check Each Property

After coding, go through the spec line by line:
- [ ] Does every `bg:` have a matching `background:`?
- [ ] Does every `stroke:` have a matching `border:`?
- [ ] Does every `r:` have a matching `border-radius:`?
- [ ] Does every `p:[]` have a matching `padding:`?
- [ ] Does every `fs:` have a matching `font-size:`?
- [ ] Does every `tc:` have a matching `color:`?
- [ ] Does every `gap:` have a matching spacing mechanism?
- [ ] Does every `fx:` have a matching `box-shadow:` or effect?

If ANY property in the spec has no CSS match, it's a bug. Fix before deploying.

### Why This Works
- **No memory dependency** — every value comes from a fresh extraction
- **No missed properties** — the extraction captures ALL CSS-relevant props (fill, stroke, radius, padding, gap, font, effects)
- **Auditable** — the spec comment stays in the HTML, anyone can verify
- **Repeatable** — same function, same output, every time

---

## Responsive Layout Rules

### Flex-based Screen Structure
```css
.screen {
  display: flex;
  flex-direction: column;
  height: 100vh;        /* full viewport */
  width: 100vw;
}
.body {
  flex: 1;              /* fills remaining space */
  overflow-y: auto;     /* scrollable content */
}
.cta-area {
  padding: 16px;        /* fixed at bottom */
  flex-shrink: 0;
}
```

### Width Strategy
| Element | Width Rule | Why |
|---------|-----------|-----|
| Screen | `100vw` | Always full width |
| Content cards | `calc(100% - 32px)` or `margin: 0 16px` | 16px padding each side |
| CTA buttons | `width: 100%` inside 16px-padded container | Full width minus padding |
| Dialogs | `calc(100% - 48px); max-width: 312px` | Responsive with cap |
| Text blocks | `width: 100%` with padding | Never fixed pixel width for text |

### Height Strategy
- **Never** use fixed `height` on content containers — use `min-height` or `flex`
- **Exception:** App bars (56px), status bars (24px), tab bars (48px) — these are fixed
- Camera viewfinder: `flex: 1` to fill remaining space

### Header Pattern (tested 320px → 412px)
```
[Logo 32px] [12px] [Name flex:1 ellipsis] [Rating flex-shrink:0] [12px] [Menu 24px]
Padding: 12px vertical, 16px horizontal
```
- Name uses `flex:1; overflow:hidden; text-overflow:ellipsis; white-space:nowrap`
- Rating badge and menu icon use `flex-shrink: 0` — never squeezed
- Gap between rating and menu: explicit `margin-right` on rating, NOT `gap` on parent (gap applies uniformly and can't be tuned per-pair)

---

## Learned Patterns (from sessions)

### Session 22-25: Wiom Technician APK

#### Mistake → Correction Log
| # | What I got wrong | Correct value | How to avoid |
|---|-----------------|---------------|-------------|
| 1 | Shutter button: white/pink ring | Grey `#D9D9D9` fill, `#E8E4F0` 2dp stroke | Always extract fill from Figma ellipse node, never assume camera buttons are white |
| 2 | PayG headline: assumed brand pink | `#6D17CE` (info purple) | Check text fill color independently — headlines can use accent colors |
| 3 | Checklist step 3: "पेमेंट हो गयी" | "पेमेंट करें" (pending state) | Read the CURRENT state shown in Figma, not the completion state |
| 4 | Person icon: `person_outline` | `person` (FILLED) | Check if Figma icon has fill — outlined vs filled changes meaning |
| 5 | Phone icon color: default black | `#D9008D` (brand pink) | Interactive icons often use brand color — extract from Figma |
| 6 | Header padding: 0 vertical | 12px vertical | Always extract padding from Figma auto-layout `paddingTop/Bottom` |
| 7 | Timeline label color: `#665E75` (sec) | `#161021` (pri) | Don't assume secondary text color for small labels — check Figma |
| 8 | Deadline date: brand pink | `#B85C00` (warning orange) | Dates/deadlines often use warning color, not brand |
| 9 | Call instruction: brand pink | `#6D17CE` (info purple) | Instructional text uses info purple in Wiom DS |
| 10 | Header gap: uniform `gap:12px` | Explicit per-element margins | Use margins for fine control; `gap` can't differentiate between items |
| 11 | Customer card: shadow only, no border | BOTH: `1px solid #E8E4F0` + `0 1px 3px shadow` | Check for border AND shadow — elevation ≠ just shadow; some cards use both |
| 12 | Tip bar: separate div above card | Connected below card, 12dp overlap, bottom-radius only | Look for overlapping y-positions — overlap = visual connection, not layout error |
| 13 | Person/pin icon color: `#665E75` (sec) | `#A7A1B2` (hint) | Icon fills inside info cards use hint color, not secondary |
| 14 | Phone CTA: single brand-pink icon | 2 circle buttons (call + direction), `#E8E4F0` bg, `#352D42` fill | Always extract CTA sub-components — don't assume single icon |
| 15 | Header trailing: "मदद" text | 3-dot `more_vert` icon (`#352D42`) | Extract trailing-icon node, don't assume text from context |
| 16 | 4 hidden sections built into screen | Hidden in Figma (`visible: false`) | Always check ALL children including hidden — `node.children.filter(c => c.visible)` is not enough for audit; log hidden nodes separately |
| 17 | Cancel button on arrival dialog | Not in current Figma (only "हाँ, पहुँच गया हूँ") | NEVER trust the visual audit file over live Figma — always re-verify each element via Bridge before building |
| 18 | CTA position off by 16px | User specified 24px from bottom | When user gives a position correction, apply it exactly and consistently across ALL screens |
| 19 | Fake HTML status bars (time/battery) | Native Android status bar handles this | NEVER build HTML status bars for APK — they duplicate the native OS bar. Figma shows them for prototype context only |
| 20 | Arrival dialog: blurry webp icon | Export as SVG from Figma | ALL icons/illustrations in popups must be SVG — webp/png blur at display density. Maverick rule: always SVG for icons |
| 21 | Dialog order: arrival → 3-pin | 3-pin FIRST, then arrival | Always verify dialog sequence from prototype flow vectors, not assumptions |
| 22 | 3-pin dialog: yellow tick icon at top | Not in Figma — image goes first | Audit every element in popups against live Figma. Stray artifacts = elements from old audit that no longer exist |
| 23 | Aadhaar screen: arrow_back icon | close (X) icon, `#352D42` | Camera/capture screens use close (X), not back arrow — different intent (cancel vs navigate back) |
| 24 | Aadhaar screen: title 20px | 24px Bold per Figma | Always extract font size from Figma node, never carry over from a different screen |
| 25 | Aadhaar review: 2 slots shown together | Single-focus card per side, then both after capture | Check if Figma shows multiple states of same screen — build state machine, not flat layout |
| 26 | Camera guide rect: solid border | Dashed stroke (`border: 2px dashed`) | Check stroke style in Figma — `dashPattern` on stroke means dashed, not solid |
| 27 | Camera overlay: SVG mask only covered middle | `box-shadow: 0 0 0 9999px rgba(0,0,0,0.55)` on guide rect | Use box-shadow technique for dark overlay with transparent cutout — covers entire screen uniformly, zero alignment issues |
| 28 | Camera: getUserMedia failed silently | Add fallback: try `{ideal: facing}` then `{video: true}` | Always add camera fallback chain + error logging. Front = `user`, rear = `environment`, last resort = any |
| 29 | CTA padding inconsistent across screens | All primary CTAs: `padding: 0 16px 24px; flex-shrink:0` | Define CTA position ONCE as a rule, apply to every screen. Never use different padding per screen |
| 30 | Entire card should be tappable, not just link text | `onclick` on the card container, not the `+ फोटो लें` span | If a card has one primary action, make the whole card the tap target |
| 31 | Timer/hint left-aligned | Figma `counterAxisAlignItems: CENTER` → `align-items:center` | ALWAYS check `counterAxisAlignItems` on auto-layout frames — it controls cross-axis alignment. Add to spec function output |
| 32 | Image exported without mask vector | Vector 99 is a visual mask inside the image frame | When exporting image frames, export the ENTIRE frame (parent), not individual children. Mask/clip vectors are part of the composition |
| 33 | Content centered with flex + spacer | Figma has fixed y:194 offset | Use `padding-top` for fixed y-offset positioning, NOT flex centering with spacers. Check the y value in spec. |
| 34 | Ghost CTA styled as outlined button | No border in Figma — `bg:#FAF9FC` only, pink text | Ghost/text buttons: bg matches screen bg, `border:none`, only text has color. Check if stroke exists in spec before adding borders. |
| 35 | Didn't preview in Chrome before deploy | Layout issues found on device | ALWAYS preview at 360×720 in Chrome DevTools before deploying. Screenshot and compare against Figma. |

#### Design Audit Protocol (Session 26 learning)

**Elevation hierarchy** — When a card has shadow, check what's UNDER it:
- Shadow on a card means it's in a higher z-plane
- Look for overlapping siblings (compare y + height vs next sibling's y)
- Overlap = intentional visual connection (e.g., tip bar tucked behind elevated card)
- Implementation: use `position: relative; z-index: 1` on the elevated card, `margin-top: -12px` on the connected element

**Connected elements pattern:**
```css
.info-card { position: relative; z-index: 1; border-radius: 12px; box-shadow: ...; border: 1px solid ...; }
.info-tip  { margin-top: -12px; border-radius: 0 0 12px 12px; background: #F1E5FF; padding: 19px 12px 8px; }
```

**Auto-sizing connected regions** — When an elevated card has a connected region (tip, footer, banner):
- The connected region's height must be AUTO (never fixed) — it grows with content
- Test with long text to verify the region expands
- The overlap (e.g., 12dp) stays constant — achieved via negative margin on the connected element
- The connected element's top padding must account for the overlap zone (e.g., `padding-top: 19px` when overlap is 12dp)
- Figma tells you: `primaryAxisSizingMode: "AUTO"` on the connected frame = it hugs content

**Component state audit** — When extracting an INSTANCE node:
1. Get the main component via `inst.getMainComponentAsync()`
2. Check if parent is COMPONENT_SET → list ALL variants
3. Document what changes between variants (height, content, detail rows, visibility toggles)
4. The screen you're building might need to support multiple states, not just the one shown

**Hidden layer audit** — When extracting a screen:
1. First pass: get ALL children (including `visible: false`)
2. Log hidden nodes separately with their content
3. **DO NOT assume hidden = "don't build".** Hidden layers often represent STATE-DEPENDENT content:
   - Content that reveals after a user action (e.g., call → return → family number revealed)
   - Content for a different step in the flow (pre-call vs post-call)
   - Toggled sections (expand/collapse)
4. Check the prototype flow to understand WHEN hidden content becomes visible
5. Ask the user about hidden layers: "These are hidden — are they a future state or truly excluded?"
6. Build hidden content into the screen with `display:none` and toggle via JS state changes

#### Device Compatibility Findings
- **Android WebView** renders CSS identically to Chrome — test in Chrome DevTools at target width
- **Status bar:** NEVER build HTML status bars — use Android system status bar (set color via `window.statusBarColor`). Figma shows status bars for prototype context only; in APK they duplicate the native bar
- **Camera (live viewfinder):** Use `getUserMedia` + `<video>` element. Requires `onPermissionRequest` in WebChromeClient to grant web camera permission. Request runtime CAMERA permission at app startup.
- **Camera fallback chain:** `{facingMode:{ideal:facing}}` → `{video:true}` → log error. Front camera = `"user"`, rear = `"environment"`.
- **Camera (file capture):** Use `WebChromeClient.onShowFileChooser` + `FileProvider` for photo file input
- **Camera overlay pattern:** Use `box-shadow: 0 0 0 9999px rgba(0,0,0,0.55)` on the guide rect for dark overlay. NOT SVG masks — they have viewBox scaling issues.
- **Fonts:** Google Fonts CDN works in WebView but adds load time — consider bundling for offline

#### APK Build Chain
- Android WebView shell → single `index.html` with all screens as `<div class="scr">`
- Navigation: `go('screenId')` toggles `.on` class
- Dialogs: `.ov` overlay divs with `showOv()`/`hideOv()`
- Gradle build → `app-debug.apk` → `adb install`

---

## Visual Audit Checklist (run before declaring a screen "done")

- [ ] Every color matches Figma hex (not "close enough")
- [ ] Font family correct (Devanagari vs Latin)
- [ ] Font weight matches (400/500/600/700 — check Figma `fontWeight`)
- [ ] Font size in px matches Figma dp
- [ ] Line height explicitly set (not browser default)
- [ ] Border radius matches (common values: 4, 8, 12, 16, 24, 36, 50%)
- [ ] Padding/margin matches Figma auto-layout values
- [ ] Icons are correct variant (filled/outlined) and correct color
- [ ] CTA button states: enabled (brand) / disabled (muted) / pressed (opacity)
- [ ] Checkbox component: unchecked border + checked fill + tick
- [ ] Tested at 320px width (no overflow/clipping)
- [ ] Tested at 412px width (no excessive stretching)
- [ ] Scrollable areas scroll, fixed areas stay fixed
- [ ] Dialog overlays dim background and center content
- [ ] Text wraps correctly (no horizontal scroll)
- [ ] **Elevation check:** Cards with shadow — what sits behind/below them? Look for overlap connections
- [ ] **Border + shadow:** Check if element has BOTH (border for definition, shadow for elevation)
- [ ] **Connected elements:** Adjacent elements with overlapping y — implement with negative margin + z-index
- [ ] **Hidden layer scan:** Run `node.children` without visibility filter, log all hidden nodes
- [ ] **Component variants:** For every INSTANCE node, check parent COMPONENT_SET for all states/variants
- [ ] **CTA sub-components:** Phone buttons, action icons — extract each child, don't assume single icon
- [ ] **No fake status bars:** HTML must NOT include time/battery/network bars — native OS handles this
- [ ] **All CTAs at consistent position:** `padding: 0 16px 24px; flex-shrink:0` — same on every screen
- [ ] **Icons are SVG, not webp/png:** Especially in dialogs/popups — raster icons blur at device density
- [ ] **Dialog content verified against live Figma:** Every text, button, icon — re-extract via Bridge, don't trust cached audit
- [ ] **Tap targets:** If a card has one primary action, make the entire card tappable, not just the text link
- [ ] **Camera overlay:** Dark region covers ALL areas outside guide rect (including header and footer areas)
- [ ] **Multi-state screens:** Check if the screen has multiple states (prompt → captured → review) — build all states with JS toggling
- [ ] **Chrome preview at 360×720:** Screenshot every screen in Chrome before deploying to device
- [ ] **Ghost buttons:** If CTA has `bg` same as screen bg and no `stroke` in spec → `border:none`, text color only
- [ ] **Fixed positioning:** If element has explicit y offset in Figma → use `padding-top`, not flex centering

---

## Stack Reference

### Current: HTML/CSS/JS → Android WebView APK
- Single-page app, all screens in one HTML file
- CSS custom properties for design tokens
- Material Icons via Google Fonts CDN
- Noto Sans / Noto Sans Devanagari via Google Fonts CDN
- Android: WebView + WebChromeClient + FileProvider
- Build: Gradle → APK

### Future expansion candidates
- React + Tailwind (component-based)
- Flutter (cross-platform native)
- React Native (cross-platform with native rendering)

---

## Review Tool — Design Review Console

**Location:** `{{PROJECT_ROOT}}/wiom-apk/app/src/main/assets/review.html`
**Server:** `{{PROJECT_ROOT}}/wiom-apk/review-server.js`

### Startup (run at every Maverick session)
```bash
cd "{{PROJECT_ROOT}}/wiom-apk" && node review-server.js &
```
Then open `http://localhost:8080/review.html` in Chrome.

### How It Works
1. **Review mode:** Click elements → annotate with corrections → paste Figma URLs
2. **Interact mode:** Use app normally; tapping non-interactive elements prompts "what should happen?"
3. **Submit Review:** Saves `design-review.json` → Matrix rain overlay → I read + fix + signal reload
4. **Deploy APK:** Button builds + installs on connected device via ADB
5. **Auto-backup:** APK + index.html versioned every 30 min to `backups/`

### Review Loop (automated)
```
User reviews → Submit → design-review.json written
                         ↓
File watcher detects change → I read the JSON
                         ↓
Fix all items → curl http://localhost:8080/api/reload
                         ↓
Review tool: matrix stops → iframe reloads → "Ready for review" notification
```

### File Watcher Command (run in background each session)
```bash
cd "{{PROJECT_ROOT}}/wiom-apk" && LAST_MTIME=$(stat -c %Y design-review.json 2>/dev/null) && while true; do MTIME=$(stat -c %Y design-review.json 2>/dev/null); if [ "$MTIME" != "$LAST_MTIME" ]; then echo "REVIEW_SUBMITTED"; cat design-review.json; break; fi; sleep 3; done
```

### Server Endpoints
| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/review` | POST | Save review JSON |
| `/api/review` | GET | Read current review |
| `/api/reload` | GET | Signal iframe reload |
| `/api/check-reload` | GET | Polled by tool for reload flag |
| `/api/deploy` | POST | Build APK + install on device |

---

## Wiom Solution Design Engine Integration

Maverick produces Figma-accurate code. The engine skills ensure that code is also **design-system consistent and UX-correct** even when Figma has gaps.

**Before building any Wiom screen, consult these 3 engine skills:**

| Skill | File | Consult for |
|-------|------|-------------|
| `wiom-interaction-patterns` | `skills/wiom-interaction-patterns/SKILL.md (in this repo)` | Which pattern to use (bottom sheet type, CTA style, dialog variant), navigation behavior, state machines, progress patterns |
| `wiom-visual-craft` | `skills/wiom-visual-craft/SKILL.md (in this repo)` | Color token application (WHEN to use each), radius rules (16 CTA, 12 card, 24 dialog), typography mapping, elevation, micro-polish |
| `wiom-ux-copy` | `skills/wiom-ux-copy/SKILL.md (in this repo)` | CTA labels (outcome-first), error messages, toast copy, state-dependent text (pending vs complete), tonal patterns, bilingual conventions |

**Integration protocol:**
1. Extract from Figma (Maverick's spec-first protocol — unchanged)
2. Cross-check against engine skills for consistency:
   - Is the CTA radius 16.dp? (not 12)
   - Is the secondary CTA filled #FFE5F6? (not outlined)
   - Is the dialog radius 24.dp? (not 16)
   - Is the input active stroke #352D42? (not brand/600)
   - Is the CTA text the right copy pattern? (outcome-first, correct tense)
3. If Figma and engine skill disagree: **Figma is truth for THIS screen**, but flag the inconsistency for the user
4. If Figma has gaps (missing states, no error state, no empty state): **engine skills fill the gaps**

**This makes Maverick's output both pixel-accurate AND design-system consistent.**

---

## How to Use This Skill

When the user says **"maverick"** or asks to implement a Figma design:
1. Load this skill file
2. **Load engine skills** (interaction-patterns, visual-craft, ux-copy) for Wiom projects
3. **Start the review server** (`node review-server.js`)
4. **Start the file watcher** (background task)
5. Extract design specs from Figma using `figmaSpec()` function
6. Check the Mistake→Correction Log before coding
7. **Cross-check against engine skills** for DS consistency
8. Follow the Spec-First Build Protocol
9. Preview in Chrome at 360×720 before deploying
10. Run the Visual Audit Checklist before declaring done
11. After user corrections, ADD the new pattern to this file

**This file grows with every session. That's the whole point.**
