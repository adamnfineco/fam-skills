# Web-Native Conventions, Semantics & Responsive Behavior

When to follow Figma literally, and when to use native web patterns instead. Plus the rules for semantic HTML, accessibility, responsive behavior, and browser realities.

## The Native-First Rule

When the Figma design shows something the web platform already does well, prefer the native element unless the design is intentionally custom.

Why: native elements bring keyboard behavior, semantics, accessibility APIs, and browser consistency for free. Custom implementations must rebuild all of that.

### Decision Framework

Ask these questions in order:

1. **Is there a native HTML element for this?** (`button`, `input`, `select`, `details`, `dialog`, `progress`, etc.)
   - Yes → Use native, style it to match
   - No → Build custom

2. **Is the Figma design intentionally different from the native pattern?**
   - No, it's just the designer's approximation → Use native
   - Yes, the custom behavior/appearance is the point → Build custom

3. **Can the native element be styled closely enough?**
   - Yes → Use native with CSS
   - Close but not exact → Use native, document the deviation
   - Not at all → Build custom

## Semantic HTML

### Prefer real elements over `div` soup

| UI Intent | Native Element |
|---|---|
| Main page content | `<main>` |
| Site navigation | `<nav>` |
| Major content block | `<section>` |
| Self-contained card/post/item | `<article>` |
| Header area | `<header>` |
| Footer area | `<footer>` |
| Clickable action | `<button>` |
| Navigation to another page/location | `<a>` |
| Text input | `<input>` |
| Multi-line input | `<textarea>` |
| Choice from list | `<select>` |
| Expand/collapse disclosure | `<details>` + `<summary>` |
| Dialog/modal | `<dialog>` (if project/browser support allows) |
| List of items | `<ul>` / `<ol>` + `<li>` |
| Data table | `<table>` |

**Rule:** if the thing behaves like a button, use `<button>`. If it navigates, use `<a>`. Never make a clickable `div` unless you truly have no native alternative.

### Accessibility baseline

- All interactive elements must be reachable by keyboard
- Focus must be visible with `:focus-visible`
- `<img>` needs meaningful `alt` text, or `alt=""` if purely decorative
- Icon-only buttons need an accessible name (`aria-label`)
- Form inputs need associated labels
- Use headings in order (`h1` → `h2` → `h3`)
- Landmarks should exist: `header`, `main`, `nav`, `footer`

## Common Native vs Custom Decisions

| Figma Shows | Native Option | Decision |
|---|---|---|
| Button | `<button>` | Always native unless it's actually a link |
| Text field | `<input type="text">` | Always native |
| Search field | `<input type="search">` or search form | Use native |
| Dropdown | `<select>` | Use native unless heavily custom is required |
| Accordion | `<details>` / `<summary>` | Prefer native unless animation/custom behavior is critical |
| Modal | `<dialog>` or a custom dialog with ARIA | Prefer native if supported by the project |
| Progress bar | `<progress>` | Use native |
| Toggle/switch | `<input type="checkbox">` styled | Use native input under the hood |
| Tabs | Button set + ARIA tab pattern | Usually custom, but semantic |
| Tooltip | `title` for trivial; custom ARIA tooltip for real UI | Usually custom |
| Navigation links | `<a>` inside `<nav>` | Use native |

## Responsive Design

### Core truth

Figma auto layout is **not** responsive design. It communicates content flow, not full viewport behavior.

Auto layout helps answer:
- Which items sit in a row vs column?
- Which items hug content vs fill available space?
- Where are the gaps and padding?

It does **not** answer:
- When does a two-column layout collapse to one?
- What happens between desktop and mobile?
- How should typography scale fluidly?

You must infer responsive behavior.

### Default strategy

1. **Mobile-first CSS** — base styles target narrow screens first
2. **Add breakpoints upward** with `@media (min-width: ...)`
3. **Prefer fluid sizing** with `clamp()` where appropriate
4. **Use wrapping before stacking** when the layout still reads cleanly
5. **Use Grid when the design is truly two-dimensional**

### Breakpoint heuristics

If the Figma file includes multiple breakpoints, trust them.

If it only includes one frame, infer using these patterns:

- **Horizontal group of cards** → wrap or convert to grid
- **Sidebar + content** → stack on mobile, two columns at larger widths
- **Three+ equal columns** → single column mobile, grid at tablet/desktop
- **Dense dashboard** → allow horizontal overflow only as last resort
- **Nav with many items** → collapse to menu/drawer on smaller widths

### Sensible baseline breakpoints

```css
/* mobile is the default */

@media (min-width: 768px) {
  /* tablet */
}

@media (min-width: 1024px) {
  /* small desktop */
}

@media (min-width: 1280px) {
  /* desktop */
}
```

Don't add five breakpoints because you can. Add the smallest number that makes the layout hold.

### Fluid typography and spacing

Use `clamp()` when the design wants to breathe across screen sizes:

```css
.hero-title {
  font-size: clamp(2rem, 4vw, 4rem);
  line-height: 1.05;
}

.section {
  padding-block: clamp(2rem, 5vw, 6rem);
}
```

Use it selectively. Not every body text size needs to be fluid.

### Container queries

Use container queries when a component's layout should depend on its own width, not the entire viewport.

```css
.card-grid {
  container-type: inline-size;
}

@container (min-width: 600px) {
  .card-grid-item {
    grid-template-columns: 96px 1fr;
  }
}
```

This is often better than viewport breakpoints for reusable components.

## Focus, Hover, Active, Disabled

Figma often shows only the default state. The web still needs all the others.

Every interactive element should consider:
- `:hover`
- `:active`
- `:focus-visible`
- `:disabled`

Example:

```css
.button {
  transition: background-color 150ms ease, color 150ms ease, transform 150ms ease;
}

.button:hover {
  background-color: var(--color-primary-hover);
}

.button:active {
  transform: translateY(1px);
}

.button:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

.button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

## Browser Reality

### Font rendering differences

Figma and browsers will not render text identically. Reasons:
- Different anti-aliasing
- Browser and OS font rendering differences
- Slight differences in line box metrics

Treat small text rendering differences as acceptable if hierarchy, size, and weight are correct.

### Blur and glass effects

`backdrop-filter` is supported broadly now, but not universally in every enterprise environment. If the design relies on blur, document the browser support assumption.

### Scrollbars

Browsers render scrollbars differently. Don't burn time trying to pixel-match them unless the product explicitly styles them.

### Form controls

Native form controls vary by browser and OS. If exact visual fidelity matters more than native appearance, keep the native control for semantics and visually layer a custom shell around it.

## HTML Structure Patterns

### Card

```html
<article class="card">
  <img class="card__image" src="..." alt="...">
  <div class="card__body">
    <h3 class="card__title">Title</h3>
    <p class="card__description">Description</p>
    <button class="card__action">Action</button>
  </div>
</article>
```

### Form row

```html
<div class="field">
  <label for="email" class="field__label">Email</label>
  <input id="email" name="email" type="email" class="field__input">
</div>
```

### Navigation

```html
<nav class="site-nav" aria-label="Primary">
  <a href="/">Home</a>
  <a href="/work">Work</a>
  <a href="/about">About</a>
</nav>
```

## Things That Make Output Look AI-Generated

1. Every value is a one-off magic number
2. Everything is a `div`
3. Absolute positioning everywhere
4. No responsive thought beyond one frame width
5. Missing focus states
6. Links implemented as buttons or buttons implemented as links
7. Huge nested wrappers with no semantic reason to exist

If you see those patterns, back up and rebuild more cleanly.
