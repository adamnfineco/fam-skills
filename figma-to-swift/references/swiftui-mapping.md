# Figma → SwiftUI Translation Dictionary

This is the core mapping reference. When translating Figma designs to SwiftUI, use these rules. They handle the non-obvious translations that cause "close but not right" output.

## Layout System

### Auto Layout → Stacks

| Figma Auto Layout | SwiftUI | Notes |
|---|---|---|
| Vertical, top-to-bottom | `VStack(alignment:spacing:)` | Default alignment is `.center` in SwiftUI but `.leading` in most designs — check! |
| Horizontal, left-to-right | `HStack(alignment:spacing:)` | Default alignment is `.center` |
| Wrap (auto layout wrap) | `LazyVGrid` or custom `FlowLayout` | No native wrap in SwiftUI stacks |
| Spacing between items | `spacing:` parameter on Stack | Figma "Auto" spacing ≈ `.infinity` spacer between items |
| Padding (all sides equal) | `.padding(value)` | |
| Padding (per-side) | `.padding(.leading, L).padding(.trailing, T)` etc. | Or `.padding(EdgeInsets(top:leading:bottom:trailing:))` |
| Gap between children | `spacing:` on the Stack | Not `.padding` — spacing is between siblings, padding is inside the container |

### Figma "Space Between" Modes

| Figma Setting | SwiftUI Equivalent |
|---|---|
| "Packed" (default) | Normal Stack with `spacing:` |
| "Space between" | Stack with `Spacer()` between items, or items with `frame(maxWidth: .infinity)` |
| "Space around" | No direct equivalent — use padding on each item |

### Sizing & Constraints

| Figma Sizing | SwiftUI |
|---|---|
| Fixed width/height | `.frame(width: W, height: H)` |
| Hug contents | Default behavior — don't set frame |
| Fill container (horizontal) | `.frame(maxWidth: .infinity)` |
| Fill container (vertical) | `.frame(maxHeight: .infinity)` |
| Fill container (both) | `.frame(maxWidth: .infinity, maxHeight: .infinity)` |
| Min width | `.frame(minWidth: W)` |
| Max width | `.frame(maxWidth: W)` |
| Aspect ratio constraint | `.aspectRatio(W/H, contentMode: .fit)` |

### Alignment

| Figma Alignment | SwiftUI |
|---|---|
| Top-left | `VStack(alignment: .leading)` + no vertical spacer |
| Top-center | `VStack(alignment: .center)` + no vertical spacer |
| Top-right | `VStack(alignment: .trailing)` + no vertical spacer |
| Center-left | `VStack(alignment: .leading)` + `Spacer()` top and bottom |
| Center | `.frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .center)` |
| Bottom-right | `VStack(alignment: .trailing)` + `Spacer()` at top |
| Absolute positioned overlay | `ZStack(alignment:)` or `.overlay(alignment:)` |

### Stacking & Overlap

| Figma Pattern | SwiftUI |
|---|---|
| Layers stacked (z-order) | `ZStack` — last child is on top (opposite of Figma where top layer in panel = top visually) |
| Element overlapping another | `.overlay { }` or `.background { }` depending on z-order |
| Absolute positioned badge/indicator | `.overlay(alignment: .topTrailing) { Badge() }` |
| Negative spacing (overlapping items) | `ZStack` with manual `.offset()` or negative padding |

## Visual Properties

### Colors

| Figma | SwiftUI |
|---|---|
| Solid fill #RRGGBB | `Color(hex: "RRGGBB")` or Asset Catalog color |
| Solid fill with opacity | `Color(hex: "RRGGBB").opacity(0.5)` |
| Linear gradient | `LinearGradient(colors: stops: startPoint: endPoint:)` |
| Radial gradient | `RadialGradient(colors: center: startRadius: endRadius:)` |
| Angular gradient | `AngularGradient(colors: center: startAngle: endAngle:)` |
| Multiple fills (layered) | `ZStack` with fills, or `.background { gradient }.overlay { secondFill }` |
| Design token color | **Always** use `DesignTokens.Colors.xxx` — never raw hex |

**Color hex extension** — add this to the project:
```swift
extension Color {
    init(hex: String) {
        let hex = hex.trimmingCharacters(in: CharacterSet.alphanumerics.inverted)
        var int: UInt64 = 0
        Scanner(string: hex).scanHexInt64(&int)
        let a, r, g, b: UInt64
        switch hex.count {
        case 6: (a, r, g, b) = (255, int >> 16, int >> 8 & 0xFF, int & 0xFF)
        case 8: (a, r, g, b) = (int >> 24, int >> 16 & 0xFF, int >> 8 & 0xFF, int & 0xFF)
        default: (a, r, g, b) = (255, 0, 0, 0)
        }
        self.init(.sRGB, red: Double(r) / 255, green: Double(g) / 255, blue: Double(b) / 255, opacity: Double(a) / 255)
    }
}
```

### Corner Radius

| Figma | SwiftUI |
|---|---|
| All corners equal | `.clipShape(RoundedRectangle(cornerRadius: R))` |
| Per-corner radius | `.clipShape(UnevenRoundedRectangle(topLeadingRadius: cornerRadii:))` (iOS 17+) |
| Fully rounded (pill) | `.clipShape(Capsule())` |
| Circle | `.clipShape(Circle())` |
| Smoothing (iOS-style squircle) | `.clipShape(RoundedRectangle(cornerRadius: R, style: .continuous))` — **always use `.continuous` for iOS-native feel** |

**Important:** Figma's default corner rounding is NOT the same as iOS's. iOS uses `.continuous` (superellipse/squircle). Always use `style: .continuous` for RoundedRectangle to match the native iOS feel. This is a common miss that makes implementations look subtly wrong.

### Shadows & Effects

| Figma | SwiftUI |
|---|---|
| Drop shadow | `.shadow(color: C.opacity(O), radius: R, x: X, y: Y)` |
| Inner shadow | No native SwiftUI modifier — use `.overlay` with a shadow shape or custom `InnerShadow` |
| Layer blur | `.blur(radius: R)` |
| Background blur (glass) | `.background(.ultraThinMaterial)` or `.background(.regularMaterial)` — prefer native materials over manual blur |
| Multiple shadows | Chain `.shadow()` modifiers — they stack |

### Borders & Strokes

| Figma | SwiftUI |
|---|---|
| Stroke (outside) | `.overlay(RoundedRectangle(cornerRadius:).stroke(color, lineWidth:))` |
| Stroke (inside) | `.overlay(RoundedRectangle(cornerRadius:).strokeBorder(color, lineWidth:))` |
| Stroke (center) | `.overlay(RoundedRectangle(cornerRadius:).stroke(color, lineWidth:))` — but radius needs adjustment |
| Dashed stroke | `.stroke(style: StrokeStyle(lineWidth:dash:[on,off]))` |
| Border on one side only | Custom `Rectangle().frame(height: 1)` as divider, or `.overlay(alignment: .bottom) { Divider() }` |

## Typography

### Text Styles

| Figma Property | SwiftUI |
|---|---|
| Font family | `.font(.custom("FontName", size: S))` or `.font(.system(size:weight:design:))` |
| Font weight | `.fontWeight(.regular/.medium/.semibold/.bold/.heavy)` |
| Font size | Part of `.font()` call |
| Line height | `.lineSpacing(lineHeight - fontSize)` — SwiftUI lineSpacing is additive, not absolute |
| Letter spacing (tracking) | `.tracking(value)` (in points, not em) |
| Text case (uppercase) | `.textCase(.uppercase)` |
| Text alignment | `.multilineTextAlignment(.leading/.center/.trailing)` |
| Max lines | `.lineLimit(N)` |
| Truncation | `.truncationMode(.tail/.middle/.head)` |
| Text color | `.foregroundStyle(Color.xxx)` — prefer `.foregroundStyle` over deprecated `.foregroundColor` |

### Figma Text ↔ SwiftUI System Fonts

If the design uses SF Pro (system font), map to system text styles:

| Figma Size/Weight | SwiftUI System Style |
|---|---|
| 34pt Bold | `.largeTitle` |
| 28pt Bold | `.title` |
| 22pt Bold | `.title2` |
| 20pt Semibold | `.title3` |
| 17pt Semibold | `.headline` |
| 17pt Regular | `.body` |
| 16pt Regular | `.callout` |
| 15pt Regular | `.subheadline` |
| 13pt Regular | `.footnote` |
| 12pt Regular | `.caption` |
| 11pt Regular | `.caption2` |

**Use system styles when possible** — they automatically support Dynamic Type. Custom `.font(.custom())` does not scale with Dynamic Type unless you use `@ScaledMetric` or `UIFontMetrics`.

### Line Height Calculation

Figma specifies absolute line height (e.g., 24px for 16px text).
SwiftUI `.lineSpacing()` is additive (extra space between lines).

```
swiftUI_lineSpacing = figma_lineHeight - figma_fontSize
```

Example: Figma line height 24, font size 16 → `.lineSpacing(8)`

For single-line text, `.lineSpacing` has no visible effect. Only set it for multi-line text.

**Caveat:** This formula is an approximation. SwiftUI's actual line spacing depends on font metrics (ascender, descender, leading) which vary by font. For system fonts (SF Pro) at standard sizes, this formula is close enough. For custom fonts, you may need to fine-tune by visual comparison. If the result looks wrong, try adjusting ±1-2pt.

## Images & Media

| Figma | SwiftUI |
|---|---|
| Image fill (cover) | `Image("name").resizable().aspectRatio(contentMode: .fill).clipped()` |
| Image fill (contain/fit) | `Image("name").resizable().aspectRatio(contentMode: .fit)` |
| Image with rounded corners | Chain `.clipShape(RoundedRectangle(cornerRadius:style:.continuous))` |
| Remote image | `AsyncImage(url:)` with placeholder |
| Image tint/overlay | `.overlay(Color.xxx.opacity(O))` on the image |

## Scroll & Lists

| Figma Pattern | SwiftUI |
|---|---|
| Vertical scrolling content | `ScrollView(.vertical) { VStack { ... } }` |
| Horizontal scrolling row | `ScrollView(.horizontal, showsIndicators: false) { HStack { ... } }` |
| Long list of identical items | `List` or `LazyVStack` — prefer `List` for iOS-native feel, `LazyVStack` for custom styling |
| Grid of items | `LazyVGrid(columns:spacing:)` |
| Pull to refresh | `.refreshable { }` |
| Sticky header | `Section(header:)` inside `List`, or `pinnedViews: [.sectionHeaders]` in `LazyVStack` |

## Navigation & Structure

| Figma Pattern | SwiftUI |
|---|---|
| Tab bar at bottom | `TabView` with `.tabItem { }` |
| Navigation with back button | `NavigationStack` + `NavigationLink` or `.navigationDestination` |
| Modal sheet | `.sheet(isPresented:) { }` |
| Full-screen modal | `.fullScreenCover(isPresented:) { }` |
| Bottom sheet (partial) | `.sheet` with `.presentationDetents([.medium, .large])` |
| Alert dialog | `.alert(title:isPresented:actions:message:)` |
| Action sheet | `.confirmationDialog(title:isPresented:actions:)` |
| Search bar | `.searchable(text:)` on NavigationStack |

## Common Gotchas

1. **Figma's coordinate system is top-left origin.** SwiftUI's is center-based for many modifiers. Don't translate Figma x/y directly — use alignment and spacing.

2. **Figma layers order: top of panel = front.** ZStack order: last in code = front. They're reversed.

3. **Figma shows pixel values at 1x.** iOS renders at 2x or 3x. A 1px Figma line should be `1` point in SwiftUI (which renders as 2-3 physical pixels). Don't divide by scale factor.

4. **Figma opacity on a group affects all children.** In SwiftUI, `.opacity()` on a container does the same — but if you want individual child opacity, apply it per-child.

5. **Figma "constraints" (pin to edges) are not the same as SwiftUI alignment.** Don't try to translate constraints 1:1. Think about what layout behavior they're achieving and express that in SwiftUI's layout system.

6. **Figma's auto layout padding is inside the container.** SwiftUI's `.padding()` is also inside. They match directly.

7. **Figma strokes are outside by default.** SwiftUI `.stroke()` is centered. Use `.strokeBorder()` for inside stroke, and remember to use `.overlay` for outside stroke since SwiftUI doesn't have a native "outside stroke."

8. **Text baseline alignment differs.** Figma positions text from the top of the text box. SwiftUI positions from the baseline. For pixel-perfect text positioning, you may need to adjust with `.alignmentGuide(.firstTextBaseline)`.
