# AudioWorkstation Visual Design Context

**Purpose**: UI specifications and design system for SwiftUI DAW  
**Version**: 2.0  
**Design Language**: Modern Apple Pro App Style  
**Last Updated**: 2025-01-31

## Current Visual Implementation

AudioWorkstation follows Apple's pro app design patterns seen in Logic Pro, Final Cut Pro, and Xcode. The UI is currently implemented as a functional shell with clean layouts but minimal styling.

## Layout Architecture

### Main Window Structure (macOS)
```
┌─────────────────────────────────────────────────────────┐
│ Toolbar                                    Min/Max/Close │
├─────────────┬───────────────────────────────────────────┤
│             │                                            │
│  Sidebar    │         Content Area                      │
│  (200pt)    │                                            │
│             ├───────────────────────────────┬───────────┤
│ • Browser   │                               │           │
│ • Tracks    │      Timeline/Tracks          │ Inspector │
│ • Mixer     │         (Flexible)            │  (300pt)  │
│             │                               │           │
├─────────────┴───────────────────────────────┴───────────┤
│ Transport Controls            Status Bar                 │
└─────────────────────────────────────────────────────────┘
```

### Adaptive Layouts
```swift
// Current Implementation
#if os(macOS)
NavigationSplitView {
    WorkstationSidebar()
} content: {
    TracksWorkspace()
} detail: {
    InspectorView()
}
#else
// iOS uses automatic adaptation
NavigationSplitView { ... }
    .navigationSplitViewStyle(.automatic)
#endif
```

## Component Specifications

### Sidebar (Implemented)
```yaml
Width: 200pt (resizable on macOS)
Background: System background
Sections:
  - Browser (SF Symbol: folder)
  - Tracks (SF Symbol: music.note.list)
  - Mixer (SF Symbol: slider.horizontal.3)
Style: Standard List with sections
Selection: Accent color highlight
```

### Track List (Partially Styled)
```yaml
Track Row Height: 60pt
Components:
  - Track Icon: 20x20pt (music.note or waveform)
  - Track Name: System font, primary label
  - Mute Button: 32x32pt (speaker.slash when muted)
  - Solo Button: 32x32pt (headphones)
  - Record Button: 32x32pt (record.circle)
  - Volume Slider: 100pt wide
  - Pan Knob: 44x44pt
Spacing: 8pt between controls
Background: Alternating row colors (subtle)
```

### Transport Controls (UI Complete)
```yaml
Container: 
  Height: 60pt
  Background: System secondary background
  
Buttons:
  Size: 44x44pt (large hit targets)
  Style: System default with SF Symbols
  - Play: play.fill / pause.fill (toggles)
  - Stop: stop.fill
  - Record: record.circle (red when active)
  
Time Display:
  Font: SF Mono, 18pt
  Format: "00:00.000"
  Color: Primary label
```

### Timeline Canvas (Minimal Implementation)
```yaml
Current State: Gray background with grid lines
Grid:
  Major Lines: Every beat, 0.5pt, gray.opacity(0.3)
  Minor Lines: Subdivisions, 0.25pt, gray.opacity(0.1)
  
Planned:
  Background: RGB(30, 30, 35) dark mode
  Playhead: 2pt red line
  Regions: Rounded rectangles with track color
```

## Color System

### Current Implementation
```swift
// System colors used throughout
Color.accentColor      // User's system accent
Color.primary          // Labels and text
Color.secondary        // Subdued elements
Color(.systemGray)     // Borders and dividers
Color(.systemGray2)    // Alternate row backgrounds
```

### Track Colors (Defined in Model)
```swift
let trackColors = [
    "blue": Color.blue,
    "green": Color.green,
    "red": Color.red,
    "orange": Color.orange,
    "purple": Color.purple,
    "pink": Color.pink,
    "yellow": Color.yellow,
    "gray": Color.gray
]
```

## Typography

### Current Usage
```yaml
Navigation Titles: .largeTitle (Tracks, Mixer, Browser)
Section Headers: .headline 
Track Names: .body
Control Labels: .caption
Time Display: .system(.body, design: .monospaced)
Status Text: .footnote
```

### Missing Typography
- No custom fonts
- No dynamic type support
- Limited text hierarchy

## Icons and Symbols

### SF Symbols in Use
```yaml
Navigation:
  folder: Browser
  music.note.list: Tracks  
  slider.horizontal.3: Mixer

Track Controls:
  speaker.slash: Mute (on)
  speaker.wave.2: Mute (off)
  headphones: Solo
  record.circle: Record Enable
  
Transport:
  play.fill: Play
  pause.fill: Pause
  stop.fill: Stop
  record.circle: Record

Track Types:
  music.note: MIDI Track
  waveform: Audio Track
  crown: Master Track
```

## Current Styling Gaps

### What's Missing
1. **Visual Polish**: Very utilitarian, needs refinement
2. **Dark Mode Optimization**: Uses system colors but not optimized
3. **Hover States**: No visual feedback on macOS
4. **Focus States**: Basic system focus rings only
5. **Animation**: No transitions or smooth updates
6. **Custom Drawing**: Timeline is just gray canvas
7. **Progress Indicators**: No loading or processing states

### Needed Enhancements

#### Professional Appearance
```swift
// Add subtle backgrounds
.background(Color(.systemGray6))
.clipShape(RoundedRectangle(cornerRadius: 8))

// Add shadows for depth
.shadow(color: .black.opacity(0.1), radius: 2, y: 1)

// Improve buttons
.buttonStyle(.borderless)
.controlSize(.large)
```

#### Track Lane Styling
```swift
// Planned region appearance
RoundedRectangle(cornerRadius: 4)
    .fill(trackColor.opacity(0.8))
    .overlay(
        RoundedRectangle(cornerRadius: 4)
            .strokeBorder(trackColor, lineWidth: 1)
    )
    .shadow(color: .black.opacity(0.2), radius: 1, y: 1)
```

## Platform Adaptations

### macOS Specific
- Window min size: 800x600
- Resizable panes with dividers
- Hover states on buttons
- Keyboard focus navigation
- Menu bar integration

### iOS/iPadOS Specific  
- Touch-friendly 44pt targets
- Swipe gestures planned
- Full-screen modes
- Simplified inspector

### tvOS Specific
- Focus-based navigation
- Larger text and controls
- High contrast mode
- Simplified layout

## Animation Specifications

### Current State
No animations implemented

### Planned Animations
```swift
// Smooth value changes
.animation(.easeInOut(duration: 0.2), value: volume)

// Transport state changes  
.animation(.spring(response: 0.3), value: isPlaying)

// View transitions
.transition(.asymmetric(
    insertion: .move(edge: .trailing),
    removal: .move(edge: .leading)
))
```

## Accessibility

### Current Support
- Basic VoiceOver labels
- Standard control accessibility
- System color compliance

### Needed Improvements
- Custom accessibility actions
- Meaningful control descriptions
- Keyboard shortcuts
- High contrast mode
- Reduced motion support

## Visual Performance

### Current Issues
- No view caching
- Frequent redraws
- Large memory footprint for many tracks

### Optimization Needed
```swift
// Lazy loading for track list
LazyVStack {
    ForEach(visibleTracks) { track in
        TrackRowView(track: track)
    }
}

// Canvas caching for timeline
.drawingGroup() // Flatten view hierarchy
```

## Design Tokens

### Spacing Scale
```swift
enum Spacing {
    static let micro: CGFloat = 4
    static let small: CGFloat = 8  
    static let medium: CGFloat = 16
    static let large: CGFloat = 24
    static let xlarge: CGFloat = 32
}
```

### Corner Radii
```swift
enum CornerRadius {
    static let small: CGFloat = 4   // Regions
    static let medium: CGFloat = 8  // Panels
    static let large: CGFloat = 12  // Modals
}
```

### Standard Sizes
```swift
enum Sizes {
    static let trackHeight: CGFloat = 60
    static let buttonSize: CGFloat = 44
    static let sliderWidth: CGFloat = 100
    static let sidebarWidth: CGFloat = 200
    static let inspectorWidth: CGFloat = 300
}
```

## Future Visual Enhancements

### Professional Polish
1. Gradient backgrounds for depth
2. Subtle textures and patterns
3. Glass morphism effects
4. Smooth shadows and highlights

### Advanced Visualizations
1. Waveform rendering with peaks
2. MIDI note visualization
3. Spectrum analyzers
4. VU meters

### Motion Design
1. Playhead smooth scrolling
2. Region snap animations
3. Track reorder animations
4. Zoom transitions

---

**Current Reality**: The visual design is functional but basic. It follows Apple's design patterns but lacks the polish of a professional DAW. The foundation is solid for adding visual refinements without major architectural changes.