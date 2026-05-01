---
name: figma-to-swift
description: Implement iOS SwiftUI interfaces from Figma designs with 1:1 visual fidelity. USE FOR building SwiftUI views from Figma frames, extracting design systems from Figma files, auditing Figma files for code-generation readiness, translating Figma layout/tokens to SwiftUI. DO NOT USE FOR web/React/Tailwind code generation or non-visual backend work.
---

# Figma → Swift

Implement SwiftUI interfaces from Figma designs with 1:1 visual fidelity. Not "close enough" — pixel-matched, token-accurate, iOS-native.

This skill orchestrates the Figma MCP server, XcodeBuildMCP, and SwiftUI knowledge to turn Figma frames into production SwiftUI code through a structured, verifiable workflow.

## Required Tools

- **Figma MCP** — design context, screenshots, variables, metadata, Code Connect, design system search
- **XcodeBuildMCP** — build, run on simulator, take screenshots for comparison
- **File tools** — read/write SwiftUI source files
- **Task tool (sub-agents)** — critical for managing context window with image-heavy workflows

## Context Window & Sub-Agent Strategy

Screenshots consume significant context. A single Figma screenshot + simulator comparison can use 10-20% of available context. Without careful management, you'll exhaust the window mid-implementation.

**Core rule: The parent agent orchestrates. Sub-agents implement.**

**Parent agent responsibilities:**
- Hold the Design Brief, DesignTokens.swift, and component inventory
- Run Phase 0 (setup) and Phase 1 (audit) — these are text-light
- Dispatch Phase 2 implementation to sub-agents, one component or screen per agent
- Collect punch lists from sub-agents and present to user
- Run Phase 4 verification as a final pass

**Sub-agent responsibilities (one per component/screen):**
- Pull Figma screenshot and design context for their specific scope
- Implement the SwiftUI code
- Build on simulator, take screenshot, compare visually
- Run the refinement loop (images stay in sub-agent context, not parent)
- Return: punch list (text), file paths created/modified, any uncertainties

**Sub-agent prompt template:**
```
Implement [component/screen name] in SwiftUI from this Figma frame: [URL or node reference].

Context:
- Design tokens: [path to DesignTokens.swift]
- Existing components: [list of available components with file paths]
- Design brief notes: [relevant excerpt]

Instructions:
1. Pull get_screenshot and get_design_context (with clientFrameworks: ["SwiftUI"]) from Figma MCP
2. Implement using existing design tokens and components
3. Build on simulator and take a screenshot
4. Compare against Figma screenshot — fix any visual deltas
5. Return: punch list, files created/modified, uncertainties, and any new components that should be added to the shared design system
```

**Save screenshots to disk, not context:**
- Use `get_screenshot` with a file path parameter when available
- Use XcodeBuildMCP `screenshot` with file path output
- Reference images by path in reports — only pull into context when actively comparing
- After comparison is complete, the images can leave context

**Session boundaries:**
- One sub-agent per component or per screen (not the whole app)
- If a screen has 5+ distinct components, implement components first as separate sub-agent tasks, then assemble the screen in a final sub-agent
- Keep the parent conversation lean — it should read like a project manager's log, not a code dump

## The Six Phases (0–5)

Every Figma-to-Swift task follows this sequence. Never skip phases. Never jump to implementation without completing pre-flight.

**If Figma MCP is not available** (no paid plan, connection issues): Use screenshots as primary reference, manually inspect spacing/colors/typography, and skip Code Connect steps. The workflow still applies — just with manual data gathering instead of MCP tool calls.

### Phase 0: Project Setup (once per project)

Run this once when starting a new project or onboarding a new Figma file.

1. **Verify Figma MCP connection** — call `whoami` to confirm auth
2. **Extract design language** — this is the most important step
   - Call `create_design_system_rules` to generate a rules file from the Figma design system
   - Call `search_design_system` to inventory all available components, variables, styles
   - Call `get_variable_defs` on representative screens to extract the full token set
   - Use `get_metadata` on 3-5 key screens to understand the overall structure patterns
   - Call `get_screenshot` on those same screens for visual reference
3. **Produce the Design Brief** — synthesize what you found into:
   - Spacing system (base unit, scale)
   - Type scale (sizes, weights, line heights, hierarchy roles)
   - Color palette with semantic roles (not just hex values — primary, secondary, surface, destructive, etc.)
   - Corner radius system
   - Shadow/elevation system
   - Recurring component patterns
   - Visual personality notes (dense vs spacious, rounded vs sharp, etc.)
4. **Build the SwiftUI design system foundation**
   - `DesignTokens.swift` — colors, spacing, typography, radii as Swift constants, sourced from Figma variables
   - Base components that map to the most-used Figma components
   - `#Preview` for each base component showing all variants
5. **Set up Code Connect mappings** — call `add_code_connect_map` to link Figma components to SwiftUI source files
6. **Set up snapshot testing infrastructure** — Point-Free's SnapshotTesting or Prefire for visual regression
7. **Present the Design Brief to the user for review** — they may correct role assignments, clarify intent, or flag priorities

### Phase 1: Pre-Flight Audit (before every screen/component)

Before writing any SwiftUI code for a frame, audit it. Read `references/figma-audit.md` for the full checklist.

**Quick audit sequence:**

1. Call `get_metadata` on the target frame — check layer names, structure depth, frame complexity
2. Call `get_variable_defs` on the frame — check token coverage
3. Call `get_screenshot` — save as visual reference
4. Call `get_code_connect_map` with `clientFrameworks: ["SwiftUI"]` — check what's already connected

**Flag and report:**

- **Unnamed layers** ("Group 5", "Frame 12", "Rectangle 3") — these make structure ambiguous
- **Missing auto layout** — absolute positioning is hard to translate reliably
- **Hardcoded values** — colors/spacing not using variables means no token mapping
- **Inconsistent spacing** — spacing that doesn't match the design system's grid
- **Uncomponentized repetition** — patterns repeated 3+ times that aren't Figma components
- **Missing states** — only one state shown for interactive elements (where are hover, pressed, disabled, loading, error, empty?)
- **Excessive complexity** — frames with 50+ layers should be decomposed into sub-frames

**Decide:**
- Can I proceed? Or does the Figma file need cleanup first?
- If proceeding with issues, log them in the deviation log
- If the frame is too large, propose a decomposition plan to the user before starting

### Phase 2: Implementation (the build loop)

Work **component by component, bottom-up**. Primitives first, then compositions, then screen assembly.

Read `references/swiftui-mapping.md` for the full Figma→SwiftUI translation dictionary.
Read `references/figma-mcp-tools.md` for tool orchestration patterns.

**For each component/screen:**

1. **Pull context** — `get_design_context` with `clientFrameworks: ["SwiftUI"]` parameter AND prompt "generate SwiftUI for iOS 17+" + `get_screenshot` + `get_code_connect_map` with `clientFrameworks: ["SwiftUI"]`
2. **Check existing components** — before creating anything new, check:
   - Does a Code Connect mapping exist? Use the mapped component.
   - Does a similar component exist in the SwiftUI design system? Extend or configure it.
   - Does a native iOS component handle this? (NavigationStack, TabView, List, Sheet, etc.) Prefer native.
   - Only create a new custom component if none of the above apply.
3. **Implement in SwiftUI** using:
   - Design tokens from `DesignTokens.swift` (never hardcode colors, spacing, or font sizes)
   - The Figma→SwiftUI mapping rules for layout translation
   - SF Symbols for icons when a match exists (check `references/ios-native.md`)
   - Asset Catalog for custom colors and images
4. **Generate `#Preview`** for every component with:
   - Default state
   - All interactive states (pressed, disabled, loading, error, empty)
   - Light and dark mode
   - At least 2 device sizes (SE and Pro Max)
5. **Build on simulator** after each major component — don't batch, catch issues early
6. **Compare** — simulator screenshot vs Figma screenshot. Note any deltas.

**Uncertainty protocol:**
When the Figma design is ambiguous (exact spacing unclear, font weight uncertain, behavior not specified):
- Flag it explicitly: "UNCERTAIN: Is this 8px or 10px gap between title and subtitle?"
- Use the design system's closest value as default
- Log it for user review
- Never guess silently

**Deviation log:**
When intentionally diverging from the Figma design (for iOS conventions, accessibility, technical constraints):
- Document what changed and why
- Example: "DEVIATION: Figma shows custom tab bar. Using native TabView instead for iOS convention compliance and accessibility. Visual appearance matched with .tint() modifier."

### Phase 3: Refinement (the polish loop)

After initial implementation, do a systematic quality pass. Read `references/verification.md` for full procedures.

**Visual audit checklist:**
- [ ] Spacing — all padding, gaps, margins match design tokens
- [ ] Typography — font family, size, weight, line height, letter spacing
- [ ] Colors — exact hex/RGBA match including opacity
- [ ] Corner radius — matches design system
- [ ] Shadows/elevation — offset, blur, spread, color
- [ ] Alignment — centered is centered, leading is leading
- [ ] Icon sizing — matches design, proper optical alignment
- [ ] Image aspect ratios — fill vs fit vs fixed

**Platform adaptation checklist:**
- [ ] Safe areas — content respects notch, home indicator, status bar
- [ ] Dynamic Type — text scales, layout holds at accessibility sizes
- [ ] Dark mode — colors adapt, contrast maintained, no invisible elements
- [ ] Landscape — layout doesn't break (even if not designed for it)
- [ ] Responsive — handles iPhone SE through Pro Max gracefully
- [ ] VoiceOver — all elements have appropriate labels, logical reading order
- [ ] Minimum tap targets — 44pt minimum for interactive elements
- [ ] Contrast ratios — WCAG AA minimum (4.5:1 for text, 3:1 for large text)

### Phase 4: Verification (before declaring done)

1. **Snapshot tests** — run or capture new baselines, review diffs
2. **All `#Preview` states** render correctly
3. **Build succeeds** with zero warnings related to our views
4. **Produce the punch list** — structured output:
   ```
   ## Punch List: [Screen Name]
   
   ### Visual Deltas
   - [item]: [description of difference from Figma]
   
   ### iOS Deviations (intentional)
   - [item]: [what changed + why]
   
   ### Uncertainties (needs user input)
   - [item]: [question for designer/user]
   
   ### Not Yet Implemented
   - [item]: [interaction/animation implied but not built — separate task]
   ```
5. **Present punch list to user** — they review, approve, or request changes

### Phase 5: Maintenance (ongoing)

- When Figma design updates: re-pull tokens, diff against current `DesignTokens.swift`, update
- When SwiftUI components change: verify Code Connect mappings still hold
- Periodic snapshot test runs to catch visual regressions
- If adding new screens: always start at Phase 1 (pre-flight audit), never skip

## DesignTokens.swift Template

```swift
import SwiftUI

enum DesignTokens {
    enum Colors {
        static let primary = Color("Primary")           // Asset Catalog
        static let secondary = Color("Secondary")
        static let surface = Color("Surface")
        static let surfaceSecondary = Color("SurfaceSecondary")
        static let textPrimary = Color("TextPrimary")
        static let textSecondary = Color("TextSecondary")
        static let destructive = Color("Destructive")
        static let divider = Color("Divider")
        // Add more as extracted from Figma variables
    }
    
    enum Spacing {
        static let xxs: CGFloat = 2
        static let xs: CGFloat = 4
        static let sm: CGFloat = 8
        static let md: CGFloat = 12
        static let lg: CGFloat = 16
        static let xl: CGFloat = 24
        static let xxl: CGFloat = 32
        static let xxxl: CGFloat = 48
        // Derived from the design's spacing system
    }
    
    enum Radii {
        static let sm: CGFloat = 4
        static let md: CGFloat = 8
        static let lg: CGFloat = 12
        static let xl: CGFloat = 16
        static let full: CGFloat = 9999  // Pill/capsule
    }
    
    enum Typography {
        static let largeTitle = Font.system(size: 34, weight: .bold)
        static let title = Font.system(size: 28, weight: .bold)
        static let headline = Font.system(size: 17, weight: .semibold)
        static let body = Font.system(size: 17, weight: .regular)
        static let callout = Font.system(size: 16, weight: .regular)
        static let caption = Font.system(size: 12, weight: .regular)
        static let buttonLabel = Font.system(size: 17, weight: .semibold)
        // Replace with .custom("FontName", size:) if design uses custom fonts
    }
}
```

Populate this from `get_variable_defs` output during Phase 0. Every visual property in the codebase references this file — never hardcode raw values.

## Animations & Transitions

Figma prototypes can show transitions between screens and micro-interactions, but the MCP cannot extract animation specs (duration, easing, spring parameters). Handle this:

1. **During audit:** Note any Figma prototype connections on the frame (visible in prototype mode)
2. **During implementation:** Use iOS-native defaults (`.spring()`, `.easeInOut`) as starting points
3. **In the punch list:** List all animations as "Not Yet Implemented" with notes on what the Figma prototype suggests
4. **Refine later:** Animation tuning is a separate pass after visual fidelity is locked

Common SwiftUI defaults that feel iOS-native:
- Navigation transitions: handled by `NavigationStack` automatically
- Sheet presentation: handled by `.sheet()` automatically  
- State changes: `.animation(.spring(response: 0.3, dampingFraction: 0.8), value: state)`
- Opacity fades: `.transition(.opacity)` with `.animation(.easeInOut(duration: 0.2))`

## Key Principles

1. **Design system first, screens second** — extract the language before translating individual sentences
2. **Components bottom-up, screens top-down** — build primitives first, assemble screens from them
3. **Native iOS when the design is ambiguous** — when Figma shows something that could be native or custom, prefer native
4. **Tokens over literals** — never hardcode a color, spacing value, or font size. Always reference `DesignTokens.swift`
5. **Verify visually, not just logically** — code that compiles and "looks right in your head" is not verified. Build it, screenshot it, compare it.
6. **Flag uncertainty, don't hide it** — a visible question mark is infinitely better than a silent wrong guess
7. **The Figma file quality is the ceiling** — if the file is messy, the output will be messy. Audit first, always.

## Reference Files

Load on demand based on what phase you're in:

| File | When to load |
|------|-------------|
| `references/figma-audit.md` | Phase 1 — pre-flight audit checklist and quality scoring |
| `references/swiftui-mapping.md` | Phase 2 — Figma→SwiftUI translation dictionary |
| `references/figma-mcp-tools.md` | Phase 2 — Figma MCP tool orchestration and prompting |
| `references/ios-native.md` | Phase 2/3 — iOS conventions, SF Symbols, asset handling |
| `references/code-connect.md` | Phase 0 — Code Connect setup for SwiftUI |
| `references/verification.md` | Phase 3/4 — visual verification, snapshot testing, punch lists |
