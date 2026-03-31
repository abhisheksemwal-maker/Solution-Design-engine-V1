# Wiom Solution Design Engine V1

Design system skills + audit pipeline for consistent, high-quality Wiom product output. Used by tech and product teams to make design decisions without designer sign-off on every edge case.

## Pipeline

```
PRD → Design (engine skills) → Build (frontend + maverick) → Audit (pratibimb) → Deploy
```

See [`ENGINE_WORKFLOW.md`](ENGINE_WORKFLOW.md) for the full pipeline documentation.

## Repository Structure

```
Solution-Design-engine-V1/
├── ENGINE_WORKFLOW.md              ← Pipeline: which skill at which stage, audit gate
├── README.md
├── commands/
│   └── pratibimb.md                ← Design mode activation + audit protocol
├── skills/
│   ├── ux-designer/                ← Universal UX psychology, flows, strategy
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── psychology-deep-dive.md
│   │       └── patterns-and-flows.md
│   ├── ui-designer/                ← Universal visual craft, tokens, components
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── design-tokens.md
│   │       ├── component-library.md
│   │       └── polish-and-craft.md
│   ├── wiom-interaction-patterns/  ← ENGINE: 35 Wiom interaction patterns
│   │   └── SKILL.md
│   ├── wiom-visual-craft/          ← ENGINE: DS tokens, motion, 56-point audit
│   │   └── SKILL.md
│   ├── wiom-ux-copy/               ← ENGINE: Hindi copy, 10 tonal patterns
│   │   └── SKILL.md
│   ├── maverick-developer/         ← Figma-to-code + engine integration
│   │   └── SKILL.md
│   ├── wiom-frontend-dev/          ← Kotlin/Compose standards
│   │   └── SKILL.md
│   └── wiom-design-system.md       ← Full Wiom DS reference (tokens, variable IDs)
├── project_pratibimb_framework_v2.md
└── feedback_pratibimb_v2_preservation.md
```

## Skills — What Each One Does

### Engine Skills (Wiom-specific, new)
| Skill | Sections | What it decides |
|-------|----------|----------------|
| [`wiom-interaction-patterns`](skills/wiom-interaction-patterns/SKILL.md) | 35 | Patterns (bottom sheet types, CTAs, dialogs, forms, camera, progress), navigation, state machines |
| [`wiom-visual-craft`](skills/wiom-visual-craft/SKILL.md) | 12+ | Token application (WHEN to use each), radius, typography, motion, 56-point quality audit |
| [`wiom-ux-copy`](skills/wiom-ux-copy/SKILL.md) | 17+ | CTA labels, error messages, status labels, 10 tonal patterns, bilingual AppStrings |

### Foundation Skills (universal, trained through real work)
| Skill | What it provides |
|-------|-----------------|
| [`ux-designer`](skills/ux-designer/SKILL.md) | UX psychology (cognitive load, Hick's Law, decision architecture), flow strategy, verification checklists |
| [`ui-designer`](skills/ui-designer/SKILL.md) | Visual craft (8pt grid, type scales, 60-30-10 color), component specs, polish techniques |

### Build Skills
| Skill | Role |
|-------|------|
| [`wiom-frontend-dev`](skills/wiom-frontend-dev/SKILL.md) | Kotlin/Compose standards: @Preview, Dimens.kt, sealed state, bilingual, project structure |
| [`maverick-developer`](skills/maverick-developer/SKILL.md) | Figma extraction + spec-first build + 35 corrections + engine integration protocol |

### Design System + Commands
| File | Role |
|------|------|
| [`wiom-design-system.md`](skills/wiom-design-system.md) | Full Wiom DS: color tokens, typography (22 styles), spacing, variable IDs, binding patterns |
| [`pratibimb.md`](commands/pratibimb.md) | Activation command: loads design skills, runs audit protocol, DS token binding |

## When to Load Which Skill

| Task | Load |
|------|------|
| Reading PRD, deciding screens | `wiom-interaction-patterns` + `wiom-ux-copy` |
| Building Kotlin/Compose | `wiom-frontend-dev` + `wiom-visual-craft` + `wiom-ux-copy` |
| Extracting from Figma + building | `maverick-developer` (includes engine refs) |
| Pre-deploy audit | `pratibimb` (loads all design skills) |
| Reviewing someone's build | `wiom-visual-craft` (56-point audit) |

## Install

```bash
# Clone
git clone https://github.com/abhisheksemwal-maker/Solution-Design-engine-V1.git

# Copy skills to Claude Code
cp -r skills/* ~/.claude/skills/
cp -r commands/* ~/.claude/commands/

# Restart Claude Code — skills appear automatically
```

## Personas (embedded in all engine skills)

- **Annu Bhaiyya** (CSP/Partner) — low-medium tech, earns through Wiom, trusts numbers
- **Technician Rohit** (Expert) — medium tech, sequential flows, task-completion driven
- **Verma Parivar** (Customer) — low tech, Hindi-first, recharge mental model

## Grounded In

- Wiom Design System (`T0klEs1aPBk7BOVZonc8JC`)
- 280+ real PA flow screens extracted via Figma Bridge
- 35 mistake-correction entries from Maverick builds
- 10 tonal patterns from real Hindi copy
- First audit: 6.5/10 → fix → 7.5/10 (proved the gate works)

## Preservation Rule

Engine skills are INDEPENDENT files. They never rewrite, merge into, or dilute existing trained skills (ux-designer, ui-designer, maverick core). See `feedback_pratibimb_v2_preservation.md`.
