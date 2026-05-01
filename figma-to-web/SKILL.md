---
name: figma-to-web
description: Implement web interfaces from Figma designs with 1:1 visual fidelity. USE FOR building HTML/CSS or React components from Figma frames, extracting design systems from Figma files, auditing Figma files for code-generation readiness, translating Figma layout and tokens to CSS. DO NOT USE FOR iOS/SwiftUI code generation or non-visual backend work.
---

# Figma → Web

Implement web interfaces from Figma designs with 1:1 visual fidelity. Not "close enough" — token-accurate, semantically correct, responsive-ready.

This skill orchestrates the Figma MCP server, a browser + Playwright for visual verification, and deep CSS/HTML knowledge to turn Figma frames into production web code through a structured, verifiable workflow.

Works with React (primary) and plain HTML/CSS (fallback). The core workflow is identical — only the code output format differs.

## Required Tools

- **Figma MCP** — design context, screenshots, variables, metadata, Code Connect, design system search
- **Browser / Playwright** — render components, take screenshots for comparison
- **File tools** — read/write HTML, CSS, JS/TS source files
- **Task tool (sub-agents)** — critical for managing context window with image-heavy workflows

## Framework Paths

This skill supports two output paths. Determine which to use at the start of Phase 0.

**React path** — JSX components, CSS Modules or CSS-in-JS, design tokens as JS constants or CSS custom properties. Use when the project is React-based.

**HTML/CSS path** — semantic HTML, plain CSS with custom properties, no framework dependency. Use when the project is vanilla, or when components need to be framework-agnostic.

The Figma MCP's `get_design_context` defaults to React + Tailwind output. Treat this as an **intermediate representation**, not final code. Translate it to the target path — never output Tailwind classes unless the project actually uses Tailwind.

## Context Window & Sub-Agent Strategy

Screenshots consume significant context. A single Figma screenshot + browser comparison can use 10–20% of available context. Without careful management, you'll exhaust the window mid-implementation.

**Core rule: The parent agent orchestrates. Sub-agents implement.**

**Parent agent responsibilities:**
- Hold the Design Brief, design tokens file, and component inventory
- Run Phase 0 (setup) and Phase 1 (audit) — these are text-light
- Dispatch Phase 2 implementation to sub-agents, one component or screen per agent
- Collect punch lists from sub-agents and present to user
- Run Phase 4 verification as a final pass

**Sub-agent responsibilities (one per component/screen):**
- Pull Figma screenshot and design context for their specific scope
- Implement the HTML/CSS or React code
- Render in browser, take screenshot, compare visually
- Run the refinement loop (images stay in sub-agent context, not parent)
- Return: punch list (text), file paths created/modified, any uncertainties

**Sub-agent prompt template:**
```
Implement [component/screen name] from this Figma frame: [URL or node reference].

Context:
- Framework: [React / HTML+CSS]
- Design tokens: [path to tokens file]
- Existing components: [list with file paths]
- Design brief notes: [relevant excerpt]

Instructions:
1. Pull get_screenshot and get_design_context from Figma MCP
2. Implement using existing design tokens (no hardcoded values)
3. Render in browser and take a screenshot at [width]px
4. Compare against Figma screenshot — fix any visual deltas
5. Return: punch list, files created/modified, uncertainties
```

**Save screenshots to disk, not context:**
- Use `get_screenshot` with a file path parameter
- Use Playwright with `path` option: `await page.screenshot({ path: 'comparison.png' })`
- Reference images by path in reports — only pull into context when actively comparing
- After comparison is complete, images can leave context

**Session boundaries:**
- One sub-agent per component or per screen (not the whole app)
- If a screen has 5+ distinct components, implement components first as separate tasks, then assemble
- Keep the parent conversation lean — project manager's log, not a code dump

## The Six Phases (0–5)

Every Figma-to-web task follows this sequence. Never skip phases. Never jump to implementation without completing pre-flight.

**If Figma MCP is not available:** Use screenshots as primary reference, manually inspect spacing/colors/typography, and skip Code Connect steps. The workflow still applies — just with manual data gathering.

### Phase 0: Project Setup (once per project)

Run once when starting a new project or onboarding a new Figma file.

1. **Determine framework path** — React or HTML/CSS. Ask if not clear from the project.
2. **Verify Figma MCP connection** — call `whoami` to confirm auth
3. **Extract design language** — the most important step
   - Call `create_design_system_rules` to generate a baseline rules file
   - Call `search_design_system` to inventory all available components, variables, styles
   - Call `get_variable_defs` on representative screens to extract the full token set
   - Use `get_metadata` on 3–5 key screens to understand overall structure patterns
   - Call `get_screenshot` on those same screens for visual reference
4. **Produce the Design Brief** — synthesize what you found into:
   - Spacing system (base unit, scale)
   - Type scale (sizes, weights, line heights, roles)
   - Color palette with semantic roles (primary, surface, text-primary, destructive, etc.)
   - Corner radius system
   - Shadow/elevation system
   - Recurring component patterns
   - Visual personality (dense vs spacious, rounded vs sharp, etc.)
5. **Build the design token foundation**

   **React path — `tokens.css` + `tokens.js`:**
   ```css
   /* tokens.css */
   :root {
     --color-primary: #6366f1;
     --color-surface: #ffffff;
     --color-text-primary: #111827;
     --color-text-secondary: #6b7280;
     --color-destructive: #ef4444;
     --color-divider: #e5e7eb;

     --spacing-xxs: 2px;
     --spacing-xs: 4px;
     --spacing-sm: 8px;
     --spacing-md: 12px;
     --spacing-lg: 16px;
     --spacing-xl: 24px;
     --spacing-xxl: 32px;
     --spacing-xxxl: 48px;

     --radius-sm: 4px;
     --radius-md: 8px;
     --radius-lg: 12px;
     --radius-xl: 16px;
     --radius-full: 9999px;

     --shadow-sm: 0 1px 2px rgb(0 0 0 / 0.05);
     --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
     --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
   }
   ```

   **HTML/CSS path** — same `tokens.css`, referenced via `<link>` or `@import`.

6. **Set up Code Connect mappings** — call `add_code_connect_map` to link Figma components to source files. Read `references/code-connect.md`.
7. **Present the Design Brief to the user for review** — they may correct role assignments or flag priorities.

### Phase 1: Pre-Flight Audit (before every screen/component)

Before writing any code for a frame, audit it. Read `references/figma-audit.md` for the full checklist.

**Quick audit sequence:**

1. Call `get_metadata` on the target frame — check layer names, structure depth, complexity
2. Call `get_variable_defs` on the frame — check token coverage
3. Call `get_screenshot` — save as visual reference
4. Call `get_code_connect_map` with `clientFrameworks: ["React"]` — check what's already connected

**Flag and report:**

- **Unnamed layers** ("Group 5", "Frame 12") — make structure ambiguous
- **Missing auto layout** — absolute positioning is hard to translate to responsive CSS
- **Hardcoded values** — colors/spacing not using variables means no token mapping
- **No states shown** — only default state for interactive elements
- **Excessive complexity** — frames with 50+ layers should be decomposed

### Phase 2: Implementation (the build loop)

Work **component by component, bottom-up**. Primitives first, then compositions, then screen assembly.

Read `references/css-mapping.md` for the Figma→CSS translation dictionary.
Read `references/figma-mcp-tools.md` for tool orchestration patterns.
Read `references/web-native.md` for semantic HTML and responsive conventions.

**For each component/screen:**

1. **Pull context** — `get_design_context` with "generate [React/HTML+CSS] — do not use Tailwind" + `get_screenshot` + `get_code_connect_map` with `clientFrameworks: ["React"]`
2. **Check existing components** — before creating anything new:
   - Does a Code Connect mapping exist? Use the mapped component.
   - Does a similar component exist in the project? Extend or configure it.
   - Does a native HTML element handle this? (`<button>`, `<input>`, `<select>`, `<details>`, etc.) Prefer native.
   - Only create a new custom component if none of the above apply.
3. **Implement** using:
   - Design tokens from `tokens.css` (never hardcode colors, spacing, or font sizes)
   - CSS Flexbox/Grid for layout — no `position: absolute` unless the design explicitly requires it
   - Semantic HTML elements (`<nav>`, `<main>`, `<section>`, `<article>`, `<button>`, `<a>`, etc.)
   - The Figma→CSS mapping rules for layout translation
   - SVG or `<img>` for assets — use URLs from the Figma MCP directly, never create placeholders
4. **Add all interactive states** in CSS:
   - `:hover`, `:active`, `:focus-visible`, `:disabled`
   - Loading and error states as separate CSS classes or component props
5. **Add responsive behavior:**
   - Mobile-first — base styles are for mobile, `@media (min-width: X)` for larger
   - Infer breakpoints from the design's auto layout structure (read `references/web-native.md`)
   - Use `clamp()` for fluid typography and spacing where appropriate
6. **Build and render** — serve locally, open in browser, compare at the Figma design width
7. **Compare** — browser screenshot vs Figma screenshot. Note any deltas.

**Uncertainty protocol:**
When the Figma design is ambiguous — flag it explicitly:
> "UNCERTAIN: Is this 8px or 10px gap between title and subtitle?"

Use the design system's closest token as default. Log it for user review. Never guess silently.

**Deviation log:**
When intentionally diverging from the Figma design:
> "DEVIATION: Figma shows a custom dropdown. Using native `<select>` instead for keyboard accessibility and form semantics. Visual styling matched with CSS."

### Phase 3: Refinement (the polish loop)

After initial implementation, do a systematic quality pass. Read `references/verification.md` for full procedures.

**Visual audit checklist:**
- [ ] Spacing — all padding, gaps, margins match design tokens
- [ ] Typography — font family, size, weight, line height, letter spacing
- [ ] Colors — exact values including opacity
- [ ] Corner radius — matches design system
- [ ] Shadows — offset, blur, spread, color match
- [ ] Alignment — leading/center/trailing matches design
- [ ] Icon sizing and optical alignment
- [ ] Image aspect ratios and `object-fit` behavior

**Web-specific quality checklist:**
- [ ] Semantic HTML — correct element choices throughout
- [ ] No hardcoded px values — everything references `tokens.css`
- [ ] No `position: absolute` without justification
- [ ] `min-width: 0` set on flex children that should shrink
- [ ] `flex-shrink: 0` set on items that should hold their size
- [ ] Focus styles visible and correct (`:focus-visible`)
- [ ] Color contrast WCAG AA minimum (4.5:1 text, 3:1 large text/UI)
- [ ] Interactive elements are keyboard accessible
- [ ] All `<img>` have meaningful `alt` text
- [ ] Responsive — layout holds at 375px, 768px, 1280px, 1440px
- [ ] Dark mode — colors adapt if the design includes a dark theme

### Phase 4: Verification (before declaring done)

1. **Playwright snapshot tests** — capture new baselines or compare against existing. Read `references/verification.md`.
2. **All component states** render correctly in browser
3. **No console errors** in the browser
4. **Produce the punch list:**
   ```
   ## Punch List: [Screen Name]

   ### Visual Deltas
   - [item]: [description of difference from Figma]

   ### Web Deviations (intentional)
   - [item]: [what changed + why]

   ### Uncertainties (needs user input)
   - [item]: [question for designer/user]

   ### Not Yet Implemented
   - [item]: [interaction/animation implied but not built — separate task]
   ```
5. **Present punch list to user** — they review, approve, or request changes

### Phase 5: Maintenance (ongoing)

- When Figma design updates: re-pull tokens, diff against current `tokens.css`, update
- When components change: verify Code Connect mappings still hold
- Periodic Playwright snapshot runs to catch visual regressions
- If adding new screens: always start at Phase 1 (pre-flight audit), never skip

## Design Tokens Template

Populate from `get_variable_defs` output during Phase 0. Every visual property in the codebase references this file — never hardcode raw values.

```css
/* tokens.css */
:root {
  /* Colors — populate from Figma variables */
  --color-primary: ;
  --color-primary-hover: ;
  --color-secondary: ;
  --color-surface: ;
  --color-surface-secondary: ;
  --color-text-primary: ;
  --color-text-secondary: ;
  --color-text-disabled: ;
  --color-destructive: ;
  --color-success: ;
  --color-warning: ;
  --color-divider: ;
  --color-overlay: ;

  /* Spacing — derive from the design's base unit */
  --spacing-xxs: 2px;
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 12px;
  --spacing-lg: 16px;
  --spacing-xl: 24px;
  --spacing-xxl: 32px;
  --spacing-xxxl: 48px;

  /* Radii */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-full: 9999px;

  /* Typography */
  --font-family-base: 'Inter', system-ui, sans-serif;
  --font-size-xs: 11px;
  --font-size-sm: 13px;
  --font-size-base: 16px;
  --font-size-md: 17px;
  --font-size-lg: 20px;
  --font-size-xl: 24px;
  --font-size-2xl: 28px;
  --font-size-3xl: 34px;

  --font-weight-regular: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;

  --line-height-tight: 1.2;
  --line-height-normal: 1.5;
  --line-height-relaxed: 1.75;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);

  /* Transitions */
  --transition-fast: 150ms ease;
  --transition-base: 200ms ease;
  --transition-slow: 300ms ease;
}

/* Dark mode — populate if the design includes a dark theme */
@media (prefers-color-scheme: dark) {
  :root {
    --color-surface: ;
    --color-text-primary: ;
    /* ... */
  }
}
```

## Animations & Transitions

Figma prototypes show transitions but don't export animation specs. Handle this:

1. **During audit:** Note any Figma prototype connections on the frame
2. **During implementation:** Use CSS transitions as starting points — `transition: var(--transition-base)`
3. **In the punch list:** List all animations as "Not Yet Implemented" with notes on what the prototype suggests
4. **Refine later:** Animation tuning is a separate pass after visual fidelity is locked

Common sensible defaults:
- Hover state changes: `transition: background-color var(--transition-fast), color var(--transition-fast)`
- Modal/sheet entry: `transition: opacity var(--transition-base), transform var(--transition-base)`
- Accordion expand: `transition: height var(--transition-base)` (use `grid-template-rows` trick for smooth height)

## Key Principles

1. **Design system first, screens second** — extract the language before translating individual sentences
2. **Components bottom-up, screens top-down** — build primitives first, assemble screens from them
3. **Native HTML when the design is ambiguous** — prefer `<button>`, `<input>`, `<select>` over custom reimplementations
4. **Tokens over literals** — never hardcode a color, spacing value, or font size. Always reference `tokens.css`
5. **Verify visually, not just logically** — render it, screenshot it, compare it
6. **Flag uncertainty, don't hide it** — a visible question is infinitely better than a silent wrong guess
7. **The Figma file quality is the ceiling** — audit first, always
8. **Responsive is not optional** — infer breakpoints from the design, implement mobile-first

## Reference Files

Load on demand based on what phase you're in:

| File | When to load |
|------|-------------|
| `references/figma-audit.md` | Phase 1 — pre-flight audit checklist and quality scoring |
| `references/css-mapping.md` | Phase 2 — Figma→CSS/HTML translation dictionary |
| `references/figma-mcp-tools.md` | Phase 2 — Figma MCP tool orchestration and prompting |
| `references/web-native.md` | Phase 2/3 — semantic HTML, responsive design, browser conventions |
| `references/code-connect.md` | Phase 0 — Code Connect setup for React/HTML |
| `references/verification.md` | Phase 3/4 — visual verification, Playwright snapshot testing, punch lists |
