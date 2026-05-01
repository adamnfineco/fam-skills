# figma-to-swift

Implement iOS SwiftUI interfaces from Figma designs with 1:1 visual fidelity. Not "close enough" — pixel-matched, token-accurate, iOS-native.

## When to use

**USE FOR:**
- Building SwiftUI views from Figma frames
- Extracting design systems from Figma files
- Auditing Figma files for code-generation readiness
- Translating Figma layout and tokens to SwiftUI

**DO NOT USE FOR:**
- Web/React/Tailwind code generation
- Non-visual backend work

## Required tools

- **Figma MCP** — design context, screenshots, variables, metadata, Code Connect, design system search
- **XcodeBuildMCP** — build, run on simulator, take screenshots for comparison
- **File tools** — read/write SwiftUI source files
- **Task tool (sub-agents)** — critical for managing context window in image-heavy workflows

## How it works

The skill runs a six-phase workflow. Every task follows this sequence — no skipping phases, no jumping to implementation:

| Phase | Name | What happens |
|-------|------|-------------|
| 0 | Project Setup | Extract design language, build `DesignTokens.swift`, set up Code Connect |
| 1 | Pre-Flight Audit | Assess frame quality before writing any code |
| 2 | Implementation | Build components bottom-up using design tokens |
| 3 | Refinement | Systematic visual and platform quality pass |
| 4 | Verification | Snapshot tests, punch list, user review |
| 5 | Maintenance | Sync on design updates, manage token drift |

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Primary skill file — start here |
| `references/figma-audit.md` | Pre-flight audit checklist and quality scoring (Phase 1) |
| `references/swiftui-mapping.md` | Figma→SwiftUI translation dictionary (Phase 2) |
| `references/figma-mcp-tools.md` | Figma MCP tool orchestration and prompting patterns (Phase 2) |
| `references/ios-native.md` | iOS conventions, SF Symbols, asset handling (Phase 2/3) |
| `references/code-connect.md` | Code Connect setup for SwiftUI (Phase 0) |
| `references/verification.md` | Visual verification, snapshot testing, punch lists (Phase 3/4) |

## Key principles

1. **Design system first, screens second** — extract the language before translating individual sentences
2. **Components bottom-up, screens top-down** — build primitives first, assemble screens from them
3. **Native iOS when the design is ambiguous** — prefer native components over custom rebuilds
4. **Tokens over literals** — never hardcode a color, spacing value, or font size
5. **Verify visually, not just logically** — build it, screenshot it, compare it
6. **Flag uncertainty, don't hide it** — a visible question is infinitely better than a silent wrong guess
7. **The Figma file quality is the ceiling** — audit first, always
