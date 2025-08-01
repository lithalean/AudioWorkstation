# AudioWorkstation Navigation Context

**Purpose**: Document actual navigation implementation and user flows  
**Version**: 2.0  
**Navigation Pattern**: NavigationSplitView with Sidebar  
**Last Updated**: 2025-01-31

## Current Navigation Implementation

AudioWorkstation uses SwiftUI's NavigationSplitView for a three-column layout on macOS/iPadOS and adaptive layouts on iOS. The sidebar provides primary navigation between main work areas.

## Navigation Structure

### Implemented Hierarchy
```
AudioWorkstationApp
└── ContentView (NavigationSplitView)
    ├── Sidebar (WorkstationSidebar)
    │   ├── Browser
    │   ├── Tracks (default)
    │   └── Mixer
    ├── Content Area
    │   ├── BrowserView (placeholder)
    │   ├── TracksWorkspace (functional)
    │   └── MixerView (placeholder)
    └── Detail (Inspector)
        └── InspectorView (UI only)
```

### Current Implementation
```swift
struct ContentView: View {
    @State private var selectedView: WorkstationView = .tracks
    
    var body: some View {
        NavigationSplitView {
            WorkstationSidebar(selection: $selectedView)
        } content: {
            switch selectedView {
            case .browser:
                BrowserView()
            case .tracks:
                TracksWorkspace()
            case .mixer:
                MixerView()
            }
        } detail: {
            InspectorView()
        }
    }
}
```

## Navigation States

### Current State Management
```swift
enum WorkstationView: String, CaseIterable {
    case browser = "Browser"
    case tracks = "Tracks"  
    case mixer = "Mixer"
    
    var icon: String {
        switch self {
        case .browser: return "folder"
        case .tracks: return "music.note.list"
        case .mixer: return "slider.horizontal.3"
        }
    }
}
```

### Selection Behavior
- Single selection in sidebar
- Persists during session
- No deep linking support
- No navigation history

## Platform Behaviors

### macOS
```swift
#if os(macOS)
.navigationSplitViewStyle(.balanced)
.frame(minWidth: 800, minHeight: 600)
// Three columns always visible
// Resizable sidebar and inspector
#endif
```

### iPadOS
```swift
#if os(iOS)
.navigationSplitViewStyle(.automatic)
// Sidebar collapses in compact size
// Inspector as overlay in compact
// Full three-column in regular size
#endif
```

### iOS (iPhone)
- Single column navigation
- Tab bar style in compact
- Full screen views
- Modal inspector

## User Flows

### Current Working Flows

#### 1. Switch Between Views
```
User Action: Click sidebar item
System Response: Content area updates
State Change: selectedView updates
Animation: None (instant switch)
```

#### 2. View Track List
```
Default State → Tracks selected → Track list visible
- Can see all tracks
- Can see controls (not functional)
- Can add/remove tracks (partially)
```

### Non-Functional Flows

#### 1. Create New Project
```
Menu → New Project → ❌ Not implemented
Currently: Template project always loaded
```

#### 2. Audio Playback
```
Select Track → Load Audio → Press Play → ❌ No audio
Transport exists but doesn't control audio
```

#### 3. Region Editing
```
Timeline → Select Region → ❌ No regions displayed
Canvas exists but empty
```

## Navigation Components

### WorkstationSidebar
```swift
struct WorkstationSidebar: View {
    @Binding var selection: WorkstationView
    
    var body: some View {
        List(WorkstationView.allCases, id: \.self, 
             selection: $selection) { view in
            Label(view.rawValue, systemImage: view.icon)
        }
        .navigationTitle("AudioWorkstation")
        .frame(minWidth: 150, idealWidth: 200)
    }
}
```

### Content Views Status
- **BrowserView**: Empty placeholder
- **TracksWorkspace**: Functional layout
- **MixerView**: "Coming Soon" message

## Keyboard Navigation

### Implemented Shortcuts
None currently implemented

### Planned Shortcuts
```swift
// In AudioWorkstationApp commands
.keyboardShortcut("1", modifiers: .command) // Tracks
.keyboardShortcut("2", modifiers: .command) // Browser  
.keyboardShortcut("3", modifiers: .command) // Mixer
.keyboardShortcut("i", modifiers: .command) // Inspector
```

## Focus Management

### Current State
- System default focus behavior
- No custom focus guides
- Basic tab navigation

### tvOS Focus (Partially Implemented)
```swift
#if os(tvOS)
// Simplified navigation
// Focus moves between sidebar and content
// No inspector on tvOS
#endif
```

## State Persistence

### Current Implementation
- No state persistence
- Resets to Tracks view on launch
- No project memory
- No window restoration

### Needed Implementation
```swift
@AppStorage("selectedView") 
private var selectedView: WorkstationView = .tracks

@SceneStorage("projectID")
private var currentProjectID: UUID?
```

## Navigation Animations

### Current State
- No custom transitions
- Instant view switches
- No loading states
- No progress indicators

### Planned Animations
```swift
.transition(.asymmetric(
    insertion: .move(edge: .trailing),
    removal: .move(edge: .leading)
))
.animation(.easeInOut, value: selectedView)
```

## Error States

### Current Handling
None - no error states defined

### Needed Error Flows
- Project load failure
- Audio engine errors
- Missing file handling
- Permission denials

## Inspector Panel

### Current Implementation
```swift
struct InspectorView: View {
    var body: some View {
        Form {
            Section("Track Properties") {
                // Placeholder controls
            }
        }
        .frame(minWidth: 250, idealWidth: 300)
    }
}
```

### Planned Context Sensitivity
- Track selected → Track properties
- Region selected → Region properties
- Nothing selected → Project properties

## Menu Bar Integration

### Current Commands
```swift
.commands {
    CommandGroup(replacing: .newItem) {
        Button("New Project") { /* Not implemented */ }
        Button("Open Project...") { /* Not implemented */ }
    }
    CommandGroup(replacing: .saveItem) {
        Button("Save Project") { /* Not implemented */ }
    }
}
```

## Navigation Performance

### Current Issues
- Full view recreation on switch
- No view caching
- Heavy view hierarchies

### Optimization Needed
```swift
// Cache views
@StateObject private var browserView = BrowserView()
@StateObject private var tracksView = TracksWorkspace()
@StateObject private var mixerView = MixerView()
```

## Deep Linking (Not Implemented)

### Planned URL Scheme
```
audioworkstation://project/{id}
audioworkstation://project/{id}/tracks
audioworkstation://project/{id}/track/{trackId}
audioworkstation://project/{id}/mixer
```

## Navigation Testing

### Test Scenarios Needed
1. View switching reliability
2. State preservation
3. Platform-specific layouts
4. Focus navigation (tvOS)
5. Keyboard shortcuts
6. Error state handling

## Future Enhancements

### Advanced Navigation
1. **History Stack**: Back/forward navigation
2. **Breadcrumbs**: Show navigation path
3. **Quick Switcher**: Cmd+K style jumper
4. **Tabs**: Multiple projects open

### Contextual Navigation
1. Double-click region → Open editor
2. Right-click menus → Context actions
3. Drag & drop → Between views
4. Gesture navigation → Swipe between views

---

**Current Reality**: Basic navigation works with sidebar selection driving content view changes. The structure is clean but lacks polish, animations, persistence, and keyboard shortcuts. Inspector exists but isn't context-aware. Many navigation flows are designed but not implemented.