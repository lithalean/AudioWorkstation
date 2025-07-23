AudioWorkstation Context

objective
Build a modern SwiftUI-based Digital Audio Workstation optimized for Darwin ARM64 (Apple Silicon). Focus on track-based sequencing, simple region editing, and a future SFZ-based sampler. The app is designed as a macOS-first workstation but runs on iPadOS, iOS, and tvOS. This is an ALPHA experimental build and not feature-complete.

scope
macOS: primary development platform, full track editing and inspector.
iPadOS: adaptive layout, basic editing, no advanced inspector interactions.
iOS: lightweight editing and playback only.
tvOS: playback only, focus-based navigation, simplified zoom and inspector.

architecture
audio engine: AVAudioEngine and AVAudioUnitSampler. Multi-sampler support for per-track instruments. Future SFZ and DLS Synth playback with advanced routing.
data layer: SwiftData for persistent projects, tracks, regions, MIDI notes, SFZ presets.
- Project: id, name, createdDate, array of Track.
- Track: id, name, color, volume, pan, isMuted, isSolo, array of Region.
- Region: id, name, duration, startTime, array of MIDINote.
timeline and inspector: real-time state updates via @Published and Combine.
cross-platform logic:
- VSplitView and HSplitView on macOS.
- VStack and HStack fallbacks on iOS and tvOS.
- No file importing on tvOS, UI-only regions.

design
WWDC25 pro app design language.
Glassmorphic panels using ultraThinMaterial.
Dynamic hover states and focus indicators for macOS and tvOS.
Inspector panel with expandable sections.
Toolbar with zoom, track inspector toggle, transport controls.

what we tried
1. Full SFZ integration early: rejected. Blocked until Phase 2 due to stability and engine maturity.
2. Unified VSplitView across all platforms: failed. Unavailable on iOS and tvOS. Resolved by adding fallback layouts with VStack and HStack.
3. Advanced Mixer View early: rejected. Placeholder only. Focus shifted to track and timeline basics.
4. Complex Timeline Drag and Drop: deferred. Region placement currently static with dummy notes.

known issues
No real audio playback. Transport is UI-only, engine planned for Phase 2.
Region editing is visual only, no live MIDI or automation support.
SwiftData relationships sometimes require manual array conversions in Lists.
tvOS inspector performance may degrade with many tracks due to focus navigation.

current phase
Phase 1: Core Editing and Layouts.
Focus areas: stable cross-platform UI, basic track and region editing interactions, persistent project and track data via SwiftData.

future
Phase 2: Audio Engine and Playback.
- DLS Synth playback for MIDI regions.
- SFZ loading and basic sampler integration on macOS and iOS first.
- Mixer view with basic inserts and sends.

Phase 3: Advanced DAW Features.
- MIDI editing and automation lanes.
- Audio recording and waveform regions.
- AUv3 and custom DSP integration using Metal.

strict exclusions
No cloud sync or online services.
No VST or AU hosting outside AUv3.
No Windows or Linux support.
No analytics or telemetry.
