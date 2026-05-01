# Figma File Audit — Pre-Flight Checklist

Run this audit before implementing any frame. The quality of the Figma file is the ceiling for output quality. A messy file guarantees messy code.

## Audit Procedure

### Step 1: Pull Metadata

```
get_metadata on the target frame
```

Examine the XML for:
- Layer naming quality
- Nesting depth
- Frame vs group usage
- Component instance usage
- Total layer count

### Step 2: Pull Variables

```
get_variable_defs on the target frame
```

Check:
- Are colors bound to variables or hardcoded?
- Are spacing values using tokens?
- Are typography styles defined?
- What percentage of visual properties use variables vs raw values?

### Step 3: Pull Screenshot

```
get_screenshot of the target frame
```

Visual check:
- Does the design look complete or are there placeholder elements?
- Are there obvious alignment issues in the source design?
- Is the visual hierarchy clear?

### Step 4: Pull Code Connect

```
get_code_connect_map with clientFrameworks: ["SwiftUI"]
```

Check:
- Which components already have SwiftUI mappings?
- Are the mappings current (pointing to files that exist)?
- What percentage of instances are connected?

## The Audit Checklist

Score each item: PASS / WARN / FAIL

### Layer Naming

| Check | Pass | Warn | Fail |
|-------|------|------|------|
| Meaningful names | All layers have semantic names | Some generic names ("Group 5") | Mostly auto-generated names |
| Consistent convention | camelCase or kebab-case throughout | Mixed but readable | No pattern |
| Describes content | "profileAvatar", "searchInput" | "avatar", "input" (vague but ok) | "Frame 12", "Rectangle 3" |

**If FAIL:** Ask user to run Figma's AI rename layers feature before proceeding. This single step dramatically improves output quality.

### Auto Layout

| Check | Pass | Warn | Fail |
|-------|------|------|------|
| Root frame uses auto layout | Yes | N/A | No — absolute positioning |
| Child containers use auto layout | All or nearly all | Most (>70%) | Minority or none |
| No unnecessary absolute positioning | No elements manually positioned | Few exceptions | Many elements positioned manually |
| Spacing is consistent | Uses spacing tokens or consistent values | Mostly consistent | Random spacing values |

**If FAIL:** The frame will be very difficult to translate to SwiftUI layout. Auto layout maps cleanly to VStack/HStack/ZStack; absolute positioning requires manual frame() calculations that are fragile across screen sizes. Strongly recommend fixing the Figma file.

### Design Tokens & Variables

| Check | Pass | Warn | Fail |
|-------|------|------|------|
| Colors use variables | >90% of fills/strokes | 50-90% | <50% or none |
| Spacing uses variables | Padding and gaps use tokens | Some use tokens | All hardcoded |
| Typography uses styles | Text styles defined and applied | Some text styles | No text styles, all manual |
| Corner radius uses variables | Radius values are tokenized | Some tokenized | All hardcoded |

**If WARN/FAIL:** I can still implement, but I'll need to infer the design system from repeated values. This is error-prone. Document inferred values in the Design Brief and flag for user confirmation.

### Component Usage

| Check | Pass | Warn | Fail |
|-------|------|------|------|
| Repeated patterns are components | Buttons, cards, list rows, etc. are component instances | Most are | Copy-pasted, not componentized |
| Components have variants | Size, state, style variants defined | Some variants | No variants |
| Component properties are used | Props for text, icons, toggles | Some props | Everything hardcoded in variants |
| Nesting is clean | 2-3 levels of component nesting | 4-5 levels | >5 levels or flat with duplication |

**If FAIL:** Without components, I have to treat every instance as unique. This means no Code Connect, no component reuse, and much more code to write and maintain.

### Structure & Organization

| Check | Pass | Warn | Fail |
|-------|------|------|------|
| Logical grouping | Related elements are grouped logically | Mostly logical | Flat or random nesting |
| No empty/hidden layers | All visible layers are intentional | Few hidden layers | Many hidden/empty layers |
| Frame hierarchy matches visual hierarchy | Parent-child matches what you'd expect | Mostly | Structure contradicts visual hierarchy |
| Reasonable complexity | <50 visible layers per screen | 50-100 layers | >100 layers — needs decomposition |

### Annotations & Intent

| Check | Pass | Warn | Fail |
|-------|------|------|------|
| Interactive elements are identifiable | Buttons look like buttons, inputs like inputs | Mostly clear | Ambiguous — is this tappable? |
| States are shown | Multiple states per interactive element | Some states | Only default state |
| Navigation intent is clear | Flow connections or annotations exist | Some hints | No indication of what happens on tap |
| Behavioral notes | Annotations for loading, error, empty states | Some annotations | No behavioral context |

**If FAIL on states/behavior:** I'll implement the visual default state and produce a list of questions about missing states. This is acceptable — but the user should know upfront that the implementation will be visually complete but behaviorally incomplete.

## Quality Score

Count your PASS / WARN / FAIL across all categories:

| Score | Meaning | Action |
|-------|---------|--------|
| All PASS | Excellent file quality | Proceed confidently |
| Mostly PASS, few WARN | Good enough | Proceed, document inferred values |
| Mixed PASS/WARN/FAIL | Proceed with caution | Flag all issues to user, expect more iteration |
| Mostly WARN/FAIL | File needs work | Recommend cleanup before implementation. List specific fixes. |
| Many FAIL | Not ready | Stop. The file needs significant work. Provide a cleanup checklist. |

## Common Fix Recommendations

When the audit reveals problems, give the user specific, actionable fixes:

1. **Generic layer names** → "Run Figma's AI rename feature on this frame: select all layers → right-click → Rename layers with AI"
2. **Missing auto layout** → "Add auto layout to [specific frame]. Select the frame → Shift+A. Set spacing to [inferred value]."
3. **Hardcoded colors** → "These colors should be variables: [list hex values and their roles]. Create color variables in Figma and apply them."
4. **No components** → "These elements repeat 3+ times and should be components: [list]. Select one instance → right-click → Create component."
5. **Too complex** → "This frame has [N] layers. I recommend splitting it into: [proposed sub-frames]. I'll implement each separately and compose them."

## Design System Extraction

Even when the audit reveals issues, extract what you can. Every Figma file contains an implicit design system — the audit helps you find it.

### Spacing Pattern Detection

Scan all padding and gap values across the frame:
- Group by frequency
- Identify the base unit (usually 4px or 8px)
- Map to a scale: xs(4), sm(8), md(12/16), lg(20/24), xl(32), xxl(48+)
- Flag outliers that don't fit the scale

### Color Pattern Detection

Scan all fill and stroke colors:
- Group by similarity (within 5% hue/saturation)
- Identify roles: primary brand, secondary brand, text primary, text secondary, surface, divider, destructive, success, warning
- Flag colors used only once (likely one-offs or mistakes)
- Check if dark mode variants exist

### Typography Pattern Detection

Scan all text layers:
- Group by size + weight combination
- Identify hierarchy: title/headline, subtitle, body, caption, overline, button
- Check line height ratios (typically 1.2-1.5x font size)
- Flag inconsistencies (e.g., two body styles that are 1px apart — probably should be the same)

### Component Pattern Detection

Look for visual repetition that isn't componentized:
- Cards with the same structure but different content
- List rows with consistent layout
- Button-like elements with similar shape/padding
- Header/footer patterns across screens
- Icon + text combinations

Report these as "recommended components" — things the user could componentize for better Code Connect support.

### Guiding Principles Extraction

Beyond raw values, try to articulate the design's character:

- **Density**: spacious (lots of whitespace) or compact (information-dense)?
- **Shape language**: rounded (large radii, pill shapes) or geometric (small radii, sharp corners)?
- **Typography feel**: modern (sans-serif, tight tracking) or classic (serif, generous leading)?
- **Color strategy**: monochrome with accent, full palette, gradient-heavy?
- **Hierarchy method**: size-based, weight-based, color-based, spatial?
- **Interaction style**: flat (no elevation), layered (shadows/cards), material (blur/transparency)?

These principles guide decisions when the design is ambiguous. If the design is "spacious and rounded," and I encounter an ambiguous gap, I lean toward the larger value.
