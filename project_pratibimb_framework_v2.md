---
name: Wiom Solution Design Engine V1
description: Architecture for a skills+agents design process engine — 4 layers, 6 stages, 3 contributor lanes, dual-mode skills deployable across Product/Design/Tech teams. New skills are independent of existing trained Pratibimb skills.
type: project
---

# Wiom Solution Design Engine V1

**Workstream name:** Wiom Solution Design Engine V1
**Started:** 2026-03-30 (Session 36)
**Status:** Architecture finalized, naming decided, ready to build
**Goal:** A two-pronged system (skills + agents) that streamlines design thinking across all stages, deployable to Product, Design, and Tech teams for consistent output.

**Relationship to Pratibimb:** Pratibimb remains the activation trigger and the existing skill layer (ux-designer, ui-designer, maverick-developer, wiom-frontend-dev). The Engine adds orchestration + new process skills on top — independent files, never merged into existing skills.

---

## Core Thesis

**Shared skills + shared project context = consistent output across teams.**

Not "design reviews everyone's work." Instead: every team produces aligned output because they reason from the same principles, tokens, and rules. The skill carries the decision, not the person.

---

## Source Models (synthesized)

| Model | Contribution |
|-------|-------------|
| **Design Thinking** (d.school) | Empathy-first, diverge-then-converge, prototype to learn |
| **Goal-Directed Design** (Cooper/About Face) | Rigorous personas, mental models, goal→task→action decomposition |
| **TARI** (Hooked/Nir Eyal) | Trigger→Action→Reward→Investment behavioral lens |

---

## 4-Layer Architecture

```
LAYER 0: PROJECT CONTEXT (single source of truth)
  tokens + voice + brand rules + constraints
  One file per project. All skills read it.
  ─────────────────────────────────────────
LAYER 1: DESIGN LEAD (orchestrator)
  Intake → Validate → Route → Activate → Transition
  Pipeline mode only. Optional — skills work without it.
  ─────────────────────────────────────────
LAYER 2: STAGE PROTOCOLS (6 stages)
  Discover → Define → Frame → Ideate → Craft → Validate
  Each: entry gate, techniques, artifacts, exit gate
  ─────────────────────────────────────────
LAYER 3: SKILL REGISTRY (pluggable, dual-mode)
  Each skill = PRINCIPLES (universal) + CONTEXT (project)
  Deployable to: Product | Design | Tech | QA
  Same skill + same context = same output across teams
```

---

## 6-Stage Pipeline (non-linear, loops back)

```
DISCOVER → DEFINE → FRAME → IDEATE → CRAFT → VALIDATE
    ↑                                            |
    └──────────────── iterate ──────────────────┘
```

| Stage | Core Question | Techniques | From DT | From Cooper | From TARI |
|-------|--------------|------------|---------|-------------|-----------|
| **Discover** | Who and why? | First Principles, 5 Whys | Empathize | Research | Identify triggers |
| **Define** | What's the real problem? | 5 Whys → RCA tree, persona modeling | Define | Modeling | Map action barriers |
| **Frame** | What are goals and requirements? | Goal hierarchy (Cooper 3-tier) | — | Requirements | Design reward + investment |
| **Ideate** | What solutions? | First Principles on each option | Ideate | Framework | Prototype hook cycle |
| **Craft** | How does it look/feel? | DS compliance, spec-first | Prototype | Refinement | Embed triggers in UI |
| **Validate** | Does it work? | Heuristic eval, 5 Whys on failures | Test | Support | Measure loop completion |

### Baseline Artifacts (mandatory at each stage)

| Artifact | Produced in | Format |
|----------|------------|--------|
| **User Needs** | Discover/Define | Behavioral needs, not feature requests |
| **User Goals** | Define/Frame | Cooper 3-tier: Life → Experience → End goals |
| **Constraints** | Define/Frame | 4 types: Technical, Business, Regulatory, Resource |
| **Assumptions** | Define | Tagged: validated / unvalidated / risky |
| **User Journeys** | Frame/Ideate | Emotional arc + touchpoints + TARI overlay |

### Core Techniques (baked into discovery stages)

- **First Principles Thinking** — decompose to fundamental truths, no reasoning by analogy. Fires in Discover (challenge brief), Define (challenge problem), Frame (challenge requirements).
- **5 Whys** — every problem drilled 5 levels. Output = root cause chain. Fires in Discover, Define, Validate.

---

## 3 Contributor Lanes (multi-role pipeline)

```
PRODUCT INPUT              DESIGN PROCESS              TECH INPUT
─────────────             ─────────────────           ──────────
RCA, painpoints     ──→   DISCOVER
Boundary conditions ──→   DEFINE
High-level solution ──→   FRAME / IDEATE
                          CRAFT              ←──  Kotlin/Compose patterns
                          VALIDATE           ←──  DS implementation rules
```

**Framework almost never starts from zero.** Product has done thinking. Design Lead's first job = assess what exists, validate it (First Principles check on Product's RCA), pick right entry point.

### Design Lead Intake Protocol

```
1. WHAT'S BEEN PROVIDED? (checklist)
   □ Problem statement  □ RCA  □ User painpoints
   □ Boundary conditions  □ Solution direction  □ Research/data  □ Existing screens

2. QUALITY CHECK (First Principles on the input)
   - Product's RCA: root cause or symptom?
   - Boundary conditions: hard constraint or assumption-disguised-as-constraint?
   - Painpoints: observed behavior or inferred?

3. ENTRY POINT
   Has RCA + painpoints + constraints → enter at FRAME
   Has problem statement only → enter at DEFINE
   Has problem + solution direction → enter at IDEATE (but validate problem first)
   Has nothing → enter at DISCOVER
   Has screens to build → enter at CRAFT (but check upstream exists)
```

---

## Skill Taxonomy — Solution Design Engine V1

### Primary Skills (directly used by Tech and Product teams)

These three skills are the engine's usable surface. Tech and product teams invoke them directly to make design decisions without designer sign-off on every edge case.

| Skill | File | What it decides | Who uses it |
|-------|------|----------------|-------------|
| `wiom-interaction-patterns` | `~/.claude/skills/wiom-interaction-patterns/SKILL.md` | What pattern (bottom sheet/dialog/toast/screen), CTA positioning, navigation/recovery, menus, confirmations, feature introduction, form patterns, edge cases | Tech building screens, Product reviewing flows |
| `wiom-visual-craft` | `~/.claude/skills/wiom-visual-craft/SKILL.md` | Typography application (which of 22 styles WHERE), color token usage rules, elevation/shadow, strokes/borders, visual hierarchy, micro-polish, responsive width | Tech styling screens, Product reviewing visual output |
| `wiom-ux-copy` | `~/.claude/skills/wiom-ux-copy/SKILL.md` | CTA labels, error messages, toasts, empty states, confirmations, hints, status labels, bilingual string patterns, persona tone calibration | Tech writing copy, Product reviewing messaging |

**Output format:** Compose-ready. Every skill produces Kotlin/Compose specs, not Figma specs. Users expect native APKs as the first interaction with engine output.

**Persona context:** All three personas (Annu Bhaiyya CSP, Technician Rohit, Verma Parivar Customer) are embedded in each skill as decision filters. No separate persona invocation needed.

**Design decisions baked in:** Feature introduction strategy, solution translation from Product input, and persona lens are reference layers WITHIN the three skills — not standalone skills. Product has already done solutioning; tech handles edge cases autonomously using these skills.

### Existing Skills (Pratibimb layer — unchanged)

| Skill | Role | Relationship to Engine |
|-------|------|----------------------|
| `ux-designer` | Universal UX psychology, flows, strategy | Foundation. Engine skills reference its principles |
| `ui-designer` | Universal visual craft, tokens, components | Foundation. Engine skills apply its rules to Wiom context |
| `maverick-developer` | Figma extraction, spec-first build | Abhishek's personal tool. NOT in engine pipeline |
| `wiom-frontend-dev` | Kotlin/Compose patterns, bilingual | Execution layer. Engine skills produce specs, this builds code |
| `wiom-design-system` | DS token definitions, variable IDs | Reference. Engine skills tell you WHEN to use each token |

### Preservation Rule
New engine skills are INDEPENDENT files. They never rewrite, merge into, or dilute existing trained skills. See `feedback_pratibimb_v2_preservation.md`.

---

## Project Context File (consistency backbone)

Each project registers a context file containing: brand colors, typography, spacing, component specs, voice/tone, brand rules. All skills read this file. Different team members loading the same skill + same context = same output.

```yaml
project: wiom-partner-app
brand:
  primary: "#D9008D"
  ...
typography:
  families: ["Noto Sans", "Noto Sans Devanagari"]
  scale: [10, 12, 14, 16, 20, 24, 32, 40, 48]
spacing:
  base: 4px
  scale: [4, 8, 12, 16, 24, 32, 40, 48, 64, 72, 84]
voice:
  tone: conversational, not corporate
  cta-labels: outcome-first
components:
  cta-height: 48px
  card-radius: 12px
  ...
```

---

## Reference: Owl-Listener/designer-skills

**Repo:** https://github.com/Owl-Listener/designer-skills
**Author:** MC Dean
**Stats:** 396 stars, 63 skills, 27 commands, 8 plugins, MIT license
**Structure:** Each plugin has `skills/<name>/SKILL.md` + `commands/<name>.md`

### Plugins inventory

| Plugin | Skills | Commands | Key skills |
|--------|--------|----------|------------|
| design-research | 10 | 4 | user-persona, empathy-map, journey-map, interview-script, summarize-interview, usability-test-plan, card-sort-analysis, diary-study-plan, affinity-diagram, jobs-to-be-done |
| design-systems | 8 | 3 | design-token, component-spec, accessibility-audit, theming-system, naming-convention, pattern-library, documentation-template, icon-system |
| ux-strategy | 8 | 3 | competitive-analysis, design-principles, experience-map, stakeholder-alignment, design-brief, metrics-definition, north-star-vision, opportunity-framework |
| ui-design | 9 | 4 | color-system, typography-scale, layout-grid, responsive-design, spacing-system, visual-hierarchy, dark-mode-design, data-visualization, illustration-style |
| interaction-design | 7 | 3 | micro-interaction-spec, state-machine, animation-principles, error-handling-ux, feedback-patterns, gesture-patterns, loading-states |
| prototyping-testing | 8 | 4 | prototype-strategy, heuristic-evaluation, a-b-test-design, wireframe-spec, test-scenario, user-flow-diagram, accessibility-test-plan, click-test-plan |
| design-ops | 7 | 3 | design-critique, handoff-spec, design-sprint-plan, team-workflow, design-qa-checklist, design-review-process, version-control-strategy |
| designer-toolkit | 6 | 3 | design-rationale, presentation-deck, case-study, ux-writing, design-system-adoption, design-token-audit |

### Key skill content highlights (for reference when building our skills)

**user-persona:** Cooper's About Face model. Behavioral variables, 3-tier goals, day-in-life scenario. 2-4 personas, primary identified.
**journey-map:** 5-7 stages, emotional curve, moments of truth, top 3-5 opportunities ranked by impact.
**jobs-to-be-done:** Christensen/Ulwick. Functional + emotional + social. 8 job stages. "When [situation], I want to [motivation], so I can [outcome]."
**heuristic-evaluation:** Nielsen's 10. 4 severity levels. 3-5 evaluators. Per-issue: heuristic, description, location, severity, recommendation.
**state-machine:** States, Events, Transitions, Actions, Guards. Eliminates impossible states.
**micro-interaction-spec:** Trigger → Rules → Feedback → Loops/Modes. 100-500ms duration. Respect reduced-motion.
**design-token:** 3-tier: global → alias → component. Naming: `{category}-{property}-{variant}-{state}`.
**component-spec:** 8 sections: Overview, Anatomy, Variants, Props, States, Behavior, A11y, Usage.
**ux-writing:** Categories: microcopy, errors, empty states, confirmations, onboarding, CTAs. Clear > clever, concise > comprehensive.

---

## Build Plan — Status

| # | Task | Status | Notes |
|---|------|--------|-------|
| 1 | Workstream naming | DONE | "Wiom Solution Design Engine V1" |
| 2 | Skill taxonomy | DONE | 3 primary skills (interaction-patterns, visual-craft, ux-copy) + existing skills unchanged |
| 3 | Output format decision | DONE | Compose-ready APKs, not Figma. Figma/maverick = Abhishek's personal tools |
| 4 | `wiom-interaction-patterns` SKILL.md | DONE | Pattern decision trees, CTA rules, nav/recovery, menus, confirmations, forms, feature intro, edge cases |
| 5 | `wiom-visual-craft` SKILL.md | DONE | Typography application, color token rules, spacing, elevation, strokes, visual hierarchy, micro-polish |
| 6 | `wiom-ux-copy` SKILL.md | DONE | Tone calibration, CTA labels, error messages, toasts, empty states, confirmations, status labels, bilingual patterns |
| 7 | Preservation rule | DONE | `feedback_pratibimb_v2_preservation.md` — new skills never touch existing ones |
| 8 | Figma DS extraction | DONE | Extracted atoms from DS doc (slides 06-15): Actions 10 variants, Input-Field 18 variants, Dialog 3 variants, Top App Bar 6 variants |
| 9 | DS corrections applied | DONE | CTA r:16 (not 12), CTA text 16/Bold (not 14/SemiBold), secondary CTA #FFE5F6 filled (not outlined), dialog r:24, input active stroke #352D42 |
| 10 | PA flow extraction (7 sections) | DONE | Device→NetBox, Hindi, Pickup Handshake, Net Box Swap, Notifications, ISP Recharge, PAY-G Early Churn |
| 11 | Pattern library from flows | DONE | 25 sections in interaction-patterns: stacked CTA, checkbox CTA, ticket header/timeline, tabs, progress bars, education flow, action landing, hero card, radio list, urgency banner, data-heavy list, rich notifications, processing states |
| 12 | Bottom sheet taxonomy | DONE | 9 real variations → 5 types (A-E) with 3-layer decision framework |
| 13 | UX copy tonal patterns | DONE | 10 patterns from real flows + input/OTP/photo/summary copy conventions |
| 13b | Input/OTP/progress screens (31 screens) | DONE | OTP boxes, photo review, summary cards, wallet display, PayG info tags, step progress states, amount input, condition assessment |
| 13c | Maverick cross-pollination | DONE | 8 patterns from Maverick's 35 corrections → engine skills. Maverick now references engine skills as consultation layer |
| 13d | Motion design system | DONE | Easing curves, duration scale, choreography patterns (enter/exit/stagger/dialog/sheet), state transitions with Compose code |
| 13e | Form & input mastery | DONE | Scan>Autofill>Select>Type hierarchy, field-specific patterns (phone/Aadhaar/amount/OTP), inline validation timing, scan-first pattern |
| 13f | Design critique audit | DONE | 56-point 3-layer audit: Pixel (16) + UX (18) + Polish (22). Ship-it/Fix-first/Needs-review scoring |
| 14 | Push to pratibimb-skills git repo | PENDING | Sync new skill files to GitHub |
| 15 | Test skills on a real screen build | PENDING | Build a screen using only engine skills (no Figma, no maverick) to validate |
| 16 | Pratibimb command update | PENDING | Update `pratibimb.md` to reference engine skills alongside existing design skills |
| 17 | Owl-Listener skill adoption | DEFERRED | Decide per-skill as needed during real work, not as a separate phase |

---

## Pending Decisions

- **Owl-Listener adoption:** Deferred — evaluate per-skill overlap during real work, not as a separate planning phase
- **Orchestrator/Design Lead:** Architecture exists but not yet needed — the 3 skills work standalone. Build orchestrator when pipeline mode is actually needed
- **Project context template:** Schema designed but no file created yet — build when a second project needs the engine
