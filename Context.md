# AudioWorkstation Context.md

## objective
Build a modern, SwiftUI-based **Digital Audio Workstation (DAW)** optimized for **Darwin ARM64 (Apple Silicon)**, focusing on **track-based sequencing, simple region editing, and a future SFZ-based sampler**.  
The app is designed as a **macOS-first workstation** but runs on iPadOS, iOS, and tvOS.  
This is an **ALPHA experimental build** — not feature-complete.

## scope
- **macOS**: primary development platform, full track editing and inspector.  
- **iPadOS**: adaptive layout, basic editing, no advanced inspector interactions.  
- **iOS**: lightweight editing and playback only.  
- **tvOS**: playback only, focus-based navigation, simplified zoom and inspector.  

## architecture
- **audio engine**: placeholder engine (`AudioEngineService`) for future SFZ and DLS Synth playback.  
- **data layer**: SwiftData for persistent **projects, tracks, regions, MIDI notes, SFZ presets**.  
  - Project: id, name, createdDate, [Track].  
  - Track: id, name, color, volume, pan, isMuted, isSolo, [Region].  
  - Region: id, name, duration, startTime, [MIDINote].  
- **timeline & inspector**: real-time state via `@Published` + Combine.  
- **cross-platform logic**:  
  - `VSplitView` + `HSplitView` on macOS.  
  - `VStack` + `HStack` fallbacks on iOS & tvOS.  
  - No file importing on tvOS (UI-only regions).

## design
- WWDC25 **pro app design language**.  
- Glassmorphic panels using `.ultraThinMaterial`.  
- Dynamic hover states and focus indicators (macOS + tvOS).  
- Inspector panel with expandable sections.  
- Toolbar with **zoom, track inspector toggle, transport controls**.

## what we tried
1. **Full SFZ integration early**  
   - rejected. blocked until Phase 2 due to stability and engine maturity.  

2. **Unified VSplitView across all platforms**  
   - failed; unavailable on iOS and tvOS.  
   - resolved by adding fallback layouts with VStack/HStack.

3. **Advanced Mixer View early**  
   - rejected. placeholder only; focus shifted to track/timeline basics.  

4. **Complex Timeline Drag & Drop**  
   - deferred; region placement currently static with dummy notes.

## known issues
- **No real audio playback** — transport is UI-only (engine to be implemented in Phase 2).  
- **Region editing is visual only**; no live MIDI or automation support.  
- **SwiftData relationships** sometimes require manual array conversions in Lists.  
- **tvOS inspector performance** may degrade with many tracks due to focus navigation.

## current phase
**Phase 1: Core Editing & Layouts**  
Focus:  
- Stable cross-platform UI.  
- Basic track/region editing interactions.  
- Persistent project & track data via SwiftData.

## future
**Phase 2: Audio Engine & Playback**  
- DLS Synth playback for MIDI regions.  
- SFZ loading and basic sampler integration (macOS/iOS first).  
- Mixer view with basic inserts/sends.  

**Phase 3: Advanced DAW Features**  
- MIDI editing & automation lanes.  
- Audio recording & waveform regions.  
- AUv3 and custom DSP integration (Metal-based).  

## strict exclusions
- No cloud sync or online services.  
- No VST/AU hosting outside AUv3.  
- No Windows or Linux support.  
- No analytics or telemetry.  
