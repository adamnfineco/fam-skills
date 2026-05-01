# Code Connect for SwiftUI

Code Connect links Figma components to your actual SwiftUI code. When set up, the Figma MCP returns real implementation details instead of guessing.

## Why Code Connect Matters

Without Code Connect:
- `get_design_context` returns generic CSS-like properties
- The agent invents component names and APIs
- Every screen re-creates components from scratch
- No reuse, no consistency

With Code Connect:
- `get_design_context` returns your actual SwiftUI struct names, props, and usage
- `get_code_connect_map` provides file paths and snippet examples
- The agent reuses existing components instead of creating new ones
- Consistent API surface across the entire app

## Requirements

- Figma Organization or Enterprise plan (Code Connect UI)
- OR Code Connect CLI (open source, works with any plan for publishing)
- A Figma component library with defined components

## Setup via Figma MCP

The simplest approach using the MCP tools directly:

### Step 1: Build your SwiftUI components first

Create the base components in your project. Example:

```swift
// Sources/UI/Components/PrimaryButton.swift
struct PrimaryButton: View {
    let title: String
    var icon: String? = nil
    var isDisabled: Bool = false
    var isLoading: Bool = false
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            HStack(spacing: DesignTokens.Spacing.sm) {
                if isLoading {
                    ProgressView()
                        .tint(.white)
                } else {
                    if let icon {
                        Image(systemName: icon)
                    }
                    Text(title)
                        .font(DesignTokens.Typography.buttonLabel)
                }
            }
            .frame(maxWidth: .infinity)
            .padding(.vertical, DesignTokens.Spacing.md)
            .padding(.horizontal, DesignTokens.Spacing.lg)
            .background(isDisabled ? DesignTokens.Colors.surfaceDisabled : DesignTokens.Colors.primary)
            .foregroundStyle(.white)
            .clipShape(RoundedRectangle(cornerRadius: DesignTokens.Radii.md, style: .continuous))
        }
        .disabled(isDisabled || isLoading)
    }
}
```

### Step 2: Map components using add_code_connect_map

For each SwiftUI component, use the Figma MCP to create a mapping:

```
Call: add_code_connect_map
- nodeId: [Figma component node ID from get_metadata]
- source: "Sources/UI/Components/PrimaryButton.swift"
- componentName: "PrimaryButton"
```

The node ID comes from the Figma component. You can find it by:
1. Running `get_metadata` on a frame containing the component
2. Looking for the component instance in the XML
3. Using the node ID from the instance

### Step 3: Verify mappings

```
Call: get_code_connect_map
- clientFrameworks: ["SwiftUI"]
```

This should return your mappings with the component names and file paths.

## Setup via Code Connect CLI (Advanced)

For more control, use Figma's Code Connect CLI with Swift support.

### Install

Code Connect for SwiftUI lives in the `figma/code-connect` repo. The SwiftUI integration is in the `swiftui/` directory.

```bash
# Install the CLI via npm (used for publishing mappings)
npm install --save-dev @figma/code-connect
# Or use the Figma MCP add_code_connect_map tool directly (no CLI needed)
```

Note: For most workflows, you don't need the CLI package at all — the Figma MCP `add_code_connect_map` tool handles mapping directly. The CLI is only needed for advanced `.figma.swift` file workflows.

Repo: https://github.com/figma/code-connect

### Create .figma.swift files

Code Connect for SwiftUI uses the `FigmaConnect` protocol:

```swift
import CodeConnect

struct PrimaryButtonFigmaConnect: FigmaConnect {
    let component = "https://figma.com/file/xxx/Component?node-id=123:456"
    
    @FigmaString("Label")
    var title: String
    
    @FigmaBoolean("Show Icon")
    var showIcon: Bool
    
    @FigmaEnum("State", mapping: [
        "Default": false,
        "Disabled": true
    ])
    var isDisabled: Bool
    
    var body: some View {
        PrimaryButton(
            title: title,
            icon: showIcon ? "arrow.right" : nil,
            isDisabled: isDisabled,
            action: {}
        )
    }
}
```

### Property Wrappers

| Wrapper | Figma Property Type | Swift Type |
|---------|-------------------|------------|
| `@FigmaString` | Text | `String` |
| `@FigmaBoolean` | Boolean | `Bool` |
| `@FigmaEnum` | Variant / Instance swap | Any (with mapping dict) |
| `@FigmaInstance` | Instance slot | `some View` (for nested components) |

### Variant Mapping

For components with Figma variants:

```swift
@FigmaEnum("Size", mapping: [
    "Small": ButtonSize.small,
    "Medium": ButtonSize.medium,
    "Large": ButtonSize.large,
])
var size: ButtonSize

@FigmaEnum("Style", mapping: [
    "Primary": ButtonStyle.primary,
    "Secondary": ButtonStyle.secondary,
    "Destructive": ButtonStyle.destructive,
])
var style: ButtonStyle
```

### Nested Components

For components containing other components:

```swift
@FigmaInstance("Icon")
var icon: some View

var body: some View {
    HStack {
        icon
        Text(title)
    }
}
```

## Multiple Framework Support

Code Connect supports mapping the same Figma component to multiple frameworks. If your Figma library serves both web and iOS:

- Set label "SwiftUI" for your Swift mappings
- Set label "React" for your web mappings
- Use `clientFrameworks: ["SwiftUI"]` in MCP calls to get only Swift mappings

## What to Map

Priority order for mapping:

1. **Buttons** — most reused, highest impact
2. **Text styles** — if you have text components (heading, body, caption)
3. **Input fields** — text inputs, search bars, pickers
4. **Cards / list rows** — high repetition
5. **Navigation components** — nav bars, tab bars
6. **Icons** — if using a custom icon component wrapper
7. **Indicators** — badges, tags, pills, status dots
8. **Modals / sheets** — if you have standardized sheet layouts

## Maintaining Mappings

- **When you rename a component:** Update the Code Connect mapping source path
- **When you change props:** Update the property wrappers in the .figma.swift file
- **When Figma components change:** Re-run `get_code_connect_suggestions` to detect drift
- **When you add a new component:** Map it immediately — don't batch

## Using get_code_connect_suggestions

This Figma-prompted tool detects Figma components that could be mapped but aren't yet. Run it periodically:

```
Call: get_code_connect_suggestions
```

It will suggest mappings. Review them, then confirm with:

```
Call: send_code_connect_mappings
```

This is especially useful after:
- Adding new screens to the Figma file
- Adding new SwiftUI components
- Refactoring component structure
