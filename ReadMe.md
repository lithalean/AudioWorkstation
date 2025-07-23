# AudioWorkstation
*An Open Source, modern Darwin ARM64 Digital Audio Workstation (DAW) following WWDC25 macOS, iPadOS & tvOS Design Principles*

![Platform Support](https://img.shields.io/badge/platform-macOS%20%7C%20iOS%20%7C%20iPadOS%20%7C%20tvOS-blue)  
![Swift Version](https://img.shields.io/badge/swift-6.2+-orange)  
![iOS Version](https://img.shields.io/badge/iOS-18.0+-green)  
![License](https://img.shields.io/badge/license-MIT-blue)

---

## ✨ Current Status: **ALPHA – NOT YET WORKING**

This is an **early ALPHA build**. Core functionality is under heavy development and subject to major changes.

---

## 🎯 Project Vision

AudioWorkstation aims to be a **simple, native-feeling DAW** for Apple platforms. It follows **WWDC25 design principles**, delivering a clean macOS-first interface that scales to **iPadOS, iOS, and tvOS**.

This is the foundation for a future full-featured SFZ-based sampler and sequencing engine.

---

## 🚀 Features (Planned / In Progress)

### 🎶 Core DAW Functionality (Phase 1)
- ⏳ **Track-based timeline** (basic regions, dummy data)  
- ⏳ Per-track **mute/solo/volume/pan** controls  
- ⏳ Inspector for editing track properties  
- ⏳ **Timeline regions** with note counts (visual only)

### 📚 Cross-Platform Layouts
- ✅ **macOS** – Split-view editor with draggable inspector  
- ✅ **iPadOS** – Adaptive layouts with floating toolbars  
- ✅ **iPhone** – Compact inspector with responsive scaling  
- ✅ **tvOS** – Remote-friendly controls (stepper-style zoom, focusable UI)

### 🎨 Modern Interface
- ✅ **SwiftUI Adaptive Layouts** using `.ultraThinMaterial`  
- ✅ Hover states & dynamic selection (macOS/iPadOS)  
- ✅ **Darwin ARM64-optimized**

---

## 🛠 Technical Overview

### Architecture
- **SwiftUI 6.2+** – Reactive, adaptive design  
- **@Published & Combine** – Real-time track state updates  
- **SwiftData** – Persistent projects, tracks & regions  
- **Platform Checks** – Conditional logic for tvOS vs macOS/iOS

### Tech Stack
- **Audio Engine** – Placeholder playback service (future DLS/SFZ support)  
- **Data** – SwiftData for Projects, Tracks, Regions  
- **Cross-Platform** – Single codebase for macOS, iOS, iPadOS, tvOS

---

## 📦 Installation & Setup

### Prerequisites
- **Xcode 26+**  
- **iOS 18+, iPadOS 18+, macOS 15+, tvOS 18+**  
- **Swift 6.2+**  
- Runs best on **Darwin ARM64 (Apple Silicon)**

### Run the Project
1. Clone the repository:
   ```bash
   git clone https://github.com/YOUR_USERNAME/AudioWorkstation.git
   ```
2. Open `AudioWorkstation.xcodeproj` in Xcode 26+  
3. Build & run on your preferred platform.

---

## ⌨️ Controls

### macOS / iPadOS / iOS
- **Play/Stop** – Transport control buttons  
- **Zoom** – Toolbar magnifying glass buttons / slider  
- **Inspector** – Toggle via info button

### tvOS
- **Play/Stop** – Siri Remote select button  
- **Zoom** – +/- stepper-style buttons  
- **Navigation** – Remote swipe/focus

---

## 🗺 Development Roadmap

### ✅ Phase 1 (Core Editing) – In Progress
- Basic timeline, tracks, inspector, cross-platform layouts

### 🚧 Phase 2 (Audio Engine)
- DLS Synth playback for MIDI regions  
- Basic sampler playback (SFZ loading on macOS/iOS)

### 🔮 Future (Planned)
- Full SFZ sampler engine  
- Advanced MIDI editing & automation  
- Mixer view with inserts & sends  
- Audio recording & waveform regions  
- Plugin support (AUv3, Metal-based DSP)
