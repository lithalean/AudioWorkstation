# AudioWorkstation Navigation Context

**Purpose**: Screen flow, user journeys, and state management  
**Version**: 1.0  
**Navigation Pattern**: Sidebar + Detail with Adaptive Layouts  
**Last Updated**: 2025-01-31

## Navigation State Machine
```
App Launch
	├─→ Empty State (no project)
	│     └─→ Create Project → Track View
	└─→ Last Project → Track View
		   ├─→ Browser View
		   ├─→ Track View (default)
		   └─→ Mixer View
```

## Screen Hierarchy
```
ContentView (Root)
├── WorkstationSidebar
│   ├── Project Info
│   ├── Browser
│   ├── Tracks (default)
│   └── Mixer
└── Detail View
	├── BrowserView
	├── TrackView
	│   ├── Timeline
	│   ├── Track Lanes
	│   └── Inspector
	└── MixerView
```

## Platform Navigation Patterns

### macOS
- NavigationSplitView with persistent sidebar
- Detail view with split panes (timeline + inspector)
- Keyboard shortcuts for view switching
- Multi-window support (future)

### iPadOS
- NavigationSplitView with collapsible sidebar
- Slide-over inspector panel
- Gesture-based navigation between views
- Split view app support

### iOS
- Tab bar or segmented control navigation
- Full-screen views with modal inspector
- Swipe gestures for view switching
- Simplified navigation stack

### tvOS
- Focus-based sidebar navigation
- Directional navigation between panes
- Simplified view hierarchy
- Play/pause button for transport

## User Journey Flows

### Creating First Track
```
Empty State → Create Project → Empty Timeline → Add Track Button → Track Added → Select Track → Inspector Opens
```

### Basic Editing Flow
```
Track View → Select Region → Inspector Updates → Adjust Properties → Timeline Updates → Save
```

### Playback Flow (Future)
```
Any View → Transport Play → Timeline Scrolls → Playhead Moves → Audio Plays → Transport Stop
```

## Navigation Quick Reference

| From | To | Trigger | Duration | Type |
|------|-----|---------|----------|------|
| Sidebar | Track View | Tap/Click | 300ms | Replace |
| Sidebar | Browser | Tap/Click | 300ms | Replace |
| Sidebar | Mixer | Tap/Click | 300ms | Replace |
| Track | Inspector | Select | 200ms | Reveal |
| Timeline | Region Edit | Double-tap | 0ms | In-place |

## Focus Navigation (tvOS)

### Focus Areas
1. Sidebar (List)
2. Timeline (Custom grid)
3. Inspector (Grouped controls)
4. Transport (Button group)

### Focus Movement Rules
```
Sidebar ←→ Timeline ←→ Inspector
   ↓                        ↓
Transport ←────────────────→
```

## State Management

### Navigation State
```swift
@StateObject navigationState:
  - currentView: ViewType
  - selectedTrack: Track?
  - selectedRegion: Region?
  - inspectorVisible: Bool
  - sidebarVisible: Bool
```

### View Transitions
- macOS/iPadOS: Animated with spring
- iOS: Navigation stack push/pop
- tvOS: Focus-driven with crossfade

## Keyboard Shortcuts (macOS)

| Action | Shortcut | Context |
|--------|----------|---------|
| Toggle Sidebar | Cmd+Opt+S | Global |
| Toggle Inspector | Cmd+I | Track View |
| Switch to Tracks | Cmd+1 | Global |
| Switch to Browser | Cmd+2 | Global |
| Switch to Mixer | Cmd+3 | Global |

## Deep Linking Support (Future)
```
audioworkstation://project/{id}/track/{trackId}
audioworkstation://project/{id}/mixer
audioworkstation://browser/preset/{presetId}
```
