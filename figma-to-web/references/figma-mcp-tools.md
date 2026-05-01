# Figma MCP Tool Orchestration for Web

How to use each Figma MCP tool effectively for web implementation. Includes the prompting patterns that produce useful output for React or plain HTML/CSS, rather than accidentally inheriting the MCP's default React+Tailwind output as final code.

## Tool Reference

### get_design_context

**What it returns:** Structured design data for a selected layer or frame — layout, styling, text content, component info.

**Critical:** Defaults to React + Tailwind output. Treat that as an intermediate representation unless the project actually uses Tailwind.

**Correct invocation pattern:**
- Include in your prompt: `generate React component without Tailwind` or `generate plain HTML and CSS`
- If Code Connect is set up: add `clientFrameworks: ["React"]`
- If targeting existing components: `using components from [path]`

**When to use:**
- Primary tool for every implementation pass
- After pulling a screenshot
- On individual components within large frames, not the whole frame at once

**Tips:**
- For large frames, use `get_metadata` first, then `get_design_context` on sub-frames
- Combine with `get_code_connect_map` before inventing a new component
- The better the Figma file, the better the output — named layers and auto layout matter a lot

### get_screenshot

**What it returns:** PNG screenshot of the selected frame/layer.

**When to use:**
- Always before implementation
- For visual comparison after browser rendering
- To capture component states or individual sections

**Tips:**
- Save to disk with meaningful names: `hero-figma.png`, `pricing-card-figma.png`
- Pull screen-level and component-level screenshots for different comparison granularity
- Screenshots preserve optical alignment and visual weight that structured output misses

### get_variable_defs

**What it returns:** Design tokens used in the selection.

**When to use:**
- Phase 0: Pull from representative screens or the whole file to build `tokens.css`
- Phase 1: Check token coverage
- When syncing after Figma updates

**Tips:**
- Pull at the file level during setup when possible
- Variable names often map cleanly to CSS custom property names
- Watch for variable modes — these often map to light/dark themes

### get_code_connect_map

**What it returns:** Mapping of Figma component instances to codebase files/components.

**Parameters:**
- `clientFrameworks: ["React"]` — for React projects
- `clientLanguages: ["typescript", "javascript"]` — optional filtering

**When to use:**
- Phase 1: Audit what's already connected
- Phase 2: Before creating any new component
- After adding new mappings

**What the response contains:**
- `componentName`
- `source`
- `snippet`
- `snippetImports`
- `label`

For plain HTML/CSS projects, Code Connect may still point to a React implementation if that's the design system source of truth. Use it as reference, not law.

### add_code_connect_map

**What it does:** Creates a mapping between a Figma component node ID and a web source file.

**When to use:**
- Phase 0: After building base components
- After creating any reusable React component
- After stabilizing a reusable HTML/CSS partial/pattern in a project that maps components that way

**Best practice:** Map components as you build them, not all at the end.

### get_metadata

**What it returns:** Sparse XML with layer IDs, names, types, positions, sizes.

**When to use:**
- Phase 0: Scan multiple screens for structure patterns
- Phase 1: Assess frame complexity before full context pulls
- For large frames: outline first, context second

**Tips:**
- Count layers to assess complexity
- Look for repeated structures that suggest reusable components
- Look for poorly named layers — they'll hurt output quality

### search_design_system

**What it returns:** Components, variables, and styles from connected design libraries matching a text query.

**When to use:**
- Phase 0: Inventory available assets
- Phase 2: Before creating something new, search whether it exists already

**Tips:**
- Search by role: `button`, `card`, `alert`, `search input`, `modal`
- Try multiple phrasings if the first search is sparse

### create_design_system_rules

**What it does:** Generates a rules file to help the agent understand the design system context.

**When to use:**
- Phase 0: Run once during setup
- Re-run if the design system changes materially

**Tips:**
- Output is framework-agnostic — add web-specific notes to your Design Brief

### use_figma

**What it does:** General-purpose write tool for Figma objects.

**When to use:**
- Creating annotations in Figma about implementation decisions
- Inspecting edge-case layer properties not exposed cleanly elsewhere
- Updating component properties if the design system needs a tweak

**When NOT to use:**
- Don't use it as a replacement for `get_design_context`
- Don't create or alter designs without user approval

## Orchestration Patterns

### Pattern 1: Single Component Implementation

```
1. get_screenshot (component)
2. get_code_connect_map with clientFrameworks: ["React"]
   → If mapped: use existing component
   → If not: continue
3. get_design_context with `generate React component without Tailwind`
   OR `generate plain HTML and CSS`
4. Implement using tokens.css
5. Render in browser
6. Compare browser screenshot to Figma screenshot
7. add_code_connect_map for the new component
```

### Pattern 2: Full Screen Implementation

```
1. get_screenshot (full screen)
2. get_metadata
   → If >50 layers: decompose into sections
3. get_code_connect_map with clientFrameworks: ["React"]
4. For each unmapped sub-component:
   a. get_design_context on that sub-component
   b. Implement
   c. add_code_connect_map
5. Assemble screen from components
6. Render and compare at the target width
7. Refinement pass
```

### Pattern 3: Design Token Sync

```
1. get_variable_defs on the full file or key screens
2. Compare against existing tokens.css
3. Update token values and names if needed
4. Run the app/build to catch broken references
5. Update components that reference changed tokens
```

### Pattern 4: Design System Bootstrap

```
1. create_design_system_rules
2. search_design_system for: button, input, card, nav, modal, badge
3. get_variable_defs on the file
4. get_metadata on 3–5 representative screens
5. get_screenshot on those same screens
6. Synthesize Design Brief
7. Build tokens.css
8. Build base components
9. add_code_connect_map for each base component
```

## Prompting Tips

### Good prompts for get_design_context

- `Implement this Figma frame as a React component using existing CSS tokens. Do not use Tailwind.`
- `Generate plain HTML and CSS for this component. Use semantic HTML and CSS custom properties.`
- `Implement this screen in React using components from src/components/ and styles from tokens.css.`

### Bad prompts

- `Generate code for this design` — too vague, defaults badly
- `Make this` — no framework or constraints
- `Convert to web` — ambiguous and underspecified

### Large-frame strategy

When a frame has 100+ layers:
1. Pull metadata
2. Identify sections: header, hero, content blocks, footer, navigation
3. Pull `get_design_context` on each section separately
4. Implement bottom-up
5. Compose at the end
