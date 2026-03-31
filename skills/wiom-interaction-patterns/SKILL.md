# Wiom Interaction Patterns — Solution Design Engine V1

The interaction design bible for Wiom products. Decides WHAT pattern to use, WHERE, WHEN, and what states to handle — so tech and product teams can build screens without designer sign-off on every edge case.

**Output format:** Compose-ready. Every pattern maps to a Jetpack Compose component, state machine, or navigation action.

---

## Who Uses This

- **Tech** building screens: "Should this be a BottomSheet or AlertDialog? What happens on back press? What about the empty state?"
- **Product** reviewing flows: "Is this the right surface for this interaction? Are we handling the edge cases?"

You do NOT need a designer to approve every edge case. This skill gives you the decision framework. Follow it.

---

## The Three Wiom Users

Every pattern decision filters through the primary persona for the screen being built.

### Annu Bhaiyya (CSP/Partner App)
- Tech comfort: low-medium. Learned apps by rote, not exploration
- Thinks in tasks: "aaj kya karna hai" — not features
- Error tolerance: LOW — one confusing screen = support call
- Will NOT discover hidden features. Everything must be explicit
- Trusts numbers and visual proof over text promises

### Technician Rohit (Expert/Partner App)
- Tech comfort: medium. Younger, phone-native, follows sequential flows
- Thinks in checklists: "step 1, 2, 3... done"
- Error tolerance: medium — will retry once, then call support
- Follows guided flows. Does not explore settings
- Comfortable with English tech terms (WiFi, OTP, ISP)

### Verma Parivar (Customer App)
- Tech comfort: low. First-time home internet. Uses WhatsApp/UPI by muscle memory
- Thinks in recharge cycles: "₹23/day wala net"
- Error tolerance: VERY LOW — will abandon and call
- Zero proactive exploration. Responds to Vyom bot prompts only
- Trusts the person (Vyom, technician) over the interface

---

## 1. Surface Selection — What Pattern, When

### Decision Tree

```
Does the user MUST acknowledge before continuing?
  YES → Is it destructive or high-stakes?
    YES → AlertDialog (confirmation pattern)
    NO  → Is it blocking the flow?
      YES → AlertDialog (gate pattern: PayG consent, permission request)
      NO  → BottomSheet (info that enriches but doesn't block)
  NO  → Is it a selection from a list?
    YES → Are there ≤5 options?
      YES → BottomSheet (picker)
      NO  → Full-screen list with search
    NO  → Is it transient feedback?
      YES → Snackbar/Toast (auto-dismiss)
      NO  → Is it a new context entirely?
        YES → New screen (navigate)
        NO  → Inline expansion (accordion, reveal)
```

### Pattern → Compose Component Map

| Pattern | Compose Component | When to use |
|---------|------------------|-------------|
| **Bottom Sheet** | `ModalBottomSheet` | Selections, pickers, options that don't need full context switch. ISP method picker, language selector, filter options, payment method selection |
| **Alert Dialog** | `AlertDialog` | Confirmations, warnings, gates that MUST be acknowledged. PayG consent, delete confirmation, Happy Code entry, exit warnings |
| **Full-screen** | New `Screen` composable via NavGraph | New context entirely. Camera capture, form entry with >3 fields, onboarding education, detailed views |
| **Snackbar** | `SnackbarHost` on `Scaffold` | Transient feedback, no action required. "सेव हो गया", "कॉपी हो गया", "भेज दिया" — auto-dismiss 3s |
| **Snackbar with action** | `Snackbar(action = ...)` | Transient feedback WITH undo or retry. "हटा दिया" + "वापस लाएं" — dismiss 5s |
| **Inline expansion** | `AnimatedVisibility` | Detail that enriches current view. FAQ accordion, tip reveal, expandable card sections |
| **Inline banner** | `Card` with semantic color | Persistent info within a screen. Warnings, tips, status messages that stay visible |

### Bottom Sheet Rules
- **Max 5 items.** Beyond 5, use a full-screen list with search
- **Always include a drag handle** (`dragHandle` parameter)
- **Dismiss on outside tap** for non-critical selections
- **Never nest:** no bottom sheet from within a bottom sheet
- **Annu/Verma:** add a title explaining what to pick. Never a bare list
- **Rohit:** title optional if the context is obvious from the flow

### Alert Dialog Rules — DS Component Specs
```
Dialog: 312px wide, bg:#FAF9FC, r:24, p:[32,24,32,24], DROP_SHADOW
  Head: VERTICAL gap:24
    Optional icon: 48×48 or 96×96, bg:#F1EDF7 circle
    Title: T2_24_Bold #161021
    Description: B1_16_Regular #161021
  Action bar: VERTICAL gap:12, gap:48 from Head
    Primary CTA: 264×48, bg:#D9008D, r:16
    Secondary CTA: 264×48, bg:#FFE5F6, r:16
  Variants: Icon=No | Icon=48x48 | Icon=96x96
```
- **Two buttons maximum.** Primary stacked above secondary (VERTICAL layout, NOT side-by-side)
- **Button labels are VERBS, not "OK"/"Cancel."** Use "हटाएं"/"रहने दें", "भेजें"/"रुकें"
- **Destructive action button:** always `negative/600` color, never primary position
- **Dialog radius: 24.dp** (rounder than cards at 12.dp)
- **Never auto-dismiss.** User must explicitly choose
- **Gate dialogs** (PayG consent, permissions): only ONE button ("समझ गया" / "आगे बढ़ें") — no dismiss option
- **Icon choice:** No icon for simple confirmations, 48×48 for warnings/info, 96×96 for major moments (success, error)

### Snackbar Rules
- **Position:** bottom, above CTA if CTA exists
- **Auto-dismiss:** 3s for info, 5s for actions, never for errors
- **Never use for errors.** Errors need persistent inline display
- **Max 2 lines of text.** If more, use an inline banner instead
- **One snackbar at a time.** Queue if multiple trigger in sequence

---

## 2. CTA Positioning — Consistent Across Every Screen

### Primary CTA
```kotlin
// ALWAYS bottom-anchored, full-width, inside a Surface with padding
Surface(
    modifier = Modifier.fillMaxWidth(),
    shadowElevation = 8.dp // subtle top shadow for separation
) {
    Button(
        onClick = { /* action */ },
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp, vertical = 12.dp)
            .height(48.dp),
        shape = RoundedCornerShape(16.dp), // DS: r:16 for standard CTAs
        colors = ButtonDefaults.buttonColors(
            containerColor = BrandPrimary // #D9008D
        )
    ) {
        Text(
            text = strings.ctaLabel, // outcome-first label
            fontSize = 16.sp,
            fontWeight = FontWeight.Bold // DS: Full CTA = T4_16_Bold
        )
    }
}
```

### DS Action Variants Reference
| Variant | Size | Radius | Text | BG | Use |
|---------|------|--------|------|-----|-----|
| Full CTA (primary) | 328×48 | 16.dp | 16/Bold #FAF9FC | #D9008D | Main screen CTA |
| Full CTA (secondary) | 328×48 | 16.dp | 16/Bold #D9008D | #FFE5F6 | Alternative action |
| Dialog CTA | 264×48 | 16.dp | 16/Bold | same | Inside dialogs |
| Half CTA | 156×48 | 16.dp | 16/Bold | same | Side-by-side pair |
| Special Small CTA | 128×44 | 12.dp | 14/SemiBold | same | Compact actions with dropdown |
| Coupon CTA | 80×28 | 8.dp | 14/SemiBold #D9008D | #FAF9FC + #FFE5F6 stroke | Inline apply buttons |

### CTA Hierarchy (one screen, top to bottom of importance)
| Level | Treatment | Compose | Example |
|-------|-----------|---------|---------|
| **Primary** | Solid fill `brand/600` (#D9008D), full-width, bottom-anchored, r:16 | `Button` | "₹300 जमा करें" |
| **Secondary** | Tonal fill `#FFE5F6` (light pink), full-width, above primary, r:16 | `FilledTonalButton` | "बाद में करें" |
| **Tertiary/Ghost** | Text-only, `brand/600` color, inline | `TextButton` | "शर्तें देखें" |
| **Destructive** | Tonal fill `#FFE5F6` with `negative/600` text, never primary position | `FilledTonalButton` with error colors | "रद्द करें" |

**Key DS fact:** Secondary CTAs are **filled tonal** (#FFE5F6), NOT outlined. The DS does not use `OutlinedButton` for secondary actions.

### Rules
- **ONE primary CTA per screen.** If you need two, one of them is actually secondary
- **Primary CTA is always visible** without scrolling — use `Scaffold` with `bottomBar`
- **Standard CTA: 48.dp height, 16.dp radius.** Small CTA: 44.dp height, 12.dp radius. Coupon CTA: 28.dp height, 8.dp radius
- **CTA text: 16.sp Bold** for standard CTAs, **14.sp SemiBold** for small/coupon CTAs
- **Disabled state:** `containerColor = neutral/300`, no click handler, `alpha = 0.5f`
- **Loading state:** `CircularProgressIndicator` replaces text, same button dimensions
- **CTA text:** outcome-first, see `wiom-ux-copy` skill for label conventions

---

## 3. Navigation and Recovery

### Back Button vs Close

| Icon | Compose | When |
|------|---------|------|
| `arrow_back` | `Icons.AutoMirrored.Filled.ArrowBack` | Navigating back within a flow (screen 3 → screen 2) |
| `close` | `Icons.Default.Close` | Exiting a context entirely (camera, modal, overlay, sub-flow) |

**Rule:** If the screen is part of a sequential flow, use back. If the screen is an overlay or branched context, use close.

### Input Preservation on Back

```
User fills 4 fields → goes to next screen → presses back
→ ALL 4 fields must retain their values
```

- **ViewModel state:** Store all form inputs in ViewModel, not in composable `remember`. ViewModel survives navigation
- **When to clear:** ONLY on explicit "reset" action or successful submission
- **On error:** Preserve ALL valid fields. Only highlight the errored field. Never clear the form
- **Annu/Verma:** clearing their inputs is a catastrophic UX failure — they may not remember what they typed

### Confirmation on Back with Unsaved Changes

```
Does the screen have user-entered data?
  NO  → navigate back silently
  YES → Has the user spent >30 seconds or filled >2 fields?
    YES → Show confirmation dialog: "बदलाव सेव नहीं होंगे। वापस जाएं?"
            Buttons: "वापस जाएं" (secondary) / "यहीं रहें" (primary)
    NO  → navigate back silently (low investment, not worth interrupting)
```

### System Back Button (Android)
- Same behavior as app bar back/close button — never a different action
- On root screen of a flow: exit the flow (not the app)
- On app root: show exit confirmation only if there's unsaved state

---

## 4. Menu Patterns

### Hamburger Menu (Global Navigation)
- **Use for:** App-level navigation in Partner app. Settings, earn money, trust line, agreement, pathshala, logout
- **Compose:** `DrawerValue` + `ModalNavigationDrawer`
- **Trigger:** hamburger icon (`Icons.Default.Menu`) in app bar, top-left
- **Width:** 280.dp (Material standard)
- **Items:** Icon (24.dp) + Label + optional badge/count
- **Active item:** background highlight with `brand/600` tint

### Triple Dot Menu (Contextual Actions)
- **Use for:** Actions on a specific item. Task options, card actions, more actions on a specific entity
- **Compose:** `IconButton` + `DropdownMenu` + `DropdownMenuItem`
- **Trigger:** `Icons.Default.MoreVert` in top-right of app bar OR trailing edge of a card/list item
- **Max items:** 5. Beyond 5, use a full-screen "More options" view
- **Destructive items:** at the bottom, `negative/600` text color, with separator above

### Never Both on Same Screen
- Hamburger = global app navigation. Used on HOME/DASHBOARD screens
- Triple dot = contextual actions. Used on DETAIL/ITEM screens
- A screen with a hamburger menu does NOT have a triple dot, and vice versa

### DS Top App Bar Component (6 variants)
```
Top App Bar: 360×80 (Normal/Brand) or 360×136 (Big), VERTICAL gap:0
  Status Bar: 360×24, p:[8,8,8,8]
  Header: 360×56, HORIZONTAL gap:4, p:[4,4,4,4]
    Left: back arrow (48×48 touch target) + title
    Trailing: help label "मदद" + language icon + more_vert (each 48×48)
  Big variant adds: Title section 360×56, p:[8,16,16,16], T2_24_Bold

Variants:
  Header=Normal, Gradient=No  → white bg, dark icons, back arrow + "Title" fs:16/Bold
  Header=Normal, Gradient=Yes → gradient bg, white icons, simplified trailing
  Header=Big, Gradient=No     → adds large title below (fs:24/Bold)
  Header=Big, Gradient=Yes    → gradient + large title
  Header=Brand, Gradient=No   → Wiom logo replaces back arrow (home screens)
  Header=Brand, Gradient=Yes  → gradient + Wiom logo
```
- **Normal:** for all flow screens with back navigation
- **Big:** for section landing screens that need a prominent title
- **Brand:** for app home/dashboard (no back arrow, Wiom logo instead)
- **Gradient:** for customer app screens with brand-colored header

### Chevron Rows (Settings Pattern)
- **Use for:** navigating to sub-screens within settings or menus
- **Structure:** Icon circle (48.dp) + Title + optional subtitle + Chevron (`›`)
- **Chevron:** `brand/600` color, `Body/B2_14_Semibold`, trailing edge
- **Tap target:** entire row, not just the chevron

---

## 5. Secondary Flow Surfacing

How to make secondary functionality discoverable WITHOUT cluttering the primary flow.

| Technique | When to use | Compose pattern |
|-----------|------------|-----------------|
| **Inline link** | Action embedded in content ("शर्तें देखें" inside a card) | `TextButton` or `ClickableText` with `brand/600` color |
| **Chevron row** | Navigating to a sub-screen (settings, detailed view) | `Row` with icon + text + `›` + `clickable` modifier |
| **"सब देखें" link** | Dashboard section with limited preview | `TextButton` at section header trailing edge |
| **Card action** | Secondary action on a specific card | `TextButton` or `IconButton` within card footer |
| **Info icon** | Tooltip/explanation for a term or value | `IconButton(Icons.Outlined.Info)` → shows dialog or bottom sheet |

### Anti-patterns (never use for any persona)
- Long-press to reveal actions — Annu/Verma will never discover this
- Swipe gestures on list items — unreliable discovery
- Hidden behind scroll — if it matters, it must be visible on first load
- "Learn more" links without explaining what they'll learn

---

## 6. Confirmation Patterns

### Decision Matrix

| Action type | Reversible? | User effort to redo | Pattern |
|-------------|-------------|---------------------|---------|
| Toggle a setting | Yes | 1 tap | No confirmation. Apply immediately |
| Delete a saved item | No | High | Alert dialog with consequence ("यह हमेशा के लिए हट जाएगा") |
| Submit a form | No | High (re-entry) | Preview/review screen before final submit |
| Cancel a booking | No | Very high (re-booking) | Two-step: explain consequence → confirm with specific verb |
| Recharge/payment | No | Medium (re-enter amount) | Summary screen with amount + breakdown → "₹[X] रिचार्ज करें" |
| Logout | Reversible | Low (re-login) | Single confirmation: "लॉग आउट करें?" |
| Exit mid-flow | Reversible | High (re-enter data) | "बदलाव सेव नहीं होंगे" dialog |

### Payment Confirmation Pattern
```
Screen N: Fill details
    ↓
Screen N+1: Summary/Review
    Shows: amount, breakdown, what they're paying for
    CTA: "₹[amount] [action verb]" — e.g., "₹300 जमा करें"
    ↓
Processing overlay (CircularProgressIndicator + "प्रोसेस हो रहा है...")
    ↓
Success screen OR error inline
```
- **Never** go directly from input to processing without a review screen
- **Always** show the amount on the final CTA button itself
- **Annu/Verma:** the review screen IS the trust moment. Skipping it breaks trust

### Two-Step Destructive Confirmation
```
Step 1: AlertDialog
    Title: "बुकिंग रद्द करें?"
    Body: "आपकी ₹100 बुकिंग फीस [X दिनों में] वापस आ जाएगी। रद्द करने के बाद दोबारा बुक करना होगा।"
    Buttons: "नहीं, रहने दें" (primary) / "हाँ, रद्द करें" (destructive outline)

Step 2 (if confirmed): Processing → Result
```
- The "safe" option ("रहने दें") is ALWAYS the primary/prominent button
- The destructive option is ALWAYS secondary/outline with `negative/600` text

---

## 7. Loading, Empty, and Error States

Every screen MUST handle all three. No exceptions.

### Loading State
```kotlin
// Skeleton shimmer — matches actual content layout
ShimmerBox(modifier = Modifier.fillMaxWidth().height(200.dp))
// NOT a centered spinner. Skeleton shows structure.
```
- Use skeleton shimmer that matches the actual content layout
- Spinners (`CircularProgressIndicator`) ONLY for overlay processing (payment, submission)
- **Never** a blank white screen while loading

### Empty State
```kotlin
Column(
    modifier = Modifier.fillMaxWidth().padding(32.dp),
    horizontalAlignment = Alignment.CenterHorizontally
) {
    Icon(/* relevant illustration or icon */, tint = Neutral500)
    Spacer(modifier = Modifier.height(16.dp))
    Text("कोई टास्क नहीं है", style = Title_T4_16_Bold)
    Spacer(modifier = Modifier.height(8.dp))
    Text("नए टास्क आने पर यहाँ दिखेंगे", style = Body_B2_14_Regular, color = Neutral700)
    // Optional CTA if user can take action
    Spacer(modifier = Modifier.height(24.dp))
    OutlinedButton(onClick = { /* refresh or navigate */ }) {
        Text("रिफ्रेश करें")
    }
}
```
- **Always:** icon/illustration + title + explanation + optional action
- **Never:** just "कोई डेटा नहीं" or a blank screen
- **Annu:** explain WHY it's empty and WHEN it will have content
- **Verma:** Vyom bot handles empty states conversationally

### Error State — Inline
```kotlin
Card(
    colors = CardDefaults.cardColors(containerColor = NegativeLight), // light red tint
    shape = RoundedCornerShape(12.dp)
) {
    Row(modifier = Modifier.padding(16.dp)) {
        Icon(Icons.Default.ErrorOutline, tint = Negative600)
        Spacer(modifier = Modifier.width(12.dp))
        Column {
            Text("कनेक्शन में दिक्कत है", style = Body_B2_14_Semibold)
            Text("इंटरनेट चेक करें और दोबारा कोशिश करें", style = Body_B2_14_Regular)
        }
    }
}
```
- **Inline, persistent.** Never a toast for errors
- **Always:** what happened + what to do
- **Retry button** if the error is retriable (network, timeout)
- **Never blame the user.** "कनेक्शन में दिक्कत है" not "आपका इंटरनेट नहीं चल रहा"
- See `wiom-ux-copy` skill for error message templates

---

## 8. Feature Introduction Patterns

When introducing a new feature or concept to any persona.

### Decision Framework

```
Is this a concept the user has never encountered?
  YES → Education flow (2-3 screens before the feature)
  NO  → Does it change an existing behavior?
    YES → Change announcement (inline banner, one-time)
    NO  → Contextual discovery (surface in natural position)
```

### Education Flow (for genuinely new concepts)
- **Annu:** 2-3 screen walkthrough. Illustration + 2-line Hindi explanation + "समझ गया" CTA. NO skip option on PayG-critical flows
- **Rohit:** Single info card before the step that uses the feature. IVR audio reinforcement
- **Verma:** Vyom bot introduces it conversationally. Never a static screen

### Contextual Discovery (for additions to existing flows)
- New menu item: appears in natural position with a small `info/600` dot badge (first 3 sessions, then remove)
- New action on existing screen: `neutral/100` background highlight for first 3 uses, then standard
- New dashboard section: no announcement. Content speaks for itself in the natural position
- **Never:** modal popup before the user encounters the feature naturally
- **Never:** forced tour blocking the primary flow
- **Never:** "What's New" changelog screen

### Coach Marks
- **Only for non-obvious gestures** (and even then, reconsider if the gesture is necessary)
- **Only for Rohit.** Annu and Verma don't benefit from gesture coaching — simplify the interaction instead
- Compose: `Popup` anchored to the element + arrow + 2-line instruction + "OK" dismiss

---

## 9. Form Patterns

### Single-Screen Forms (≤4 fields)
- All fields visible. Vertical stack with 16.dp between fields
- Label above each field, 4.dp gap
- CTA at bottom of screen (scrolls with content if fields fill the screen)
- Real-time validation: check on focus loss, clear error on new input

### Multi-Screen Forms (>4 fields)
- One logical group per screen (personal details, address, payment)
- Progress indicator at top: `StepIndicator` showing current step / total steps
- "Back" preserves ALL entered data (ViewModel state)
- Review/summary screen before final submission

### DS Input-Field Component Specs (18 variants: 9 states × 2 types)
```
Input-Field: 328×80 (Normal) or 328×48 (Chat Bubble), VERTICAL gap:8
  Title: fs:16, fw:700 (default/error) or fw:400 (active/filled/success), #161021
  Field: 328×48, bg:#FAF9FC, r:12, HORIZONTAL gap:12, p:[12,16,12,16]
    Leading icon: 24×24, tint #A7A1B2
    Text area: placeholder #A7A1B2 fs:16/400, value #161021 fs:16/700
  Error subtext: fs:14 fw:400 #D92130 (adds 28px → total 108px height)

State strokes:
  Default:    #D7D3E0 (neutral/300)
  Active:     #352D42 (neutral/800) — NOT brand/600
  Filled:     #D7D3E0
  Success:    #008043 (positive/600)
  Error:      #D92130 (negative/600)
  Pre-Filled: #D7D3E0, field bg:#D7D3E0, text:#665E75
  Disabled:   #D7D3E0, field bg:#D7D3E0, text:#A7A1B2

Active cursor: "|" character in #D9008D (brand/600)
```

### Field-Level Compose Pattern
```kotlin
OutlinedTextField(
    value = value,
    onValueChange = {
        value = it
        error = "" // clear error on typing
    },
    label = { Text(strings.fieldLabel, fontWeight = if (error.isNotEmpty()) FontWeight.Bold else FontWeight.Normal) },
    isError = error.isNotEmpty(),
    supportingText = if (error.isNotEmpty()) {
        { Text(error, color = WiomColors.Negative600, fontSize = 14.sp) }
    } else null,
    colors = OutlinedTextFieldDefaults.colors(
        focusedBorderColor = WiomColors.Neutral800,      // #352D42 — DS active state
        unfocusedBorderColor = WiomColors.Neutral300,     // #D7D3E0 — DS default
        errorBorderColor = WiomColors.Negative600,        // #D92130
        cursorColor = WiomColors.BrandPrimary,            // #D9008D
        focusedContainerColor = WiomColors.NeutralWhite,  // #FAF9FC
        unfocusedContainerColor = WiomColors.NeutralWhite
    ),
    shape = RoundedCornerShape(12.dp), // DS: r:12 for input fields
    keyboardOptions = KeyboardOptions(
        keyboardType = fieldType,
        imeAction = if (isLastField) ImeAction.Done else ImeAction.Next
    ),
    modifier = Modifier.fillMaxWidth()
)
```

### Validation Rules
- Validate on focus loss (not on every keystroke)
- Clear error when user starts re-typing
- On submission with errors: scroll to first errored field
- Show all errors at once, not one at a time
- Error text: what's wrong + what's expected (see `wiom-ux-copy`)
- **Pre-Filled fields** (grey bg #D7D3E0, text #665E75): for system-provided data the user cannot edit
- **Disabled fields** (grey bg, placeholder text): for fields not yet available in the flow

---

## 10. Stacked Dual CTA Pattern (Wiom Standard)

The standard Wiom CTA pairing is **vertically stacked**, NOT side-by-side. Primary on top, secondary below.

```kotlin
// Wiom stacked dual CTA — used in ticket details, installation, dialogs
Column(
    modifier = Modifier
        .fillMaxWidth()
        .padding(horizontal = 16.dp),
    verticalArrangement = Arrangement.spacedBy(12.dp) // DS: gap:12 between CTAs
) {
    // Primary CTA (top)
    Button(
        onClick = { /* primary action */ },
        modifier = Modifier.fillMaxWidth().height(48.dp),
        shape = RoundedCornerShape(16.dp),
        colors = ButtonDefaults.buttonColors(containerColor = WiomColors.BrandPrimary)
    ) {
        Text(strings.primaryAction, fontSize = 16.sp, fontWeight = FontWeight.Bold)
    }
    // Secondary CTA (below)
    Button(
        onClick = { /* secondary action */ },
        modifier = Modifier.fillMaxWidth().height(48.dp),
        shape = RoundedCornerShape(16.dp),
        colors = ButtonDefaults.buttonColors(
            containerColor = WiomColors.BrandTonal, // #FFE5F6
            contentColor = WiomColors.BrandPrimary   // #D9008D
        )
    ) {
        Text(strings.secondaryAction, fontSize = 16.sp, fontWeight = FontWeight.Bold)
    }
}
```

### Real examples from PA flows:
| Screen | Primary CTA | Secondary CTA |
|--------|------------|---------------|
| Pickup Ticket | "टिकट बंद करें" | "कस्टमर को रिचार्ज करना है" |
| Installation Progress | Primary action | Secondary route |
| Payment Done dialog | "अन्य टास्क देखें" | "रिचार्ज @ ISP खोलें" |
| Pending Popup | "काम जारी रखें" | "अन्य काम देखें" |

### Checkbox CTA Variant (Confirm Delivery pattern)
For explicit user confirmation before a consequential action:
```kotlin
OutlinedButton(
    onClick = { /* confirm */ },
    modifier = Modifier.fillMaxWidth().height(48.dp),
    shape = RoundedCornerShape(12.dp), // r:12, not r:16
    border = BorderStroke(1.dp, WiomColors.BrandPrimary)
) {
    Icon(Icons.Default.CheckBoxOutlineBlank, tint = WiomColors.BrandPrimary, modifier = Modifier.size(24.dp))
    Spacer(Modifier.width(12.dp))
    Text("Yes, I've received 10 net boxes", color = WiomColors.BrandPrimary, fontSize = 16.sp, fontWeight = FontWeight.Bold)
}
```
- Used when the action implies physical-world confirmation (received goods, arrived at location)
- Outlined style (not filled) — user must consciously check/confirm

---

## 11. Ticket Header and Timeline

### Ticket Header Pattern
For service ticket / task detail screens:
```
PA_Headers: 360×80, bg:#443152
  Status Bar: 360×24
  Header: 360×56, HORIZONTAL gap:106, p:[12,16,12,16]
    Left group: arrow_back + Status dot (24×24, semantic color) + ticket title + separator dot (6×6 #FAF9FC) + ticket ID
    Right: more_vert (24×24)
```
- **Status dot colors:** Green (active), Orange (pending), Red (escalated)
- **Ticket ID** shown in regular weight (fs:16/400) after separator
- **Title** in bold (fs:16/700)

### Ticket Timeline Bar
Appears below header in ticket flows — shows progress and deadline:
```kotlin
Surface(
    modifier = Modifier.fillMaxWidth().height(88.dp),
    color = WiomColors.Neutral100 // #F1EDF7
) {
    Column(modifier = Modifier.padding(horizontal = 16.dp, vertical = 12.dp), verticalArrangement = Arrangement.spacedBy(6.dp)) {
        // Progress dots + connecting line
        Row { /* start dot → line → end dot */ }
        // Date labels
        Row(horizontalArrangement = Arrangement.spacedBy(12.dp)) {
            Text("7 अगस्त", fontSize = 12.sp, fontWeight = FontWeight.Normal)
            Row {
                Text("15 अगस्त", fontSize = 12.sp, fontWeight = FontWeight.SemiBold)
                Text(" तक काम पूरा करें", fontSize = 12.sp, fontWeight = FontWeight.Normal)
            }
        }
    }
}
```

---

## 12. Bottom Sheet Taxonomy (from 9 real PA variations)

### Common Structure (all types)
```
Drag handle: 120×4, bg:#D7D3E0, r:4, centered
  Container: p:[12,120,24,120] — 12 top, 24 bottom before content
Optional heading: 360×48, p:[8,16,8,16]
  Title: fs:24/Bold #161021
Content: p:[24,16,24-48,16] — bottom padding varies
```

### Type A: Confirmation/Success Sheet
No heading bar — **icon is the hero**. Used for success confirmations and payment info.
```
[Drag handle]
[Icon circle 72×72 or Illustration 100×100]
[Title fs:24/700]
[Subtitle fs:16/400 or fs:16/500]
[Stacked dual CTA]
```
- Icon circle bg: `#E1FAED` (success green), `#FFE5F6` (brand)
- Optional **info tag**: `bg:#F1E5FF r:4, text #6D17CE fs:16/700` — labels a state ("इस कस्टमर ने पेमेंट की हुई है")
- Subtitle can use `#665E75 fw:500` for explanatory text
- Content padding: p:[24,16,24,16], gap:24 between sections

### Type B: Choice Sheet (Cards)
Heading bar + selectable option cards. Used for method selection, quantity input.
```
[Drag handle]
[Heading: "ऑफिस कैसे भेजना चाहते हैं?" fs:24/700]
[Side-by-side cards OR stepper card]
[Optional single CTA]
```
- **Side-by-side cards**: 156×164 each, HORIZONTAL gap:16
  - Icon circle 48×48 (bg:#FFE5F6) + label fs:16/700
  - Unselected: bg:#FAF9FC stk:#E8E4F0 r:16
  - Selected: bg:#FFCCED stk:#D9008D r:16
- **Stepper card**: 328×72, contains quantity control (120×40 bg:#D9008D r:12) + value display
- Content padding: p:[24,16,48,16] — extra bottom padding

### Type C: Form Input Sheet (with Keyboard)
Info context card + input field + CTA. Sheet stretches when keyboard appears.
```
[Drag handle]
[Info card: context about what's being edited]
[Divider: stk:#E8E4F0]
[Form fields]
[CTA — enabled or disabled]
[Keyboard pushes content up]
```
- **Disabled CTA**: bg:#A7A1B2 (neutral/500), NOT dimmed brand. Activates on valid input
- Info card: stk:#E8E4F0 r:12, with icon+label identifying the entity
- Optional info tag: bg:#F1EDF7 r:4, small contextual label
- Gap between info card and form: 24.dp via divider

### Type D: Selection List Sheet
Title + radio list for single-select. Used for ISP selection, category choice.
```
[Drag handle]
[Info card: current selection context]
[Divider]
[List title: "ISP बदलें" fs:24/700]
[Radio options: vertical list]
[Add-new link: "नया पोर्टल खोलें" fs:16/700 #D9008D]
```
- Radio items: icon + name (fs:16/400) + subtitle (fs:12/400 #665E75), HORIZONTAL gap:8
- Add-new link: brand pink text, no button container — just tappable text

### Type E: Information Display Sheet
Read-only detail view. Used for customer info, device details.
```
[Drag handle]
[Heading: "कस्टमर डिटेल्स" fs:24/700]
[Info card: icon+label+value pairs]
  Name row: user icon + name (fs:14/600)
  Address row: pin icon + address (fs:14/400, wraps)
  Detail row: phone icon + value + label below
[Divider within card: stk:#D7D3E0]
[Additional details section]
```
- Card: stk:#E8E4F0 r:16, p:[16,16,16,16]
- Icon+label pairs: icon 24×24 #A7A1B2 + value fs:14 + label below fs:12/400 #A7A1B2
- No CTA — dismiss by dragging down or tapping outside

### Bottom Sheet Decision Framework

**Layer 1: Should this be a bottom sheet at all?**
```
Can the user complete this without losing context of the screen behind?
  YES → bottom sheet
  NO  → full screen (navigate)
If sheet would cover >70% of screen or need scrolling → full screen
```

**Layer 2: Which type?**

| User's cognitive state | Their unasked question | Sheet type | Visual hierarchy principle |
|---|---|---|---|
| Just completed an action | "Did it work?" | **Type A** | Icon answers BEFORE they read. Green circle = yes. Title confirms with specifics. CTA asks "what next?" |
| At a fork in the flow | "Which way?" | **Type B** | Heading frames the question. Cards are EQUAL weight — no visual bias. Selected state is dramatic (#FFCCED + #D9008D) |
| Editing a specific value | "What do I type?" | **Type C** | Info card gives "for whom" context. Input field is single focus. Disabled CTA (#A7A1B2) → enabled (#D9008D) IS the feedback |
| Scanning to pick one | "Which of these?" | **Type D** | Context card shows what they're changing FROM. Radio items equal weight. Add-new link is tertiary |
| Wants to see details | "Show me" | **Type E** | Heading is a noun. Content is scannable pairs. No CTA = "this is for looking, not doing" |

**Layer 3: The meta-principle**

**Visual hierarchy always answers the user's first question without them having to read.**
- Success? → Green icon
- Choice? → Equal cards + question heading
- Input? → Active field draws the eye
- Selection? → Radio buttons signal "pick one"
- Info? → No CTA signals "just looking"

Typography ranks within: fs:24 = primary message, fs:16 = supporting context, fs:14/12 = metadata. Weight does emphasis at same size: Bold for what matters, Regular for everything else.

### Quick Reference Matrix
| Content type | Sheet type | Has heading? | Has CTA? |
|-------------|-----------|-------------|----------|
| Success/result confirmation | Type A | No (icon is hero) | Yes (stacked dual) |
| Payment/status info | Type A | No (info tag instead) | Yes (stacked dual) |
| Choose between 2-3 options | Type B | Yes | Optional |
| Quantity/stepper input | Type B | Yes | Yes (single) |
| Edit a value (text input) | Type C | No (info card instead) | Yes (enabled/disabled) |
| Select from radio list | Type D | No (info card + title) | No (add-new link instead) |
| View entity details | Type E | Yes | No |

---

## 13. Triple Dot Dropdown Menu

```
Menu container: 200×variable, bg:#FAF9FC, r:4 (SHARP — exception to rounded rule)
  Item: 200×56, p:[16,12,16,12]
    Label: fs:16/400 #161021
  Destructive item (if any): same but text color #D92130, separator above
```

```kotlin
DropdownMenu(
    expanded = expanded,
    onDismissRequest = { expanded = false },
    modifier = Modifier.width(200.dp)
) {
    DropdownMenuItem(
        text = { Text("नेट बॉक्स का हिसाब", fontSize = 16.sp) },
        onClick = { /* action */ },
        contentPadding = PaddingValues(horizontal = 12.dp, vertical = 16.dp)
    )
    DropdownMenuItem(
        text = { Text("सेटिंग", fontSize = 16.sp) },
        onClick = { /* action */ },
        contentPadding = PaddingValues(horizontal = 12.dp, vertical = 16.dp)
    )
}
```
- **r:4 is an intentional exception** — dropdown menus use sharp corners, unlike everything else in the DS
- Max 5 items. Destructive items at bottom with `negative/600` text

---

## 14. Tab Pattern

```
Tabs container: 360×48, HORIZONTAL
  Selected tab: 180×48, stk:#E8E4F0 bottom, text fs:14/SemiBold #FAF9FC (on dark header)
  Deselected tab: 180×48, stk:#E8E4F0 bottom, text fs:14/Regular #FAF9FC (on dark header)
```

```kotlin
TabRow(
    selectedTabIndex = selectedTab,
    containerColor = WiomColors.BrandSecondary, // #443152
    contentColor = WiomColors.NeutralWhite,
    divider = { Divider(color = WiomColors.Neutral200) }
) {
    Tab(
        selected = selectedTab == 0,
        onClick = { selectedTab = 0 },
        text = {
            Text(
                "नेटवर्क बढ़ाएं",
                fontWeight = if (selectedTab == 0) FontWeight.SemiBold else FontWeight.Normal,
                fontSize = 14.sp
            )
        }
    )
    Tab(
        selected = selectedTab == 1,
        onClick = { selectedTab = 1 },
        text = { Text("पैसे कमाएं", fontWeight = if (selectedTab == 1) FontWeight.SemiBold else FontWeight.Normal, fontSize = 14.sp) }
    )
}
```

---

## 15. Segmented Progress Bar

Used in sequential flows (installation, education, pickup):
```kotlin
@Composable
fun SegmentedProgressBar(currentStep: Int, totalSteps: Int = 4) {
    Row(
        modifier = Modifier.fillMaxWidth().padding(horizontal = 12.dp),
        horizontalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        repeat(totalSteps) { index ->
            Box(
                modifier = Modifier
                    .weight(1f).height(2.dp)
                    .background(
                        color = if (index < currentStep) WiomColors.Positive600  // #008043 (or BrandPrimary for education)
                               else WiomColors.Neutral200, // #E8E4F0
                        shape = RoundedCornerShape(4.dp)
                    )
            )
        }
    }
}
```
- **Active segment color:** `positive/600` (#008043) for task completion, `brand/600` (#D9008D) for education flows
- **Remaining:** `neutral/200` (#E8E4F0)
- **Height:** 2.dp, **gap:** 8.dp, **radius:** 4.dp

---

## 16. Education Flow Template

For introducing new features or providing step-by-step guidance:
```
Screen structure:
  Top app bar (Normal, back arrow only)
  Info card: customer/context details (328×152, stk:#E8E4F0, r:12)
  Progress bar: segmented, brand/600 active
  Content area:
    Illustration (120×120)
    Title + description
    Optional video preview (stk:#D6B2FF, r:8 — info/purple tint border)
  "इसके बाद →" inline navigation link (brand/600 text + rotated arrow)
```
- **No bottom CTA** — navigation via inline "next" link
- **Progress bar** above content, not in header
- **Video preview** uses purple border (#D6B2FF) to signal educational content
- **Rohit-oriented:** step-by-step, visual, minimal text

---

## 17. Horizontal Step Progress (Radio Button Stepper)

For multi-step flows shown inline (not as a top bar but as a card within the screen):
```
Card: 328×112, bg:#F1EDF7, r:16, HORIZONTAL gap:8, p:[24,16,24,16]
  Each step: VERTICAL gap:8
    Radio button: 24×24
      Active:   filled #D9008D (inner dot 10×10)
      Inactive: outline #A7A1B2
    Label: fs:12
      Active:   fw:600 #161021
      Inactive: fw:400 #A7A1B2
  Progress line between steps: 56×2
    Completed: bg:#D9008D or #A7A1B2
    Remaining: bg:#D7D3E0
```
- Use for **swap/replacement flows** where the user sees all steps at once
- Different from segmented progress bar (Section 15): this has labeled steps with radio indicators
- Placed BELOW app bar, above main content

---

## 18. Action Landing Screen Pattern

For flow entry points — "explain what this flow does before starting":
```
Screen structure:
  Top app bar (Normal, back + trailing help)
  Step progress card (if multi-step flow)
  Centered content:
    Illustration: 120×120, circular bg:#F1E5FF (info tint)
    Title: T3_20_Bold #161021, centered
    Subtitle: B1_16_Regular #665E75, centered
  CTA: bottom-anchored, full-width
```

```kotlin
Column(
    modifier = Modifier.fillMaxWidth(),
    horizontalAlignment = Alignment.CenterHorizontally
) {
    // Illustration
    Box(
        modifier = Modifier.size(120.dp)
            .background(Color(0xFFF1E5FF), CircleShape),
        contentAlignment = Alignment.Center
    ) { /* icon/illustration */ }
    Spacer(Modifier.height(24.dp))
    Text("नेट बॉक्स बदलें", fontSize = 20.sp, fontWeight = FontWeight.Bold, textAlign = TextAlign.Center)
    Spacer(Modifier.height(8.dp))
    Text("यह प्रक्रिया आपको स्टेप-बाय-स्टेप पूरी करनी है", fontSize = 16.sp, color = WiomColors.Neutral700, textAlign = TextAlign.Center)
}
```
- Used for: Net Box Swap, Installation Start, any multi-step flow entry
- **Rohit:** concise title + subtitle is enough context
- **Annu:** same pattern but may need more reassurance in subtitle

---

## 19. Hero Action Card (Home Dashboard)

Large branded card at the top of dashboard/home screens:
```
328×204, bg:#FFE5F6, stk:#D9008D, r:16, p:[24,24,24,24]
  HORIZONTAL gap:16
    Content area (left): title + stats + description
    CTA circle (right): 72×72, large tappable action button
```
- Used on home/dashboard as the primary attention-getter
- Large CTA circle (72×72) — not the standard 48×48 button
- Below hero card: divider (#E8E4F0) + stat line ("कुल एक्टिव कस्टमर 276")
- Stat line: label fs:16/400 + value fs:20/700

---

## 20. Header Badge Pill

Notification count or status badge in the app bar:
```
55×28, bg:#60506C (dark purple tint of brand/secondary), r:20 (pill)
```
- Positioned in header trailing area, before the menu icon
- Shows unread count or status indicator
- Pill shape (r:20 on 28.dp height = fully rounded)

---

## 21. Radio Selection List Pattern

For single-select from a list of options (team assignment, plan selection):
```
Card (unselected): 328×56-92, bg:#FAF9FC, stk:#E8E4F0, r:16, p:[16,16,16,16]
  Radio: 24×24, outline #A7A1B2
  Details: name + optional badge/tag

Card (selected): 328×56-92, bg:#FFCCED, stk:#D9008D, r:16, p:[16,16,16,16]
  Radio: 24×24, filled #D9008D (inner dot 10×10)
  Details: same layout, highlighted border
```
- Cards stacked VERTICALLY with 16.dp gap
- **Selected card changes BOTH bg (#FFCCED) AND stroke (#D9008D)** — not just the radio button
- CTA appears only after selection (bottom-anchored)
- Optional "+ नया टीम मेंबर जोड़ें" add-link below list (brand/600, fs:14/600)

### Urgency Banner Pattern
For driving action through social proof / urgency:
```
Overlapping avatars: 5× photo circles (32×32), gap:-8 (overlap)
Urgency text: fs:20/700, brand color (#D92B90)
  "10 more customers are waiting..."
```
- Avatar stack uses **negative gap** (-8.dp) for overlapping effect
- Placed above the list to create urgency context
- Used for team assignment, queue-based flows

---

## 22. Data-Heavy List Pattern (ISP Recharge)

For screens with multiple data items requiring selection and action:
```
Screen bg: #F1EDF7 (not white — subtle differentiation)

Header row: "3 कनेक्शन" (count) + "सब चुनें" (select all) + checkbox
  Count: fs:16/700, Select All: fs:16/700 + trailing checkbox

Item card: 328×72, bg:#FAF9FC, stk:#E8E4F0, r:12, HORIZONTAL g:8, p:[24,16,24,16]
  Left: user icon (24×24, #A7A1B2) + name (fs:16/400)
  Right: amount (fs:14/500, #008043 positive) + checkbox

Expanded detail card: 328×152, bg:#FAF9FC, stk:#E8E4F0, r:12, p:[16,16,16,16]
  Header: name + checkbox
  Detail grid: 2×2 icon+label+value pairs
    Icon: 20×20, #A7A1B2
    Value: fs:14/400 #161021
    Label: fs:12/400 #A7A1B2
```
- **Amount in positive green** (#008043) — signals the earning opportunity
- **Select all** pattern: count + toggle at top for batch operations
- **Expandable cards:** tapping a list item reveals the detail card with full info grid
- Screen bg uses `neutral/100` to differentiate from card bg `neutral/white`

---

## 23. Push Notification Card (Rich)

For app notifications shown as overlay on dark background:
```
Dark overlay: bg:#000000

Card: 328×128-138, bg:#FAF9FC, r:12-16, p:[16,12,16,12]
  Logo: 40×40, r:24 (circular), brand colors
  Content: VERTICAL gap:8
    Title: fs:16/700 #161021 (max 2 lines)
    Body: fs:14/400 #665E75 (max 2 lines)
  Action link: "अभी चैक करें" or "देखें" — fs:14/600 #D9008D (brand pink link)
```
- **Action link is inline text**, not a button — just brand-colored text with SemiBold weight
- **Success notification:** prepend ✅ emoji to title. "✅ रिचार्ज सफल"
- **Body copy uses #665E75** (neutral/700) not primary text — secondary emphasis
- Card appears on dark overlay, not within app UI

---

## 24. Processing/Loading State with Progress Items

For operations processing multiple items simultaneously:
```
Screen bg: #FAF9FC
Title: "पोर्टल पर लॉगिन हो रहा है..." fs:20/400 (Regular, not Bold — it's in-progress)

Item rows: VERTICAL gap:12
  Each: HORIZONTAL gap:8
    Spinner: 24×24 (animated loader)
    User icon: 24×24 #A7A1B2
    Name: fs:16/400 #665E75 (secondary text — not yet confirmed)

Trust banner: 360×72 bg:#F1EDF7
  Shield icon + "आपका ID-पासवर्ड सुरक्षित है" fs:16/400
```
- **Title is Regular weight** during processing — switches to Bold on completion
- **Names are #665E75** (secondary) — not yet confirmed results
- **Trust banner** at bottom for sensitive operations (login, payment)
- **Close (X) button** instead of back arrow — user can cancel/dismiss

---

## 25. Form & Input Design Mastery

### The Input Hierarchy: Scan > Autofill > Select > Type

For every input field, ask: "Can we avoid typing?" The hierarchy from least to most friction:

| Method | Friction | When to use | Wiom example |
|--------|---------|-------------|-------------|
| **Scan** (camera/QR) | Lowest | Structured data on a physical object | Net Box ID, Aadhaar, QR code |
| **Autofill** (from system/API) | Low | Data system already knows | Customer name, phone, ISP details |
| **Select** (radio/dropdown) | Medium | Finite known options | ISP type, plan, device condition |
| **Type** (keyboard) | Highest | Free-form or unique data | WiFi password, amount, notes |

**Rule:** Never ask the user to TYPE something the system can SCAN or AUTOFILL. If Rohit is entering a Net Box ID that's printed on the device, camera scan is the first option, manual entry is the fallback.

### Multi-Step Form Pattern
```
Form with >4 fields → split into logical screens:
  Screen 1: Identity (name, phone, Aadhaar) — 2-3 fields max
  Screen 2: Address/location — 2-3 fields
  Screen 3: Technical (ISP, plan, device) — 2-3 fields
  Screen 4: Review/Summary — read-only, edit links per section
  Screen 5: Confirmation CTA

Progress: segmented bar or radio stepper at top
Back: preserves ALL entered data (ViewModel)
```

### Field-Specific Input Patterns

**Phone number (10 digits):**
```kotlin
OutlinedTextField(
    keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Phone),
    visualTransformation = PhoneVisualTransformation(), // format as XXX-XXX-XXXX
    // Validate: starts with 6-9, exactly 10 digits
)
```
- Leading icon: phone (24×24, #A7A1B2)
- Placeholder: "उदाहरण: 9876543210"
- Auto-advance to next field on 10th digit

**Aadhaar (12 digits):**
```
Input method priority: Camera scan → Manual entry
Camera: rear camera, guide rect for card, dashed border
Manual: 12-digit numeric field, format as XXXX-XXXX-XXXX
Mask display: *****52041 (show last 5 only after entry)
```
- fs:16/400 for masked value, fs:16/700 for unmasked during entry
- Privacy: mask immediately after field loses focus

**Amount (₹):**
```
Prefix: "₹" baked into placeholder and value display
Numeric keyboard only (not full QWERTY)
Format: comma-separated thousands (₹5,000 not ₹5000)
Context card: wallet balance shown above/below field
CTA: includes the amount ("₹5,000 भेजें")
```

**WiFi password (min 8 chars):**
```
Show/hide toggle (eye icon trailing)
Min 8 characters, English only
Real-time char count or validation indicator
```

**OTP / Happy Code (4-6 digits):**
See Section 26 below — segmented boxes, not single field.

### Inline Validation Rules

| Timing | Behavior |
|--------|----------|
| **On keystroke** | Only for format hints (phone: show dash after 3 digits) |
| **On focus loss** | Full validation — show error if invalid |
| **On CTA tap** | Validate ALL fields, scroll to first error |
| **On re-focus** | Clear the error — let user retry without the red staring at them |

### Smart Defaults & Autofill
- **Pre-fill from previous sessions:** if Rohit entered ISP details for this customer before, show them as pre-filled (grey bg #D7D3E0, text #665E75)
- **Pre-fill from scan:** after QR/barcode scan, populate relevant fields automatically
- **Default selections:** if there's a "most likely" option (e.g., ISP plan, device type), pre-select it with radio filled
- **Keyboard type per field:** Phone=numeric, Amount=numeric, Name=text, OTP=numeric, Password=text with visibility toggle

### Scan-First Pattern (QR/Barcode/Camera)
```
Screen structure:
  App bar with close (X) — not back arrow
  Title: instruction ("नेट बॉक्स को फ्रेम के बीच में रखें") — T3_20_Bold or Big header
  Camera viewfinder: flex:1 fills available space
    Guide rect: 328×320 r:16, dashed stroke 2dp
    Dark overlay: box-shadow 0 0 0 9999px rgba(0,0,0,0.55)
  Shutter button: 64×64, fill #D9D9D9, stroke #E8E4F0 2dp
  OR auto-scan (no shutter — detects automatically)

Fallback: "मैन्युअल एंट्री" text link below camera
```
- **Auto-scan preferred** for QR codes (no shutter needed)
- **Manual shutter** for photos (Aadhaar, device photo) where framing matters
- **Always provide manual fallback** — camera may fail, lighting may be bad
- **Guide rect shape matches the target:** card-shaped for Aadhaar, square for QR, rectangular for device label

---

## 26. OTP / Pin Code Input Pattern

Segmented individual digit boxes — NOT a single text field:
```
OTP container: HORIZONTAL gap:8
  Each box: 76×48, bg:#FAF9FC, stk:#E8E4F0, r:12, p:[12,16,12,16]
    Content: fs:16/400 #A7A1B2 (empty), fs:16/700 #161021 (filled)

Purple OTP display (for sharing, not input):
  Container: bg:#F1E5FF, stk:#6D17CE, r:variable, p:[32,12,32,12]
    Each digit: 36×64, bg:#6D17CE, r:4, white text
```

### Two OTP contexts:

**A. User enters OTP (Happy Code, verification):**
- 4 separate boxes (76×48 each), gap:8
- Empty: neutral border (#E8E4F0), placeholder
- Active: darker border on current box
- Filled: value in Bold, neutral border
- Numeric keypad below (not full QWERTY)
- Hint card below: bg:#F9DFEE r:12, p:[8,8,8,8], instruction text fs:14/400

**B. System shows OTP (for user to share with customer):**
- 4 digit holders in purple containers (bg:#6D17CE r:4)
- Wrapped in info card (bg:#F1E5FF stk:#6D17CE)
- Large display — user reads and shares verbally
- Warning info card below: bg:#F1EDF7 r:16, warning_outline icon + explanation

### Hint Card Pattern (below OTP/inputs)
```
bg:#F9DFEE (light pink) or bg:#F1EDF7 (lavender), r:12
p:[8,8-12,8,8-12], HORIZONTAL gap:8
  Indicator icon: 17×24 (custom)
  Instruction text: fs:14/400 #191919 (near-black, not neutral/900)
```
- "कस्टमर से उनकी ऐप में आए हैप्पी कोड को मांगे"
- "यूजर से व्योम ऐप में आए चार अंकों का हैप्पी कोड मांगें"
- Always 1-2 lines. Practical instruction, not explanation

---

## 26. Photo Review / Confirmation Pattern

For camera capture review before submission:
```
Screen structure:
  App bar (back arrow, not close — still in flow)
  Title question: "क्या यह फोटो ठीक है?" fs:20/700
  Subtitle: "देख लें की नेट बॉक्स ID और QR साफ़ दिख रहे हों" fs:16/400
  Photo: 328×328 (large) or 180×180 (compact), r:24 or r:12
    Close/retake button: 24×24 on photo, bg transparent
    Optional highlight rect: 110×110 or 64×64, r:16/r:12 (focus area marker)
  Segmented progress bar
  CTA: confirmation statement as button text
```
- **Photo r:24** for full-size review, **r:12** for compact view
- CTA text is a confirmation STATEMENT: "फोटो में नेटबॉक्स साफ़ दिख रहा है" — not "Submit" or "OK"
- Close/retake overlay on photo top-right (24×24, white icon on dark photo)

---

## 27. Summary Card Pattern (Key-Value Pairs)

For displaying confirmed data before or after an action:
```
Card: bg:#F1EDF7 or bg:#FFFFFF, r:16, p:[8-12,16,8-12,16]
  Row: HORIZONTAL gap:8, p:[4-10,0,4-10,0]
    Label: fs:13/400 #665E75 (secondary)
    Value: fs:13/700 #161021 (primary) or #008043 (positive/new)
  Divider: 296×1, bg:#E8E4F0 (between rows)
```

### Color-coding values:
| Value type | Color | Example |
|-----------|-------|---------|
| Standard data | #161021 (primary) | "NB-4821-7A3F", "वर्मा परिवार" |
| New/positive data | #008043 (positive) | "NB-9102-3B7E" (new device ID) |
| Label | #665E75 (secondary) | "कनेक्शन का नाम:", "पुराना नेट बॉक्स:" |

- Labels end with colon in Hindi: "कनेक्शन का नाम:"
- fs:13 (not 12 or 14) is the summary card scale — tighter than body text
- Dividers between rows (1px #E8E4F0), not between label and value

---

## 28. Wallet / Balance Display Pattern

```
Card: 328×92, bg:#F1EDF7, r:16, p:[16,16,16,16]
  HORIZONTAL layout:
    Left: VERTICAL gap:4
      Label: "Wallet balance" / "वॉलेट बैलेंस" fs:16/400 #161021
      Amount: "₹17,330" fs:24/700 #161021
    Right: illustration (60×60)
```
- Amount is ALWAYS the largest text (fs:24/700)
- Label above amount, not below
- Illustration on right provides visual anchor without competing with the number
- Used above forms where the balance is context (send money, recharge)

---

## 29. PayG Info Tag Pattern

Purple info card for system-level education moments:
```
Container: bg:#F1E5FF, r:8, p:[16,12,16,12]
  Tag: bg:#6D17CE, r:12, p:[4,12,4,12]
    Text: fs:14/600 #FAF9FC
  Content: stk:#6D17CE (left border), p:[0,0,0,8]
    Title: fs:14/600 #161021
    Body: fs:14/400 #161021
```
- **Tag** is a label chip — "PayG – सिस्टम जानकारी"
- Content has a **left purple border** (not full border) — signals "info from system"
- Used at critical PayG consent points
- IVR audio may accompany

---

## 30. Step Progress States (within radio stepper)

| State | Radio icon | Radio color | Label weight | Label color |
|-------|-----------|------------|-------------|------------|
| **Current** | Filled dot (10×10) inside ring | #D9008D (brand) | 600 (SemiBold) | #161021 |
| **Completed** | Tick (checkmark) | #008043 (positive) | 400 (Regular) | #161021 |
| **Pending** | Empty ring | #A7A1B2 (hint) | 400 (Regular) | #A7A1B2 |
| **Half-complete** | Split bar (half filled) | #008043/#E8E4F0 | 400 | #A7A1B2 |

- Completed steps use **tick icon** (not filled radio) — different shape signals "done"
- Current step is the ONLY one with SemiBold label — visual weight draws the eye
- Pending steps are fully muted (#A7A1B2 for both icon and text)
- Connection bars between steps: completed=#D7D3E0, in-progress=#A7A1B2

---

## 31. Dual Action Circle Buttons (from Maverick Session 22-25)

For screens where two equal-weight actions are available (call + directions, call + WhatsApp):
```
Two circles side by side: 48×48 each, bg:#E8E4F0 (neutral/200), r:24
  Icon: 24×24, fill #352D42 (neutral/800)
  Gap: 12-16dp between circles
```
- Both circles are EQUAL size and weight — neither is primary
- bg:#E8E4F0 is the standard circle button background (not white, not brand)
- Icon fill is #352D42 (dark neutral), NOT brand pink
- Used on customer info cards, contact sections

---

## 32. Camera Capture State Machine

Multi-step captures (Aadhaar front+back, device photo+confirmation) follow a state machine, not a flat layout:

```
State 1: CAPTURE (single focus)
  Camera viewfinder fills available space
  Guide rect: dashed stroke (2px dashed, NOT solid) — border-style important
  Dark overlay: box-shadow: 0 0 0 9999px rgba(0,0,0,0.55) on guide rect
  Shutter button: 64×64, fill #D9D9D9, stroke #E8E4F0 2dp
  Close (X) icon — NOT back arrow (cancel intent, not navigate)

State 2: REVIEW (single photo)
  Photo displayed large (328×328 r:24 or 180×180 r:12)
  Retake overlay button on photo (24×24 close/retake icon, white on dark)
  Focus highlight rect on photo (110×110 or 64×64, r:16/r:12)
  CTA: confirmation statement ("फोटो में नेटबॉक्स साफ़ दिख रहा है")

State 3: BOTH (multi-capture complete)
  Both photos shown as smaller cards
  Combined review before final submission

Transition: State 1 → capture → State 2 → confirm → (State 1 for next side OR State 3)
```

**Key Maverick learning:** Build all states with JS/Compose state toggling. Do NOT build them as separate screens — it's one screen with state-dependent content.

---

## 33. Connected Elements (Elevated Card + Tip Bar)

When a card has a visually connected element below (tip, footer, info banner):
```
Elevated card:
  position: relative / zIndex(1)
  Card with shadow + border
  Full border-radius (12.dp all corners)

Connected tip bar:
  Negative margin: offset(y = -12.dp) — overlaps behind the card
  Bottom-only radius: RoundedCornerShape(bottomStart = 12.dp, bottomEnd = 12.dp)
  Extra top padding: 20.dp (to account for 12dp overlap zone)
  bg: #F1E5FF (info purple tint) or #F1EDF7 (neutral/100)
  Height: AUTO — grows with content, never fixed
```
- The overlap is INTENTIONAL — creates visual connection without a border
- Always test with long text to verify auto-sizing works
- The 12dp overlap stays constant regardless of content length

---

## 34. Ghost CTA Pattern

For tertiary actions that shouldn't compete with primary/secondary:
```
Ghost CTA: bg matches screen background (#FAF9FC), NO border, NO stroke
  Text: brand/600 (#D9008D), fs:16/700 or fs:14/600
  Touch target: full width or content-width, 48.dp height
```
- **NEVER add a border** — if Figma shows no stroke, implement with `border:none`
- Ghost CTA on white screen = white bg + pink text
- Ghost CTA on colored screen = matching bg + pink text
- Visually the lightest CTA — only the text color identifies it as interactive

---

## 35. Screen Transitions

### Navigation Transitions
| Transition | When | Compose |
|-----------|------|---------|
| Slide left | Forward in flow (screen 2 → 3) | `slideInHorizontally(initialOffsetX = { it })` |
| Slide right | Back in flow (screen 3 → 2) | `slideOutHorizontally(targetOffsetX = { it })` |
| Fade | Switching tabs or sections | `fadeIn() + fadeOut()` |
| Bottom-up slide | Opening bottom sheet or new context | `slideInVertically(initialOffsetY = { it })` |
| Scale + fade | Opening dialog | `scaleIn(initialScale = 0.9f) + fadeIn()` |

### Duration Rules
- Forward/back navigation: 300ms
- Dialog enter: 250ms, exit: 200ms (exit always faster)
- Bottom sheet: 300ms
- Tab switch: 200ms fade
- Skeleton shimmer: 1500ms loop
- **Easing:** `FastOutSlowInInterpolator` for enter, `FastOutLinearInInterpolator` for exit

---

## Checklist Before Declaring a Screen "Done"

- [ ] Surface selection justified (why this pattern, not another?)
- [ ] Primary CTA bottom-anchored, full-width, 48.dp height, 16.dp radius, text 16/Bold
- [ ] Only ONE primary CTA per screen
- [ ] Dual CTA: stacked vertically (primary top, secondary below, gap:12), NOT side-by-side
- [ ] Secondary CTA uses #FFE5F6 tonal fill (not outlined)
- [ ] Back button behavior defined (preserve inputs? show confirmation?)
- [ ] Loading state: skeleton shimmer matching content layout
- [ ] Empty state: icon + title + explanation + optional action
- [ ] Error state: inline card, what happened + what to do, retry if applicable
- [ ] All interactive elements ≥ 48.dp touch target
- [ ] Snackbar/toast positioned above CTA
- [ ] No hidden features — everything discoverable through explicit UI
- [ ] Dialog uses r:24 (not r:12), 312px wide, p:[32,24,32,24]
- [ ] Bottom sheet has drag handle (120×4, #D7D3E0, r:4), title fs:24/Bold
- [ ] Triple dot dropdown uses r:4 (sharp corners exception)
- [ ] Sequential flow has segmented progress bar (2dp height, gap:8, r:4)
- [ ] Ticket screens have timeline bar (88dp, bg:#F1EDF7) with deadline
- [ ] Persona check: would [Annu/Rohit/Verma] understand this screen in 5 seconds?
