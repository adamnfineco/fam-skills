# figma-to-web

Implement web interfaces from Figma designs with 1:1 visual fidelity. Not "close enough" — token-accurate, semantically correct, responsive-ready.

## When to use

**USE FOR:**
- Building React components from Figma frames
- Building plain HTML/CSS from Figma frames
- Extracting design systems from Figma files
- Auditing Figma files for code-generation readiness
- Translating Figma layout and tokens to CSS

**DO NOT USE FOR:**
- iOS/SwiftUI code generation
- Non-visual backend work

## Required tools

- **Figma MCP** — design context, screenshots, variables, metadata, Code Connect, design system search
- **Browser / Playwright** — render code and capture screenshots for comparison
- **File tools** — read/write HTML, CSS, JS/TS source files
- **Task tool (sub-agents)** — useful for image-heavy component implementation passes

## How it works

The skill runs a six-phase workflow:

| Phase | Name | What happens |
|-------|------|-------------|
| 0 | Project Setup | Extract design language, build `tokens.css`, set up Code Connect |
| 1 | Pre-Flight Audit | Assess frame quality before writing code |
| 2 | Implementation | Build components bottom-up using semantic HTML and design tokens |
| 3 | Refinement | Systematic visual, accessibility, and responsive quality pass |
| 4 | Verification | Playwright snapshots, punch list, user review |
| 5 | Maintenance | Sync token drift and catch regressions |

## Framework paths

This skill supports two output paths:

- **React path** — primary path for component-based web apps
- **HTML/CSS path** — fallback for framework-agnostic or vanilla projects

The workflow is shared. Only the code output format changes.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Primary skill file — start here |
| `references/figma-audit.md` | Pre-flight audit checklist and quality scoring |
| `references/css-mapping.md` | Figma→CSS/HTML translation dictionary |
| `references/figma-mcp-tools.md` | Figma MCP orchestration and prompting patterns |
| `references/web-native.md` | Semantic HTML, responsive behavior, browser conventions |
| `references/code-connect.md` | Code Connect setup for React/web |
| `references/verification.md` | Visual verification, Playwright snapshots, punch lists |

## Key principles

1. **Design system first, screens second**
2. **Tokens over literals**
3. **Semantic HTML over div soup**
4. **Responsive is not optional**
5. **Verify visually, not just logically**
6. **Flag uncertainty instead of hiding it**
7. **The Figma file quality is the ceiling**
