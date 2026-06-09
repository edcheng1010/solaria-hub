# Solaria Architecture

> **Version:** 2.0  
> **Last Updated:** 2026-05-25  
> **Status:** Canonical reference for all Solaria contributors

---

## Overview

Solaria defines the **Solaria Standard Protocol (SSP)** — a shared communication specification between programming environments (clients) and robotics hardware (bridges). SSP is a **developer and infrastructure unification layer**: it ensures that any Solaria client can talk to any SSP-compatible firmware, and that the robot capabilities available to students are consistent across every supported platform combination.

SSP does **not** make student code portable. A student's App Inventor blocks are stateful and component-based; a student's Scratch blocks are event-driven and sequential. The code looks different, the logic patterns differ, and the blocks are platform-specific. What SSP guarantees is that the underlying robot capabilities — motor control, sensor reading, real-time feedback, AI integration — are the same regardless of which client and hardware combination a school chooses. The project's mission is **capability parity, not code portability**.

To support the vast diversity of robotics hardware — from open-source microcontrollers to closed-ecosystem commercial products — Solaria uses a **Hybrid Architecture**. This document explains the two hardware integration types, how clients interact with each, the data flow through the system, and the repository structure that organizes the codebase.

---

## Core Concepts

### Client
The software environment where the student's program executes. The client:
- Runs on the student's device (smartphone, laptop, tablet, or browser)
- Contains all program logic, decision-making, and AI/ML processing
- Initiates and manages connections to one or more hubs
- Has access to device-native capabilities (camera, microphone, network, GPU, onboard sensors)

### Hub
The physical robotics hardware controlled by the client. The hub:
- Executes motor commands and reports sensor readings
- Does not run student logic — it acts as a controlled peripheral
- Implements SSP (or a protocol library that translates SSP ↔ proprietary commands) over one or more transports
- May be any microcontroller, single-board computer, or commercial robotics kit that participates in SSP communication
- Some hubs have limited onboard AI (e.g., face detection, speech recognition, object tracking); within Solaria, these are treated as additional sensor inputs available to the client

### SSP (Solaria Standard Protocol)
The communication contract between client and hub. SSP defines:
- Message format and command vocabulary (motor, sensor, LED, sound, system commands)
- State reporting and event patterns
- Connection lifecycle (discovery, capability handshake, session, disconnect)

SSP is **transport-agnostic**. The same logical protocol operates identically whether the physical layer is Bluetooth Low Energy, WiFi TCP/UDP, USB serial, infrared, or any other byte-stream transport.

A single client instance may maintain concurrent connections to multiple hubs over different transports simultaneously, enabling multi-device coordination from one program.

---

## Table of Contents

1. [The Hybrid Architecture Model](#the-hybrid-architecture-model)
2. [TYPE 1: Native SSP Hardware](#type-1-native-ssp-hardware)
3. [TYPE 2: Protocol Bridge Hardware](#type-2-protocol-bridge-hardware)
4. [Solaria Standard Protocol (SSP)](#solaria-standard-protocol-ssp)
5. [Data Flow Diagrams](#data-flow-diagrams)
6. [Client Platform Architecture](#client-platform-architecture)
7. [Repository Structure](#repository-structure)
8. [The Generic ESP32 Firmware: Highest-Leverage Asset](#the-generic-esp32-firmware-highest-leverage-asset)
9. [Separation of Concerns](#separation-of-concerns)
10. [Building a New Integration](#building-a-new-integration)

---

## The Hybrid Architecture Model

The Solaria ecosystem is built around a four-layer model with a clear separation of concerns. The SSP sits in the middle, acting as the universal translator between what the user sees (the client) and what the robot does (the hardware). The critical architectural insight is that hardware falls into two fundamentally different categories based on firmware openness, and Solaria handles each differently.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Layer 3: AI / Agent Layer (Future)                                         │
│  Natural language → SSP commands. Hardware-agnostic intelligence.           │
│  GUI Electronics Configuration: visual drag-and-drop hardware setup.       │
├─────────────────────────────────────────────────────────────────────────────┤
│  Layer 2: Clients                                                           │
│  App Inventor │ MicroBlocks │ Python │ Scratch™ │ Web (JS) │ MakeCode®        │
├─────────────────────────────────────────────────────────────────────────────┤
│  Layer 1: SSP + Bridge Layer                                                │
│  ┌─────────────────────────────┐  ┌──────────────────────────────────────┐  │
│  │  TYPE 1: Native SSP FW      │  │  TYPE 2: Protocol Bridge Libraries   │  │
│  │  (Runs on open hardware)    │  │  (Translates SSP ↔ proprietary)      │  │
│  │  • ESP32 Solaria Firmware   │  │  • SPIKE Prime protocol lib          │  │
│  │  • micro:bit™ SSP firmware   │  │  • Powered Up (LWP3) protocol lib    │  │
│  │  • StackChan SSP firmware   │  │  • toio protocol lib                 │  │
│  │  • Raspberry Pi SSP service │  │  • uGot protocol lib                 │  │
│  │  • CyberBrick SSP firmware  │  │                                      │  │
│  │  • mBot2/CyberPi SSP fw     │  │                                      │  │
│  └─────────────────────────────┘  └──────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────────────┤
│  Layer 0: Hardware                                                          │
│  Motors │ Sensors │ Actuators │ LEDs │ Speakers │ Displays                  │
└─────────────────────────────────────────────────────────────────────────────┘
```

**The key insight:** Solaria owns Layer 1. Everything above (clients) and below (hardware) is interchangeable from the protocol's perspective. The Hybrid Architecture ensures that both open and closed hardware can participate in the ecosystem, while maximizing the leverage of each development dollar spent. Importantly, "interchangeable" applies at the infrastructure level — a developer building firmware or a client extension only needs to speak SSP. It does not mean student code is portable across client platforms; each platform has its own blocks and interaction model, designed to feel native to that environment.

---

## TYPE 1: Native SSP Hardware

### Definition

TYPE 1 devices are hardware platforms where you **CAN flash custom firmware**. These devices run SSP firmware natively — they speak the Solaria Standard Protocol directly over their communication interfaces (BLE, WiFi, or Serial).

### Hardware Platforms (TYPE 1)

| Platform | MCU / Base | Transport | Notes |
|----------|-----------|-----------|-------|
| **Generic ESP32 "Solaria Firmware"** | ESP32 / ESP32-S3 | BLE + WiFi | Universal firmware for any cheap robot chassis |
| **BBC micro:bit™ / MicroBlocks** | nRF52833 | BLE | Open source, millions deployed in schools globally |
| **StackChan (M5Stack)** | ESP32 | BLE + WiFi | Desktop companion robot, open firmware |
| **Raspberry Pi** | ARM Linux | WiFi + BLE + Serial | Runs SSP as a systemd service |
| **CyberBrick** | MicroPython (ESP32) | BLE + WiFi | Open-source controller core, modular building blocks |
| **Makeblock mBot2 (CyberPi)** | ESP32 (Arduino/PlatformIO) | BLE + WiFi | Open-source Arduino library (GPL-3.0), popular in schools |

### How It Works

Once the SSP firmware is flashed onto a TYPE 1 device, **ANY client that speaks SSP can connect directly**. There is zero per-client work required. The firmware handles:

1. Advertising/discovery (BLE service UUID, mDNS, or serial handshake)
2. Capability declaration (reporting what motors, sensors, etc. are attached)
3. SSP command parsing and execution
4. Sensor data streaming back to the client

### Development Economics

Building a TYPE 1 firmware is a **one-time investment** that instantly unlocks connectivity with ALL existing and future SSP clients:

- **Effort:** 100–200 hours
- **Cost:** $1,000–$2,500 (hardware + AI tooling + testing)
- **Ongoing:** Zero per-client maintenance
- **ROI multiplier:** 1 firmware × N clients = N integrations for free

---

## TYPE 2: Protocol Bridge Hardware

### Definition

TYPE 2 devices have **CLOSED firmware that you CANNOT modify**. They use proprietary communication protocols. To integrate them into Solaria, a translation layer must convert SSP commands into the device's native protocol and translate sensor data back into SSP format.

### Hardware Platforms (TYPE 2)

| Platform | Protocol Type | Transport | Notes |
|----------|--------------|-----------|-------|
| **LEGO® SPIKE™ Prime** | Proprietary BLE protocol | BLE | Closed firmware, partially documented |
| **LEGO Powered Up (LWP3)** | LEGO Wireless Protocol 3 | BLE | Documented BLE protocol, multiple hub types |
| **Sony® toio™** | Published BLE specification | BLE | Precision cubes, well-documented spec |
| **UBTECH® uGot** | Proprietary protocol | BLE / WiFi | Modular AI robotics kit, undocumented |

### How It Works

Because the hardware firmware cannot be modified, the translation logic must live on the **client side**. This creates a two-layer structure within each client:

1. **Protocol Library (built once per hardware):** The core translation logic that converts SSP ↔ proprietary protocol. This is hardware-specific and client-agnostic in its logic, but must be implemented in a language compatible with the target client.
2. **Client Wrapper (built per hardware × client combination):** A thin adapter that packages the protocol library into the specific client platform's extension/plugin format.

### Development Economics

TYPE 2 hardware requires **linear investment** — each new client platform requires a new wrapper:

- **Protocol Library (one-time per hardware):** 150–300 hours, $1,500–$4,000
- **Client Wrapper (per hardware × client):** 40–80 hours, $500–$1,200
- **Total for one hardware across all 6 clients:** $4,500–$11,200

### App Inventor Packaging Strategy

For MIT App Inventor specifically, TYPE 2 hardware results in **separate `.aix` extension files**:

- `Solaria.aix` — Universal SSP client (connects to any TYPE 1 device)
- `SolariaSpikePrime.aix` — SPIKE Prime protocol bridge wrapper
- `SolariaPoweredUp.aix` — Powered Up protocol bridge wrapper
- `SolariaToio.aix` — Sony® toio™ protocol bridge wrapper
- *(etc.)*

The recommendation is **multiple `.aix` files** rather than one monolithic extension, for modularity and download size.

---

## Solaria Standard Protocol (SSP)

SSP is the transport-agnostic language spoken between Layer 2 (clients) and Layer 1 (bridges/firmware). It is designed to be simple enough for a student to read, yet expressive enough to control complex robotic systems.

### The Four Concerns of SSP

#### 1. Device Discovery

How a client finds and connects to a Solaria-compatible device:

| Transport | Discovery Method |
|-----------|-----------------|
| BLE | Standardized advertising name prefix (`Solaria-*`) and service UUID |
| WiFi | mDNS service type `_solaria._tcp` |
| Serial | Handshake sequence on connection (`SSP?` → `SSP!`) |

#### 2. Capability Declaration

Once connected, the device reports what hardware is attached:

```json
{
  "device": "esp32-robot-car",
  "firmware": "solaria-fw-esp32 v1.2.0",
  "ports": [
    {"id": "A", "type": "motor", "features": ["speed", "position"]},
    {"id": "B", "type": "motor", "features": ["speed", "position"]},
    {"id": "C", "type": "ultrasonic", "features": ["distance"]},
    {"id": "D", "type": "line_sensor", "features": ["reflected"]}
  ]
}
```

Clients use this to dynamically build their UI, block palette, or API surface.

#### 3. Command Format

Standardized instruction messages from client → device:

```json
{"cmd": "motor.run", "port": "A", "speed": 75}
{"cmd": "motor.stop", "port": "A"}
{"cmd": "led.set", "port": "status", "color": [0, 255, 0]}
{"cmd": "sound.beep", "freq": 440, "duration": 500}
{"cmd": "servo.angle", "port": "S1", "angle": 90}
```

#### 4. Sensor Streaming

Device → client, continuous or on-change:

```json
{"event": "sensor", "port": "C", "type": "distance", "value": 23.5, "unit": "cm"}
{"event": "sensor", "port": "D", "type": "line", "value": 850}
{"event": "button", "port": "btn", "state": "pressed"}
```

> Full specification: [spec/SSP-v0.8.md](spec/SSP-v0.8.md)

---

## Data Flow Diagrams

### TYPE 1: Native SSP (Direct Connection)

```
┌──────────────────┐         SSP over BLE/WiFi/Serial          ┌──────────────────┐
│                  │ ════════════════════════════════════════> │                  │
│   CLIENT         │                                           │   HARDWARE       │
│   (Any platform) │ <════════════════════════════════════════ │   (SSP Firmware) │
│                  │         SSP sensor events                 │                  │
└──────────────────┘                                           └──────────────────┘

Example: Python SDK ←→ ESP32 Robot Car running Solaria Firmware

Step 1: Client discovers device via BLE scan (finds "Solaria-ESP32-Car")
Step 2: Client connects, receives capability declaration
Step 3: Client sends SSP commands: {"cmd": "motor.run", "port": "A", "speed": 50}
Step 4: Hardware executes command, streams sensor data back
Step 5: Client receives: {"event": "sensor", "port": "C", "type": "distance", "value": 15.2}
```

### TYPE 2: Protocol Bridge (Translated Connection)

```
┌──────────────────┐    SSP     ┌───────────────────┐   Proprietary    ┌──────────────────┐
│                  │ ────────>  │  PROTOCOL BRIDGE  │ ═══════════════> │                  │
│   CLIENT CORE    │            │  (inside client)  │                  │   CLOSED         │
│   (SSP logic)    │ <────────  │  Translates:      │ <═══════════════ │   HARDWARE       │
│                  │   SSP      │  SSP ↔ Proprietary│   Proprietary    │                  │
└──────────────────┘            └───────────────────┘                  └──────────────────┘

Example: App Inventor + SolariaSpikePrime.aix ←→ LEGO® SPIKE™ Prime Hub

Step 1: Wrapper scans for SPIKE Prime hub (proprietary BLE advertising)
Step 2: Wrapper connects using SPIKE Prime's proprietary handshake
Step 3: Client sends SSP: {"cmd": "motor.run", "port": "A", "speed": 75}
Step 4: Wrapper translates to SPIKE Prime protocol bytes and sends over BLE
Step 5: SPIKE Prime executes, sends proprietary sensor packet back
Step 6: Wrapper translates to SSP: {"event": "sensor", "port": "C", "type": "color", "value": "red"}
Step 7: Client receives standard SSP event (identical format to TYPE 1)
```

### Combined Ecosystem View

```
                    ┌─────────────────────────────────────────────────┐
                    │              CLIENT PLATFORM                    │
                    │  ┌───────────────────────────────────────────┐  │
                    │  │  Universal SSP Client Module              │  │
                    │  │  (connects to ANY Type 1 device)          │  │
                    │  └───────────────────┬───────────────────────┘  │
                    │                      │                          │
                    │  ┌──────────┐ ┌──────────┐ ┌──────────┐         │
                    │  │SPIKE     │ │Powered Up│ │toio      │ ...     │
                    │  │Wrapper   │ │Wrapper   │ │Wrapper   │         │
                    │  └────┬─────┘ └────┬─────┘ └────┬─────┘         │
                    └───────┼────────────┼────────────┼───────────────┘
                            │            │            │
            ┌───────────────┼────────────┼────────────┼───────────────┐
            │               ▼            ▼            ▼               │
            │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐     │
            │  │ESP32    │  │SPIKE    │  │Powered  │  │toio     │     │
            │  │(Type 1) │  │Prime    │  │Up Hub   │  │Cube     │     │
            │  │SSP FW   │  │(Type 2) │  │(Type 2) │  │(Type 2) │     │
            │  └─────────┘  └─────────┘  └─────────┘  └─────────┘     │
            │                     HARDWARE LAYER                      │
            └─────────────────────────────────────────────────────────┘
```

---

## Client Platform Architecture

Each client platform implements the same logical structure, adapted to the platform's native extension/plugin mechanism:

| Client Platform | Extension Format | Language | Transport API | TYPE 1 Support | TYPE 2 Support |
|-----------------|-----------------|----------|---------------|----------------|----------------|
| MIT App Inventor | `.aix` Java extension | Java | Android BLE API | `Solaria.aix` | Separate `.aix` per hardware |
| MicroBlocks | Built-in library | MicroBlocks | On-device BLE | Native blocks | Wrapper blocks per hardware |
| Python | `pip` package | Python | `bleak` / `socket` | `solaria` package | Sub-modules per hardware |
| Scratch™ | Scratch™ extension | JavaScript | Web Bluetooth | Solaria extension | Sub-extensions per hardware |
| Web (JavaScript) | JS library / npm | JavaScript | Web Bluetooth API | `solaria.js` | Modules per hardware |
| MakeCode® | MakeCode® extension | TypeScript | micro:bit™ radio/BLE | Solaria package | Packages per hardware |
| Arduino IDE | Arduino library | C/C++ | Serial / BLE | `SolariaSSP` library | Modules per hardware |

### Client Programming Models & Platform Constraints

While the SSP **wire format** is unified across all clients, the **user-facing block logic and programming model** differs by host environment. "Same capabilities, every platform" is the accurate framing: the robot can do the same things regardless of which client a student uses, but the blocks, event patterns, and code structure are intentionally platform-specific. SSP unifies the infrastructure; each client extension is designed to feel native to its host environment.

#### Two programming models

| Model | Platform | Pattern | Example |
|---|---|---|---|
| **Event-driven** | App Inventor | Fire-and-forget command + `When…Read` event handler block | `GetColor()` fires `sensor.read`; `When ColorRead` block receives the value |
| **Synchronous reporter** | Scratch (TurboWarp) | `async` reporter awaits the SSP response via `requestEvent` before returning | `color` reporter sends `sensor.read` and awaits the matching event |

System metrics (battery, temperature, charging) in Scratch use a **command + cache** variant: a
`request*` command fires `system.read`; `routeSSP` caches the value; the reporter returns the cache
synchronously. This is a workaround for the latency of battery reads and mirrors the App Inventor
fire-and-forget pattern most closely.

A client code block cannot be translated 1:1 across these models; the underlying protocol messages are
identical, but the host-environment interaction pattern differs.

#### Platform-specific hard limits

| Capability | App Inventor | Scratch (Web Bluetooth) | Reason |
|---|---|---|---|
| BLE device scanning | `StartScanning`, `HubList`, `ConnectToHub(index)` | Browser-native chooser replaces all scanning blocks | Web Bluetooth prohibits programmatic scan/enumeration for security |
| RSSI (signal strength) | `GetRSSI` / `RSSIRead` (Android BLE stack) | Not available | Web Bluetooth API does not expose connected-device RSSI |
| Named connection flow | Explicit index-based `ConnectToHub` | Single user-gesture connect | Same browser security model |

These are platform constraints, not SSP gaps. The wire protocol is identical; only the host API
differs.

#### Naming note

The App Inventor component is named **`Connectivity`** (a software-object class name:
`LegoSpikeConnectivity`). The Scratch palette label is **`Connection`** (a user-facing section header
for students). This is an intentional, platform-appropriate difference — the Java class cannot be
renamed without breaking existing `.aia` project files.

---

## Repository Structure

The Solaria ecosystem is modular. Each component lives in its own repository with a clear naming convention:

### Repository Naming Convention

| Category | Pattern | Example | Contents |
|----------|---------|---------|----------|
| **Hub** | `solaria-hub` | `solaria-hub` | SSP spec, architecture, roadmap, ecosystem coordination |
| **Type 1 Firmware** | `solaria-fw-<hardware>` | `solaria-fw-esp32` | Firmware source, build configs, flashing tools |
| **Type 2 Protocol Library** | `solaria-lib-<hardware>` | `solaria-lib-spike-prime` | Core protocol translation logic, tests |
| **Universal Client** | `solaria-client-<platform>` | `solaria-client-python` | Platform SSP client (connects to all Type 1) |
| **Type 2 Wrapper** | `solaria-<platform>-<hardware>` | `solaria-appinventor-spike-prime` | Packages protocol lib into client format |
| **Spec** | `solaria-hub/spec/` | `SSP-v0.6.md` | Protocol specification documents |
| **Tools** | `solaria-tools` | `solaria-tools` | CLI utilities, testing harnesses, simulators |

### Current Repositories

| Repository | Status | Description |
|------------|--------|-------------|
| `solaria-hub` | Active | Central coordination, SSP spec, docs |
| `solaria-appinventor-spike-prime` | Released (Gen 1) | SPIKE Prime integration (combined lib + App Inventor wrapper) |

### Planned Repositories (Gen 2+)

| Repository | Priority | Unlocks |
|------------|----------|---------|
| `solaria-fw-esp32` | **Highest** | All 6 clients × Generic ESP32 (free) |
| `solaria-fw-microbit` | High | All 6 clients × micro:bit™ (free) |
| `solaria-fw-stackchan` | Medium | All 6 clients × StackChan (free) |
| `solaria-fw-rpi` | Medium | All 6 clients × Raspberry Pi (free) |
| `solaria-client-python` | High | Python access to all Type 1 hardware |
| `solaria-client-web` | High | Browser access to all Type 1 hardware |
| `solaria-client-scratch` | Medium | Scratch™ access to all Type 1 hardware |
| `solaria-client-microblocks` | Medium | MicroBlocks access to all Type 1 hardware |
| `solaria-client-makecode` | Lower | MakeCode® access to all Type 1 hardware |
| `solaria-lib-powered-up` | Medium | Core LWP3 protocol translation |
| `solaria-lib-toio` | Medium | Core toio BLE protocol translation |
| `solaria-lib-ugot` | Lower | Core uGot protocol translation |
| `solaria-fw-cyberbrick` | Lower | SSP firmware for CyberBrick Multi-Function Core Board |
| `solaria-fw-mbot2` | Lower | SSP firmware for Makeblock mBot2 (CyberPi ESP32) |

---

## The Generic ESP32 Firmware: Highest-Leverage Asset

The Generic ESP32 "Solaria Firmware" (`solaria-fw-esp32`) is the single highest-leverage component in the entire ecosystem. Here is why:

**The Problem:** Most affordable robot kits ($15–$50 from AliExpress, Amazon, etc.) ship with proprietary apps or require Arduino IDE knowledge. Students and educators cannot easily connect them to visual programming environments.

**The Solution:** Flash the Solaria Firmware onto the ESP32 inside any cheap robot. It instantly becomes a native SSP device, connectable from App Inventor, Python, Scratch™, Web Bluetooth, or any other SSP client — with zero additional development.

**The Math:**
- Cost to build: ~$2,000 (one-time)
- Hardware it unlocks: Thousands of ESP32-based robot kits worldwide
- Clients it works with: All 6 platforms (once their Universal Clients exist)
- Per-client wrapper cost: $0 (it's TYPE 1)
- Effective integrations created: 6 clients × unlimited hardware = ∞

This is why the ESP32 firmware is the recommended **first priority** for Gen 2 development.

---

## Separation of Concerns

The layered architecture enforces strict separation of concerns:

| Layer | Knows About | Does NOT Know About |
|-------|-------------|---------------------|
| **Hardware / Firmware** | Hardware registers, GPIO, motor drivers, SSP parsing | Which client sent the command, UI details |
| **SSP Protocol** | Message format, transport framing, capability schema | Specific hardware registers, specific client UI |
| **Client Extension** | User-facing blocks/API, SSP generation, transport connection — **platform-specific by design** | Hardware internals, motor driver registers, other clients' block patterns |
| **Protocol Bridge (Type 2)** | Both SSP format AND proprietary protocol format | Client UI, other hardware types |
| **Student Code** | Platform-native blocks and interaction patterns (stateful in App Inventor, event-driven in Scratch) | SSP wire format, hardware registers — **student code is not portable across clients** |
| **AI Agent (Gen 3)** | Intent parsing, goal decomposition, SSP command generation, GUI electronics configuration | Transport details, hardware registers, client UI — **this layer is where platform differences fade** |

**Architectural guarantees:**

- A new TYPE 1 firmware requires **ZERO changes** to existing clients.
- A new Universal Client requires **ZERO changes** to existing firmware.
- A new TYPE 2 protocol library requires wrappers per client, but **ZERO changes** to the client's core SSP logic.
- The AI agent (Gen 3) will work with **ANY** bridge + **ANY** client combination — and is the layer where platform-specific block differences become invisible to the student.

**What SSP does not guarantee:** Student code written for App Inventor cannot be run on Scratch without rewriting. Each client platform has its own blocks, its own event model, and its own interaction patterns. SSP guarantees that the robot capabilities are consistent; it does not make code portable.

---

## Building a New Integration

### Adding a TYPE 1 Device (New Firmware)

1. Read the SSP specification: [spec/SSP-v0.6.md](spec/SSP-v0.6.md)
2. Create repository: `solaria-fw-<hardware>`
3. Implement transport listener (BLE GATT server, WiFi socket, or Serial)
4. Implement SSP discovery advertising
5. Implement capability declaration for attached hardware
6. Implement SSP command parser → hardware driver mapping
7. Implement sensor streaming (continuous or on-change)
8. Test with the reference App Inventor client (Solaria.aix)
9. Document pin mappings, supported hardware configurations
10. Submit PR to `solaria-hub` to add your firmware to the ecosystem registry

### Adding a TYPE 2 Device (New Protocol Library)

1. Read the SSP specification: [spec/SSP-v0.6.md](spec/SSP-v0.6.md)
2. Reverse-engineer or document the proprietary protocol
3. Create repository: `solaria-lib-<hardware>`
4. Implement bidirectional translation: SSP ↔ proprietary protocol
5. Write comprehensive protocol documentation
6. Create at least one client wrapper (recommended: App Inventor or Python)
7. Test end-to-end with physical hardware
8. Submit PR to `solaria-hub` to add your library to the ecosystem registry

### Adding a New Client Platform

1. Read the SSP specification: [spec/SSP-v0.6.md](spec/SSP-v0.6.md)
2. Create repository: `solaria-client-<platform>`
3. Implement transport connection (BLE/WiFi/Serial for the target platform)
4. Implement SSP capability declaration parsing
5. Build user-facing interface (blocks, API methods, UI components)
6. Implement SSP command generation from user actions
7. Implement sensor event handling and display
8. Test with at least one TYPE 1 device (ESP32 recommended)
9. Submit PR to `solaria-hub` to add your client to the ecosystem registry

---

## Summary

The Solaria Hybrid Architecture maximizes development efficiency by recognizing that hardware falls into two categories:

- **TYPE 1 (Native SSP):** Open hardware where firmware investment pays dividends across ALL clients simultaneously.
- **TYPE 2 (Protocol Bridge):** Closed hardware requiring per-client wrappers, with costs scaling linearly.

This insight drives the project's strategic priorities: invest in TYPE 1 firmware first (especially the Generic ESP32) to create maximum ecosystem value, then selectively build TYPE 2 integrations based on community demand and funding.

**The ecosystem promise:** Every supported hardware/software combination exposes the same robot capabilities. A teacher can choose App Inventor because their students know it, or Scratch because it is already in the curriculum, and the learning goals are consistent. A student who learns motor control and sensor reading on one platform understands the concepts that transfer to any other. When Gen 3 (the AI agent layer) is complete, even the interaction model converges — students describe what they want in natural language, and the platform differences disappear entirely.
