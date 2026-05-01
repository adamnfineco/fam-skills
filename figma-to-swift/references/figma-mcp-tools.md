# Figma MCP Tool Orchestration

How to use each Figma MCP tool effectively for SwiftUI implementation. Includes the exact prompting patterns that produce SwiftUI output (not the default React+Tailwind).

## Tool Reference

### get_design_context

**What it returns:** Structured design data for a selected layer or frame — layout, styling, text content, component info.

**Critical:** Defaults to React + Tailwind output. You MUST prompt for SwiftUI explicitly.

**Correct invocation pattern:**
- Include in your prompt: "generate SwiftUI for iOS 17+" or "implement this in SwiftUI"
- If Code Connect is set up: add `clientFrameworks: ["SwiftUI"]` parameter
- If targeting specific existing components: "using components from [path]"

**When to use:**
- Primary tool for every implementation pass
- After pulling screenshot (so you have both visual and structural reference)
- On individual components within a large frame (not the whole frame at once)

**Tips:**
- For large frames, use `get_metadata` first to understand structure, then `get_design_context` on sub-frames
- Combine with `get_code_connect_map` to know which components are already mapped
- The output quality depends heavily on Figma file quality — auto layout and named layers produce much better results

### get_screenshot

**What it returns:** PNG screenshot of the selected frame/layer.

**When to use:**
- ALWAYS — before every implementation pass
- For visual comparison after building
- To capture specific component states
- To document the "source of truth" visual reference

**Tips:**
- **Always save to disk** with meaningful names: `home-screen-figma.png`, `button-primary-states-figma.png` — avoid keeping images in context longer than needed
- Pull screenshots at component level AND screen level for different granularity of comparison
- Screenshots preserve visual fidelity information that structured data misses (optical alignment, visual weight, whitespace rhythm)
- In sub-agent workflows, each agent pulls its own screenshots — they never flow to the parent context

### get_variable_defs

**What it returns:** All design tokens (colors, spacing, typography, etc.) used in the selection.

**When to use:**
- Phase 0 (setup): Pull from representative screens to build `DesignTokens.swift`
- Phase 1 (audit): Check token coverage per frame
- When updating tokens after Figma design changes

**Tips:**
- Pull at the FILE level (not frame level) during setup to get the complete token set
- Variable names in Figma often map well to Swift naming: `colors/primary` → `DesignTokens.Colors.primary`
- Look for variable modes — these often correspond to light/dark themes
- If variables have multiple modes, generate both light and dark token values

### get_code_connect_map

**What it returns:** Mapping of Figma component instances to codebase files/components.

**Parameters:**
- `clientFrameworks: ["SwiftUI"]` — ALWAYS pass this for iOS projects
- `clientLanguages: ["swift"]` — optional, for additional filtering

**When to use:**
- Phase 1 (audit): Check what's already connected
- Phase 2 (implementation): Before creating any new component, check if it's mapped
- After adding new Code Connect mappings

**What the response contains:**
- `componentName` — the Swift struct/view name
- `source` — file path in your codebase
- `snippet` — usage example code
- `snippetImports` — import statements needed
- `label` — framework label (should be "SwiftUI")

### add_code_connect_map

**What it does:** Creates a mapping between a Figma component node ID and a SwiftUI source file.

**When to use:**
- Phase 0 (setup): After building base SwiftUI components
- After creating any new reusable component
- When refactoring components to a new location

**Best practice:** Map components as you build them, not all at once at the end. This way subsequent screens benefit from the mappings immediately.

### get_metadata

**What it returns:** Sparse XML with layer IDs, names, types, positions, sizes. Lightweight.

**When to use:**
- Phase 0 (setup): Scan multiple screens to understand overall structure patterns
- Phase 1 (audit): Assess frame complexity before pulling full design context
- For very large frames: get the outline first, then `get_design_context` on sub-sections
- When you need layer names and structure without the full styling data

**Tips:**
- Count total layers to assess complexity
- Look for repeating structures (same pattern of nested layers = component candidate)
- Check for layers named "Group" or "Frame" — these signal poor naming

### search_design_system

**What it returns:** Components, variables, and styles from connected design libraries matching a search query.

**When to use:**
- Phase 0 (setup): Inventory all available design system assets
- Phase 2 (implementation): Before creating a new component, search if one exists in the library
- When you encounter an unfamiliar component in a frame

**Tips:**
- Search is text-based — try multiple terms (e.g., "button", "btn", "cta")
- Results include components from ALL connected libraries
- Use this to avoid duplicating existing library components

### create_design_system_rules

**What it does:** Generates a rules file that helps the agent understand the design system context.

**When to use:**
- Phase 0 (setup): Run once to generate baseline rules
- Save the output to your project's rules/instructions directory
- Re-run if the design system significantly changes

**Tips:**
- The output is framework-agnostic — review it and add SwiftUI-specific notes
- Pair with the Design Brief you create manually for maximum context

### use_figma

**What it does:** General-purpose write tool. Can create, edit, delete, or inspect any Figma object.

**When to use (for our workflow):**
- Creating annotation frames in Figma to document implementation decisions
- Updating component properties after implementation reveals needed changes
- Inspecting specific layer properties not available through other tools

**When NOT to use:**
- Don't use it as a replacement for `get_design_context` — it's for writing, not reading design context
- Don't create new designs without user approval

## Orchestration Patterns

### Pattern 1: Single Component Implementation

```
1. get_screenshot (component in Figma)
2. get_code_connect_map with clientFrameworks: ["SwiftUI"]
   → If mapped: use the existing SwiftUI component
   → If not mapped: continue to step 3
3. get_design_context with "generate SwiftUI for iOS 17+"
4. Implement in SwiftUI using DesignTokens
5. Build + preview on simulator
6. Compare simulator screenshot to Figma screenshot
7. add_code_connect_map to link new component
```

### Pattern 2: Full Screen Implementation

```
1. get_screenshot (full screen)
2. get_metadata (to understand structure and assess complexity)
   → If >50 layers: decompose into sub-frames
   → If manageable: continue
3. get_code_connect_map with clientFrameworks: ["SwiftUI"]
   → Identify which sub-components are already mapped
4. For each unmapped sub-component (bottom-up):
   a. get_design_context on just that component
   b. Implement
   c. add_code_connect_map
5. Assemble screen from components
6. Build + preview on simulator
7. Full-screen comparison
8. Refinement pass
```

### Pattern 3: Design Token Sync

```
1. get_variable_defs on the full file or key screens
2. Compare against existing DesignTokens.swift
3. Update tokens (add new, modify changed, flag removed)
4. Run build to check for compilation errors from removed/renamed tokens
5. Update any component code that references changed tokens
```

### Pattern 4: Design System Bootstrap

```
1. create_design_system_rules → save rules file
2. search_design_system for common terms: "button", "input", "card", "nav", "tab", "icon"
3. get_variable_defs on the full file
4. get_metadata on 3-5 representative screens
5. get_screenshot on those same screens
6. Synthesize Design Brief
7. Build DesignTokens.swift
8. Build base components
9. add_code_connect_map for each base component
```

## Prompting Tips

### For get_design_context

**Good prompts:**
- "Implement this Figma frame in SwiftUI for iOS 17+. Use my existing design tokens from DesignTokens.swift."
- "Generate SwiftUI code for this component. Prefer SF Symbols for icons. Use .continuous corner radius style."
- "Implement this screen in SwiftUI using components from Sources/UI/Components/"

**Bad prompts:**
- "Generate code for this design" (will default to React)
- "Make this" (too vague)
- "Convert to Swift" (ambiguous — could mean UIKit)

### For search_design_system

**Good searches:**
- "primary button" "text input" "avatar" "navigation bar" "tab bar"
- Search by role, not appearance: "call to action" not "blue rounded rectangle"

### For get_metadata on large frames

When a frame has 100+ layers, use metadata to plan your attack:
1. Pull metadata
2. Identify logical sections (header, content, footer, nav)
3. Pull `get_design_context` on each section separately
4. Implement section by section
5. Compose at the end
