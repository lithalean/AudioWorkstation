# AudioWorkstation Data Models Documentation

**Purpose**: Complete SwiftData schema documentation for DAW entities  
**Version**: 1.0  
**Status**: Models Defined, Persistence Not Implemented  
**Last Updated**: 2025-01-31

## Overview

AudioWorkstation uses SwiftData for its persistence layer with a hierarchical model structure designed for a professional DAW. The models are fully defined with proper relationships but are not yet actively used for persistence.

## Model Hierarchy

```
Project (Root Entity)
├── Track[] (Audio/MIDI/Master tracks)
│   └── Region[] (Time-based clips)
│       └── MIDINote[] (Note events)
└── Metadata (Tempo, time signature, dates)
```

## Core Models

### Project Model
```swift
@Model
class Project {
    // Identity
    var id: UUID = UUID()
    var name: String = "Untitled Project"
    
    // Metadata
    var createdDate: Date = Date()
    var modifiedDate: Date = Date()
    
    // Musical Properties
    var tempo: Double = 120.0  // BPM
    var timeSignature: String = "4/4"
    
    // Relationships
    @Relationship(deleteRule: .cascade, inverse: \Track.project)
    var tracks: [Track] = []
    
    init(name: String) {
        self.name = name
    }
}
```

**Key Design Decisions:**
- UUID for unique identification
- Cascade delete for data integrity
- Default tempo follows industry standard
- Simple string for time signature (consider TimeSignature type later)

### Track Model
```swift
@Model
class Track {
    // Identity
    var id: UUID = UUID()
    var name: String = "Track"
    var index: Int = 0  // Display order
    
    // Track Properties
    var trackType: TrackType = .audio
    var color: String = "blue"  // Color name for UI
    
    // Mix Parameters
    var volume: Float = 0.0      // dB: -∞ to +12
    var pan: Float = 0.0         // -1.0 (L) to +1.0 (R)
    var isMuted: Bool = false
    var isSolo: Bool = false
    var isRecordEnabled: Bool = false
    
    // Relationships
    var project: Project?
    
    @Relationship(deleteRule: .cascade, inverse: \Region.track)
    var regions: [Region] = []
    
    init(name: String, type: TrackType) {
        self.name = name
        self.trackType = type
    }
}

enum TrackType: String, Codable {
    case audio = "audio"
    case midi = "midi" 
    case master = "master"
}
```

**Mix Parameter Ranges:**
- Volume: -∞ to +12 dB (0.0 = unity gain)
- Pan: -1.0 (hard left) to +1.0 (hard right)
- Consider using `Measurement<UnitAudioLevel>` in future

### Region Model
```swift
@Model
class Region {
    // Identity
    var id: UUID = UUID()
    var name: String = "Region"
    
    // Timeline Position
    var startTime: TimeInterval = 0.0  // Seconds from project start
    var duration: TimeInterval = 1.0   // Length in seconds
    
    // Content Type
    var regionType: RegionType = .midi
    
    // Audio Properties (future)
    var audioFileURL: String?  // Path to audio file
    var audioOffset: TimeInterval = 0.0  // Start offset within file
    
    // Visual Properties
    var color: String?  // Optional override of track color
    
    // Relationships
    var track: Track?
    
    @Relationship(deleteRule: .cascade, inverse: \MIDINote.region)
    var midiNotes: [MIDINote] = []
    
    init(startTime: TimeInterval, duration: TimeInterval) {
        self.startTime = startTime
        self.duration = duration
    }
}

enum RegionType: String, Codable {
    case audio = "audio"
    case midi = "midi"
    case automation = "automation"  // Future
}
```

**Timeline Considerations:**
- Using TimeInterval (seconds) for simplicity
- Consider musical time (bars/beats) in future
- Audio offset enables non-destructive editing

### MIDINote Model
```swift
@Model
class MIDINote {
    // Identity
    var id: UUID = UUID()
    
    // MIDI Properties
    var pitch: Int = 60          // MIDI note number (0-127)
    var velocity: Int = 80       // Strike velocity (0-127)
    var startTime: TimeInterval = 0.0  // Relative to region
    var duration: TimeInterval = 0.25   // Note length
    
    // Extended Properties
    var channel: Int = 0         // MIDI channel (0-15)
    
    // Relationships
    var region: Region?
    
    init(pitch: Int, velocity: Int, startTime: TimeInterval, duration: TimeInterval) {
        self.pitch = pitch
        self.velocity = velocity
        self.startTime = startTime
        self.duration = duration
    }
}
```

**MIDI Standards:**
- Pitch: Middle C = 60
- Velocity: 0 = note off, 127 = maximum
- Times relative to containing region

## Relationship Design

### Cascade Delete Rules
```
Project ─┬─(delete)──> Track[] ─┬─(delete)──> Region[] ─┬─(delete)──> MIDINote[]
         │                      │                       │
         └── Deletes all ───────┴── Deletes all ───────┴── Deletes all
```

### Inverse Relationships
- Track ←→ Project (many-to-one)
- Region ←→ Track (many-to-one)
- MIDINote ←→ Region (many-to-one)

## SwiftData Configuration

### Model Container Setup
```swift
// In AudioWorkstationApp.swift
.modelContainer(for: Project.self)
```

### Schema Migration Strategy (Future)
```swift
enum DataSchemaV1: VersionedSchema {
    static var versionIdentifier = Schema.Version(1, 0, 0)
    static var models: [any PersistentModel.Type] {
        [Project.self, Track.self, Region.self, MIDINote.self]
    }
}
```

## Query Patterns

### Basic Queries
```swift
// All projects
@Query var projects: [Project]

// Projects sorted by modified date
@Query(sort: \Project.modifiedDate, order: .reverse) 
var recentProjects: [Project]

// Tracks for current project
@Query var tracks: [Track]
let projectTracks = tracks.filter { $0.project?.id == currentProject.id }
```

### Complex Queries (Planned)
```swift
// All MIDI regions in project
let midiRegions = project.tracks
    .flatMap { $0.regions }
    .filter { $0.regionType == .midi }

// Solo'd tracks
let soloTracks = project.tracks.filter { $0.isSolo }

// Notes in time range
let notesInRange = region.midiNotes.filter { note in
    let absoluteTime = region.startTime + note.startTime
    return absoluteTime >= rangeStart && absoluteTime <= rangeEnd
}
```

## Performance Considerations

### Lazy Loading
- Regions load on demand when track expands
- MIDI notes load when region opens for editing
- Audio waveform data cached separately

### Batch Operations
```swift
// Batch update track indices after reordering
try await modelContext.transaction {
    for (index, track) in reorderedTracks.enumerated() {
        track.index = index
    }
}
```

### Memory Management
- Large audio files referenced by URL, not stored
- Waveform peak data in separate cache
- Limit in-memory MIDI notes for large projects

## Data Validation

### Model Constraints
```swift
extension Track {
    var isValid: Bool {
        !name.isEmpty &&
        volume >= -60 && volume <= 12 &&
        pan >= -1 && pan <= 1
    }
}

extension MIDINote {
    var isValid: Bool {
        pitch >= 0 && pitch <= 127 &&
        velocity >= 0 && velocity <= 127 &&
        duration > 0
    }
}
```

### Business Rules
1. Only one master track per project
2. Track names unique within project
3. Regions cannot have negative duration
4. MIDI notes must be within region bounds

## Future Enhancements

### Planned Model Additions
```swift
@Model
class AutomationPoint {
    var time: TimeInterval
    var value: Float
    var curveType: CurveType
}

@Model
class Marker {
    var time: TimeInterval
    var name: String
    var color: String
}

@Model
class AudioFile {
    var url: URL
    var duration: TimeInterval
    var sampleRate: Double
    var channelCount: Int
    var peakData: Data?  // Cached waveform
}
```

### Relationship Extensions
- Track ← → AudioUnit (effects chain)
- Region ← → AutomationLane
- Project ← → MixerSnapshot

### CloudKit Sync (Future)
```swift
extension Project {
    var cloudKitRecord: CKRecord {
        // Convert to CloudKit format
    }
}
```

## Current Implementation Gaps

### What's Missing
1. **No Persistence Operations**: Models defined but not saved/loaded
2. **No Migration Setup**: Version 1 schema not established
3. **No Validation**: Constraints not enforced
4. **No Indexing**: No performance optimization
5. **No Batch Operations**: Individual updates only

### Required Implementations
```swift
// Save project
func saveProject(_ project: Project) async throws {
    modelContext.insert(project)
    try await modelContext.save()
}

// Load recent projects
func loadRecentProjects() async throws -> [Project] {
    let descriptor = FetchDescriptor<Project>(
        sortBy: [SortDescriptor(\.modifiedDate, order: .reverse)]
    )
    return try await modelContext.fetch(descriptor)
}
```

## Testing Considerations

### Model Tests Needed
- Relationship integrity
- Cascade delete behavior  
- Migration compatibility
- Query performance
- Thread safety

### Test Data Factory
```swift
extension Project {
    static func makeTestProject() -> Project {
        let project = Project(name: "Test Project")
        
        let audioTrack = Track(name: "Audio 1", type: .audio)
        let midiTrack = Track(name: "MIDI 1", type: .midi)
        
        project.tracks = [audioTrack, midiTrack]
        return project
    }
}
```

---

**Current Status**: The data model layer is well-designed and complete, following SwiftData best practices. However, no persistence operations are implemented yet. The models exist in code but aren't actively used for saving or loading projects.