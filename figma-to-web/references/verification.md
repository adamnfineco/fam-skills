# Visual Verification & Snapshot Testing for Web

How to verify that a web implementation matches the Figma design. The goal is 1:1 — not "looks about right."

## Verification Loop

After every implementation pass:

```
1. Render the component/screen in the browser
2. Take a browser screenshot to file
3. Pull the Figma screenshot to file
4. Compare side by side
5. Identify deltas
6. Fix deltas
7. Repeat until matched
```

**Context note:** Save screenshots to disk first, then load only the active comparison into context.

## What to Compare

### Tier 1: Must Match Exactly

- Colors, including opacity
- Typography size and weight
- Content and labels
- Visual hierarchy
- Corner radius values
- Primary icon choice

### Tier 2: Must Match Closely

- Spacing and margins
- Line height and letter spacing
- Shadows
- Alignment
- Optical sizing of icons

### Tier 3: Acceptable Differences

- Browser font rendering differences
- Native scrollbar rendering
- Native form control chrome, if intentionally preserved
- Minor anti-aliasing differences

## Manual Comparison Technique

1. **Overall scan** — step back and check shape/weight first
2. **Section pass** — header, body, footer, component-by-component
3. **Vertical rhythm** — spacing issues are the #1 reason it feels off
4. **Horizontal alignment** — edges should line up cleanly
5. **Typography** — size, weight, and line height separately
6. **Fine details** — borders, shadows, small indicators, icon weight

## Playwright Snapshot Testing

Playwright is the practical default for web visual regression.

### Basic example

```ts
import { test, expect } from '@playwright/test'

test('home page matches baseline', async ({ page }) => {
  await page.goto('http://localhost:3000')
  await expect(page).toHaveScreenshot('home-page.png', {
    fullPage: true,
    maxDiffPixels: 100,
  })
})
```

### Component example

```ts
import { test, expect } from '@playwright/test'

test('primary button states', async ({ page }) => {
  await page.goto('http://localhost:3000/components/button')
  await expect(page.locator('[data-test="button-default"]')).toHaveScreenshot('button-default.png')
  await expect(page.locator('[data-test="button-disabled"]')).toHaveScreenshot('button-disabled.png')
  await expect(page.locator('[data-test="button-loading"]')).toHaveScreenshot('button-loading.png')
})
```

### Stabilizing screenshots

Use Playwright's options to reduce false diffs:

```ts
await expect(page).toHaveScreenshot('hero.png', {
  animations: 'disabled',
  caret: 'hide',
  maxDiffPixels: 150,
})
```

If needed, inject CSS to hide dynamic elements before capture.

## Practical limits of automated comparison

There is no mainstream tool that perfectly compares Figma to browser output automatically. The reliable workflow is:

1. Figma screenshot from MCP
2. Browser screenshot from Playwright
3. Human-guided comparison
4. Playwright baselines for future regressions in code

Use automation for regression within the codebase. Use human eyes for Figma-vs-browser truth.

## Punch List Format

```markdown
## Punch List: [Screen/Component Name]

### Status: [Ready for Review / Needs Iteration / Blocked]

### Visual Match Score: [Excellent / Good / Needs Work]

### Visual Deltas
- [ ] [Element]: [Description of difference]. [Severity: minor/moderate/significant]

### Web Deviations (intentional)
- [Element]: [What changed] — [Why: semantics / accessibility / browser reality / technical constraint]

### Uncertainties (needs input)
- [Element]: [Question]. Current implementation: [what I did]. Alternative: [other option].

### Not Yet Implemented
- [Item]: [What's needed] — [Complexity: trivial/moderate/complex]

### Snapshot Test Status
- [ ] Reference screenshots captured for: [list]
- [ ] All snapshots passing: [yes/no]

### Files Created/Modified
- `path/to/file` — [brief description]
```

## Suggested Viewports to Test

At minimum:

| Width | Why |
|------|-----|
| 375px | Small mobile |
| 768px | Tablet / small landscape |
| 1280px | Standard desktop |
| 1440px | Large desktop / common Figma desktop frame |

If the product is app-like, test the exact frame width from Figma too.

## When to Re-Verify

- After any component change that affects the screen
- After updating tokens.css
- After a Figma design update
- After changing fonts or assets
- Before release
