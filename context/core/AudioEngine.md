# AudioWorkstation Audio Engine Architecture

**Purpose**: Detailed documentation of AVAudioEngine implementation and audio pipeline  
**Version**: 1.0  
**Status**: Structure Implemented, Processing Pending  
**Last Updated**: 2025-01-31

## Overview

AudioWorkstation uses AVAudioEngine as its core audio processing system. The current implementation establishes the architecture with a singleton service pattern but doesn't yet process audio. This document describes both the implemented structure and the planned audio pipeline.

## Current Implementation Status

### ✅ Implemented Structure
```swift
@MainActor
class AudioEngineService: ObservableObject {
    static let shared = AudioEngineService()
    
    // Published State
    @Published var isPlaying: Bool = false
    @Published var isRecording: Bool = false
    @Published var currentPosition: TimeInterval = 0
    @Published var recordingTime: TimeInterval = 0
    
    // Core Components
    private let engine = AVAudioEngine()
    private let mainMixer = AVAudioMixerNode()
    private var trackMixers: [UUID: AVAudioMixerNode] = [:]
    
    // Placeholder Methods
    func play() { isPlaying = true }
    func pause() { isPlaying = false }
    func stop() { 
        isPlaying = false
        currentPosition = 0
    }
}
```

### ❌ Not Yet Implemented
- Audio file loading and playback
- Recording from inputs
- Mixer routing connections
- Real-time position updates
- Audio format conversions
- Buffer management

## Architecture Design

### Singleton Pattern Rationale
```swift
// Single audio engine instance for entire app
// @MainActor ensures UI thread safety for published properties
@MainActor
class AudioEngineService: ObservableObject {
    static let shared = AudioEngineService()
    private init() {
        setupEngine()
    }
}
```

**Benefits:**
- Single audio graph for entire application
- Centralized state management
- Easy SwiftUI integration via @Published
- Prevents multiple engine instances

### Node Graph Architecture

#### Current Structure (Implemented)
```
AudioEngineService
├── AVAudioEngine
├── Main Mixer Node
└── Track Mixers (Dictionary)
```

#### Planned Full Architecture
```
┌─────────────────────────────────────────────────────────┐
│                    Audio Engine                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  File Players          Track Mixers        Main Bus    │
│  ┌──────────┐         ┌───────────┐       ┌─────────┐ │
│  │ Track 1  │────────▶│ Mixer 1   │──┐    │         │ │
│  │ Player   │         │ Vol/Pan   │  │    │  Main   │ │
│  └──────────┘         └───────────┘  ├───▶│ Mixer   │ │
│                                      │    │         │ │
│  ┌──────────┐         ┌───────────┐  │    └────┬────┘ │
│  │ Track 2  │────────▶│ Mixer 2   │──┘         │     │
│  │ Player   │         │ Vol/Pan   │            │     │
│  └──────────┘         └───────────┘            ▼     │
│                                          ┌──────────┐ │
│  Input                                   │ Output   │ │
│  ┌──────────┐         ┌───────────┐     │ Device   │ │
│  │ Audio In │────────▶│ Input Mix │     └──────────┘ │
│  └──────────┘         └───────────┘                  │
└─────────────────────────────────────────────────────────┘
```

### Track Mixer Management

#### Current Implementation
```swift
private var trackMixers: [UUID: AVAudioMixerNode] = [:]

func createMixerForTrack(_ track: Track) {
    let mixer = AVAudioMixerNode()
    trackMixers[track.id] = mixer
    
    // TODO: Connect to engine
    // TODO: Set initial volume/pan
    // TODO: Connect to main mixer
}
```

#### Planned Implementation
```swift
func createMixerForTrack(_ track: Track) {
    let mixer = AVAudioMixerNode()
    trackMixers[track.id] = mixer
    
    // Attach to engine
    engine.attach(mixer)
    
    // Set initial parameters
    mixer.volume = track.volume
    mixer.pan = track.pan
    
    // Connect to main mixer
    engine.connect(mixer, 
                   to: mainMixer, 
                   format: standardFormat)
}
```

## Audio Format Standards

### Standard Format Definition
```swift
let standardFormat = AVAudioFormat(
    standardFormatWithSampleRate: 48000,
    channels: 2
)!
```

### Format Conversion Strategy
- Input: Accept any format
- Processing: Convert to 48kHz/24-bit
- Output: Match device format
- Files: Preserve original quality

## Transport Control Implementation

### Current State (UI Only)
```swift
func play() {
    isPlaying = true
    // TODO: Start all file players
    // TODO: Begin position updates
}

func pause() {
    isPlaying = false
    // TODO: Pause all file players
    // TODO: Store current position
}

func stop() {
    isPlaying = false
    currentPosition = 0
    // TODO: Stop all file players
    // TODO: Reset all positions
}
```

### Planned Transport System
```swift
class TransportController {
    private var displayLink: CADisplayLink?
    private var startTime: TimeInterval = 0
    
    func startPlayback(from position: TimeInterval) {
        // Schedule all players
        scheduleFilePlayers(startingAt: position)
        
        // Start position updates
        startPositionUpdates()
        
        // Start engine
        try engine.start()
    }
    
    private func startPositionUpdates() {
        displayLink = CADisplayLink(
            target: self,
            selector: #selector(updatePosition)
        )
        displayLink?.add(to: .main, forMode: .common)
    }
}
```

## File Playback Architecture

### Planned Implementation
```swift
class AudioFilePlayer {
    let player = AVAudioPlayerNode()
    var file: AVAudioFile?
    var startFrame: AVAudioFramePosition = 0
    
    func loadFile(url: URL) async throws {
        file = try AVAudioFile(forReading: url)
        // Generate waveform data
        // Cache format info
    }
    
    func schedulePlayback(at engineTime: AVAudioTime) {
        guard let file = file else { return }
        
        player.scheduleFile(
            file,
            at: engineTime,
            completionHandler: nil
        )
    }
}
```

### Buffer Management Strategy
- Pre-buffer 2 seconds ahead
- Use ring buffers for recording
- Pool buffers to reduce allocation
- Background thread for loading

## Recording Architecture

### Input Configuration
```swift
func configureInput() throws {
    let input = engine.inputNode
    let inputFormat = input.inputFormat(forBus: 0)
    
    // Create recording mixer
    let recordMixer = AVAudioMixerNode()
    engine.attach(recordMixer)
    
    // Connect input to record mixer
    engine.connect(input,
                   to: recordMixer,
                   format: inputFormat)
}
```

### Recording Pipeline
```
Audio Input ──▶ Input Mixer ──▶ Monitor Mix ──▶ Main Output
                     │
                     ▼
                Record Buffer ──▶ File Writer
```

## Real-time Constraints

### Thread Safety Rules
1. **UI Updates**: Always on main thread via @MainActor
2. **Audio Processing**: Never block render thread
3. **File I/O**: Always on background queue
4. **State Changes**: Atomic operations only

### Performance Guidelines
```swift
// ❌ NEVER in audio thread
func audioCallback() {
    DispatchQueue.main.async { // BLOCKS!
        updateUI()
    }
}

// ✅ CORRECT approach
private var pendingUIUpdate = false

func audioCallback() {
    pendingUIUpdate = true // Atomic flag
}

func updateLoop() {
    if pendingUIUpdate {
        updateUI()
        pendingUIUpdate = false
    }
}
```

## Error Handling Strategy

### Audio Interruption Handling
```swift
func setupInterruptionHandling() {
    NotificationCenter.default.addObserver(
        self,
        selector: #selector(handleInterruption),
        name: AVAudioSession.interruptionNotification,
        object: nil
    )
}

@objc private func handleInterruption(_ notification: Notification) {
    guard let info = notification.userInfo,
          let type = info[AVAudioSessionInterruptionTypeKey] as? UInt,
          let interruption = AVAudioSession.InterruptionType(rawValue: type)
    else { return }
    
    switch interruption {
    case .began:
        pause()
    case .ended:
        // Optionally resume
    @unknown default:
        break
    }
}
```

### Route Change Handling
- Monitor for device changes
- Reconfigure on sample rate changes
- Handle headphone plug/unplug
- Graceful Bluetooth transitions

## Memory Management

### Buffer Pooling (Planned)
```swift
class AudioBufferPool {
    private var availableBuffers: [AVAudioPCMBuffer] = []
    private let bufferSize: AVAudioFrameCount = 4096
    private let format: AVAudioFormat
    
    func getBuffer() -> AVAudioPCMBuffer {
        if let buffer = availableBuffers.popLast() {
            return buffer
        }
        return AVAudioPCMBuffer(
            pcmFormat: format,
            frameCapacity: bufferSize
        )!
    }
    
    func returnBuffer(_ buffer: AVAudioPCMBuffer) {
        buffer.frameLength = 0  // Reset
        availableBuffers.append(buffer)
    }
}
```

## Future Enhancements

### Phase 2: Basic Playback
1. Implement file loading
2. Create playback scheduling
3. Add position tracking
4. Enable transport controls

### Phase 3: Recording
1. Configure audio inputs
2. Implement record buffers
3. Add monitoring path
4. Create file writers

### Phase 4: Advanced Features
1. Plugin hosting (AUv3)
2. Time stretching
3. Pitch correction
4. Advanced routing

### Phase 5: Performance
1. GPU waveform rendering
2. Predictive buffering
3. Latency compensation
4. Multi-core processing

## Testing Strategy

### Unit Tests (Needed)
- Mock audio engine for UI tests
- Test mixer configurations
- Verify state transitions
- Test error conditions

### Integration Tests
- Full playback pipeline
- Recording workflow
- Format conversions
- Performance benchmarks

### Audio-Specific Tests
- Latency measurements
- Dropout detection
- Format compatibility
- Memory leak detection

---

**Current Reality**: The AudioEngineService exists as a well-structured singleton with proper state management, but all audio processing is TODO. The architecture is sound and ready for implementation. Next step is connecting the node graph and implementing basic file playback.