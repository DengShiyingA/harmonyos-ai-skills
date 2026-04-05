---
name: adopting-liquid-glass
description: Comprehensive guide to adopting Apple's Liquid Glass material in apps built with SwiftUI, UIKit, and AppKit for iOS 26, iPadOS 26, macOS Tahoe 26, watchOS 26, and tvOS 26. Use this skill when the user is updating an Apple-platform app for the new Liquid Glass design language, asks about `glassEffect`, `GlassEffectContainer`, `UIGlassEffect`, `NSGlassEffectView`, `.buttonStyle(.glass)`, `UIButton.Configuration.glass()`, scroll edge effects, background extension effect, tab bar minimize behavior, concentric corners, search tabs, or app icon layering (Icon Composer). Also use when auditing custom backgrounds, toolbars, sidebars, sheets, or when configuring `UIDesignRequiresCompatibility` for backward compatibility.
---

# Adopting Liquid Glass

Liquid Glass is Apple's dynamic material that blends the optical properties of glass with fluid motion. It forms a distinct functional layer for controls and navigation that adapts to surrounding content, focus state, and accessibility settings. Source: https://developer.apple.com/documentation/technologyoverviews/adopting-liquid-glass

## When to apply this skill

- Modernizing an iOS / iPadOS / macOS / watchOS / tvOS app against the latest SDKs
- Fixing custom toolbars, tab bars, navigation bars, split views, sidebars, or sheets that look wrong after rebuilding
- Adding a glass look to custom controls (buttons, pills, floating actions)
- Building morphing/animating glass UI with `GlassEffectContainer`, `glassEffectID`, `GlassEffectTransition`
- Designing layered app icons with Icon Composer
- Opting out temporarily via `UIDesignRequiresCompatibility`

## Core adoption workflow (do this first)

When rebuilding an existing app against iOS 26 / macOS Tahoe SDKs, follow this order:

1. **Rebuild with latest Xcode.** Standard components adopt Liquid Glass automatically — no code changes required for `NavigationStack`, `NavigationSplitView`, `TabView`, `.toolbar`, `UINavigationBar`, `UITabBar`, `UIToolbar`, `UISplitViewController`, `NSToolbar`, `NSSplitView`.
2. **Audit and remove custom backgrounds** on bars, split views, tab bars, toolbars. Custom `backgroundColor`, `UIVisualEffectView`, or `.background(...)` modifiers placed on these containers will fight the system material. Remove them and let the system render the glass.
3. **Don't hard-code layout metrics** for controls. Control sizes/shapes changed (rounder forms, extra-large option). Let Auto Layout / SwiftUI layout drive sizing.
4. **Review section headers.** Lists now use title-style capitalization — don't upper-case them manually.
5. **Test with Reduce Transparency and Reduce Motion enabled.** The system adapts automatically for standard components; verify custom glass views respect these settings.
6. **Apply custom glass sparingly.** Overusing `glassEffect` distracts from content. Reserve it for the most important floating / functional elements.

## SwiftUI API reference

### Glass effect on custom views

```swift
// Basic - default Glass.regular in a Capsule
Text("Hello")
    .padding()
    .glassEffect()

// Custom shape
Text("Hello")
    .padding()
    .glassEffect(in: .rect(cornerRadius: 16))

// Tinted for prominence
.glassEffect(.regular.tint(.orange))

// Interactive (responds to touch/pointer like a button)
.glassEffect(.regular.tint(.orange).interactive())
```

Signature: `func glassEffect(_ glass: Glass = .regular, in shape: some Shape = Capsule()) -> some View`

**Critical:** apply `.glassEffect(...)` AFTER any modifiers that change appearance — the modifier captures rendered content.

### Button styles

```swift
Button("Action") { }.buttonStyle(.glass)
Button("Primary") { }.buttonStyle(.glassProminent)
```

Available as `PrimitiveButtonStyle.glass`, `.glassProminent`, and `.glass(_:)` with a configurable `Glass` value.

### GlassEffectContainer — combining & morphing

Combines multiple glass shapes into a single rendered shape for performance AND enables morphing between them.

```swift
GlassEffectContainer(spacing: 40.0) {
    HStack(spacing: 40) {
        Image(systemName: "scribble.variable")
            .frame(width: 80, height: 80)
            .glassEffect()
        Image(systemName: "eraser.fill")
            .frame(width: 80, height: 80)
            .glassEffect()
    }
}
```

- `spacing` controls how effects interact. When container spacing > interior layout spacing, nearby glass shapes blend/merge at rest and morph during animation.

### Morphing with glassEffectID + namespace

```swift
@State private var isExpanded = false
@Namespace private var namespace

GlassEffectContainer(spacing: 40) {
    HStack(spacing: 40) {
        Image(systemName: "scribble.variable")
            .frame(width: 80, height: 80)
            .glassEffect()
            .glassEffectID("pencil", in: namespace)

        if isExpanded {
            Image(systemName: "eraser.fill")
                .frame(width: 80, height: 80)
                .glassEffect()
                .glassEffectID("eraser", in: namespace)
        }
    }
}
Button("Toggle") { withAnimation { isExpanded.toggle() } }
    .buttonStyle(.glass)
```

Signatures:
- `func glassEffectID(_ id: (some Hashable & Sendable)?, in namespace: Namespace.ID) -> some View`
- `func glassEffectUnion(id: (some Hashable & Sendable)?, namespace: Namespace.ID) -> some View` — unions geometries at rest (useful for dynamically generated views).

### GlassEffectTransition

- `.matchedGeometry` (default) — morph between shapes within container spacing.
- `.materialize` — fade content + materialize glass without geometry matching. Use for effects farther apart than container spacing.

### Scroll edge effect

System bars get this automatically. For custom floating bars overlaying a scroll view:

```swift
ScrollView { /* ... */ }
    .safeAreaBar(edge: .top) {
        MyCustomBar()
    }
// or customise with:
.scrollEdgeEffectStyle(.soft, for: .top)
```

### Concentric shapes

```swift
RoundedRectangle(cornerRadius: 16, style: .continuous)    // or
ConcentricRectangle()                                     // aligns to parent corners
.rect(corners: .init(...), isUniform: false)              // per-corner radii
```

Use these so nested controls visually nest inside rounder containers.

### Background extension (edge-to-edge hero content under sidebars)

```swift
Image("hero")
    .backgroundExtensionEffect()
```

Creates the impression content extends under sidebars/inspectors without moving the actual content.

### Tab bar behavior

```swift
TabView(selection: $selection) {
    Tab("Home", systemImage: "house", value: .home) { HomeView() }
    Tab(role: .search) { SearchView() }     // semantic search tab, placed trailing
}
.tabViewStyle(.sidebarAdaptable)
.tabBarMinimizeBehavior(.onScrollDown)
```

## UIKit API reference

### Glass effects

```swift
// Applied to a UIView
let glassEffect = UIGlassEffect()
// Merge/contain multiple effects for morphing + perf
let container = UIGlassContainerEffect()
```

### Button configurations

```swift
var config = UIButton.Configuration.glass()           // or
UIButton.Configuration.prominentGlass()
UIButton.Configuration.clearGlass()
UIButton.Configuration.prominentClearGlass()
let button = UIButton(configuration: config, primaryAction: action)
```

### Scroll edge element container

For custom bars overlaying scroll content — descendants (labels, images, glass views, controls) automatically shape the scroll edge effect.

```swift
let interaction = UIScrollEdgeElementContainerInteraction()
customBar.addInteraction(interaction)
```

### Background extension

```swift
let bgExt = UIBackgroundExtensionView()
bgExt.contentView = heroImageView
```

### Popovers & action sheets

Set `sourceView` / `sourceItem` so action sheets originate from the initiating control:

```swift
controller.modalPresentationStyle = .popover
controller.popoverPresentationController?.sourceItem = senderButton
```

### Corner configuration

```swift
view.cornerConfiguration = UICornerConfiguration.uniformCorners(radius: 16)
```

### Toolbar items

- Use `UIBarButtonItem.isHidden` to hide an entire toolbar item (not the view inside).
- Use `UIBarButtonItem.fixedSpace(_:)` between logical groups so items within a group share a glass background, and groups are visually separated.

## AppKit API reference

```swift
NSButton.BezelStyle.glass              // button bezel

let glassView = NSGlassEffectView()    // wrap custom content in glass
glassView.contentView = myView

let bgExt = NSBackgroundExtensionView()
bgExt.contentView = heroImageView

// Split view with inspector
let inspector = NSSplitViewItem(inspectorWithViewController: inspectorVC)
splitViewController.addSplitViewItem(inspector)

// Toolbar items
NSToolbarItem.Identifier.space         // fixed spacer
item.isHidden = true                   // hide whole item
view.cornerConfiguration = ...         // concentric corners
```

## Toolbar grouping — the rule

Related actions go in one group (they share a single glass background); unrelated actions go in separate groups with a fixed spacer between. Don't mix text and icon buttons in the same group.

```swift
.toolbar {
    ToolbarItemGroup(placement: .primaryAction) {
        Button { } label: { Image(systemName: "arrow.uturn.backward") }
        Button { } label: { Image(systemName: "arrow.uturn.forward") }
    }
    ToolbarSpacer()
    ToolbarItemGroup(placement: .primaryAction) {
        Button { } label: { Image(systemName: "pencil.tip.crop.circle") }
        Button { } label: { Image(systemName: "ellipsis") }
    }
}
```

Always provide `accessibilityLabel` for icon-only buttons.

## Sheets, popovers, action sheets

- Sheets get increased corner radius and are inset; half sheets allow content peek-through and become more opaque when fully expanded. Audit content that sits near corners.
- Popovers adopt glass — **remove custom background views** inside them.
- Action sheets originate from the initiating element and allow interaction with the rest of the UI. Always set the source view/item.

## Platform-specific notes

| Platform | Notes |
|---|---|
| **iOS / iPadOS** | Full Liquid Glass on floating tab bars, adaptive sidebars, morphing sheets, action sheets from origin, continuous iPad window resizing. |
| **macOS Tahoe** | Toolbars, split views, title bars updated. Use `.buttonStyle(.glass)` / `NSButton.BezelStyle.glass`. |
| **watchOS** | Changes minimal and automatic. Adopt standard button styles and toolbar APIs. |
| **tvOS** | Standard controls show Liquid Glass **on focus**. Custom focusable controls should adopt the effect on focus. Apple TV 4K (2nd gen)+ required for effects. |

## Backward compatibility

To build against the latest SDKs while temporarily keeping the previous look:

```xml
<!-- Info.plist -->
<key>UIDesignRequiresCompatibility</key>
<true/>
```

This is a transitional opt-out — adopt Liquid Glass in a follow-up milestone.

## App icons (Icon Composer)

- Design in layered form (foreground / middle / background) with solid, filled, semi-transparent overlapping shapes.
- **Don't bake in** reflections, refraction, shadows, blur, highlights — the system applies them.
- Use Icon Composer (in Xcode 26, or via Apple Design Resources) to assemble layers, adjust opacity, and preview Default / Dark / Clear / Tinted appearances for both light and dark.
- Keep elements centered — system masks to rounded rectangle (iOS/iPadOS/macOS) or circle (watchOS).

## Performance checklist

- Limit simultaneous `GlassEffectContainer`s onscreen.
- Prefer containing multiple glass shapes in one container over many independent glass views.
- Apply `.glassEffect(...)` after content-shaping modifiers.
- Profile with Instruments (Animation Hitches, SwiftUI). See WWDC25 "Optimize SwiftUI performance with Instruments."

## Audit checklist (use this when updating an existing app)

- [ ] Removed custom background colors / visual effects from navigation bars, tab bars, toolbars, split views
- [ ] No hard-coded control sizes or corner radii that fight new metrics
- [ ] Section headers use title case (not uppercased strings)
- [ ] Sheet / popover content isn't clipped by new corner radii
- [ ] Action sheets have `sourceView`/`sourceItem` set
- [ ] Custom floating bars use `safeAreaBar` or `UIScrollEdgeElementContainerInteraction`
- [ ] Toolbar items grouped by function with fixed spacers between groups
- [ ] Icon-only buttons have accessibility labels
- [ ] Tested with Reduce Transparency and Reduce Motion
- [ ] Tested on oldest supported device in each platform family
- [ ] App icon re-authored in Icon Composer with layered design
- [ ] Decided on `UIDesignRequiresCompatibility` (usually: do not set)

## Related Apple documentation

- Adopting Liquid Glass: https://developer.apple.com/documentation/technologyoverviews/adopting-liquid-glass
- Applying Liquid Glass to custom views (SwiftUI): search `applying-liquid-glass-to-custom-views` in the SwiftUI documentation
- Human Interface Guidelines — Materials
- WWDC25: "Meet Liquid Glass", "Build a SwiftUI app with the new design", "Get to know the new design system"
