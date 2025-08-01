# AudioWorkstation Architecture Context

**Purpose**: Technical blueprint for SwiftUI-based DAW with AVAudioEngine  
**Version**: 2.0  
**Status**: ~40% Implemented (UI Shell + Audio Structure)  
**Last Updated**: 2025-01-31

## Executive Summary

AudioWorkstation is a modern Digital Audio Workstation built entirely in SwiftUI with AVAudioEngine for audio processing. The architecture prioritizes clean separation of concerns, reactive state management, and platform adaptability. Currently implements the UI shell and audio engine structure without actual audio processing.

## Key Architecture Patterns

### Core Stack
```
SwiftUI + AVAudioEngine + SwiftData + Combine
├── Pure SwiftUI (no UIKit/AppKit)
├── Singleton Audio Engine (@MainActor)
├── SwiftData Models (Project/Track/Region/MIDI)
└── Observable State Management
```

### Design Principles
1. **SwiftUI-First**: Every UI component is pure SwiftUI
2. **Reactive State**: @Published properties drive UI updates
3. **Platform Adaptive**: Single codebase, platform-specific enhancements
4. **Type Safety**: Strong types throughout, no stringly-typed APIs
5. **Prepared for Scale**: Architecture supports full DAW complexity

## System Architecture

### Layer Architecture
```
┌─────────────────────────────────────┐
│         App Layer (SwiftUI)         │
├─────────────────────────────────────┤
│  Presentation Layer (ViewModels)    │
├─────────────────────────────────────┤
│      Core Layer (Business Logic)    │
├─────────────────────────────────────┤
│     Infrastructure (Audio/Data)     │
└─────────────────────────────────────┘
```

### Component Architecture
```
AudioWorkstationApp
├── ContentView (Main Workspace)
│   ├── WorkstationSidebar
│   │   ├── Project Browser
│   │   ├── Track List
│   │   └── Media Browser
│   └── TracksWorkspace
│       ├── TimelineView
│       ├── TrackLanesView
│       └── InspectorView
├── TransportControls
└── MenuCommands

Core Services
├── AudioEngineService (Singleton)
│   ├── AVAudioEngine
│   ├── Mixer Architecture
│   └── Track Management
└── ProjectManager
    ├── SwiftData Container
    └── File Management
```

## Audio Engine Architecture

### Singleton Pattern
```swift
@MainActor
class AudioEngineService: ObservableObject {
    static let shared = AudioEngineService()
    
    @Published var isPlaying = false
    @Published var isRecording = false
    @Published var currentPosition: TimeInterval = 0
    
    private let engine = AVAudioEngine()
    private let mainMixer = AVAudioMixerNode()
}
```

### Mixer Node Graph (Planned)
```
Input Sources                    Track Mixers              Main Mix
┌──────────┐                    ┌─────────┐              ┌─────────┐
│ File     │───────────────────▶│ Track 1 │──────┐       │         │
│ Player   │                    └─────────┘      │       │  Main   │
└──────────┘                                     ├──────▶│ Mixer   │──▶ Output
┌──────────┐                    ┌─────────┐      │       │         │
│ Audio    │───────────────────▶│ Track 2 │──────┘       └─────────┘
│ Input    │                    └─────────┘
└──────────┘                    
                                ┌─────────┐
                                │ Track N │──────┘
                                └─────────┘
```

### Thread Safety Strategy
- Main thread UI updates via @MainActor
- Audio processing on real-time thread
- Lock-free audio buffers (future)
- Dispatch queues for file I/O

## SwiftUI Architecture

### View Composition Pattern
```swift
struct ContentView: View {
    @StateObject private var audioEngine = AudioEngineService.shared
    @State private var selectedTrack: Track?
    
    var body: some View {
        NavigationSplitView {
            WorkstationSidebar()
        } content: {
            TracksWorkspace()
        } detail: {
            InspectorView(track: selectedTrack)
        }
    }
}
```

### State Management Hierarchy
```
App State (@StateObject)
├── AudioEngineService (Singleton)
├── ProjectDocument (Current Project)
└── WorkspaceState (UI State)

View State (@State)
├── Selection State
├── Zoom/Scroll Position
└── UI Toggles

Environment Values
├── ColorScheme
├── SizeClass
└── Platform Adaptations
```

### Platform Adaptations
```swift
#if os(macOS)
    // Desktop-optimized with hover states, resizable panes
    HSplitView { ... }
#elseif os(iOS)
    // Touch-optimized with gestures, full-screen modes
    NavigationStack { ... }
#endif
```

## Data Architecture

### SwiftData Schema
```swift
@Model
class Project {
    var id = UUID()
    var name: String
    var tempo: Double = 120.0
    @Relationship(deleteRule: .cascade) 
    var tracks: [Track] = []
}

@Model
class Track {
    var id = UUID()
    var name: String
    var trackType: TrackType
    var volume: Float = 0.0
    var pan: Float = 0.0
    @Relationship(deleteRule: .cascade)
    var regions: [Region] = []
}
```

### Persistence Strategy
- SwiftData for project/session data
- File-based audio storage
- CloudKit sync (future consideration)
- Versioned schema migrations

## Performance Architecture

### UI Performance
- Lazy loading for track lanes
- Canvas-based timeline rendering
- Virtualized track list
- Debounced property updates

### Audio Performance (Planned)
- Pre-buffered file playback
- Lock-free ring buffers
- Optimized DSP chains
- Background audio capability

### Memory Management
- Weak references for delegates
- Careful capture list in closures
- Audio buffer pooling (future)
- Thumbnail cache for waveforms

## Security & Sandboxing

### Entitlements Required
- `com.apple.security.files.user-selected.read-write`
- `com.apple.security.device.audio-input`
- `com.apple.security.device.microphone`

### File Access
- Sandbox-safe file bookmarks
- Security-scoped resource access
- Temporary directory for renders

## Architectural Decisions

### Decision: Pure SwiftUI
**Rationale**: Modern, unified codebase across all Apple platforms  
**Trade-offs**: Less control over custom drawing, but better platform integration  
**Result**: Clean architecture, excellent platform adaptation

### Decision: Singleton Audio Engine
**Rationale**: Single source of truth for audio state, simplified management  
**Implementation**: @MainActor singleton with published properties  
**Result**: Clear state management, easy UI binding

### Decision: SwiftData over Core Data
**Rationale**: Modern Swift-first API, better type safety, cleaner code  
**Trade-offs**: Newer technology, less documentation  
**Result**: 50% less boilerplate, better Swift integration

### Decision: Desktop-First Development
**Rationale**: Pro audio users primarily on macOS  
**Implementation**: Full features on macOS, adapted subsets on iOS  
**Result**: Professional desktop experience without compromising mobile

## Anti-Patterns to Avoid

### ❌ Direct Audio Engine Access in Views
```swift
// WRONG
struct TrackView: View {
    let engine = AVAudioEngine() // Never!
}

// CORRECT
struct TrackView: View {
    @EnvironmentObject var audioEngine: AudioEngineService
}
```

### ❌ Synchronous File Operations
```swift
// WRONG
let data = try Data(contentsOf: audioFileURL) // Blocks UI

// CORRECT
Task {
    let data = try await loadAudioFile(from: audioFileURL)
}
```

### ❌ Massive View Bodies
```swift
// WRONG
struct ContentView: View {
    var body: some View {
        // 500 lines of view code
    }
}

// CORRECT
struct ContentView: View {
    var body: some View {
        VStack {
            HeaderSection()
            MainContent()
            FooterControls()
        }
    }
}
```

## Future Architecture Considerations

### Phase 2: Audio Implementation
- Implement file playback pipeline
- Add recording capabilities
- Create waveform rendering system
- Enable real-time monitoring

### Phase 3: MIDI Support
- CoreMIDI integration
- MIDI routing architecture
- Piano roll editor
- MIDI effect chain

### Phase 4: Plugin System
- AudioUnit v3 hosting
- Effect chain architecture
- Parameter automation
- Preset management

### Phase 5: Collaboration
- CloudKit project sharing
- Real-time collaboration
- Version control integration
- Comment system

## Architecture Quality Metrics

### Current Strengths
- ✅ Clean separation of concerns
- ✅ Modern Swift patterns throughout
- ✅ Platform-adaptive from day one
- ✅ Prepared for complex audio routing
- ✅ Type-safe API design

### Areas for Improvement
- ❌ No test coverage
- ❌ Limited error handling
- ❌ No performance profiling
- ❌ Documentation gaps
- ❌ Future OS deployment targets

---

**Note**: This architecture is actively evolving. The foundation is solid but audio implementation is pending. All architectural patterns have been validated through the UI shell implementation.