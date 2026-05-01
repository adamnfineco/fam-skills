# Figma → CSS/HTML Translation Dictionary

The core mapping reference. When translating Figma designs to CSS and HTML, use these rules. They cover the non-obvious translations that cause "close but not right" output.

## Layout System

### Auto Layout → Flexbox

| Figma Auto Layout | CSS | Notes |
|---|---|---|
| Vertical, top-to-bottom | `display: flex; flex-direction: column` | |
| Horizontal, left-to-right | `display: flex; flex-direction: row` | Default — `flex-direction: row` can be omitted |
| Gap between items | `gap: Npx` | Use `var(--spacing-X)` from tokens |
| Padding (all sides equal) | `padding: Npx` | |
| Padding (per-side) | `padding: top right bottom left` | Or per-property: `padding-top`, etc. |
| Wrap (auto layout wrap) | `flex-wrap: wrap` | Also add `align-content` — see below |

### Figma "Space Between" Modes

| Figma Setting | CSS |
|---|---|
| Packed (default) | `justify-content: flex-start` (or omit — it's the default) |
| Space between | `justify-content: space-between` |
| Space around | `justify-content: space-around` |
| Space evenly | `justify-content: space-evenly` |
| Center | `justify-content: center` |

### Sizing & Constraints

| Figma Sizing | CSS |
|---|---|
| Fixed width/height | `width: Npx; height: Npx` — but add `flex-shrink: 0` when inside a flex container |
| Hug contents | Default behavior — don't set width/height |
| Fill container (horizontal) | `flex: 1` or `flex-grow: 1` or `width: 100%` (context-dependent) |
| Fill container (vertical) | `flex: 1` inside a flex column container |
| Fill container (both) | `flex: 1` in a flex container that spans the viewport |
| Min width | `min-width: Npx` |
| Max width | `max-width: Npx` |
| Aspect ratio | `aspect-ratio: W / H` |

### Critical Flexbox Gotchas

**1. Fixed-size items shrink unexpectedly.**
In Figma, "Fixed width" is a hard constraint. In CSS Flexbox, `width` is a *suggestion* — the flex algorithm can shrink below it. To replicate Figma's fixed behavior:
```css
.item {
  width: 48px;
  flex-shrink: 0; /* required */
}
```

**2. Text and content have hidden minimum sizes.**
Flex children won't shrink below their longest unbreakable word or intrinsic content size by default. This causes overflow that doesn't exist in Figma. Fix:
```css
.flex-child-that-should-shrink {
  min-width: 0; /* overrides the default min-width: auto */
  overflow: hidden; /* often paired with this */
}
```

**3. "Fill container" in a nested layout.**
When a Figma element uses "Fill container" inside a container that itself uses "Hug contents," the fill behavior depends on the outer container's resolved size. In CSS, this means you may need `width: 100%` on the child AND a resolved width on the parent — not just `flex: 1`.

**4. Wrap + alignment.**
When using `flex-wrap: wrap`, the `align-items` property controls alignment within each row, but `align-content` controls how the rows themselves are distributed. Figma doesn't expose `align-content` — infer it from the overall layout intent.

```css
/* Common wrap pattern */
.container {
  display: flex;
  flex-wrap: wrap;
  gap: var(--spacing-md);
  align-content: flex-start; /* usually what you want — rows pack to the top */
}
```

**5. "Ignore auto layout" = `position: absolute`.**
When a Figma element is set to "Ignore auto layout," it floats above the layout flow. In CSS, this maps to `position: absolute` relative to the nearest `position: relative` ancestor. This is often a design hack — prefer rethinking the layout in CSS terms over direct translation.

### CSS Grid (for 2D layouts)

Figma doesn't have a native Grid equivalent, but many designs have implicit grid structures (card grids, form layouts, data tables). When you see a horizontal auto layout wrapping with consistent column sizing, consider CSS Grid instead:

```css
/* Responsive card grid — common pattern */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: var(--spacing-xl);
}

/* Fixed column grid */
.two-col {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: var(--spacing-lg);
}

/* Sidebar + main */
.layout {
  display: grid;
  grid-template-columns: 280px 1fr;
  gap: var(--spacing-xl);
}
```

### Alignment

| Figma Alignment | CSS (in a flex container) |
|---|---|
| Top-left | `align-items: flex-start; justify-content: flex-start` |
| Top-center | `align-items: flex-start; justify-content: center` |
| Top-right | `align-items: flex-start; justify-content: flex-end` |
| Center-left | `align-items: center; justify-content: flex-start` |
| Center | `align-items: center; justify-content: center` |
| Bottom-right | `align-items: flex-end; justify-content: flex-end` |

### Stacking & Overlap

| Figma Pattern | CSS |
|---|---|
| Layers stacked (z-order) | Parent: `position: relative`. Children: `position: absolute` with `z-index` |
| Element overlapping another | `position: absolute` on the overlay element |
| Badge/indicator on a component | Parent: `position: relative`. Badge: `position: absolute; top: 0; right: 0` |
| Full bleed background | `position: absolute; inset: 0` (shorthand for top/right/bottom/left: 0) |

Note: Unlike Figma (top layer in panel = front), CSS z-index is explicit — higher values are in front.

## Visual Properties

### Colors

| Figma | CSS |
|---|---|
| Solid fill `#RRGGBB` | `color: #RRGGBB` or `background-color: #RRGGBB` |
| Solid fill with opacity | `color: rgb(R G B / 0.5)` — prefer modern syntax |
| Linear gradient | `background: linear-gradient(angle, color1 stop%, color2 stop%)` |
| Radial gradient | `background: radial-gradient(circle at X Y, color1, color2)` |
| Design token color | **Always** use `var(--color-xxx)` — never raw hex |
| P3 wide-gamut color | `color: color(display-p3 R G B)` with sRGB fallback above it |

**Modern color syntax (preferred):**
```css
/* rgba() is legacy — use space-separated with / for alpha */
background-color: rgb(99 102 241 / 0.1);
color: rgb(17 24 39 / 0.8);
```

### Corner Radius

| Figma | CSS |
|---|---|
| All corners equal | `border-radius: Npx` — use `var(--radius-X)` |
| Per-corner radius | `border-radius: top-left top-right bottom-right bottom-left` |
| Fully rounded (pill) | `border-radius: var(--radius-full)` (9999px) |
| Circle | `border-radius: 50%` (on a square element) |

**Important:** Figma's default corner rounding does NOT match CSS `border-radius` exactly for large radius values. Figma uses a "squircle" (superellipse) curve that looks smoother than the CSS circular arc. There's no direct CSS equivalent without SVG or a custom clip-path. For design systems that rely on this look (Apple-style), note it as a known deviation.

### Shadows & Effects

| Figma | CSS |
|---|---|
| Drop shadow | `box-shadow: X Y blur spread color` |
| Multiple shadows | Comma-separate: `box-shadow: shadow1, shadow2` |
| Inner shadow | `box-shadow: inset X Y blur spread color` |
| Layer blur | `filter: blur(Npx)` |
| Background blur (glass) | `backdrop-filter: blur(Npx)` — check browser support |
| Text shadow | `text-shadow: X Y blur color` |

**Shadow with opacity (correct syntax):**
```css
box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
```

### Borders & Strokes

| Figma | CSS |
|---|---|
| Stroke (inside) | `box-shadow: inset 0 0 0 1px var(--color-divider)` — preserves border-radius |
| Stroke (center) | `outline: 1px solid var(--color-divider)` — doesn't affect layout |
| Stroke (outside) | `box-shadow: 0 0 0 1px var(--color-divider)` |
| Stroke on one side only | `border-bottom: 1px solid var(--color-divider)` etc. |
| Dashed stroke | `border: 2px dashed var(--color-divider)` |

**Prefer `box-shadow` over `border` for strokes** when the element has border-radius — `border` changes the element's box model and can cause layout shifts. `box-shadow` with spread is layout-neutral.

## Typography

### Text Properties

| Figma Property | CSS |
|---|---|
| Font family | `font-family: 'Name', fallback` — use `var(--font-family-base)` |
| Font weight | `font-weight: 400/500/600/700` — use `var(--font-weight-X)` |
| Font size | `font-size: Npx` — use `var(--font-size-X)` |
| Line height (absolute px) | `line-height: N` (unitless ratio) — see calculation below |
| Letter spacing | `letter-spacing: Npx` — Figma shows px, CSS can use px or em |
| Text case (uppercase) | `text-transform: uppercase` |
| Text alignment | `text-align: left/center/right` |
| Truncate (single line) | `white-space: nowrap; overflow: hidden; text-overflow: ellipsis` |
| Truncate (multi-line) | `display: -webkit-box; -webkit-line-clamp: N; -webkit-box-orient: vertical; overflow: hidden` |
| Text color | `color: var(--color-text-primary)` |
| Text decoration | `text-decoration: underline/line-through/none` |

### Line Height Calculation

Figma specifies absolute line height (e.g., 24px for 16px text).
CSS `line-height` is best expressed as a unitless ratio.

```
css_line_height = figma_line_height / figma_font_size
```

Example: Figma line height 24px, font size 16px → `line-height: 1.5`

For single-line text, `line-height` mainly affects vertical centering within the element — set it to match the design.

### Font Weight Mapping

Figma uses named weights. CSS uses numeric values:

| Figma | CSS `font-weight` |
|---|---|
| Thin | 100 |
| Extra Light | 200 |
| Light | 300 |
| Regular | 400 |
| Medium | 500 |
| Semi Bold | 600 |
| Bold | 700 |
| Extra Bold | 800 |
| Black | 900 |

**Common miss:** Figma shows "Semi Bold" → AI outputs `font-weight: bold` (700). Incorrect. Semi Bold = 600.

## Images & Media

| Figma | CSS/HTML |
|---|---|
| Image fill (cover) | `<img>` with `object-fit: cover; width: 100%; height: 100%` |
| Image fill (contain/fit) | `object-fit: contain` |
| Image with rounded corners | `border-radius: var(--radius-X); overflow: hidden` on the container |
| Remote/dynamic image | `<img src="..." alt="..." loading="lazy">` |
| Background image | `background-image: url(...); background-size: cover; background-position: center` |
| SVG icon | Inline `<svg>` or `<img src="icon.svg">` — use from Figma MCP URL directly |

**Asset handling rule:** The Figma MCP provides asset URLs. Use them directly — never create placeholder `<img>` tags or download/copy assets. The skill's asset handoff produces an export list for the designer.

## Scroll & Lists

| Figma Pattern | HTML/CSS |
|---|---|
| Vertical scrolling content | `overflow-y: auto` on the container |
| Horizontal scrolling row | `overflow-x: auto; white-space: nowrap` or flex with `overflow-x: auto` |
| List of items | `<ul>/<ol>` for semantic lists, CSS for layout |
| Grid of cards | CSS Grid with `auto-fill`/`auto-fit` |
| Sticky header | `position: sticky; top: 0; z-index: 10` |

**Scrollbar styling (optional):**
```css
.scrollable {
  scrollbar-width: thin; /* Firefox */
  scrollbar-color: var(--color-divider) transparent;
}
.scrollable::-webkit-scrollbar { width: 6px; }
.scrollable::-webkit-scrollbar-track { background: transparent; }
.scrollable::-webkit-scrollbar-thumb { background: var(--color-divider); border-radius: var(--radius-full); }
```

## Common Gotchas

1. **Figma's coordinate system is top-left origin, CSS box model is also top-left** — but Figma x/y values are absolute positions in the canvas, not CSS positioning values. Don't translate Figma x/y directly to `left/top`. Use layout (flex/grid) and alignment instead.

2. **Figma layers: top of panel = front. CSS z-index: explicit.** Unlike Figma, there's no implied stacking order without `z-index`.

3. **Figma shows design pixels (1x). CSS uses logical pixels.** A 1px Figma border = `1px` in CSS. Don't divide by device pixel ratio.

4. **Figma opacity on a group = CSS `opacity` on the container.** Both affect all children identically. For selective child opacity, apply it per-child.

5. **Figma strokes can be inside/outside/center. CSS `border` is center by default.** Use `box-shadow` for precise stroke position control.

6. **Figma padding is inside the container. CSS `padding` is also inside.** They map directly — no conversion needed.

7. **Figma "clip content" = `overflow: hidden` on the container.**

8. **Figma text baseline ≠ CSS text baseline.** Figma positions text from the top of the text box. CSS renders text along the baseline with ascender/descender space above and below. This causes subtle vertical positioning differences, especially for centered text inside fixed-height containers. Use `display: flex; align-items: center` on the container rather than `padding` to center text reliably.

9. **Magic numbers are a failure mode.** If you're about to write `padding: 13px`, stop — find the token it should map to. If no token fits, flag it as an uncertainty.
