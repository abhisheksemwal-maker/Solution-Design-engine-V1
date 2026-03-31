# Wiom Solution Design Engine V1 — Workflow & Audit Protocol

## The Problem This Solves

Without this workflow, builds skip design quality gates. The audit just proved it: an APK was deployed with 7 critical issues (hardcoded values, missing previews, no fonts, no error states). The engine must enforce audit BEFORE deploy.

---

## Pipeline: PRD → Design → Build → Audit → Deploy

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐     ┌──────────┐     ┌────────┐
│  PRODUCT     │ ──→ │  DESIGN      │ ──→ │  BUILD      │ ──→ │  AUDIT   │ ──→ │ DEPLOY │
│  PRD input   │     │  Engine      │     │  Kotlin/    │     │  Pratibimb│     │ APK on │
│              │     │  Skills      │     │  Compose    │     │  56-point │     │ device │
└─────────────┘     └──────────────┘     └─────────────┘     └──────────┘     └────────┘
                                                                  │
                                                          FAIL? ──┤──→ FIX → re-audit
                                                                  │
                                                          PASS? ──┘──→ DEPLOY
```

**The audit gate is MANDATORY. No APK gets installed without passing it.**

---

## Stage 1: PRODUCT INPUT

**Who:** Product team (Abhishek or PM)
**What:** PRD document, Figma links, or verbal brief
**Output:** Scoped feature with user flow defined

No skills needed. This is the raw input.

---

## Stage 2: DESIGN DECISIONS (Engine Skills)

**Who:** Claude Code with engine skills loaded
**Skills used:**

| Skill | Loaded via | Decides |
|-------|-----------|---------|
| `wiom-interaction-patterns` | Auto-loaded or `/wiom-interaction-patterns` | Which pattern (bottom sheet type, CTA style, dialog variant, form type, progress pattern, camera state machine) |
| `wiom-visual-craft` | Auto-loaded or `/wiom-visual-craft` | Color tokens (WHEN to use each), radius rules, typography mapping, elevation, spacing, motion specs |
| `wiom-ux-copy` | Auto-loaded or `/wiom-ux-copy` | CTA labels, error messages, status labels, tone per persona, bilingual strings |

**Process:**
1. Read the PRD
2. Identify screens needed
3. For each screen, consult interaction-patterns: "What pattern fits this user moment?"
4. For each pattern, consult visual-craft: "What tokens, radius, spacing?"
5. For all text, consult ux-copy: "What does this say in Hindi? What tone?"
6. Output: Screen plan with patterns, specs, and copy decided

**Anti-pattern:** Do NOT skip this stage and go straight to building. Every screen must have a pattern decision before code is written.

---

## Stage 3: BUILD (Kotlin/Compose)

**Who:** Claude Code with build skills loaded
**Skills used:**

| Skill | Role in build |
|-------|--------------|
| `wiom-frontend-dev` | Kotlin/Compose standards: @Preview (2 per screen), Dimens.kt (no hardcoded values), sealed UI state, collectAsStateWithLifecycle, bilingual AppStrings pattern, project structure |
| `maverick-developer` | Integration protocol: cross-check against engine skills for DS consistency, fill gaps where PRD/Figma is incomplete, apply 35 correction patterns |

**Build rules (from wiom-frontend-dev):**
- EVERY dp/sp value goes in `Dimens.kt` — zero hardcoded values in composables
- EVERY color goes in `Color.kt` — zero hardcoded hex in composables
- EVERY string goes in `AppStrings.kt` — zero hardcoded text in composables
- 2 @Preview per screen (Hindi + English)
- Bundle Noto Sans + Noto Sans Devanagari fonts
- Sealed UI state for async operations
- Screen structure: `Column(fillMaxSize, background) { AppBar; Content(weight(1f), scroll); CTA(bottomBar) }`

**Maverick integration check (during build):**
- Is the CTA radius 16.dp? (not 12)
- Is the secondary CTA filled #FFE5F6? (not outlined)
- Is the dialog radius 24.dp? (not 16)
- Is the input active stroke #352D42? (not brand/600)
- Are icons filled or outlined correctly for context?
- Camera screens: close (X) not back arrow

**Output:** Compilable Kotlin project, all files written.

---

## Stage 4: AUDIT GATE (Pratibimb)

**Who:** Claude Code with Pratibimb loaded (`/pratibimb` or "load pratibimb")
**Skills used:**

| Skill | Audit role |
|-------|-----------|
| `pratibimb` command | Activates audit mode, loads Wiom DS reference |
| `wiom-visual-craft` (56-point audit) | 3-layer scoring: Pixel (16) + UX (18) + Polish (22) |
| `wiom-ux-copy` (copy checklist) | CTA labels, error messages, state-dependent copy, tonal patterns |
| `wiom-interaction-patterns` | Pattern compliance: correct bottom sheet type, CTA positioning, state handling |

### Audit Protocol

**Step 1: Code scan** — grep all `.kt` files for violations:
```bash
# Find hardcoded dp values (should be Dimens.*)
grep -rn "\.dp" screens/ components/ | grep -v "import\|Dimens\|//"

# Find hardcoded colors (should be InvColors.*)
grep -rn "Color(" screens/ components/ | grep -v "import\|InvColors\|//"

# Find hardcoded text (should be strings.*)
grep -rn '"[अ-ह]' screens/ | grep -v "import\|//"

# Check for @Preview
grep -rn "@Preview" screens/
```

**Step 2: Pattern compliance** — for each screen:
- [ ] Correct surface type used? (bottom sheet vs dialog vs full screen)
- [ ] CTA hierarchy correct? (one primary per screen, secondary is tonal not outlined)
- [ ] Navigation: back arrow vs close (X) correct per context?
- [ ] All states handled? (loading, empty, error, success)
- [ ] Progress indicator present for multi-step flows?

**Step 3: Visual compliance** — against DS:
- [ ] All spacing on 4px grid?
- [ ] Internal spacing ≤ external spacing?
- [ ] Radius hierarchy: 16 CTA, 12 card, 24 dialog, 8 nested?
- [ ] Typography from DS scale only (10,12,14,16,20,24,32,40,48)?
- [ ] Font family specified (not system default)?
- [ ] Touch targets ≥ 48dp?

**Step 4: Copy compliance** — against ux-copy:
- [ ] CTAs are outcome-first?
- [ ] Status labels in correct tense (pending vs completed)?
- [ ] Error messages: what happened + what to do?
- [ ] Hindi text uses Devanagari, not Romanized?
- [ ] English tech terms kept in English (WiFi, OTP, Net Box)?

### Audit Scoring

| Score | Verdict | Action |
|-------|---------|--------|
| 8-10 | **Ship it** | Deploy to device |
| 6-7.9 | **Fix first** | Fix Critical items, re-audit, then deploy |
| <6 | **Needs redesign** | Go back to Stage 2, re-evaluate pattern decisions |

### Report Format
```
PRATIBIMB DESIGN AUDIT — [Feature] — [Date]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SCORE: [X/10]
VERDICT: [Ship it / Fix first / Needs redesign]

CRITICAL: [list with screen name + issue + fix]
IMPORTANT: [list]
POLISH: [list]
WHAT'S WORKING: [list — always include positives]
```

---

## Stage 5: FIX (if audit fails)

**Who:** Claude Code with build skills
**Process:**
1. Read the audit report
2. Fix all CRITICAL items first
3. Fix IMPORTANT items
4. Rebuild: `./gradlew assembleDebug`
5. **Re-run audit** (go back to Stage 4)
6. Repeat until score ≥ 8

**Anti-pattern:** Do NOT deploy after fixing without re-auditing. The fix may have introduced new issues.

---

## Stage 6: DEPLOY

**Who:** Claude Code or developer
**Only after:** Audit score ≥ 8
**Command:**
```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

---

## Skill Loading Cheat Sheet

| When you're doing... | Load these skills |
|---------------------|-------------------|
| Reading a PRD, deciding screens | `wiom-interaction-patterns` + `wiom-ux-copy` |
| Building Kotlin/Compose code | `wiom-frontend-dev` + `wiom-visual-craft` + `wiom-ux-copy` |
| Extracting from Figma + building | `maverick-developer` (includes engine integration) |
| Running pre-deploy audit | `pratibimb` (activates all design skills) |
| Reviewing someone else's build | `wiom-visual-craft` (56-point audit section) |

---

## What the First Audit Caught (Device Inventory build, 31 Mar 2026)

Score: **6.5/10 — Fix first**

| # | Category | Issue | Root cause |
|---|----------|-------|-----------|
| 1-6 | Hardcoded values | 14 raw `.dp` values in composables | Build skipped Dimens centralization |
| 7 | DS conflict | Pratibimb command says CTA r:12, engine says r:16 | Pratibimb command file outdated |
| 8 | Missing previews | Zero @Preview on any screen | Build skipped wiom-frontend-dev rule 1 |
| 9 | Missing fonts | No Noto Sans bundled | Build used system font default |
| 10 | Off-scale type | 13.sp used (not in DS 22 styles) | Engine pattern used without DS validation |
| 11-12 | Missing states | No loading, no error states | Build only handled happy path |
| 13 | Fragile code | Object identity comparison for language | Code shortcut, not pattern-compliant |

**If the audit had run before deploy, items 1-6 and 8 would have been caught and fixed in ~15 minutes.**

---

## Updating the Engine

When a real build reveals a new pattern, correction, or rule:

1. **Add to the relevant skill file** (interaction-patterns, visual-craft, or ux-copy)
2. **Push to the engine repo** (`Solution-Design-engine-V1`)
3. **Copy to local skills** (`~/.claude/skills/`)
4. **Do NOT modify existing trained skills** (ux-designer, ui-designer, maverick core) — preservation rule

When the audit catches a recurring issue:

1. **Add it to the audit checklist** in `wiom-visual-craft` (56-point audit section)
2. **Add a grep pattern** to the code scan protocol above
3. **Note it as a feedback memory** if it's a behavioral correction
