# AudioWorkstation Architecture Context

**Purpose**: Technical blueprint and system design rationale  
**Version**: 1.0  
**Status**: Alpha Experimental  
**Last Updated**: 2025-01-31

## Key Concepts (AI Quick Reference)

### Core Architecture Pattern
```
SwiftUI + AVAudioEngine + SwiftData = Modern DAW
```

### Critical Design Elements
1. **Cross-Platform SwiftUI** - Unified codebase for macOS/iOS/iPadOS/tvOS
2. **AVAudioEngine Core** - Professional audio processing backbone
3. **SwiftData Persistence** - Modern data layer for projects/tracks/regions
4. **Adaptive Layouts** - Platform-specific UI optimizations

## System Architecture

### Core Design Pattern
Track-based sequencing architecture with region-based MIDI editing. Designed as macOS-first workstation with adaptive layouts for other Apple platforms.

### Component Architecture
```
App Layer (SwiftUI)
├── ContentView (Main Container)
├── WorkstationSidebar (Navigation)
├── TrackView (Timeline)
├── TransportControls
└── Inspector (Context-sensitive)

Core Layer
├── AudioEngineService (AVAudioEngine wrapper)
├── Models (SwiftData entities)
└── Timeline State Management

Data Layer (SwiftData)
├── Project
├── Track
├── Region
└── MIDINote
```

## Technical Architecture

### State Management
- Timeline state via @Published and Combine
- Project data via SwiftData with @Query
- Audio engine state centralized in AudioEngineService

### Component Communication
- SwiftUI environment objects for global state
- Combine publishers for real-time updates
- SwiftData for persistent state

## Design Patterns

### Pattern 1: Adaptive Platform Layouts
```
PURPOSE: Single codebase, platform-optimized UX
IMPLEMENTATION: Conditional compilation and view modifiers
BENEFITS: Native feel on each platform
```

### Pattern 2: Future-Ready Audio Architecture
```
PURPOSE: Support planned SFZ sampler and DLS synth
IMPLEMENTATION: Abstracted audio engine interface
BENEFITS: Easy integration of new audio sources
```

## Anti-Patterns to Avoid

### ❌ NEVER: Force Desktop Paradigms on Mobile
```
// WRONG
VSplitView on iOS (crashes)

// CORRECT
#if os(macOS)
	VSplitView { ... }
#else
	VStack { ... }
#endif
```

### ❌ NEVER: Tight Audio-UI Coupling
```
// WRONG
Direct AVAudioEngine calls in Views

// CORRECT
AudioEngineService abstraction layer
```

## Architectural Decisions Log

### Decision: SwiftData over Core Data
**Rationale**: Modern, Swift-native persistence with better type safety
**Implementation**: Models as @Model classes with relationships
**Result**: Cleaner code, better Swift integration

### Decision: macOS-First Development
**Rationale**: Pro audio users primarily on desktop
**Implementation**: Full features on macOS, adaptive subsets elsewhere
**Result**: Professional desktop experience without compromising mobile

### Decision: Defer Complex Audio Features
**Rationale**: Establish stable UI/UX foundation first
**Implementation**: Phase 1 focuses on visual editing, Phase 2 adds audio
**Result**: More stable alpha build, clearer development path
