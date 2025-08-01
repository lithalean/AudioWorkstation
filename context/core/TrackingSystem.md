# AudioWorkstation Track and Region System

**Purpose**: Documentation of track management, regions, and timeline architecture  
**Version**: 1.0  
**Status**: UI Structure Implemented, Interaction Pending  
**Last Updated**: 2025-01-31

## System Overview

The track system is the core organizational structure of AudioWorkstation, managing the vertical arrangement of tracks and horizontal timeline of regions. Currently implements visual structure without interaction capabilities.

## Track Architecture

### Track Types and Capabilities

```swift
enum TrackType {
    case audio    // Audio file playback and recording
    case midi     // MIDI note sequencing  
    case master   // Main output bus (singleton)
}
```

#### Audio Tracks
- **Purpose**: Record and play audio files
- **Current**: Visual representation only
- **Planned**: AVAudioPlayerNode per track
- **Features**: Volume, pan, mute, solo, record enable

#### MIDI Tracks  
- **Purpose**: Sequence and play MIDI notes
- **Current**: Data model only
- **Planned**: Connect to sampler/synth
- **Features**: Note editing, velocity, MIDI routing

#### Master Track
- **Purpose**: Final mix output
- **Current**: Concept only
- **Planned**: Main mixer node output
- **Constraint**: One per project

### Track List Management

#### Current Implementation
```swift
struct TrackView: View {
    @Query private var projects: [Project]
    @State private var selectedTrack: Track?
    
    var body: some View {
        List {
            ForEach(currentProject.tracks) { track in
                TrackRowView(track: track)
                    .tag(track.id)
            }
        }
    }
}
```

#### Track Row Components
```
┌─────────────────────────────────────────────┐
│ [▶] Track Name   [M][S][R] ══○══ <○>       │
│  ↑     ↑          ↑  ↑  ↑    ↑    ↑        │
│ Icon  Name      Mute │  │  Volume │        │
│                     Solo │        Pan       │
│                      Record                 │
└─────────────────────────────────────────────┘
```

### Track State Management

#### Current State Properties
```swift
class Track {
    // Visual State
    @Published var isExpanded: Bool = false
    @Published var height: CGFloat = 60.0
    @Published var isSelected: Bool = false
    
    // Mix State
    var volume: Float = 0.0      // dB
    var pan: Float = 0.0         // -1 to +1
    var isMuted: Bool = false
    var isSolo: Bool = false
    var isRecordEnabled: Bool = false
}
```

#### Missing Implementations
- Track reordering (drag & drop)
- Track grouping/folders
- Track templates
- Automation lanes
- Track freeze/bounce

## Region System

### Region Concept
Regions are time-based containers that hold audio references or MIDI data. They exist on tracks and define when content plays.

### Region Properties
```swift
class Region {
    // Timeline Position
    var startTime: TimeInterval   // Seconds from project start
    var duration: TimeInterval    // Length in seconds
    
    // Content
    var regionType: RegionType    // .audio, .midi
    var name: String
    
    // Visual
    var color: String?           // Optional track color override
    
    // Audio Specific (future)
    var audioFileURL: String?    // Reference to audio file
    var audioOffset: TimeInterval // Start point within file
    var fadeInTime: TimeInterval = 0
    var fadeOutTime: TimeInterval = 0
}
```

### Region Rendering (Not Implemented)
```swift
// Planned implementation
struct RegionView: View {
    let region: Region
    let pixelsPerSecond: CGFloat
    
    var body: some View {
        RoundedRectangle(cornerRadius: 4)
            .fill(Color(region.color))
            .frame(
                width: region.duration * pixelsPerSecond,
                height: 50
            )
            .overlay(
                Text(region.name)
                    .lineLimit(1)
            )
            .position(x: region.startTime * pixelsPerSecond)
    }
}
```

### Region Interaction (Planned)

#### Selection
- Click to select
- Shift-click for multiple
- Marquee selection
- Select all in track

#### Manipulation
- Drag to move position
- Drag edges to resize
- Option-drag to copy
- Snap to grid

#### Editing
- Double-click to open editor
- Split at playhead
- Crossfade between regions
- Time stretch/compress

## Timeline Architecture

### Current Timeline Implementation
```swift
struct TimelineView: View {
    @State private var zoomLevel: CGFloat = 100 // pixels per second
    @State private var scrollOffset: CGFloat = 0
    
    var body: some View {
        ScrollView(.horizontal) {
            Canvas { context, size in
                // Grid drawing only
                drawGrid(context: context, size: size)
                // TODO: Draw regions
                // TODO: Draw playhead
                // TODO: Draw markers
            }
        }
    }
}
```

### Timeline Coordinate System
```
Time (seconds) ←→ Pixels conversion:
pixels = timeInSeconds * zoomLevel
timeInSeconds = pixels / zoomLevel

Zoom levels:
- 1 px/sec = Full project view
- 100 px/sec = Normal editing
- 1000 px/sec = Detailed editing
```

### Grid System (Partially Implemented)
```swift
func drawGrid(context: GraphicsContext, size: CGSize) {
    // Beat lines every 0.5 seconds (120 BPM)
    let beatInterval = 60.0 / project.tempo
    let pixelsPerBeat = beatInterval * zoomLevel
    
    // Draw vertical lines
    var x: CGFloat = 0
    while x < size.width {
        context.stroke(
            Path { path in
                path.move(to: CGPoint(x: x, y: 0))
                path.addLine(to: CGPoint(x: x, y: size.height))
            },
            with: .color(.gray.opacity(0.3))
        )
        x += pixelsPerBeat
    }
}
```

## MIDI Architecture

### MIDI Note Storage
```swift
class MIDINote {
    var pitch: Int         // 0-127 (C-1 to G9)
    var velocity: Int      // 0-127
    var startTime: TimeInterval  // Relative to region
    var duration: TimeInterval
    var channel: Int       // 0-15
}
```

### MIDI Display (Not Implemented)
```
Piano Roll View:
┌────┬────────────────────────┐
│ C4 │ ████  ██████          │
│ B3 │                        │
│ A3 │    ████    ████  ████  │
│ G3 │                        │
└────┴────────────────────────┘
     0    1    2    3    4  (beats)
```

### MIDI Editing Operations (Planned)
- Note creation (click & drag)
- Velocity editing
- Quantization
- Transpose
- Scale correction
- Humanization

## Synchronization System

### Timeline Synchronization
All timeline-dependent views must sync:
- Track lanes horizontal scroll
- Timeline ruler
- Playhead position
- Region positions

### Current Sync Strategy (Not Implemented)
```swift
class TimelineState: ObservableObject {
    @Published var scrollOffset: CGFloat = 0
    @Published var zoomLevel: CGFloat = 100
    @Published var playheadPosition: TimeInterval = 0
    
    func pixelToTime(_ pixel: CGFloat) -> TimeInterval {
        return (pixel + scrollOffset) / zoomLevel
    }
    
    func timeToPixel(_ time: TimeInterval) -> CGFloat {
        return (time * zoomLevel) - scrollOffset
    }
}
```

## Selection System

### Selection Types (Planned)
```swift
enum SelectionType {
    case none
    case track(Track)
    case region(Region)
    case regions([Region])
    case midiNotes([MIDINote])
    case timeRange(start: TimeInterval, end: TimeInterval)
}
```

### Selection Behavior
- Single click: Select item
- Cmd+click: Toggle selection
- Shift+click: Range select
- Drag: Marquee selection
- Cmd+A: Select all

## Snap and Grid System

### Snap Modes (Planned)
```swift
enum SnapMode {
    case off
    case bar
    case beat
    case subdivision(Int)  // 1/8, 1/16, etc.
    case seconds
    case frames
    case samples
}
```

### Snap Implementation
```swift
func snapTime(_ time: TimeInterval, mode: SnapMode) -> TimeInterval {
    switch mode {
    case .beat:
        let beatLength = 60.0 / project.tempo
        return round(time / beatLength) * beatLength
    // ... other modes
    }
}
```

## Performance Considerations

### View Optimization
- Virtualize track list (only render visible)
- LOD for regions (less detail when zoomed out)
- Canvas caching for static elements
- Debounced updates during playback

### Memory Management
```swift
// Region thumbnail cache
class RegionCache {
    private var thumbnails: [UUID: CGImage] = [:]
    private let maxCacheSize = 100 // MB
    
    func thumbnail(for region: Region) -> CGImage? {
        if let cached = thumbnails[region.id] {
            return cached
        }
        // Generate and cache
        let thumbnail = generateThumbnail(region)
        thumbnails[region.id] = thumbnail
        return thumbnail
    }
}
```

### Rendering Strategy
1. **Static Layer**: Grid, track headers
2. **Dynamic Layer**: Regions, playhead
3. **Overlay Layer**: Selection, markers
4. **Update Frequency**: 60 FPS for playhead, 30 FPS for UI

## Edit Operations

### Basic Operations (Not Implemented)
```swift
protocol EditOperation {
    func execute()
    func undo()
    var description: String { get }
}

struct MoveRegionOperation: EditOperation {
    let region: Region
    let oldPosition: TimeInterval
    let newPosition: TimeInterval
    
    func execute() {
        region.startTime = newPosition
    }
    
    func undo() {
        region.startTime = oldPosition
    }
}
```

### Planned Edit Operations
- **Move**: Change region position
- **Resize**: Adjust duration
- **Split**: Divide at position
- **Join**: Merge adjacent regions
- **Duplicate**: Copy with offset
- **Delete**: Remove from track
- **Crossfade**: Blend overlapping regions

## Track Routing Architecture

### Audio Signal Flow (Planned)
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Region    │────▶│    Track    │────▶│   Master    │
│   Player    │     │    Mixer    │     │    Mixer    │
└─────────────┘     └─────────────┘     └─────────────┘
                           │                     │
                           ▼                     ▼
                    ┌─────────────┐       ┌─────────┐
                    │   Effects   │       │ Output  │
                    │    Chain    │       │ Device  │
                    └─────────────┘       └─────────┘
```

### Mixer Node Management
```swift
extension AudioEngineService {
    func setupTrackMixer(for track: Track) {
        let mixer = AVAudioMixerNode()
        
        // Configure mixer
        mixer.volume = track.volume
        mixer.pan = track.pan
        
        // Store reference
        trackMixers[track.id] = mixer
        
        // Connect to main mixer
        engine.attach(mixer)
        engine.connect(mixer, to: mainMixer, format: nil)
    }
}
```

## Automation System (Future)

### Automation Lanes
```swift
struct AutomationLane {
    let parameter: AutomatableParameter
    var points: [AutomationPoint]
    var isVisible: Bool
    var isActive: Bool
}

enum AutomatableParameter {
    case volume
    case pan
    case send(Int)
    case pluginParameter(pluginId: UUID, parameterId: Int)
}
```

### Automation Recording
- Touch mode: Write while adjusting
- Latch mode: Write from touch to stop
- Write mode: Always writing
- Read mode: Playback only

## Current Implementation Gaps

### What's Missing
1. **Region Rendering**: No visual regions on timeline
2. **Region Interaction**: Can't select, move, or edit
3. **Playhead Display**: No current position indicator
4. **Timeline Scrolling**: No sync between views
5. **MIDI Display**: No piano roll or note rendering
6. **Track Reordering**: Can't drag tracks
7. **Zoom Controls**: UI exists but not functional

### Required Implementations

#### 1. Region Rendering
```swift
// Add to TimelineView
ForEach(track.regions) { region in
    RegionView(region: region)
        .position(
            x: region.startTime * zoomLevel,
            y: trackYPosition
        )
}
```

#### 2. Selection System
```swift
@State private var selectedRegions: Set<UUID> = []

func handleRegionTap(_ region: Region) {
    if isMultiSelectMode {
        selectedRegions.toggle(region.id)
    } else {
        selectedRegions = [region.id]
    }
}
```

#### 3. Playhead Implementation
```swift
struct PlayheadView: View {
    @ObservedObject var audioEngine: AudioEngineService
    let zoomLevel: CGFloat
    
    var body: some View {
        Rectangle()
            .fill(Color.red)
            .frame(width: 2)
            .position(x: audioEngine.currentPosition * zoomLevel)
    }
}
```

## Testing Strategies

### Unit Tests Needed
- Track CRUD operations
- Region timeline calculations
- MIDI note quantization
- Snap grid accuracy
- Selection logic

### Integration Tests
- Multi-track synchronization
- Timeline zoom/scroll
- Region move/resize
- Undo/redo operations
- Audio routing setup

### Performance Tests
- Large project loading (100+ tracks)
- Timeline rendering with many regions
- Playback with multiple tracks
- Memory usage during long sessions

## Usage Examples

### Creating a Track
```swift
func addAudioTrack() {
    let track = Track(name: "Audio \(project.tracks.count + 1)", type: .audio)
    track.color = "blue"
    track.index = project.tracks.count
    project.tracks.append(track)
    
    // Setup audio routing
    audioEngine.setupTrackMixer(for: track)
}
```

### Adding a Region
```swift
func addRegion(to track: Track, at time: TimeInterval) {
    let region = Region(startTime: time, duration: 4.0)
    region.name = "New Region"
    region.regionType = track.trackType == .midi ? .midi : .audio
    track.regions.append(region)
}
```

### Timeline Navigation
```swift
func zoomToFit() {
    let projectDuration = calculateProjectDuration()
    let viewWidth = timelineView.frame.width
    zoomLevel = viewWidth / projectDuration
}

func scrollToPlayhead() {
    let playheadPixel = audioEngine.currentPosition * zoomLevel
    scrollOffset = playheadPixel - (viewWidth / 2)
}
```

## Future Enhancements

### Advanced Track Types
- **Bus Tracks**: Submix routing
- **Folder Tracks**: Organization
- **VCA Tracks**: Group control
- **Video Tracks**: Movie sync

### Smart Features
- **Auto-Colorization**: Based on track type
- **Track Templates**: Preconfigured setups
- **Track Presets**: Save/load configurations
- **AI Mix Assistant**: Suggested levels

### Collaboration
- **Track Locking**: Prevent edits
- **Track Ownership**: User assignment
- **Change Tracking**: Who edited what
- **Conflict Resolution**: Merge changes

---

**Current Reality**: The track and region system architecture is well-designed with proper data models and UI structure. However, the timeline is just a gray canvas with no actual region rendering or interaction. The track list shows tracks but controls aren't wired to the audio engine. This document represents both what exists (structure) and what's planned (functionality).