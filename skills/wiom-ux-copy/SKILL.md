# Wiom UX Copy — Solution Design Engine V1

The copy and tone bible for Wiom products. Decides WHAT every screen says — error messages, CTA labels, toasts, empty states, confirmations, hints, instructions — so tech teams can write production-ready copy without waiting for a copywriter.

**Output format:** Ready-to-use Hindi and English strings for `AppStrings` data class or `strings.xml`.

---

## Who Uses This

- **Tech** building screens: "What should this error say? What's the CTA label? What does the empty state show?"
- **Product** reviewing copy: "Is the tone right for this persona? Is the message clear?"

Every string in the Wiom app is a design decision. Bad copy makes good UI feel broken.

---

## Core Principle: Wiom Voice

Wiom speaks like a **helpful neighbor** — warm, direct, practical. Not a corporation. Not a chatbot trying to be clever.

| Wiom IS | Wiom is NOT |
|---------|-------------|
| Direct ("ये करें") | Formal ("कृपया यह कार्य सम्पन्न करें") |
| Warm ("आपकी कमाई आ गई!") | Cold ("Transaction successful") |
| Practical ("इंटरनेट चेक करें") | Vague ("Something went wrong") |
| Reassuring ("चिंता न करें, पैसे वापस आ जाएंगे") | Dismissive ("Error occurred") |
| Outcome-first ("₹300 जमा करें") | Action-first ("Submit payment") |
| Bilingual-natural ("WiFi Setup करें") | Forced Hindi ("बेतार जाल स्थापित करें") |

### 10 Tonal Patterns (extracted from real PA flows)

**1. First-Person CTAs — speak AS the user, not TO the user**
```
"फिर से चेक करता हूँ"     ← CTA is in user's voice ("I'll check again")
"कन्फर्म होने का इंतज़ार करें" ← instruction voice ("wait for confirmation")
"काम जारी रखें"           ← user's intent ("continue working")
```
When the CTA represents the USER'S decision, write it in first person. When it's an instruction from the system, use imperative.

**2. Loss Aversion + Opportunity Framing**
```
"नया नेट बॉक्स सेट अप करने का मौका ना खोएं"  ← "don't lose the opportunity"
"जल्दी से नेट बॉक्स पिकअप करें और 1 कैच पकड़ें"  ← opportunity + gamification
```
Frame inaction as LOSING something, not just missing something. Works for Annu (earnings) and Rohit (tasks).

**3. Specificity in Reassurance — always exact amounts, timeframes, names**
```
"अगर कस्टमर 1 महीने के अंदर नेट बंद करता है, तब ₹300 की भरपाई Wiom करेगा"
  ← exact timeframe (1 month) + exact amount (₹300) + who pays (Wiom)
"5:02PM तक पक्का कन्फर्म हो जाएगा"
  ← exact time, not "जल्दी"
"@Varun_pm का पोर्टल पर रिचार्ज सफल हुआ"
  ← names the specific person
```
NEVER vague reassurance. Annu trusts numbers. Specificity = trust.

**4. Earning Angle — connect every action to कमाई**
```
"अभी जा कर अपना कमाई का हिस्सा ले लें"  ← recharge success → "collect your earnings"
"पैसे कमाएं"  ← tab label is the OUTCOME, not "ISP Recharge" or "Tasks"
"1 कैच पकड़ें"  ← gamification of earnings
```
Every notification, every CTA where earnings are involved — mention the earning, not just the task.

**5. Gamification Language — cricket metaphor**
```
"1 कैच पकड़ें"  ← "catch 1 catch" — cricket metaphor for completing a pickup task
```
Wiom uses cricket metaphors for task completion. "कैच" = completed task/bonus captured. Natural for the Hindi-belt audience.

**6. Processing = Continuous Tense + Ellipsis**
```
"पोर्टल पर लॉगिन हो रहा है..."  ← "...हो रहा है..." + "..."
"प्रोसेस हो रहा है..."
"कन्फर्म हो रहा है..."
```
Three dots (...) at end is the Hindi convention for "still happening." Title uses Regular weight during processing, Bold on completion.

**7. Questions Frame Choices, Statements Frame Actions**
```
Questions (user must decide):
  "ऑफिस कैसे भेजना चाहते हैं?"
  "आपको कितने नेट बॉक्स चाहिए?"
  "क्या आप नेट बॉक्स वापस ले आए?"

Statements (system is acting or informing):
  "ISP बदलें"
  "कस्टमर डिटेल्स"
  "नेट बॉक्स बदलें"
```
If the user needs to make a choice → question. If the system is providing info or the action is clear → statement.

**8. Tab/Section Labels Are Outcome Words**
```
"नेटवर्क बढ़ाएं"  ← "grow network" — what Annu GETS
"पैसे कमाएं"     ← "earn money" — what Annu GETS
NOT: "इंस्टॉलेशन टास्क" or "ISP रिचार्ज"
```
Labels name the OUTCOME for Annu, not the feature or system concept. He navigates by what he'll achieve, not what the system calls it.

**9. Screen Titles Are Possessive/Relational**
```
"मेरे नेट बॉक्स"  ← "MY net boxes" — possessive
"कस्टमर डिटेल्स" ← "customer's details" — relational
"टीम को पिकअप टिकट भेजें" ← "send to team" — relational
NOT: "नेट बॉक्स लिस्ट" or "डिटेल पेज"
```
Titles establish relationship between the user and the content. Never use system-language ("list", "page", "screen").

**10. Consequence Statements Are Specific + Actionable**
```
"आपको इस टिकट पे दोबारा काम करना होगा"
  ← what WILL happen + what THEY'LL need to do
"बिना फोटो सबमिट किये टिकट को बंद करना चाहते हैं?"
  ← names the specific thing they're skipping
"रद्द करने के बाद दोबारा बुक करना होगा"
  ← consequence + effort to redo
```
Every warning/consequence statement must answer: what happens + what they'll need to do because of it.

---

### Language Rules
- **Primary:** Hindi (Devanagari) for all customer-facing text in Tier 2/3 context
- **Natural mixing:** English tech terms in Devanagari sentence is CORRECT. "WiFi", "OTP", "Net Box", "ISP", "Recharge" stay in English
- **Never** translate universally understood English terms to Hindi. "WiFi" not "बेतार संजाल"
- **Romanized Hindi** (transliteration) is acceptable ONLY in code comments and variable names, never in user-facing UI
- **Numbers:** always in standard numerals (1, 2, 3), never Devanagari numerals (१, २, ३)
- **Currency:** always "₹" prefix with no space: "₹300", "₹23/दिन"

---

## Persona Tone Calibration

### Annu Bhaiyya (CSP/Partner)
- **Address as:** respectful but direct. Not formal
- **Tone:** encouraging, earnings-focused, practical
- **Vocabulary:** simple Hindi + common English business terms (payment, bonus, team, rating)
- **Numbers:** always prominent. Annu trusts numbers
- **Urgency:** gentle nudges, never pressure. "आज का टास्क बाकी है" not "जल्दी करें!"
- **Example:** "आज की कमाई: ₹450 | 3 टास्क बाकी हैं"

### Technician Rohit (Expert/Partner)
- **Address as:** colleague. Slightly more direct
- **Tone:** instructional, step-focused, efficient
- **Vocabulary:** Hindi + technical English freely (ISP, PPPoE, DHCP, WiFi, Net Box, OTP)
- **Numbers:** step counts matter. "स्टेप 3/8" gives him progress context
- **Urgency:** task completion oriented. "अगला स्टेप: WiFi कनेक्ट करें"
- **Example:** "Net Box ID डालें — डिवाइस के पीछे लिखा है"

### Verma Parivar (Customer)
- **Address as:** family. Warm, protective, patient
- **Tone:** conversational (via Vyom bot), reassuring, simple
- **Vocabulary:** simplest possible Hindi. Avoid ALL technical terms except "WiFi", "Net Box", "Recharge"
- **Numbers:** always in context of benefit. "₹23 में पूरे दिन का इंटरनेट"
- **Urgency:** never. Patience. "जब आपको सही लगे, तब करें"
- **Example:** "आपका नेट बॉक्स लग गया है! 2 दिन का फ्री इंटरनेट चालू हो गया"

---

## 1. CTA Labels — Outcome-First

The button tells the user WHAT WILL HAPPEN, not what they're doing to the system.

### Pattern: `[outcome] + [context if needed]`

| Bad (action-first) | Good (outcome-first) | Why |
|----|------|-----|
| Submit | ₹300 जमा करें | Shows the outcome (deposit) + amount |
| Next | WiFi सेटअप करें | Shows what the next step achieves |
| Confirm | हाँ, पहुँच गया हूँ | States the confirmation in the user's voice |
| Save | नंबर सेव करें | Names what's being saved |
| Cancel | बुकिंग रद्द करें | Names what's being cancelled |
| OK | समझ गया | Acknowledges understanding in user's voice |
| Delete | हटाएं | Direct verb for the action |
| Login | लॉग इन करें | Direct verb (English term natural in Hindi) |
| Send | OTP भेजें | Names what's being sent |

### CTA Anti-patterns
- **Never "OK"** — always a specific verb or acknowledgment
- **Never "Cancel" as a dismiss label** — use "रहने दें", "बाद में", or "वापस जाएं"
- **Never English-only CTAs** in Hindi-primary screens (exceptions: "WiFi", "OTP" as part of Hindi sentence)
- **Never vague:** "आगे बढ़ें" is acceptable ONLY when the next step is self-evident from context. Otherwise name the outcome
- **Loading state CTA:** text becomes "प्रोसेस हो रहा है..." with spinner. Never leave the button text unchanged during processing

### Primary vs Secondary CTA Copy (Stacked Dual CTA)
Wiom uses vertically stacked CTA pairs — primary on top, secondary below. The copy pattern:
```
Primary (solid #D9008D):    The main action / safe path
Secondary (tonal #FFE5F6):  The alternative route / different path
```

### Real Dual CTA Examples (from PA flows)
| Screen | Primary (top) | Secondary (below) | Logic |
|--------|--------------|-------------------|-------|
| Pickup Ticket | "टिकट बंद करें" | "कस्टमर को रिचार्ज करना है" | Close ticket vs route to ISP |
| Payment Done | "अन्य टास्क देखें" | "रिचार्ज @ ISP खोलें" | Navigate out vs continue flow |
| Pending Popup | "काम जारी रखें" | "अन्य काम देखें" | Stay in flow vs exit |
| Confirm Delivery | (checkbox CTA) "Yes, I've received 10 net boxes" | — | Physical confirmation |
| Installation | Action CTA | Alternative route | Flow progression |

**Pattern:** Primary = the expected next step. Secondary = a valid alternative the user might need. Both are outcome-first, both are specific verbs.

- **Never** generic secondary like "बाद में" when a specific alternative exists
- Destructive CTA: clear consequence. "हाँ, रद्द करें" not just "रद्द करें"

---

## 2. Error Messages — What Happened + What To Do

### Template
```
[क्या हुआ — 1 line] + [क्या करें — 1 line]
```

### Error Message Library

| Error type | Message (Hindi) | English fallback |
|-----------|----------------|-----------------|
| **No internet** | कनेक्शन में दिक्कत है। इंटरनेट चेक करें और दोबारा कोशिश करें | Connection issue. Check your internet and try again |
| **Server error** | कुछ गड़बड़ हो गई। थोड़ी देर बाद दोबारा कोशिश करें | Something went wrong. Try again in a moment |
| **Timeout** | जवाब आने में ज़्यादा वक़्त लग रहा है। दोबारा कोशिश करें | Taking too long. Please try again |
| **Invalid OTP** | OTP गलत है। दोबारा डालें या नया OTP भेजें | Incorrect OTP. Re-enter or request a new one |
| **Invalid phone** | फ़ोन नंबर सही नहीं है। 10 अंकों का नंबर डालें | Invalid phone number. Enter a 10-digit number |
| **Invalid Aadhaar** | आधार नंबर सही नहीं है। 12 अंकों का नंबर डालें | Invalid Aadhaar. Enter a 12-digit number |
| **Empty required field** | यह भरना ज़रूरी है | This field is required |
| **Password too short** | पासवर्ड कम से कम 8 अक्षर का होना चाहिए | Password must be at least 8 characters |
| **Device not found** | यह Net Box ID नहीं मिला। दोबारा चेक करें | Net Box ID not found. Please recheck |
| **Camera permission denied** | कैमरा इस्तेमाल करने की परमिशन दें | Please allow camera permission |
| **Location permission denied** | लोकेशन ऑन करें — सर्विस एरिया चेक करने के लिए ज़रूरी है | Turn on location — needed to check service area |
| **Payment failed** | पेमेंट नहीं हो पाई। दोबारा कोशिश करें या दूसरा तरीक़ा चुनें | Payment failed. Try again or choose another method |
| **File too large** | फ़ाइल बहुत बड़ी है। 5MB से छोटी फ़ोटो चुनें | File too large. Choose a photo under 5MB |
| **No service area** | अभी आपके एरिया में सर्विस उपलब्ध नहीं है। जल्द आ रही है! | Service not available in your area yet. Coming soon! |
| **Session expired** | सेशन ख़त्म हो गया। दोबारा लॉग इन करें | Session expired. Please log in again |

### Error Copy Rules
- **Never blame the user.** "कनेक्शन में दिक्कत है" not "आपने इंटरनेट बंद कर रखा है"
- **Always tell what to do next.** Every error ends with an action step
- **Be specific.** "10 अंकों का नंबर डालें" not "सही नंबर डालें"
- **Retry-able errors** get a "दोबारा कोशिश करें" button. Non-retry-able errors explain what to do instead
- **Annu/Verma:** if the error is technical, translate to consequence. "Net Box कनेक्ट नहीं हो पा रहा" not "Provisioning failed"
- **Never use error codes** in user-facing text. Log them for tech, show human language to users

---

## 3. Toast/Snackbar Copy

### Template
```
[Verb past tense] — max 2 lines
```

### Toast Library

| Action | Toast (Hindi) | Duration |
|--------|--------------|----------|
| Saved | सेव हो गया | 3s |
| Copied | कॉपी हो गया | 3s |
| Sent | भेज दिया | 3s |
| Deleted (with undo) | हटा दिया · वापस लाएं | 5s |
| Downloaded | डाउनलोड हो गया | 3s |
| Updated | अपडेट हो गया | 3s |
| Photo captured | फ़ोटो ले ली गई | 3s |
| Connection restored | इंटरनेट वापस आ गया | 3s |
| Recharge successful | रिचार्ज हो गया! ₹[X] | 3s |

### Toast Copy Rules
- **Past tense** — confirms what already happened. "हो गया", "ले ली गई", "भेज दिया"
- **Max 2 lines.** If you need more, it's not a toast — it's an inline banner
- **No period/full stop** at the end
- **Action toasts:** include the undo/action label after `·` separator: "हटा दिया · वापस लाएं"
- **Never use toasts for errors.** Errors need persistent inline display
- **Success toasts for money:** include the amount. "₹300 जमा हो गए!" not just "पेमेंट हो गई"

---

## 4. Empty State Copy

### Template
```
[Icon or illustration]
[Title: what's missing — T4_16_Bold]
[Explanation: when/why it'll appear — B2_14_Regular, neutral/700]
[Optional CTA if user can take action]
```

### Empty State Library

| Screen | Title | Explanation | CTA |
|--------|-------|-------------|-----|
| Task list | कोई टास्क नहीं है | नए टास्क आने पर यहाँ दिखेंगे | रिफ्रेश करें |
| Earnings | अभी कोई कमाई नहीं है | टास्क पूरे करने पर कमाई यहाँ दिखेगी | टास्क देखें |
| Team list | अभी टीम में कोई नहीं है | नए सदस्य जोड़ने पर यहाँ दिखेंगे | सदस्य जोड़ें |
| Notifications | कोई नई सूचना नहीं | नई सूचना आने पर यहाँ दिखेगी | — |
| Search results | कुछ नहीं मिला | दूसरे शब्दों से खोजें | — |
| Recharge history | अभी कोई रिचार्ज नहीं हुआ | रिचार्ज करने पर यहाँ दिखेगा | रिचार्ज करें |
| Chat history | अभी कोई मैसेज नहीं | Vyom से बात करने के लिए नीचे लिखें | — |

### Empty State Rules
- **Never** just "कोई डेटा नहीं" — always explain WHEN content will appear
- **Tone:** reassuring, not apologetic. "अभी" implies it's temporary
- **CTA:** only if the user can directly take action to fill the empty state
- **Illustration/icon:** use a relevant icon from the DS, tinted `neutral/500`

---

## 5. Confirmation Dialog Copy

### Template
```
Title:  [क्या करना चाहते हैं?]
Body:   [consequence — what will happen]
Primary button:   [safe option — रहने दें / नहीं]
Secondary button: [action — specific verb]
```

### Confirmation Library

| Context | Title | Body | Primary (safe) | Secondary (action) |
|---------|-------|------|----------------|-------------------|
| Cancel booking | बुकिंग रद्द करें? | आपकी ₹100 बुकिंग फीस [X] दिनों में वापस आ जाएगी | नहीं, रहने दें | हाँ, रद्द करें |
| Exit with unsaved data | वापस जाएं? | बदलाव सेव नहीं होंगे | यहीं रहें | वापस जाएं |
| Logout | लॉग आउट करें? | — | रहने दें | लॉग आउट करें |
| Delete item | [item] हटाएं? | यह हमेशा के लिए हट जाएगा | रहने दें | हटाएं |
| Submit form | जानकारी सबमिट करें? | सबमिट करने के बाद बदलाव नहीं होगा | वापस जाएं | सबमिट करें |
| Gate (PayG consent) | PayG मॉडल | कस्टमर सिर्फ़ ₹300 सिक्योरिटी फीस देगा... | — | समझ गया |
| Permission | [X] की परमिशन चाहिए | [X] के लिए ज़रूरी है | बाद में | परमिशन दें |

### Confirmation Copy Rules
- **Safe option is ALWAYS the primary (prominent) button** — the user who taps without reading should stay safe
- **Destructive option is ALWAYS secondary** with `negative/600` text color
- **Gate dialogs** (consent, education) have only ONE button — no escape path
- **Never "OK" / "Cancel"** — always specific verbs
- **Body text:** state the consequence, not just repeat the title. "₹100 वापस आ जाएगी" adds information. "आप बुकिंग रद्द कर रहे हैं" adds nothing

### Dialog Title Size Encodes Severity
| Dialog type | Title size | Example |
|------------|-----------|---------|
| Success | `T2_24_Bold` (fs:24) | "10 नए नेट बॉक्स की रिक्वेस्ट भेज दी गई है" |
| Success (payment) | `T2_24_Bold` (fs:24) | "पेमेंट हो गयी" |
| Warning/pending | `T3_20_Bold` (fs:20) | "बिना फोटो सबमिट किये टिकट को बंद करना चाहते हैं?" |

**Rule:** Success = fs:24 (celebration, bigger). Warning = fs:20 (caution, slightly smaller, more body text space for consequence explanation).

---

## 6. Instruction and Guidance Copy

### In-Flow Instructions (Rohit's sequential steps)
```
[What to do] + [Where/how to find it] — max 2 lines
```

| Step | Instruction |
|------|------------|
| Device ID entry | Net Box ID डालें — डिवाइस के पीछे लिखा है |
| Selfie capture | Wiom T-शर्ट पहनकर सेल्फी लें |
| 3-pin plug | 3-पिन प्लग लगाएं — फ़ोटो में प्लग दिखना चाहिए |
| WiFi setup | WiFi का नाम डालें — कस्टमर इसी नाम से कनेक्ट करेगा |
| Happy Code | कस्टमर से 4-अंकों का Happy Code लें |

### PayG System Info Cards
```
Card bg: info/600 tint (light purple)
Tag: "PayG — सिस्टम जानकारी" in info/600 solid tag
Body: [explanation in B2_14_Regular]
```
- Used to explain PayG model rules at critical steps
- IVR audio instruction may accompany these — note in spec

### Tooltip/Info Copy
- Triggered by `(i)` icon tap
- Max 3 lines
- Explain the term or value in simple Hindi
- Example: "रेटिंग बोनस → अच्छी रेटिंग मिलने पर हर महीने बोनस मिलता है"

---

## 7. Push Notification Copy

### Rich Notification Template
```
Title: [outcome/event] — fs:16/700, max 2 lines
Body: [context + what to do] — fs:14/400, #665E75, max 2 lines
Link: [action verb] — fs:14/600, brand pink
```

### Notification Copy Library
| Event | Title | Body | Link |
|-------|-------|------|------|
| New pickup ticket | "आपको एक नई पिकअप टिकट मिली है" | "जल्दी से नेट बॉक्स पिकअप करें और 1 कैच पकड़ें" | "अभी चैक करें" |
| Recharge success | "✅ रिचार्ज सफल" | "@[name] का पोर्टल पर रिचार्ज सफल हुआ, अभी जा कर अपना कमाई का हिस्सा ले लें" | "देखें" |
| New setup task | "नया नेट बॉक्स सेट अप मिला है" | (with illustration) | "अभी देखें" |
| Device shipped | "आपके आर्डर किये नेट बॉक्स अब पैक हो रहे हैं" | — | — |

### Notification Copy Rules
- **Title = what happened.** Body = what to do about it
- **Success:** prepend ✅ emoji to title
- **Earning angle:** mention "कमाई" whenever earnings are involved — Annu cares
- **Link text:** always a verb. "देखें", "चैक करें", "अभी देखें" — never "यहाँ क्लिक करें"
- **Body uses #665E75** (secondary text) — the title carries the weight

### Urgency Copy
For social proof / queue-based urgency:
```
"10 more customers are waiting..."  — fs:20/700, brand color
```
- Overlapping avatar stack provides visual urgency
- Use sparingly — only when genuine queue/competition exists
- **Never fake urgency.** If there's no queue, don't show this pattern

---

## 8. Processing State Copy

### In-Progress Title Convention
```
During processing: "पोर्टल पर लॉगिन हो रहा है..." — fs:20/Regular (not Bold)
After completion:  "लॉगिन हो गया" — fs:20/Bold
```
- **Regular weight during processing** — signals "not done yet"
- **Bold weight on completion** — signals finality
- **Ellipsis (...)** at the end during processing — Hindi convention for ongoing action

### Trust Banner Copy
For sensitive operations (ISP login, payment processing):
```
"आपका ID-पासवर्ड सुरक्षित है" — with shield icon, bg:#F1EDF7
```
- Placed at bottom of processing screen
- Always present during credential-related operations
- Reassures without interrupting the flow

---

## 9. Bottom Sheet Copy Patterns

### Sheet Headings (fs:24/700, always a question or noun)
| Sheet type | Heading example |
|-----------|----------------|
| Choice | "ऑफिस कैसे भेजना चाहते हैं?" |
| Request | "आपको कितने नेट बॉक्स चाहिए?" |
| Confirm pickup | "क्या आप नेट बॉक्स वापस ले आए?" |
| Customer info | "कस्टमर डिटेल्स" |
| ISP change | "ISP बदलें" |

**Pattern:** Questions for action sheets, nouns for info sheets.

### Info Tag Copy (inside sheets)
Small contextual label: bg:#F1E5FF, text #6D17CE, r:4
```
"इस कस्टमर ने पेमेंट की हुई है"  — states a fact about the entity
"Bharti Airtel Ltd., Telemedia Services"  — identifies the current selection
```
- Used to label context without a full sentence
- fs:16/700 for state labels, fs:12/400 for entity identifiers

### Add-New Link Copy
At bottom of selection lists in sheets:
```
"+ नया टीम मेंबर जोड़ें"  — fs:14/600 #D9008D
"नया पोर्टल खोलें"  — fs:16/700 #D9008D
```
- Always prefixed with "+" or "नया"
- Brand pink, no button container
- Action verb + what's being created

### Reassurance Copy (in payment/commitment sheets)
```
"आप बेफिक्र होकर इंस्टॉलेशन करें"  — fs:24/700, reassurance headline
"अगर कस्टमर 1 महीने के अंदर नेट बंद करता है, तब ₹300 की भरपाई Wiom करेगा"
  — fs:16/500 #665E75, specific guarantee
```
- Lead with reassurance ("बेफिक्र"), follow with specific guarantee
- Mention the exact amount and timeframe — Annu trusts specifics

---

## 10. Input Screen Copy Patterns

### Amount Input
```
Title (default): "Enter amount" / "अमाउंट डालें" — fs:16/700 (default), fs:16/400 (active)
Placeholder: "₹100" — fs:16/400 #A7A1B2
Filled value: "₹5,000" — fs:16/700 #161021
CTA: "Confirm your bank details" / "अपने बैंक की डिटेल्स कन्फर्म करें"
```
- **Title weight changes by state:** Bold when empty (draws attention to the field), Regular when active (focus shifts to the value being typed)
- Placeholder includes ₹ symbol + example amount
- CTA is about the NEXT step, not "Submit"

### OTP/Pin Copy
```
Title: "लॉटरी पाने के लिए हैप्पी कोड डालें" — fs:24/700 (Big header variant)
    OR: "हैप्पी कोड डालें" — fs:24/700
Hint: "कस्टमर से उनकी ऐप में आए हैप्पी कोड को मांगे" — fs:14/400 in pink hint card
    OR: "यूजर से व्योम ऐप में आए चार अंकों का हैप्पी कोड मांगें"
```
- **Title motivates:** "लॉटरी पाने के लिए" — earning angle even in OTP entry
- **Hint is practical instruction:** tells Rohit exactly what to say to the customer
- OTP for sharing: "यह OTP शेयर करें और कस्टमर से ऐप में डालने के लिए कहें" — complete instruction in one line

### Photo Review Copy
```
Question: "क्या यह फोटो ठीक है?" — fs:20/700
Instruction: "देख लें की नेट बॉक्स ID और QR साफ़ दिख रहे हों" — fs:16/400
CTA: "फोटो में नेटबॉक्स साफ़ दिख रहा है" — confirmation STATEMENT as CTA
```
- CTA is a statement of fact, not an action verb — user is confirming what they see
- Instruction names what to check specifically (ID and QR), not "check the photo"

### Condition Assessment Copy
```
Question: "नेटबॉक्स किस हालत में है?" — fs:20/700
Subtitle: "इस जानकारी को वेरीफाई किया जायेगा" — fs:16/400 #665E75
Options: "नेटबॉक्स अच्छी हालत में है, कोई नुक्सान नहीं" — fs:14/400 inside radio card
Follow-up: "क्या एडाप्टर वापस मिला?" — fs:20/700 (second question on same screen)
```
- Multiple questions on one screen: each gets its own section with fs:20/700 title
- Subtitle in #665E75 (not primary) — it's context, not the question

### Summary Card Copy (key-value pairs)
```
Label: "कनेक्शन का नाम:" — fs:13/400 #665E75, ends with colon
Value: "वर्मा परिवार" — fs:13/700 #161021
New value: "NB-9102-3B7E" — fs:13/700 #008043 (green = new/positive)
Old value: "NB-4821-7A3F" — fs:13/700 #161021 (black = existing)
```
- Labels end with colon + space in Hindi
- New/positive data in green, existing data in primary black
- fs:13 specifically for summary cards (tighter than body text)

### Success Completion Copy
```
Title: "नेट बॉक्स सफलतापूर्वक बदल दिया गया!" — fs:24/700
CTA: "बचे हुए काम देखें" — navigates to remaining tasks
```
- "सफलतापूर्वक" (successfully) + action in past tense
- Exclamation mark for celebration
- CTA points to what's NEXT, not the completed action

---

## 11. Current vs Completion State Copy (from Maverick correction #3)

Labels and copy MUST reflect the CURRENT state, not the end state:

| Element state | Copy | NOT |
|--------------|------|-----|
| Payment pending | "पेमेंट **करें**" | ~~"पेमेंट हो गयी"~~ |
| Payment done | "पेमेंट **हो गयी**" | |
| Photo not taken | "**फोटो लें**" | ~~"फोटो ले ली गई"~~ |
| Photo captured | "**फोटो ले ली गई**" | |
| OTP not entered | "OTP **डालें**" | ~~"OTP वेरीफाई हो गया"~~ |
| OTP verified | "OTP **वेरीफाई हो गया**" | |

**The rule:** imperative ("करें") for pending actions, past tense ("हो गयी") for completed ones. Never show the completion text while the action is still pending — it creates false confidence.

This applies to:
- Checklist items in step flows
- Progress bar step labels
- Status chips
- CTA labels

---

## 12. Timeline and Deadline Copy

### Deadline Pattern
```
[start date] ............ [end date] तक काम पूरा करें
```
- Start date: fs:12/Regular — just the date, no emphasis
- End date: fs:12/SemiBold — bolded to draw attention
- Deadline phrase: fs:12/Regular — "तक काम पूरा करें" (complete work by)
- Date format: "[DD] [month in Hindi]" — "7 अगस्त", "15 अगस्त"

### Timeline Status Labels
| State | Label | Font |
|-------|-------|------|
| Start | "[date]" | fs:12/Regular |
| Deadline | "[date] तक काम पूरा करें" | date=fs:12/SemiBold, rest=fs:12/Regular |
| Overdue | "[date] तक पूरा होना चाहिए था" | fs:12/SemiBold, warning/600 color |

### Navigation Link Copy
For inline "next step" links (not CTAs):
```
"इसके बाद →"  — brand/600 color, fs:16/Bold + arrow icon
```
- Used in education flows as inline navigation
- Not a CTA — no button container, just text + arrow
- Always brand pink, always bold

---

## 8. Status Labels

### Active Status Chips

| Status | Hindi | Color | Icon |
|--------|-------|-------|------|
| Active/Running | चालू है | `positive/600` | `check_circle` |
| Pending/Incomplete | बाकी है | `warning/600` | `schedule` |
| Error/Failed | दिक्कत है | `negative/600` | `error` |
| Processing | प्रोसेस हो रहा है | `info/600` | `hourglass_empty` |
| Completed | पूरा हो गया | `positive/600` | `check_circle` |
| Cancelled | रद्द हो गया | `neutral/500` | `cancel` |
| Searching | ढूंढा जा रहा है | `info/600` | `search` |
| Refund initiated | रिफंड शुरू हो गया | `info/600` | `payments` |

### Status Copy Rules
- **Always in present tense or recent past:** "चालू है", "हो गया", "हो रहा है"
- **Never future tense for status.** "होगा" is for projections, not current state
- **Pair with color + icon** — never status text alone
- **Customer app (Verma):** status appears as chat bubble from Vyom, not as a chip

---

## 8. Bilingual String Pattern

### AppStrings Data Class Convention
```kotlin
data class AppStrings(
    // Screen: Task List
    val taskListTitle: String,
    val taskListEmpty: String,
    val taskListEmptyHint: String,
    val taskListRefresh: String,

    // Screen: Payment
    val paymentTitle: String,
    val paymentAmount: String, // with placeholder: "₹{amount} जमा करें"
    val paymentSuccess: String,
    val paymentFailed: String,
    val paymentRetry: String,
    // ...
)

val HindiStrings = AppStrings(
    taskListTitle = "टास्क",
    taskListEmpty = "कोई टास्क नहीं है",
    taskListEmptyHint = "नए टास्क आने पर यहाँ दिखेंगे",
    taskListRefresh = "रिफ्रेश करें",
    paymentTitle = "पेमेंट",
    paymentAmount = "₹%s जमा करें",
    paymentSuccess = "₹%s जमा हो गए!",
    paymentFailed = "पेमेंट नहीं हो पाई। दोबारा कोशिश करें",
    paymentRetry = "दोबारा कोशिश करें",
)

val EnglishStrings = AppStrings(
    taskListTitle = "Tasks",
    taskListEmpty = "No tasks yet",
    taskListEmptyHint = "New tasks will appear here",
    taskListRefresh = "Refresh",
    paymentTitle = "Payment",
    paymentAmount = "Pay ₹%s",
    paymentSuccess = "₹%s deposited!",
    paymentFailed = "Payment failed. Please try again",
    paymentRetry = "Try again",
)
```

### String Placeholder Convention
- Use `%s` for string substitution (amounts, names, counts)
- Use `%d` for integer-only values (step numbers, counts)
- Amount format: always "₹" prefix, no space, no decimals for round amounts
- Date format: "DD MMM YYYY" ("25 मार्च 2026") — never slash format in Hindi
- Time format: "HH:MM AM/PM" — 12-hour for user-facing, 24-hour for tech logs

---

## 9. Audio/IVR Script Conventions (Rohit's Flows)

Some screens in technician flows have IVR audio that plays alongside the visual screen.

### Convention
- Audio script matches screen text but is MORE conversational
- Screen: "3-पिन प्लग लगाएं" → Audio: "अब 3-पिन वाला प्लग लगा दीजिए। ध्यान रहे कि फ़ोटो में प्लग साफ़ दिखना चाहिए"
- Audio adds context that the screen doesn't have space for
- Mark audio screens in specs with `// 🔊 IVR audio accompanies this screen`

---

## 10. Copy Review Checklist

- [ ] Every CTA is outcome-first (names what happens, not "Submit"/"OK")?
- [ ] Every error message has: what happened + what to do?
- [ ] No error blames the user?
- [ ] All confirmation dialogs use specific verbs (no "OK"/"Cancel")?
- [ ] Safe option is the prominent button in destructive confirmations?
- [ ] Empty states explain WHEN content will appear?
- [ ] Status labels in present/recent past tense?
- [ ] Hindi text uses Devanagari script (not Romanized)?
- [ ] English tech terms kept in English (WiFi, OTP, Net Box, ISP)?
- [ ] Numbers in standard numerals (1,2,3), not Devanagari (१,२,३)?
- [ ] Currency formatted as "₹[amount]" with no space?
- [ ] Toast copy is past tense, max 2 lines, no period?
- [ ] Persona tone check: would this sound right if read aloud to [Annu/Rohit/Verma]?
