# iOS Native Conventions & Asset Handling

When to follow Figma literally, and when to use iOS-native patterns instead. Plus how to handle icons, images, colors, and fonts.

## The Native-First Rule

When the Figma design shows something that has an iOS-native equivalent, **prefer the native version** unless the design is intentionally custom.

Why: Native components get accessibility, Dynamic Type, dark mode, haptics, and platform conventions for free. Custom implementations must rebuild all of that.

### Decision Framework

Ask these questions in order:

1. **Is there a native SwiftUI component for this?** (TabView, NavigationStack, List, Sheet, etc.)
   - Yes → Use native, style it to match the Figma visual
   - No → Build custom

2. **Is the Figma design intentionally different from the native component?**
   - No, it's just the designer's approximation → Use native
   - Yes, the custom design is the point (brand-specific tab bar, unique interaction) → Build custom

3. **Can the native component be styled to match the design?**
   - Yes → Use native with styling modifiers
   - Close but not exact → Use native, document the visual deviation
   - Not at all → Build custom

### Common Native ↔ Custom Decisions

| Figma Shows | Native Option | Decision |
|---|---|---|
| Bottom tab bar | `TabView` with `.tabItem` | Use native unless tabs have unusual interaction (animated icons, morphing shapes) |
| Navigation bar with back button | `NavigationStack` with `.navigationTitle` | Use native. Style with `.toolbarBackground` and `.tint` |
| Pull-to-refresh | `.refreshable { }` | Always use native |
| Search bar | `.searchable(text:)` | Use native unless the search has a very custom visual |
| List with rows | `List` with custom row views | Use `List` for standard lists. Use `LazyVStack` in `ScrollView` only when `List` styling can't match |
| Action sheet / bottom menu | `.confirmationDialog` | Always use native |
| Alert | `.alert` | Always use native |
| Toggle / switch | `Toggle` | Always use native. Tint with `.tint()` |
| Segmented control | `Picker` with `.pickerStyle(.segmented)` | Use native unless visually very different |
| Slider | `Slider` | Use native. Customize with `.tint()` |
| Date/time picker | `DatePicker` | Always use native |
| Bottom sheet (partial height) | `.sheet` with `.presentationDetents` | Use native. Supports `.medium`, `.large`, custom heights |
| Modal full screen | `.fullScreenCover` | Use native |
| Page indicator / dots | `TabView` with `.tabViewStyle(.page)` | Use native for horizontal paging |
| Progress bar | `ProgressView` | Use native, style with `.tint()` |
| Loading spinner | `ProgressView()` (no value) | Use native |

## SF Symbols

### When to Use SF Symbols

**Always prefer SF Symbols when:**
- The icon has a conceptual match in SF Symbols (there are 6,000+)
- The design uses standard iOS iconography (share, gear, plus, chevron, etc.)
- The icon needs to scale with Dynamic Type
- The icon needs automatic dark mode adaptation

**Use custom icons when:**
- Brand-specific icons (your logo, product icons)
- The icon has no SF Symbol equivalent
- The design explicitly uses a custom icon set (and consistency with that set matters more)

### SF Symbol Selection

Search at: https://developer.apple.com/sf-symbols/

Common mappings:

| Figma Icon Description | SF Symbol Name |
|---|---|
| Home | `house` or `house.fill` |
| Search / magnifying glass | `magnifyingglass` |
| Settings / gear | `gearshape` or `gearshape.fill` |
| Profile / person | `person` or `person.fill` |
| Plus / add | `plus` |
| Close / X | `xmark` |
| Back arrow | `chevron.left` |
| Forward arrow | `chevron.right` |
| Share | `square.and.arrow.up` |
| Bookmark | `bookmark` or `bookmark.fill` |
| Heart / like | `heart` or `heart.fill` |
| Star / favorite | `star` or `star.fill` |
| Delete / trash | `trash` or `trash.fill` |
| Edit / pencil | `pencil` |
| Camera | `camera` or `camera.fill` |
| Photo | `photo` or `photo.fill` |
| Link | `link` |
| Copy | `doc.on.doc` |
| Download | `arrow.down.circle` |
| Upload | `arrow.up.circle` |
| Notification / bell | `bell` or `bell.fill` |
| Menu / hamburger | `line.3.horizontal` |
| More / ellipsis | `ellipsis` |
| Filter | `line.3.horizontal.decrease` |
| Sort | `arrow.up.arrow.down` |
| Calendar | `calendar` |
| Clock / time | `clock` |
| Location / pin | `mappin` or `location` |
| Check / done | `checkmark` |
| Warning | `exclamationmark.triangle` |
| Info | `info.circle` |
| Error / X circle | `xmark.circle` |

### SF Symbol Usage in SwiftUI

```swift
// Basic
Image(systemName: "house.fill")

// With size
Image(systemName: "house.fill")
    .font(.system(size: 24))

// With weight (matches text weight)
Image(systemName: "house.fill")
    .fontWeight(.medium)

// With color
Image(systemName: "house.fill")
    .foregroundStyle(DesignTokens.Colors.primary)

// Multicolor (some symbols support it)
Image(systemName: "chart.bar.fill")
    .symbolRenderingMode(.multicolor)

// Variable value (progress-like)
Image(systemName: "wifi", variableValue: 0.5)
```

### Optical Size Alignment

SF Symbols are optically aligned, not geometrically. When placing an SF Symbol next to text in an HStack, they should align naturally. If using custom icons alongside SF Symbols, custom icons may need `.offset()` adjustment for optical alignment.

## Asset Handling

### Colors

**Preferred: Asset Catalog colors**

Create colors in the Asset Catalog (`Assets.xcassets`) for:
- All design token colors
- Colors that need dark mode variants
- Colors referenced by name in multiple places

```swift
// In DesignTokens.swift — referencing Asset Catalog
enum Colors {
    static let primary = Color("Primary")         // Asset Catalog
    static let surface = Color("Surface")          // Asset Catalog
    static let textPrimary = Color("TextPrimary")  // Asset Catalog
}
```

Asset Catalog advantages:
- Dark mode variants defined visually in Xcode
- High contrast accessibility variants
- Named colors appear in Interface Builder and previews
- Platform-appropriate rendering

**Acceptable: Code-defined colors**

For colors that don't need dark mode adaptation, or for computed colors:

```swift
static let brandGradientStart = Color(hex: "6366F1")
static let brandGradientEnd = Color(hex: "A855F7")
static let overlay = Color.black.opacity(0.4)
```

### Images

**Export requirements from Figma:**
- Raster images: PNG at @2x and @3x (or PDF vector if possible)
- Vector illustrations: PDF (preferred for resolution independence) or SVG
- Photos: JPEG at @2x and @3x

**Asset Catalog organization:**
```
Assets.xcassets/
├── Colors/
│   ├── Primary.colorset/
│   ├── Surface.colorset/
│   └── TextPrimary.colorset/
├── Images/
│   ├── hero-banner.imageset/     (with @2x and @3x)
│   ├── onboarding-1.imageset/
│   └── profile-placeholder.imageset/
└── Icons/
    ├── custom-logo.imageset/     (only for non-SF-Symbol icons)
    └── brand-mark.imageset/
```

**Image export is currently a manual step.** The Figma MCP cannot export assets as files. The handoff protocol is:

1. I identify all images/icons needed during implementation
2. I produce an **Asset Export List** with exact names and specs
3. User (or Cristina) exports from Figma and adds to Asset Catalog
4. I reference them by name in code

**Asset Export List format:**
```
## Asset Export List: [Screen Name]

### Images (raster)
| Name | Format | Sizes | Location in Figma |
|------|--------|-------|-------------------|
| hero-banner | PNG | @2x, @3x | Home > Header > Background |
| avatar-placeholder | PNG | @2x, @3x | Profile > Avatar > Default |

### Icons (custom, no SF Symbol match)
| Name | Format | Size | Location in Figma |
|------|--------|------|-------------------|
| brand-logo | PDF | 24pt | Global > Icons > Logo |

### SF Symbols Used (no export needed)
| Usage | Symbol Name | Configuration |
|-------|-------------|---------------|
| Tab: Home | house.fill | .font(.system(size: 24)) |
| Tab: Search | magnifyingglass | .font(.system(size: 24)) |
```

### Fonts

**System font (SF Pro):** No asset needed. Use `.font(.system(size:weight:design:))` or system text styles (`.body`, `.headline`, etc.)

**Custom fonts:**
1. Add .ttf or .otf files to the Xcode project
2. Register in Info.plist under "Fonts provided by application"
3. Reference: `.font(.custom("FontName-Weight", size: S))`

**Font registration checklist:**
- [ ] Font files added to project target
- [ ] Info.plist updated with font filenames
- [ ] Font names verified (actual PostScript name, not filename)
- [ ] All weights used in design are included (Regular, Medium, Semibold, Bold, etc.)
- [ ] `@ScaledMetric` or `UIFontMetrics` used for Dynamic Type support

## Platform Conventions

### Safe Areas

Always respect safe areas unless the design explicitly bleeds to the edge:

```swift
// Respects safe areas (default)
VStack { content }

// Ignores safe areas (for background images, gradients)
VStack { content }
    .ignoresSafeArea()

// Ignores only specific edges
VStack { content }
    .ignoresSafeArea(edges: .top)
```

**Common pattern:** Background ignores safe area, content respects it:
```swift
ZStack {
    Color.background.ignoresSafeArea()
    VStack { /* content respects safe area by default */ }
}
```

### Dynamic Type

All text should scale with Dynamic Type. This means:
- Use system text styles (`.body`, `.headline`) when possible — they scale automatically
- For custom fonts, use `@ScaledMetric` for associated spacing/sizing:

```swift
@ScaledMetric(relativeTo: .body) var iconSize: CGFloat = 24
@ScaledMetric(relativeTo: .body) var cardPadding: CGFloat = 16
```

- Test at largest accessibility size to ensure layout doesn't break
- Use `.dynamicTypeSize(.large ... .accessibility3)` to cap scaling if layout breaks at extreme sizes

### Dark Mode

Checklist:
- [ ] All colors from Asset Catalog with dark mode variants
- [ ] No hardcoded white/black for text (use semantic colors)
- [ ] Shadows visible in both modes (may need different opacity)
- [ ] Images have appropriate contrast in both modes
- [ ] Dividers/borders visible in both modes
- [ ] Status bar content adapts

### Haptics

Figma can't specify haptics, but iOS users expect them:
- Button taps: `.sensoryFeedback(.impact(weight: .light), trigger: value)`
- Toggle changes: `.sensoryFeedback(.selection, trigger: value)`
- Destructive actions: `.sensoryFeedback(.warning, trigger: value)`
- Success: `.sensoryFeedback(.success, trigger: value)`

Add haptics after the visual implementation is verified. They're polish, not structure.

### Minimum Tap Targets

iOS Human Interface Guidelines: 44pt minimum for tap targets.

If the Figma design shows a 24pt icon as a button, the visual should be 24pt but the tap target should be 44pt:

```swift
Image(systemName: "xmark")
    .font(.system(size: 16))
    .frame(width: 44, height: 44)  // Tap target
    .contentShape(Rectangle())      // Ensures full frame is tappable
```
