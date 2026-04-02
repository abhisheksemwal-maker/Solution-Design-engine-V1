# SD Text Container V1 — Training Evidence

Compiled from Figma DS source components (PA-March-Sprint, Autolayout Training page).

---

## Evidence from Input-Field (18 variants)

**Text sizing observed:**
- `title-text`: FILL×HUG, fs:16/Bold (empty) or Regular (filled), autoResize:"HEIGHT"
- `content` (placeholder): HUG×HUG, fs:16/Regular, autoResize:"WIDTH_AND_HEIGHT", color:#A7A1B2
- `content` (filled): fs:16/Bold, color:#161021
- `subtext` (visible/error): FILL×HUG, fs:14/Regular, color:#D92130
- `subtext` (hidden/non-error): FIXED×FIXED — **BUG**, should be FILL×HUG

**Font weight inversion pattern (L11):**
Empty: title Bold + content Regular. Filled: title Regular + content Bold.

**OTP text:** fs:16, centered via primaryAlign:CENTER on parent box. Empty shows spaces, filled shows digit in Bold.

---

## Evidence from Dialog (3 variants)

**Text sizing observed:**
- `Title`: FILL×HUG, fs:24/Bold, color:#161021
- `description`: FILL×HUG, fs:16/Regular, color:#161021
- Gap between title and description: 12

Both text nodes inside a FILL×HUG `text` frame with VERTICAL layout, gap:12.

---

## Evidence from Bottom Sheet Header

**Text sizing observed:**
- `Title`: FILL×HUG, fs:24/Bold, color:#161021
- `subtitle` (hidden): FIXED×FIXED — **BUG**, should be FILL×HUG

---

## Evidence from Top App Bar

**Text sizing observed:**
- `headline` (Normal header): FILL×HUG, fs:16/Bold
- `headline` (Big title): FILL×HUG, fs:24/Bold
- `subheading` (Big, hidden): FIXED×FIXED — **BUG**, should be FILL×HUG

---

## Evidence from FAQ Card (Accordion)

**Text sizing observed:**
- Question `title`: FILL×HUG, fs:16/Bold, color:#161021
- Answer `description`: FILL×HUG, fs:14/Regular (collapsed) or fs:16/Regular (expanded)
- Answer description color: #444444 (collapsed variant) vs #161021 (expanded) — **inconsistency**, should be consistent

---

## Evidence from Card (Plan Selection)

**Text sizing observed:**
- Plan info text inside HORIZONTAL row
- `details` frame incorrectly uses FIXED×FIXED instead of FILL×HUG

---

## Evidence from CTA (Actions)

**Text sizing observed:**
- `action` text: FILL×HUG inside CTA, fs:16/Bold (standard) or fs:14/SemiBold (small)
- CTA text centered via parent primaryAlign:CENTER, not textAlign
- Coupon CTA text: FIXED×HUG (small bug — should be HUG×HUG for inline)

---

## Evidence from Apply Coupon

**Text sizing observed:**
- `title`: FILL×HUG, fs:16/Bold
- Applied=Yes has title+subtitle in VERTICAL frame, gap:4
- Link text ("सारे कूपन देखें"): FILL×HUG, fs:14/Regular, color:#665E75

---

## Evidence from Menu List Item

**Text sizing observed:**
- `Content` frame: FILL×HUG — correct pattern inside HORIZONTAL row

---

## Systemic Bug Pattern

Hidden text nodes consistently use FIXED×FIXED across DS source:
1. Input-Field subtext (non-error states)
2. Bottom Sheet Header subtitle
3. Top App Bar Big subheading
4. Chat Bubble title frame

All should be FILL×HUG to match their visible counterparts.
