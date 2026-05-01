# Figma File Audit — Pre-Flight Checklist for Web

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
- Auto layout presence and depth

### Step 2: Pull Variables

```
get_variable_defs on the target frame
```

Check:
- Are colors bound to variables or hardcoded?
- Are spacing values using tokens?
- Are typography styles defined?
- Are there light/dark token modes?
- What percentage of visual properties use variables vs raw values?

### Step 3: Pull Screenshot

```
get_screenshot of the target frame
```

Visual check:
- Does the design look complete or are there placeholders?
- Is the hierarchy clear?
- Are there obvious alignment issues in the source design?
- Does the frame imply mobile, tablet, desktop, or a one-off marketing canvas?

### Step 4: Pull Code Connect

```
get_code_connect_map with clientFrameworks: ["React"]
```

Check:
- Which components already have web mappings?
- Are the mappings current (files exist)?
- What percentage of repeated instances are connected?

## The Audit Checklist

Score each item: PASS / WARN / FAIL

### Layer Naming

| Check | Pass | Warn | Fail |
|-------|------|------|------|
| Meaningful names | All layers have semantic names | Some generic names | Mostly auto-generated names |
| Consistent convention | Reasonably consistent naming | Mixed but readable | No pattern |
| Describes content | `heroTitle`, `searchInput` | Vague but usable | `Frame 12`, `Rectangle 3` |

**If FAIL:** Ask the user to run Figma's AI rename layers feature before proceeding.

### Auto Layout

| Check | Pass | Warn | Fail |
|-------|------|------|------|
| Root frame uses auto layout | Yes | N/A | No |
| Child containers use auto layout | Nearly all | Most | Minority or none |
| No unnecessary absolute positioning | No manual positions | Few exceptions | Many elements manually positioned |
| Spacing is consistent | Uses spacing tokens or scale | Mostly consistent | Random spacing values |

**If FAIL:** The frame will be hard to translate to responsive CSS. Auto layout maps cleanly to Flexbox/Grid. Absolute positioning usually means brittle output.

### Design Tokens & Variables

| Check | Pass | Warn | Fail |
|-------|------|------|------|
| Colors use variables | >90% | 50–90% | <50% |
| Spacing uses variables | Most padding/gaps tokenized | Some tokenized | All hardcoded |
| Typography uses styles | Text styles defined and applied | Some styles | No text styles |
| Radius uses variables | Most tokenized | Some tokenized | All hardcoded |

**If WARN/FAIL:** You can still implement, but you'll need to infer a design system from repeated values. Document inferred values in the Design Brief.

### Component Usage

| Check | Pass | Warn | Fail |
|-------|------|------|------|
| Repeated patterns are components | Buttons/cards/rows are component instances | Most are | Copy-pasted repetition |
| Components have variants | Size/state/style variants defined | Some variants | No variants |
| Component properties are used | Props drive content/state | Some props | Variants are all hardcoded |
| Nesting is clean | 2–3 levels | 4–5 levels | Too deep or flat duplication |

**If FAIL:** Without componentization, the implementation will bloat and Code Connect loses most of its value.

### Structure & Organization

| Check | Pass | Warn | Fail |
|-------|------|------|------|
| Logical grouping | Related elements grouped clearly | Mostly logical | Flat/random nesting |
| No hidden junk | Visible layers are intentional | Few hidden layers | Many hidden/empty layers |
| Hierarchy matches visual hierarchy | Parent-child makes sense | Mostly | Contradictory |
| Reasonable complexity | <50 visible layers | 50–100 | >100 |

### Web Readiness

| Check | Pass | Warn | Fail |
|-------|------|------|------|
| Interactive intent is clear | Buttons/links/inputs are obvious | Mostly clear | Ambiguous |
| States are shown | Default + hover/focus/active/disabled where relevant | Some states | Only default |
| Responsive intent exists | Multiple breakpoints or clear auto-layout behavior | Some clues | One static canvas with no clues |
| Content is realistic | Real text lengths and images | Mostly realistic | Lorem/placeholder-heavy |

**If FAIL on responsive intent:** You'll need to infer breakpoints. That's fine, but log the assumptions.

## Quality Score

| Score | Meaning | Action |
|-------|---------|--------|
| All PASS | Excellent file quality | Proceed confidently |
| Mostly PASS, few WARN | Good enough | Proceed, document inferred values |
| Mixed PASS/WARN/FAIL | Proceed with caution | Flag issues and expect iteration |
| Mostly WARN/FAIL | File needs work | Recommend cleanup before implementation |
| Many FAIL | Not ready | Stop and provide a cleanup checklist |

## Common Fix Recommendations

1. **Generic layer names** → Run Figma AI rename on the frame
2. **Missing auto layout** → Add auto layout to the specific frame and infer spacing tokens
3. **Hardcoded colors** → Convert repeated colors into variables with semantic roles
4. **No components** → Turn repeated patterns into components before implementation
5. **Too complex** → Split the frame into logical sections/components
6. **No states** → Add hover/focus/active/disabled variants for interactive elements
7. **No responsive clues** → Add mobile/desktop frames or at least confirm how layouts should collapse

## Design System Extraction

Even when the audit reveals issues, extract what you can.

### Spacing Pattern Detection

- Group padding/gap values by frequency
- Identify base unit (usually 4px or 8px)
- Map to a clean scale: xs(4), sm(8), md(12/16), lg(20/24), xl(32), xxl(48+)
- Flag outliers that don't fit

### Color Pattern Detection

- Group fills/strokes by similarity
- Identify roles: primary, secondary, surface, text-primary, text-secondary, divider, destructive, success, warning
- Flag single-use one-offs
- Check whether dark mode values exist

### Typography Pattern Detection

- Group text by size + weight
- Identify hierarchy: display, title, heading, body, caption, overline, button
- Check line-height ratios
- Flag near-duplicates that should probably collapse into one token

### Responsive Pattern Detection

Look for:
- Horizontal groups that likely wrap on smaller screens
- Sidebars that likely stack under content on mobile
- Multi-column card sections that likely become one column
- Hero sections that need fluid typography

Document these as assumptions in the Design Brief.
