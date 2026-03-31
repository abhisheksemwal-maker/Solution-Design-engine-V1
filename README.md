# Wiom Solution Design Engine V1

Design system skills for consistent, high-quality Wiom product output. Used by tech and product teams to make design decisions without designer sign-off on every edge case.

## Skills

| Skill | What it decides | Lines |
|-------|----------------|-------|
| [`wiom-interaction-patterns`](skills/wiom-interaction-patterns/SKILL.md) | What pattern, where, when, what states | ~1100 |
| [`wiom-visual-craft`](skills/wiom-visual-craft/SKILL.md) | Tokens, hierarchy, elevation, polish, motion, audit | ~700 |
| [`wiom-ux-copy`](skills/wiom-ux-copy/SKILL.md) | Tone, copy, labels, errors, bilingual patterns | ~800 |
| [`maverick-developer`](skills/maverick-developer/SKILL.md) | Figma-to-code with engine integration | ~400 |

## Output Format

Compose-ready. Every skill produces Kotlin/Jetpack Compose specs, not Figma specs. Users expect native APKs as the first interaction with engine output.

## Personas (embedded in all skills)

- **Annu Bhaiyya** (CSP/Partner) — low-medium tech, earns through Wiom, trusts numbers
- **Technician Rohit** (Expert) — medium tech, sequential flows, task-completion driven
- **Verma Parivar** (Customer) — low tech, Hindi-first, recharge mental model

## Grounded In

- Wiom Design System (CA_Design-System, `T0klEs1aPBk7BOVZonc8JC`)
- 280+ real PA flow screens extracted via Figma Bridge
- 35 mistake-correction entries from Maverick production builds
- 10 tonal patterns from real Hindi copy

## Install

### For Claude Code CLI / Desktop / Web

1. Clone this repo anywhere on your machine
2. Copy the skill folders into your Claude Code skills directory:

**Mac/Linux:**
```bash
cp -r skills/wiom-interaction-patterns ~/.claude/skills/
cp -r skills/wiom-visual-craft ~/.claude/skills/
cp -r skills/wiom-ux-copy ~/.claude/skills/
cp -r skills/maverick-developer ~/.claude/skills/
```

**Windows (Git Bash):**
```bash
cp -r skills/wiom-interaction-patterns "$HOME/.claude/skills/"
cp -r skills/wiom-visual-craft "$HOME/.claude/skills/"
cp -r skills/wiom-ux-copy "$HOME/.claude/skills/"
cp -r skills/maverick-developer "$HOME/.claude/skills/"
```

3. Restart Claude Code. Skills will appear in the skill list automatically.

### How to Use

**Tech building screens:**
- Ask: "I need to build the ISP recharge list screen" — Claude loads `wiom-interaction-patterns` for pattern decisions, `wiom-visual-craft` for styling, `wiom-ux-copy` for all text
- Ask: "Should this be a bottom sheet or dialog?" — gets the 5-type taxonomy with decision framework
- Ask: "What's the CTA label for payment confirmation?" — gets outcome-first Hindi copy

**Product reviewing:**
- Ask: "Run the design audit on this screen" — gets the 56-point pixel/UX/polish audit
- Ask: "Is this the right pattern for team assignment?" — gets the radio selection list pattern with specs

### Updating

When skills are updated, pull from this repo and re-copy to `~/.claude/skills/`. Skills in `~/.claude/skills/` are what Claude Code reads at runtime.

## Relationship to Pratibimb

Pratibimb = activation trigger + existing design skills (ux-designer, ui-designer).
Engine = orchestration + new skills on top. Independent files, never merged into existing skills.

## Session Log

- **Session 37 (31 Mar 2026):** Full build. 3 skills created, DS-grounded, pattern library from 280+ frames, motion system, 56-point audit, Maverick integration.
