AudioWorkstation Design

overview
The design follows Apple’s pro app patterns as outlined in WWDC25 Human Interface Guidelines. The primary goals are:
1. A macOS-first professional workstation interface.
2. Consistent adaptive layouts across macOS, iPadOS, iOS, and tvOS.
3. Use of liquid glass aesthetics (glassmorphic design) with ultraThinMaterial backgrounds.
4. Optimized focus interaction for Apple TV.
5. Strict adherence to SwiftUI 6.2+ adaptive layout recommendations.

general principles
- All panels and toolbars are layered on semi-translucent glass surfaces with blurred backgrounds.
- UI elements scale proportionally across platforms using size classes and SwiftUI responsive layout APIs.
- Toolbar items use only SF Symbols or standardized pro app iconography.
- Adaptive typography: macOS uses standard system font sizes, iOS and iPadOS scale proportionally based on dynamic type, tvOS uses focus-optimized larger text.
- Avoid custom gestures that conflict with platform defaults (tvOS swipe vs iPadOS drag).

platform differences

macOS
- primary development platform.
- main layout uses NavigationSplitView for sidebar and detail views.
- tracks workspace uses VSplitView and HSplitView for true pro app split panes.
- inspector height is adjustable by drag.
- full hover states with accent-color highlights.
- dynamic cursor changes for draggable regions and sliders.
- follows WWDC25 macOS pro app guidance: persistent toolbars, resizable panels, and no full-screen modal sheets.

iPadOS
- similar to macOS but adapted for touch.
- NavigationSplitView collapses to compact navigation drawer in portrait orientation.
- inspector collapses to a floating panel, draggable via drag handles.
- Liquid Glass panels automatically reduce blur intensity for battery efficiency.
- large hit targets for all controls.
- toolbar items may collapse into a More (…) menu in compact width.

iOS (iPhone)
- compact single-column layout.
- no split view: sidebar replaced with bottom sheet navigation or segmented toolbar.
- inspector is a scrollable panel below the timeline or as a modal sheet.
- Liquid Glass is retained but heavily simplified, with reduced transparency for performance.
- zoom controls replaced by pinch gestures where possible.
- minimal pro app features due to limited space.

tvOS
- focus-driven navigation.
- no sliders, replaced with stepper or increment/decrement buttons.
- simplified timeline layout with larger hit targets and reduced inspector complexity.
- glassmorphic panels use ultraThinMaterial but with higher contrast for television readability.
- Sidebar navigation is rendered as a List with focus rings.
- follows WWDC25 tvOS Pro App guidelines: always-on focus indicator, remote-friendly interaction.

liquid glass design
- All panels and controls use Apple’s ultraThinMaterial.
- blur intensity scales by platform:
  - macOS: full intensity for pro app look.
  - iPadOS: medium intensity to balance performance and clarity.
  - iOS: minimal intensity, opaque fallback in low-power mode.
  - tvOS: high contrast variant for television readability.
- Rounded corners: 8 points on iPhone, 12 points on iPad, 16 points on Mac panels, 20 points on tvOS panels.
- Glass panels stack with subtle depth shadows, following WWDC25 depth hierarchy.

adaptive ui implementation
- uses SwiftUI size classes, platform checks, and container queries.
- macOS uses fixed resizable panes.
- iPadOS and iOS use GeometryReader and LayoutPriority for adaptive resizing.
- tvOS uses conditional HStack and VStack replacements with larger spacing.
- toolbar items dynamically reorder or collapse based on width classes.

color and typography
- primary colors use system-defined semantic colors for dark and light mode consistency.
- text styles use SwiftUI dynamic type.
- labels and icons adapt automatically to system accessibility settings.

next steps
- integrate custom LiquidGlassKit components for future phase.
- expand adaptive layouts for portrait/landscape rotations.
- prepare for Phase 2 Audio Engine controls with additional toolbar groups and inspector tabs.
