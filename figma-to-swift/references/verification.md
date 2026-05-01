# Visual Verification & Snapshot Testing

How to verify that SwiftUI implementation matches the Figma design. The goal is 1:1 — not "looks about right."

## Verification Loop

After every implementation pass:

```
1. Build on simulator (XcodeBuildMCP: build_run_sim)
2. Take simulator screenshot to file (XcodeBuildMCP: screenshot → save to disk)
3. Pull Figma screenshot to file (Figma MCP: get_screenshot → save to disk)
4. Compare side by side (load both into context only for active comparison)
5. Identify deltas
6. Fix deltas
7. Repeat until matched
```

**Context window note:** Always save screenshots to disk first, then load for comparison. Don't keep old comparison images in context — once you've identified the deltas and made fixes, the previous screenshots are no longer needed. In sub-agent workflows, all comparison images stay within the sub-agent and never reach the parent context.

## What to Compare

### Tier 1: Must Match Exactly
These should be pixel-identical. Any mismatch is a bug.

- **Colors** — exact hex values, including opacity
- **Typography** — font, size, weight (line height and tracking are Tier 2)
- **Corner radius** — exact values, and `.continuous` style
- **Icon selection** — correct SF Symbol or custom icon
- **Content** — correct text, labels, placeholder text
- **Visual hierarchy** — what's prominent, what's secondary, what's tertiary

### Tier 2: Must Match Closely
These should be within 1-2pt. Small deviations may be acceptable if iOS native behavior causes them.

- **Spacing** — padding, gaps, margins (within 1pt)
- **Line height** — SwiftUI's text rendering differs slightly from Figma
- **Letter spacing** — tracking values may render slightly differently
- **Shadow** — blur and offset should be close, exact pixel match is difficult
- **Alignment** — leading/trailing/center should be exact, but baseline alignment of mixed text sizes may differ slightly

### Tier 3: Acceptable Differences
These are expected to differ between Figma and iOS. Document them, don't fight them.

- **System chrome** — status bar, navigation bar native styling, tab bar translucency
- **Font rendering** — subpixel rendering differences between Figma and iOS
- **Scroll indicators** — iOS adds its own, Figma may not show them
- **Safe area insets** — differ by device, Figma may show one size
- **Material/blur effects** — iOS materials look different from Figma blur approximations
- **Dynamic Type scaling** — Figma shows fixed size, iOS may scale

## Manual Comparison Technique

When comparing screenshots:

1. **Overall scan** — step back, squint. Does the overall shape and weight feel the same? If it looks wrong from 5 feet away, there's a structural issue.

2. **Section-by-section** — compare header, then content area, then footer. Isolate where differences are.

3. **Vertical rhythm** — are the vertical spacings between elements consistent with the design? This is the #1 source of "looks off but I can't tell why."

4. **Horizontal alignment** — are leading edges aligned? Are elements that should be left-aligned actually left-aligned? Are centered elements truly centered?

5. **Typography** — look at text color, size, and weight independently. Wrong weight is the most common text issue.

6. **Fine details** — dividers, borders, shadows, badges, indicators. These are often missed or subtly wrong.

## Snapshot Testing Setup

### Option 1: Point-Free SnapshotTesting (Recommended)

Add to your Swift package:
```swift
// Package.swift
dependencies: [
    .package(url: "https://github.com/pointfreeco/swift-snapshot-testing", from: "1.19.0"),
]
```

Write snapshot tests:
```swift
import SnapshotTesting
import SwiftUI
import XCTest

final class HomeScreenSnapshotTests: XCTestCase {
    func testHomeScreen_iPhone16Pro() {
        let view = HomeScreen()
        let vc = UIHostingController(rootView: view)
        vc.view.frame = UIScreen.main.bounds
        assertSnapshot(of: vc, as: .image)
    }
    
    func testHomeScreen_iPhoneSE() {
        let view = HomeScreen()
        let vc = UIHostingController(rootView: view)
        vc.view.frame = CGRect(x: 0, y: 0, width: 375, height: 667)
        assertSnapshot(of: vc, as: .image)
    }
    
    func testHomeScreen_darkMode() {
        let view = HomeScreen()
            .preferredColorScheme(.dark)
        let vc = UIHostingController(rootView: view)
        vc.view.frame = UIScreen.main.bounds
        assertSnapshot(of: vc, as: .image)
    }
    
    func testPrimaryButton_allStates() {
        let states: [(String, PrimaryButton)] = [
            ("default", PrimaryButton(title: "Continue", action: {})),
            ("disabled", PrimaryButton(title: "Continue", isDisabled: true, action: {})),
            ("loading", PrimaryButton(title: "Continue", isLoading: true, action: {})),
        ]
        for (name, button) in states {
            assertSnapshot(
                of: UIHostingController(rootView: button.padding()),
                as: .image,
                named: name
            )
        }
    }
}
```

First run records reference images. Subsequent runs compare against references. Any pixel difference triggers a test failure with a diff image.

**Use `record: true` to update baselines:**
```swift
assertSnapshot(of: view, as: .image, record: true)
```

### Option 2: Prefire (Auto-generates from Previews)

Prefire automatically generates snapshot tests from your `#Preview` declarations. Less manual test writing, more reliance on comprehensive previews.

Add via SPM:
```swift
.package(url: "https://github.com/BarredEwe/Prefire", from: "2.0.0")
```

If you write good `#Preview` blocks (which you should anyway for development), Prefire turns them into snapshot tests automatically. This is powerful because:
- No separate test code to maintain
- Every preview state becomes a visual regression test
- Previews serve double duty as development aids AND test fixtures

### Recommended Approach

Use **both**:
- **Point-Free SnapshotTesting** for explicit, curated tests of key screens at specific device sizes and configurations
- **Prefire** for broad coverage from previews — catches regressions in components you might not have written explicit tests for

## Punch List Format

After verification, produce this structured output. This is the handoff document — it tells the user exactly what's done, what's different, and what needs their input.

```markdown
## Punch List: [Screen/Component Name]

### Status: [Ready for Review / Needs Iteration / Blocked]

### Visual Match Score: [Excellent / Good / Needs Work]
Brief assessment of overall fidelity.

### Visual Deltas (from Figma reference)
Items where the implementation differs from the Figma design:
- [ ] [Element]: [Description of difference]. [Severity: minor/moderate/significant]

### iOS Deviations (intentional)
Items where we intentionally diverged from Figma for iOS-native behavior:
- [Element]: [What changed] — [Why: iOS convention / accessibility / technical constraint]

### Uncertainties (needs designer/user input)
Items where the Figma design was ambiguous and I made a best guess:
- [Element]: [Question]. Current implementation: [what I did]. Alternative: [other option].

### Not Yet Implemented
Interactions, animations, or states that were implied but not yet built:
- [Item]: [What's needed] — [Complexity estimate: trivial/moderate/complex]

### Snapshot Test Status
- [ ] Reference snapshots captured for: [list devices/configurations]
- [ ] All snapshots passing: [yes/no, list failures]

### Files Created/Modified
- `path/to/File.swift` — [brief description]
```

## Visual Regression Workflow

Once snapshot testing is set up, the ongoing workflow is:

1. **Before making changes**: Run snapshot tests — all should pass (baseline is clean)
2. **Make changes**: Implement new screen or modify existing
3. **Run snapshot tests**: 
   - New components: record new baselines with `record: true`
   - Modified components: review diffs. Intentional visual changes → update baseline. Unintentional → fix code.
4. **On Figma design update**: 
   - Pull new Figma screenshots
   - Compare against current snapshots
   - Identify what changed in the design
   - Update SwiftUI code
   - Update snapshot baselines
   - Verify all other snapshots still pass (no regressions)

## Device Configurations to Test

At minimum, capture snapshots for:

| Device | Why |
|--------|-----|
| iPhone 16 Pro | Primary target, most common |
| iPhone SE (3rd gen) | Smallest screen, catches layout overflow |
| iPhone 16 Pro Max | Largest screen, catches underfill/centering |
| Dark mode (any device) | Catches invisible elements, contrast issues |

If the app supports iPad:
| iPad Pro 11" | Standard tablet layout |
| iPad mini | Smallest tablet |

## When to Re-Verify

- After any code change to a component used in the screen
- After updating DesignTokens.swift
- After a Figma design update
- After iOS SDK updates (rendering may change subtly)
- Before any release
