# AudioWorkstation Visual Design Context

**Purpose**: Complete visual design system and UI specifications  
**Version**: 1.0  
**Design Language**: WWDC25 Pro App + Liquid Glass (Glassmorphic)  
**Last Updated**: 2025-01-31

## Visual Hierarchy
```
Z-LAYERS (back→front):
├─[0] Background (app background)
├─[1] Content (timeline, tracks)
├─[2] Controls (buttons, sliders)
├─[3] Panels (inspector, sidebar)
└─[4] Overlays (tooltips, popovers)
```

## Quick Reference Tables

### Color Palette
| Element | Light Mode | Dark Mode | Notes |
|---------|------------|-----------|-------|
| **Background** | systemBackground | black | Base layer |
| **Panel Glass** | ultraThinMaterial | ultraThinMaterial | Glassmorphic blur |
| **Track Default** | systemBlue | systemBlue | User customizable |
| **Selected** | accentColor | accentColor | Platform accent |
| **Hover State** | secondary.opacity(0.2) | secondary.opacity(0.3) | macOS only |

### Spacing System
| Token | Value | Usage |
|-------|-------|-------|
| **micro** | 4pt | Icon padding |
| **small** | 8pt | Internal component padding |
| **medium** | 16pt | Component spacing |
| **large** | 24pt | Section gaps |
| **xlarge** | 32pt | Major sections |

### Typography Scale
| Style | Size | Weight | Line Height | Usage |
|-------|------|--------|-------------|-------|
| **Title** | 28pt | Bold | 34pt | Main sections |
| **Headline** | 20pt | Semibold | 24pt | Panel headers |
| **Body** | 16pt | Regular | 20pt | General text |
| **Caption** | 13pt | Regular | 16pt | Labels, hints |
| **Micro** | 11pt | Regular | 14pt | Timeline markers |

## Platform-Specific Adjustments

### macOS
```yaml
panel_corner_radius: 16pt
blur_intensity: full
hover_states: enabled
cursor_changes: enabled
resize_handles: visible
```

### iPadOS
```yaml
panel_corner_radius: 12pt
blur_intensity: medium
touch_targets: 44pt_minimum
drag_handles: visible
compact_mode: adaptive
```

### iOS
```yaml
panel_corner_radius: 8pt
blur_intensity: minimal
navigation: bottom_sheet
inspector: modal_sheet
simplified_controls: true
```

### tvOS
```yaml
panel_corner_radius: 20pt
blur_intensity: high_contrast
focus_rings: always_visible
text_size: +4pt_from_base
simplified_layout: true
```

## Component Specifications

### Track Lane
```yaml
height: 60pt
padding: 8pt
background: track_color.opacity(0.1)
border: 1pt track_color.opacity(0.3)
selected_border: 2pt accentColor
hover_overlay: white.opacity(0.05) # macOS only
```

### Region Block
```yaml
corner_radius: 4pt
background: track_color.opacity(0.8)
border: 1pt track_color
selected_border: 2pt accentColor
min_width: 20pt
resize_handles: 4pt x 20pt # future
```

### Inspector Panel
```yaml
width:
  macOS: 320pt
  iPadOS: 280pt_to_360pt
  iOS: full_width
background: ultraThinMaterial
sections:
  header_height: 44pt
  padding: 16pt
  separator: 1pt_gray.opacity(0.2)
```

### Transport Controls
```yaml
button_size:
  macOS: 32pt
  iOS/iPadOS: 44pt
  tvOS: 64pt
spacing: 8pt
background: ultraThinMaterial
padding: 12pt
```

## Glassmorphic Design Rules

1. **Layering**: Each panel must have clear visual separation through:
   - Background blur (ultraThinMaterial)
   - Subtle drop shadow (color: black.opacity(0.1), radius: 8, y: 2)
   - 1pt border (gray.opacity(0.1))

2. **Depth Perception**:
   - Closer elements: stronger shadows, higher blur
   - Further elements: softer shadows, lower blur
   - Active elements: additional glow effect

3. **Performance Optimization**:
   - iOS: Reduce to thinMaterial in low power mode
   - tvOS: Use high contrast variant for readability
   - Limit nested glass effects to 2 levels max

## Animation Specifications

### Standard Transitions
```yaml
navigation: 0.3s easeInOut
hover_states: 0.15s easeOut
selection: 0.2s easeInOut
panel_resize: realtime (no animation)
focus_change: 0.25s easeInOut # tvOS
```

### Platform-Specific Animations
- macOS: Full spring animations on hover/selection
- iOS/iPadOS: Respect reduced motion settings
- tvOS: Focus engine handles all animations

## Icon System
- Use SF Symbols exclusively
- Weight: Regular for most, Medium for selected
- Size: 20pt standard, 16pt compact, 24pt tvOS
- Color: Primary for active, Secondary for inactive
