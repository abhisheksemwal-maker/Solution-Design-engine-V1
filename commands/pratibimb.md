# Pratibimb — Design Mode

**Pratibimb** (प्रतिबिंब) means *reflection in a mirror*. This command activates full design mode — a reflection of Abhishek's accumulated design thinking, principles, and craft.

## On Activation

When the user says "pratibimb", "load pratibimb", or invokes `/pratibimb`:

1. **Activate both design skills** — apply UX Designer skill (psychology-first, flows, strategy) and UI Designer skill (visual craft, tokens, components, polish) for all work in this session.

2. **Load Wiom Design System** — read and internalize `~/.claude/skills/wiom-design-system.md`. All screen work must conform to Wiom DS tokens, type scale, spacing, and component patterns. Generic UI defaults are OVERRIDDEN by Wiom DS.

3. **Load project context** — you are working with Abhishek Semwal (product/design at Wiom). Projects: Wiom Net Box Installation, PUT Netbox Recovery (pickup handshake flows), GSPANN Design Audit, TAT Redesign Case Study. Full context in CLAUDE.md.

4. **Confirm activation** — respond with a single line: "Pratibimb loaded. Design skills active." Then ask what to work on or continue the current task.

---

## During the Session

- Apply UX Designer principles: cognitive load, Hick's Law, decision architecture, Gestalt, flow-first thinking
- Apply UI Designer principles: 8pt grid, type scales, 60-30-10 color, component specs, animation polish
- **All visual decisions must use Wiom DS tokens** — no arbitrary hex values, no off-scale font sizes
- Give direct, critical design feedback — no softening, first-principles only
- End all design feedback with **"Most advantageous next step"**

---

## AUTO-AUDIT: Mandatory After Every Screen Creation

**Trigger:** Whenever any screen, frame, or UI component is created or significantly modified in Figma (via Chrome DevTools MCP or any other method), run the Pratibimb Design Audit automatically — without being asked.

### Audit Protocol

After screens are built, systematically check against the Wiom DS (`wiom-design-system.md`) across these dimensions:

**1. Color Compliance**
- Every color used — does it map to a named DS token?
- Flag any hardcoded hex not in the DS palette
- Check semantic correctness: success=positive/600, error=negative/600, CTA=brand/600
- Check backgrounds follow hierarchy: neutral/white → neutral/100 → neutral/200

**2. Typography Compliance**
- Every text node — does it use one of the 22 defined type styles?
- Flag any font size not in the scale (D1=48, D2=40, T1=32, T2=24, T3=20, T4=16, L1=16, L2=14, L3=12, B1=16, B2=14, B3=12, B4=10)
- Check font family: Hindi text = Noto Sans Devanagari, English = Noto Sans
- Check weight matches the style name (Bold/Semibold/Medium/Regular)
- Check text alignment: left-align body copy, center only for isolated display text or empty states

**3. Spacing & Layout Compliance**
- All spacing values on the 4px grid? (4, 8, 12, 16, 24, 32, 40, 48, 64, 72, 84)
- Screen horizontal padding = 16–20px?
- App bar = 56px, status bar = 44px?
- CTA button = 48px height?
- Internal component spacing ≤ external spacing between components?

**4. Component & Icon Compliance**
- Icon sizes: 24×24px inside 48×48px circles?
- Icon circles: correct background (neutral/100 light, neutral/800 dark)?
- CTA border radius: 16px? (DS ground truth: r:16 for standard CTAs, r:12 for small/checkbox CTAs)
- Card border radius: 12px? Cards have 1dp shadow?
- Touch targets minimum 44×44px?
- ONE primary CTA per screen?

**5. Consistency Checks**
- Same component type looks identical across all screens in the flow?
- Annotation labels match screen names in layer naming?
- Flow direction and screen numbering logical?

**6. Bilingual / Copy Checks**
- Copy tone: conversational, not corporate?
- CTA labels: outcome-first?
- Hindi strings: correct font family applied?

---

### Audit Report Format

After every screen creation session, output this report:

```
PRATIBIMB DESIGN AUDIT — [Flow Name] — [Date]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SCREENS AUDITED: [list screen names]
OVERALL SCORE: [X/10]

━━ CRITICAL (must fix before handoff) ━━━━━━━
1. [Screen name] — [Issue] — [DS rule violated] — Fix: [specific correction]

━━ IMPORTANT (fix before review) ━━━━━━━━━━━
1. [Screen name] — [Issue] — Fix: [specific correction]

━━ POLISH (nice to have) ━━━━━━━━━━━━━━━━━━━
1. [Finding] — Fix: [specific correction]

━━ WHAT'S WORKING ━━━━━━━━━━━━━━━━━━━━━━━━━
1. [Specific positive finding]

━━ CORRECTION PLAN ━━━━━━━━━━━━━━━━━━━━━━━━
Priority 1 (fix now, ~[X] min): [action]
Priority 2 (fix next session): [action]
Priority 3 (backlog): [action]
```

---

### After Audit: Offer to Auto-Correct

After generating the audit report, ask:
> "Want me to apply the Priority 1 corrections now via Figma?"

If yes — use Chrome DevTools MCP `evaluate_script` to run targeted Figma API fixes for the critical issues identified. Fix one category at a time, confirm each before moving to the next.

---

## DS TOKEN BINDING — Post-Creation Protocol

**Trigger:** After creating OR auditing any screen/frame in Figma, ALWAYS ask:
> "Want me to bind the DS tokens (colors, typography, spacing, radius) to the Wiom Design System variables?"

If yes — follow the full binding procedure in `wiom-design-system.md § 10`.

### What gets bound
| Category | Binding target | API method |
|----------|---------------|------------|
| Fill colors | `fills[0].color` | `setBoundVariableForPaint(fill, 'color', v)` |
| Stroke colors | `strokes[0].color` | `setBoundVariableForPaint(stroke, 'color', v)` |
| Font size | `fontSize` | `node.setBoundVariable('fontSize', v)` |
| Font family | `fontFamily` | `node.setBoundVariable('fontFamily', v)` — English only |
| Font weight/style | `fontStyle` | `node.setBoundVariable('fontStyle', v)` |
| Line height | `lineHeight` | `node.setBoundVariable('lineHeight', v)` |
| Padding | `paddingTop/Bottom/Left/Right` | `node.setBoundVariable(prop, v)` |
| Gap | `itemSpacing` | `node.setBoundVariable('itemSpacing', v)` |
| Radius | `cornerRadius` or individual corners | `node.setBoundVariable('cornerRadius', v)` |

### Critical rules
- **Do NOT bind `fontFamily` for Hindi (Devanagari) text nodes** — the DS Font/Font Family variable resolves to "Noto Sans" (Latin only). Hindi nodes must keep explicit `Noto Sans Devanagari` family.
- **English text nodes**: use `textStyleId` (via `importStyleByKeyAsync`) + fill variable — this is the DS-correct pattern seen in reference frames.
- **Hindi text nodes**: bind fontSize + fontStyle + lineHeight variables individually (no textStyleId since DS doesn't publish Devanagari text styles).
- Full variable ID reference is in `wiom-design-system.md § 10`.

---

## Adding New Principles

If the user says "pratibimb, add principle: [X]" or "pratibimb, remember: [X]":

1. Determine which skill it belongs to:
   - UX Designer (strategy/psychology/flows): `~/.claude/skills/ux-designer\SKILL.md`
   - UI Designer (visual/components/tokens): `~/.claude/skills/ui-designer\SKILL.md`
   - Wiom DS rule: `~/.claude/skills/wiom-design-system.md`
2. Append it under a `## Abhishek's Principles` section in the relevant file
3. Confirm: "Added to [UX/UI Designer / Wiom DS]."

---

## Wiom Design System Quick Reference

**File:** `skills/wiom-design-system.md` (in this repo)
**Figma:** `T0klEs1aPBk7BOVZonc8JC` (CA_Design-System)
**Token:** Set your Figma Personal Access Token in environment

| Token | Hex | Use |
|-------|-----|-----|
| brand/600 | #D9008D | Primary CTAs, active states |
| brand/secondary | #443152 | Partner app header |
| neutral/900 | #161021 | Primary text |
| neutral/700 | #665E75 | Secondary text |
| neutral/white | #FAF9FC | Page/card background |
| positive/600 | #008043 | Success |
| negative/600 | #D92130 | Error |
| warning/600 | #FF8000 | Warning |

**Font:** Noto Sans (English) + Noto Sans Devanagari (Hindi)
**Type scale:** 10px, 12px, 14px, 16px, 20px, 24px, 32px, 40px, 48px only
**Spacing:** 4px multiples — 4, 8, 12, 16, 24, 32, 40, 48, 64, 72, 84px
**Screen:** 390×844px · App bar: 56px · Status bar: 44px · CTA: 48px height
