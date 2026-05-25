# Solaria Roadmap

> **Version:** 2.0  
> **Last Updated:** 2026-05-25  
> **Status:** Living document — priorities shift based on community votes, sponsorship, and technical feasibility.

---

## Table of Contents

1. [Strategic Phases](#strategic-phases)
2. [Hardware Target List](#hardware-target-list)
3. [Client Platform Roadmap](#client-platform-roadmap)
4. [Cost Estimation Framework](#cost-estimation-framework)
5. [Full Hardware × Client Matrix](#full-hardware--client-matrix)
6. [Dependency Map](#dependency-map)
7. [Total Ecosystem Cost Summary](#total-ecosystem-cost-summary)
8. [Recommended Build Order](#recommended-build-order)
9. [Community Voting & Sponsorship](#community-voting--sponsorship)
10. [Changelog](#changelog)

---

## Strategic Phases

### Phase 1 — Foundation ✅

**Status:** Shipped  
**Delivered:** 2025–2026

The core architecture is proven end-to-end. A student can open MIT App Inventor on their phone, connect to a LEGO SPIKE Prime hub over BLE, and control motors and read sensors using the Solaria Standard Protocol.

| Deliverable | Status | Repository |
|-------------|--------|------------|
| SSP v0.1 specification | ✅ Complete | `solaria-hub/spec/` |
| LEGO SPIKE Prime BLE bridge | ✅ Released | `solaria-bridge-spike-prime` |
| App Inventor client extension (.aix) | ✅ Released | `solaria-bridge-spike-prime` |
| End-to-end demo | ✅ Working | — |

**Actual Phase 1 costs (reference baseline):**

| Category | Cost |
|----------|------|
| AI tooling (Claude Code, Manus, API credits) | $500+ |
| Hardware (SPIKE Prime hub, motors, sensors, cables) | $400+ |
| Development time | 100+ hours |
| **Total** | **>$1,000** |

---

### Phase 2 — Ecosystem Expansion 🚧

**Status:** Active Development & Community Voting

Phase 2 expands the ecosystem along two axes simultaneously:

1. **Hardware axis:** Adding new TYPE 1 firmware and TYPE 2 protocol libraries.
2. **Client axis:** Building Universal SSP Clients for new programming platforms.

The full scope, costs, and dependencies are detailed in the [Hardware × Client Matrix](#full-hardware--client-matrix) below.

---

### Phase 3 — AI / Agent Layer 🔮

**Status:** Planned (target: after 3+ bridges are stable)

Introducing hardware-agnostic intelligence on top of SSP:

- **Vibe Coding:** Natural language → SSP command translation (e.g., "Drive forward until you see a red object" → motor + color sensor commands)
- **GUI Electronics Configuration:** Visual drag-and-drop interface for hardware setup — select your microcontroller, sensors, and motors; Solaria generates the correct wiring diagrams, pin mappings, and code scaffolding automatically. No more hunting through datasheets.
- **LLM-powered processing:** Local (M5Stack Module LLM) or cloud-based inference
- **Cross-platform intelligence:** Works with ANY bridge + ANY client combination because it generates standard SSP
- **Platform integration:** Potential integration with MIT App Inventor's AI components
- **Agentic AI overlay:** Context-aware assistant that understands the full hardware × software matrix and guides users through building, debugging, and extending projects

**Prerequisites:** At least 3 stable hardware bridges and 2 stable client platforms.

---

### Phase 4 — Solaria Flagship Robot 🌱

**Status:** Vision

A reference hardware design that showcases the full Solaria stack in one cohesive product:

- Custom PCB (ESP32-S3 based, Native SSP TYPE 1)
- BLE sphere core with wireless charging
- Solarpunk aesthetic, partially 3D-printable enclosure
- Ships with SSP firmware pre-flashed
- Works out-of-box with App Inventor, Python, Web Bluetooth, and AI agent
- The "iPhone of the ecosystem" — demonstrates everything Solaria can do

**Estimated cost:** $15,000–$25,000 (PCB design, tooling, small production run, certification)

---

## Hardware Target List

Solaria targets 9 hardware platforms across two architecture types:

### TYPE 1: Native SSP Hardware (Open Firmware)

These devices can be flashed with custom SSP firmware. Once firmware is built, ALL clients work automatically.

| # | Platform | MCU / Base | Transport | Key Advantage | Est. FW Cost |
|---|----------|-----------|-----------|---------------|-------------|
| 1 | **Generic ESP32 "Solaria Firmware"** | ESP32 / ESP32-S3 | BLE + WiFi | Universal — works with any cheap robot | $1,500–$2,500 |
| 2 | **BBC micro:bit / MicroBlocks** | nRF52833 | BLE | Millions in schools, open source | $1,000–$1,500 |
| 3 | **StackChan (M5Stack)** | ESP32 | BLE + WiFi | Desktop companion, active community | $1,500–$2,000 |
| 4 | **Raspberry Pi** | ARM Linux | WiFi + BLE + Serial | Complex robots, Linux ecosystem | $1,500–$2,500 |
| 5 | **CyberBrick** | MicroPython (ESP32) | BLE + WiFi | Modular building blocks, open-source core | $1,000–$2,000 |
| 6 | **Makeblock mBot2 (CyberPi)** | ESP32 (Arduino) | BLE + WiFi | Open-source library (GPL-3.0), popular in schools | $1,000–$2,000 |

**Total TYPE 1 firmware investment:** $7,500–$12,500

### TYPE 2: Protocol Bridge Hardware (Closed Firmware)

These devices have proprietary firmware. A protocol library must be reverse-engineered, then wrapped into each client.

| # | Platform | Protocol | Transport | Documentation Status | Est. Lib Cost |
|---|----------|----------|-----------|---------------------|--------------|
| 5 | **LEGO SPIKE Prime** | Proprietary BLE | BLE | Partially documented | ✅ Done ($1,000+) |
| 6 | **LEGO Powered Up (LWP3)** | LEGO Wireless Protocol 3 | BLE | Well-documented | $2,000–$3,000 |
| 7 | **Sony toio** | Published BLE spec | BLE | Fully documented | $1,500–$2,500 |
| 8 | **UBTECH uGot** | Proprietary | BLE / WiFi | Undocumented (reverse-eng. needed) | $2,500–$4,000 |

**Total TYPE 2 protocol library investment (remaining 3):** $6,000–$9,500

---

## Client Platform Roadmap

SSP is designed to be client-agnostic. Each Universal Client is a one-time investment that unlocks connectivity to ALL TYPE 1 hardware.

| # | Client Platform | Format | Language | Status | Est. Cost | Priority |
|---|----------------|--------|----------|--------|-----------|----------|
| 1 | **MIT App Inventor** | `.aix` extension | Java | ✅ Shipped | Done | — |
| 2 | **Python** | `pip` package | Python | 📋 Planned | $1,200–$2,000 | High |
| 3 | **Web (JavaScript)** | npm / CDN library | JavaScript | 📋 Planned | $1,200–$2,000 | High |
| 4 | **MicroBlocks** | Built-in library | MicroBlocks | 📋 Planned | $1,000–$1,500 | Medium |
| 5 | **Scratch** | Scratch extension | JavaScript | 💡 Proposed | $1,500–$2,000 | Medium |
| 6 | **MakeCode** | MakeCode package | TypeScript | 💡 Proposed | $1,200–$1,800 | Lower |
| 7 | **Arduino IDE** | Arduino library | C/C++ | 💡 Proposed | $1,000–$1,500 | Proposed |

**Total Universal Client investment (remaining 5):** $6,100–$9,300

### Client Priority Rationale

- **Python** is highest priority because it serves advanced students, university courses, and the maker community. It also enables scripting and automation workflows.
- **Web (JavaScript)** is equally high priority because Web Bluetooth enables zero-install experiences — students just open a URL and connect to their robot.
- **MicroBlocks** is a natural fit because it already runs on micro:bit and ESP32, and the MicroBlocks team is aligned with open education.
- **Scratch** has the largest K-8 user base globally but requires more complex integration (Scratch Link or custom extension server).
- **MakeCode** is lower priority because its user base overlaps heavily with micro:bit (which already gets MicroBlocks support).

---

## Cost Estimation Framework

### Baseline Reference

All estimates are calibrated against the actual SPIKE Prime Phase 1 development:

> **SPIKE Prime (1 hardware × 1 client):** $500+ AI tooling + $400+ hardware + 100+ hours dev time = **>$1,000 total**

### Standard Cost Categories

| Work Type | Description | Hours | Cost Range | Frequency |
|-----------|-------------|-------|-----------|-----------|
| **Universal Client** | Core SSP client for one platform (discovery, connection, commands, events) | 80–150 hrs | $800–$2,000 | Once per platform |
| **Type 1 Firmware** | Native SSP firmware for open hardware (transport, capability, commands, sensors) | 100–200 hrs | $1,000–$2,500 | Once per hardware |
| **Type 2 Protocol Library** | Reverse-engineering + core translation logic for closed hardware | 150–300 hrs | $1,500–$4,000 | Once per hardware |
| **Type 2 Client Wrapper** | Packaging protocol lib into a specific client platform's format | 40–80 hrs | $500–$1,200 | Per hardware × client |
| **Annual Maintenance** | Bug fixes, protocol updates, dependency updates per active repo | 20–40 hrs | $200–$500 | Per year per repo |

### Cost Breakdown Components

Each estimate includes:

- **Hardware:** Physical devices, accessories, cables, adapters for development and testing
- **AI Tooling:** Claude Code subscription, Manus sessions, API credits (a core part of the "vibe coding" workflow)
- **Development Time:** Protocol research, implementation, integration, debugging, testing
- **Documentation:** Tutorials, API references, example projects
- **Testing:** End-to-end validation across platforms and OS versions

---

## Full Hardware × Client Matrix

This is the complete integration matrix showing every possible hardware × client combination, its effort category, estimated cost, and dependencies.

### Matrix Legend

| Symbol | Meaning |
|--------|---------|
| ✅ | Shipped and available |
| **Free** | Zero additional cost (TYPE 1 firmware + Universal Client already exist) |
| **FW** | Requires TYPE 1 firmware to be built first |
| **Lib** | Requires TYPE 2 protocol library to be built first |
| **Wrap** | Requires a client-specific wrapper (protocol lib must exist) |
| **UC** | Requires Universal Client to be built first |

### Detailed Matrix: Hours and Cost per Cell

#### MIT App Inventor (✅ Universal Client Shipped)

| Hardware | Type | Effort | Hours | Cost | Dependencies | Status |
|----------|------|--------|-------|------|--------------|--------|
| Generic ESP32 | 1 | Free (after FW) | 0 | $0 | `solaria-fw-esp32` | Blocked on FW |
| BBC micro:bit | 1 | Free (after FW) | 0 | $0 | `solaria-fw-microbit` | Blocked on FW |
| StackChan | 1 | Free (after FW) | 0 | $0 | `solaria-fw-stackchan` | Blocked on FW |
| Raspberry Pi | 1 | Free (after FW) | 0 | $0 | `solaria-fw-rpi` | Blocked on FW |
| LEGO SPIKE Prime | 2 | Wrapper | — | — | — | ✅ Shipped |
| LEGO Powered Up | 2 | Wrapper | 40–80 | $500–$1,200 | `solaria-lib-powered-up` | Planned |
| Sony toio | 2 | Wrapper | 40–60 | $500–$900 | `solaria-lib-toio` | Planned |
| UBTECH uGot | 2 | Wrapper | 50–80 | $600–$1,200 | `solaria-lib-ugot` | Planned |
| CyberBrick | 1 | Free (after FW) | 0 | $0 | `solaria-fw-cyberbrick` | Blocked on FW |
| mBot2 (CyberPi) | 1 | Free (after FW) | 0 | $0 | `solaria-fw-mbot2` | Blocked on FW |

#### Python (Universal Client: $1,200–$2,000)

| Hardware | Type | Effort | Hours | Cost | Dependencies | Status |
|----------|------|--------|-------|------|--------------|--------|
| Generic ESP32 | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-esp32`, `solaria-client-python` | Planned |
| BBC micro:bit | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-microbit`, `solaria-client-python` | Planned |
| StackChan | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-stackchan`, `solaria-client-python` | Planned |
| Raspberry Pi | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-rpi`, `solaria-client-python` | Planned |
| LEGO SPIKE Prime | 2 | Wrapper | 40–80 | $500–$1,200 | `solaria-lib-spike-prime`, `solaria-client-python` | Planned |
| LEGO Powered Up | 2 | Wrapper | 40–80 | $500–$1,200 | `solaria-lib-powered-up`, `solaria-client-python` | Planned |
| Sony toio | 2 | Wrapper | 40–60 | $500–$900 | `solaria-lib-toio`, `solaria-client-python` | Planned |
| UBTECH uGot | 2 | Wrapper | 50–80 | $600–$1,200 | `solaria-lib-ugot`, `solaria-client-python` | Planned |
| CyberBrick | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-cyberbrick`, `solaria-client-python` | Planned |
| mBot2 (CyberPi) | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-mbot2`, `solaria-client-python` | Planned |

#### Web / JavaScript (Universal Client: $1,200–$2,000)

| Hardware | Type | Effort | Hours | Cost | Dependencies | Status |
|----------|------|--------|-------|------|--------------|--------|
| Generic ESP32 | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-esp32`, `solaria-client-web` | Planned |
| BBC micro:bit | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-microbit`, `solaria-client-web` | Planned |
| StackChan | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-stackchan`, `solaria-client-web` | Planned |
| Raspberry Pi | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-rpi`, `solaria-client-web` | Planned |
| LEGO SPIKE Prime | 2 | Wrapper | 40–80 | $500–$1,200 | `solaria-lib-spike-prime`, `solaria-client-web` | Planned |
| LEGO Powered Up | 2 | Wrapper | 40–80 | $500–$1,200 | `solaria-lib-powered-up`, `solaria-client-web` | Planned |
| Sony toio | 2 | Wrapper | 40–60 | $500–$900 | `solaria-lib-toio`, `solaria-client-web` | Planned |
| UBTECH uGot | 2 | Wrapper | 50–80 | $600–$1,200 | `solaria-lib-ugot`, `solaria-client-web` | Planned |
| CyberBrick | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-cyberbrick`, `solaria-client-web` | Planned |
| mBot2 (CyberPi) | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-mbot2`, `solaria-client-web` | Planned |

#### MicroBlocks (Universal Client: $1,000–$1,500)

| Hardware | Type | Effort | Hours | Cost | Dependencies | Status |
|----------|------|--------|-------|------|--------------|--------|
| Generic ESP32 | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-esp32`, `solaria-client-microblocks` | Planned |
| BBC micro:bit | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-microbit`, `solaria-client-microblocks` | Planned |
| StackChan | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-stackchan`, `solaria-client-microblocks` | Planned |
| Raspberry Pi | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-rpi`, `solaria-client-microblocks` | Planned |
| LEGO SPIKE Prime | 2 | Wrapper | 50–80 | $600–$1,200 | `solaria-lib-spike-prime`, `solaria-client-microblocks` | Planned |
| LEGO Powered Up | 2 | Wrapper | 50–80 | $600–$1,200 | `solaria-lib-powered-up`, `solaria-client-microblocks` | Planned |
| Sony toio | 2 | Wrapper | 40–60 | $500–$900 | `solaria-lib-toio`, `solaria-client-microblocks` | Planned |
| UBTECH uGot | 2 | Wrapper | 50–80 | $600–$1,200 | `solaria-lib-ugot`, `solaria-client-microblocks` | Planned |
| CyberBrick | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-cyberbrick`, `solaria-client-microblocks` | Planned |
| mBot2 (CyberPi) | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-mbot2`, `solaria-client-microblocks` | Planned |

#### Scratch (Universal Client: $1,500–$2,000)

| Hardware | Type | Effort | Hours | Cost | Dependencies | Status |
|----------|------|--------|-------|------|--------------|--------|
| Generic ESP32 | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-esp32`, `solaria-client-scratch` | Planned |
| BBC micro:bit | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-microbit`, `solaria-client-scratch` | Planned |
| StackChan | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-stackchan`, `solaria-client-scratch` | Planned |
| Raspberry Pi | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-rpi`, `solaria-client-scratch` | Planned |
| LEGO SPIKE Prime | 2 | Wrapper | 50–80 | $600–$1,200 | `solaria-lib-spike-prime`, `solaria-client-scratch` | Planned |
| LEGO Powered Up | 2 | Wrapper | 50–80 | $600–$1,200 | `solaria-lib-powered-up`, `solaria-client-scratch` | Planned |
| Sony toio | 2 | Wrapper | 40–60 | $500–$900 | `solaria-lib-toio`, `solaria-client-scratch` | Planned |
| UBTECH uGot | 2 | Wrapper | 50–80 | $600–$1,200 | `solaria-lib-ugot`, `solaria-client-scratch` | Planned |
| CyberBrick | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-cyberbrick`, `solaria-client-scratch` | Planned |
| mBot2 (CyberPi) | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-mbot2`, `solaria-client-scratch` | Planned |

#### MakeCode (Universal Client: $1,200–$1,800)

| Hardware | Type | Effort | Hours | Cost | Dependencies | Status |
|----------|------|--------|-------|------|--------------|--------|
| Generic ESP32 | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-esp32`, `solaria-client-makecode` | Planned |
| BBC micro:bit | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-microbit`, `solaria-client-makecode` | Planned |
| StackChan | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-stackchan`, `solaria-client-makecode` | Planned |
| Raspberry Pi | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-rpi`, `solaria-client-makecode` | Planned |
| LEGO SPIKE Prime | 2 | Wrapper | 40–80 | $500–$1,200 | `solaria-lib-spike-prime`, `solaria-client-makecode` | Planned |
| LEGO Powered Up | 2 | Wrapper | 40–80 | $500–$1,200 | `solaria-lib-powered-up`, `solaria-client-makecode` | Planned |
| Sony toio | 2 | Wrapper | 40–60 | $500–$900 | `solaria-lib-toio`, `solaria-client-makecode` | Planned |
| UBTECH uGot | 2 | Wrapper | 50–80 | $600–$1,200 | `solaria-lib-ugot`, `solaria-client-makecode` | Planned |
| CyberBrick | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-cyberbrick`, `solaria-client-makecode` | Planned |
| mBot2 (CyberPi) | 1 | Free (after FW + UC) | 0 | $0 | `solaria-fw-mbot2`, `solaria-client-makecode` | Planned |

### Summary Matrix (Compact View)

This table shows the effort category for each combination at a glance:

| Hardware \ Client | App Inventor | Python | Web (JS) | MicroBlocks | Scratch | MakeCode |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|
| **Generic ESP32** (T1) | Free | Free | Free | Free | Free | Free |
| **BBC micro:bit** (T1) | Free | Free | Free | Free | Free | Free |
| **StackChan** (T1) | Free | Free | Free | Free | Free | Free |
| **Raspberry Pi** (T1) | Free | Free | Free | Free | Free | Free |
| **SPIKE Prime** (T2) | ✅ Done | Wrap | Wrap | Wrap | Wrap | Wrap |
| **Powered Up** (T2) | Wrap | Wrap | Wrap | Wrap | Wrap | Wrap |
| **Sony toio** (T2) | Wrap | Wrap | Wrap | Wrap | Wrap | Wrap |
| **UBTECH uGot** (T2) | Wrap | Wrap | Wrap | Wrap | Wrap | Wrap |
| **CyberBrick** (T1) | Free | Free | Free | Free | Free | Free |
| **mBot2 (CyberPi)** (T1) | Free | Free | Free | Free | Free | Free |

**Reading this table:**
- **Free** = Zero marginal cost once the firmware (row) and Universal Client (column) are built.
- **Wrap** = Requires a wrapper ($500–$1,200) but only after the Protocol Library for that hardware exists.
- The 24 "Free" cells (4 TYPE 1 × 6 clients) represent the massive leverage of the hybrid architecture.

---

## Dependency Map

Development must follow a strict dependency chain. Nothing can be built out of order.

### Tier 0: Foundation (✅ Complete)

```
SSP Specification v0.1 ──> SPIKE Prime Protocol Lib ──> App Inventor Wrapper
                                                         (= Phase 1 shipped)
```

### Tier 1: Infrastructure (Unlocks Everything)

These items have the highest ROI because they are prerequisites for many downstream items:

```
┌─────────────────────────────────────────────────────────────────────┐
│  TIER 1: Build these FIRST (each unlocks multiple downstream items) │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  solaria-fw-esp32          ──> Unlocks 6 client connections (Free)  │
│  solaria-client-python     ──> Unlocks 4 Type 1 HW + 5 Type 2 wrap │
│  solaria-client-web        ──> Unlocks 4 Type 1 HW + 5 Type 2 wrap │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Tier 2: Expansion

```
┌─────────────────────────────────────────────────────────────────────┐
│  TIER 2: Build after Tier 1 (expands coverage)                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  solaria-fw-microbit       ──> Unlocks 6 client connections (Free)  │
│  solaria-fw-stackchan      ──> Unlocks 6 client connections (Free)  │
│  solaria-fw-rpi            ──> Unlocks 6 client connections (Free)  │
│  solaria-lib-powered-up    ──> Unlocks 6 wrappers                   │
│  solaria-lib-toio          ──> Unlocks 6 wrappers                   │
│  solaria-client-microblocks──> Unlocks 4 Type 1 HW + 5 Type 2 wrap │
│  solaria-client-scratch    ──> Unlocks 4 Type 1 HW + 5 Type 2 wrap │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Tier 3: Long Tail

```
┌─────────────────────────────────────────────────────────────────────┐
│  TIER 3: Build on demand (community-driven)                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  solaria-lib-ugot          ──> Unlocks 6 wrappers                   │
│  solaria-lib-cyberbrick    ──> Unlocks 6 wrappers                   │
│  solaria-client-makecode   ──> Unlocks 4 Type 1 HW + 5 Type 2 wrap │
│  All Type 2 wrappers      ──> Individual integrations               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Dependency Rules

1. **A TYPE 1 "Free" cell requires:** The firmware for that row AND the Universal Client for that column.
2. **A TYPE 2 "Wrap" cell requires:** The Protocol Library for that row AND the Universal Client for that column.
3. **Universal Clients are independent** of each other (can be built in parallel).
4. **TYPE 1 Firmware projects are independent** of each other (can be built in parallel).
5. **TYPE 2 Protocol Libraries are independent** of each other (can be built in parallel).
6. **Wrappers depend on BOTH** their protocol library AND their target client.

---

## Total Ecosystem Cost Summary

### By Category

| Category | Count | Hours (range) | Cost (range) | Notes |
|----------|-------|---------------|-------------|-------|
| **Universal Clients** (remaining) | 5 | 400–750 hrs | $6,100–$9,300 | Python, Web, MicroBlocks, Scratch, MakeCode |
| **Type 1 Firmware** | 6 | 600–1,200 hrs | $7,500–$12,500 | ESP32, micro:bit, StackChan, RPi, CyberBrick, mBot2 |
| **Type 2 Protocol Libraries** (remaining) | 3 | 450–900 hrs | $6,000–$9,500 | Powered Up, toio, uGot |
| **Type 2 Client Wrappers** | 23 | 920–1,840 hrs | $11,500–$27,600 | 4 hardware × 6 clients minus 1 shipped |
| **Annual Maintenance** (all repos) | ~20 repos | 400–800 hrs/yr | $4,000–$10,000/yr | Ongoing |
| | | | | |
| **TOTAL (one-time build)** | — | **2,370–4,690 hrs** | **$31,100–$58,900** | |
| **Midpoint estimate** | — | **~3,500 hrs** | **~$45,000** | |

### By Phase

| Phase | Scope | Est. Cost | Cumulative |
|-------|-------|-----------|-----------|
| Phase 1 (✅ Done) | 1 hardware × 1 client | $1,000+ | $1,000+ |
| Phase 2a (Tier 1) | ESP32 FW + Python UC + Web UC | $4,500–$6,500 | $5,500–$7,500 |
| Phase 2b (Tier 2) | 3 more FW + 2 more UC + 2 protocol libs | $10,000–$16,000 | $15,500–$23,500 |
| Phase 2c (Tier 3) | Remaining libs + all wrappers | $18,000–$42,000 | $33,500–$65,500 |
| Phase 3 (AI) | Agent layer development | $5,000–$10,000 | $38,500–$75,500 |
| Phase 4 (Flagship) | Hardware design + production | $15,000–$25,000 | $53,500–$100,500 |

### The Leverage Insight

The hybrid architecture creates dramatically different cost profiles:

| Investment | Cost | Integrations Unlocked | Cost per Integration |
|-----------|------|----------------------|---------------------|
| 1 TYPE 1 Firmware (ESP32) | ~$2,000 | 6 (one per client) | **$333 each** |
| 1 Universal Client (Python) | ~$1,500 | 4 TYPE 1 free + 5 TYPE 2 possible | **$167–$375 each** |
| 1 TYPE 2 Protocol Lib (Powered Up) | ~$3,000 | 0 (needs wrappers) | ∞ (until wrapped) |
| 1 TYPE 2 Wrapper | ~$800 | 1 | **$800 each** |

**Conclusion:** Every dollar spent on TYPE 1 firmware or Universal Clients generates 3–5× more ecosystem value than dollars spent on TYPE 2 wrappers. This is why the recommended build order prioritizes infrastructure over individual integrations.

---

## Recommended Build Order

Based on dependency analysis, leverage calculations, and community impact:

### Immediate Priority (Phase 2a)

| # | Item | Type | Cost | Rationale |
|---|------|------|------|-----------|
| 1 | **Generic ESP32 Firmware** | Type 1 FW | $2,000 | Highest leverage — unlocks ALL clients for unlimited cheap robots |
| 2 | **Python Universal Client** | Client | $1,500 | Serves advanced users, universities, enables scripting |
| 3 | **Web (JS) Universal Client** | Client | $1,500 | Zero-install browser experience, Web Bluetooth |

**Phase 2a total: ~$5,000** → Unlocks: ESP32 + Python, ESP32 + Web, ESP32 + App Inventor (3 new working combinations)

### Near-Term (Phase 2b)

| # | Item | Type | Cost | Rationale |
|---|------|------|------|-----------|
| 4 | **BBC micro:bit Firmware** | Type 1 FW | $1,200 | Millions in schools, easy win |
| 5 | **MicroBlocks Universal Client** | Client | $1,200 | Natural partner for micro:bit, on-device |
| 6 | **LEGO Powered Up Protocol Lib** | Type 2 Lib | $2,500 | Huge user base (Boost, Robot Inventor, all PU hubs) |
| 7 | **StackChan Firmware** | Type 1 FW | $1,800 | Active community, compelling demo |
| 8 | **Scratch Universal Client** | Client | $1,800 | Largest K-8 programming platform |

**Phase 2b total: ~$8,500** → Unlocks: 12+ new working combinations

### Medium-Term (Phase 2c)

| # | Item | Type | Cost | Rationale |
|---|------|------|------|-----------|
| 9 | **Raspberry Pi Firmware** | Type 1 FW | $2,000 | Complex robots, Linux ecosystem |
| 10 | **Sony toio Protocol Lib** | Type 2 Lib | $2,000 | Well-documented, K-6 market |
| 11 | **MakeCode Universal Client** | Client | $1,500 | Microsoft ecosystem, micro:bit native |
| 12 | **Powered Up wrappers** (×6) | Wrappers | $4,800 | Wrappers for all clients |
| 13 | **toio wrappers** (×6) | Wrappers | $4,200 | Wrappers for all clients |

**Phase 2c total: ~$14,500**

### Long-Term (Community-Driven)

| # | Item | Type | Cost | Rationale |
|---|------|------|------|-----------|
| 14 | **UBTECH uGot Protocol Lib** | Type 2 Lib | $3,500 | Asian education market |
| 15 | **CyberBrick Protocol Lib** | Type 2 Lib | $3,000 | Modular electronics |
| 16 | **Remaining wrappers** (×12) | Wrappers | $8,400 | Complete the matrix |
| 17 | **SPIKE Prime wrappers** (×5) | Wrappers | $4,500 | Expand SPIKE to all clients |

**Long-term total: ~$19,400**

---

## Community Voting & Sponsorship

### How Voting Works

1. Go to **[GitHub Discussions → Polls](../../discussions)** in the `solaria-hub` repository.
2. Upvote the hardware platform(s) and client platform(s) you want prioritized.
3. Comment with your use case to help inform prioritization decisions.
4. Vote results are reviewed monthly and influence the build order.

### Sponsoring a Specific Integration

Want a particular hardware or client platform supported sooner? You can **sponsor it directly** to move it to the front of the queue:

1. Check this roadmap for the item you want and its estimated cost.
2. Open a Discussion thread or contact via [GitHub Sponsors](https://github.com/sponsors/edcheng1010).
3. Sponsorship covers hardware + AI tooling + development time for that specific item.
4. Progress is tracked publicly in the relevant repository.
5. You are credited as the sponsor in the repo README.

See [FUNDING.md](FUNDING.md) for full details on sponsorship tiers and transparency commitments.

---

## Changelog

| Date | Update |
|------|--------|
| 2026-05-25 | Roadmap v2.0: Added hybrid architecture types, full hardware × client matrix, detailed cost estimates, dependency map, and build order recommendations. |
| 2026-05-24 | Initial roadmap published. Phase 1 shipped. Phase 2 voting open. |
