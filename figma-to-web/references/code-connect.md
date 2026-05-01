# Code Connect for Web

Code Connect links Figma components to your actual web code. When set up, the Figma MCP returns real implementation details instead of guessing.

## Why Code Connect Matters

Without Code Connect:
- `get_design_context` returns generic structure and styling hints
- The agent invents component names and APIs
- Every screen re-creates components from scratch
- Reuse and consistency erode quickly

With Code Connect:
- `get_design_context` can return your actual component names, props, and usage
- `get_code_connect_map` provides file paths and snippets
- The agent reuses existing components instead of duplicating them

## Requirements

- Figma Organization or Enterprise plan for the full UI, or the Code Connect CLI workflow
- A Figma component library with real components
- A web codebase with stable component boundaries

## Setup via Figma MCP

### Step 1: Build your base components first

React example:

```tsx
// src/components/PrimaryButton.tsx
type PrimaryButtonProps = {
  children: React.ReactNode
  disabled?: boolean
  loading?: boolean
  onClick?: () => void
}

export function PrimaryButton({ children, disabled = false, loading = false, onClick }: PrimaryButtonProps) {
  return (
    <button className="primaryButton" disabled={disabled || loading} onClick={onClick}>
      {loading ? 'Loading…' : children}
    </button>
  )
}
```

HTML/CSS projects may not have a component system in code, but they still have reusable patterns. Map the canonical implementation file or template partial if the project supports it.

### Step 2: Map components with add_code_connect_map

```
Call: add_code_connect_map
- nodeId: [Figma component node ID]
- source: "src/components/PrimaryButton.tsx"
- componentName: "PrimaryButton"
```

### Step 3: Verify mappings

```
Call: get_code_connect_map
- clientFrameworks: ["React"]
```

## Setup via Code Connect CLI (Advanced)

Install:

```bash
npm install --save-dev @figma/code-connect
```

Repo: https://github.com/figma/code-connect

### React example

```tsx
import { figma } from '@figma/code-connect'
import { PrimaryButton } from './PrimaryButton'

figma.connect(PrimaryButton, 'https://figma.com/file/xxx?node-id=123:456', {
  props: {
    children: figma.string('Label'),
    disabled: figma.boolean('Disabled'),
  },
  example: ({ children, disabled }) => (
    <PrimaryButton disabled={disabled}>{children}</PrimaryButton>
  ),
})
```

The exact CLI API evolves — check the upstream docs if the project already uses Code Connect files. The important thing is the mapping intent: Figma component ↔ code component.

## Multiple Framework Support

The same Figma component can map to multiple frameworks.

- Label React mappings as `React`
- If the design system also serves native, keep those mappings distinct
- Use `clientFrameworks: ["React"]` in MCP calls to limit noise

## What to Map First

1. Buttons
2. Form fields
3. Cards and list rows
4. Navigation components
5. Badges/tags/status indicators
6. Modals/sheets
7. Repeated layout shells

## Maintaining Mappings

- When a component moves, update the source path
- When props change, update the mapping example
- When new reusable components are added, map them immediately
- Re-run suggestions periodically if available in your Figma tooling

## React vs Plain HTML/CSS

React is the natural fit for Code Connect because there is a real component API.

For plain HTML/CSS projects:
- Use Code Connect only if the project still has reusable templating units
- Otherwise treat Code Connect as a source-of-truth reference from the design system, not a one-to-one binding

That's fine. The skill still works without full Code Connect coverage.
