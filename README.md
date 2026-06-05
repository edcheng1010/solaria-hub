<p align="center">
  <img src="assets/logo.png" alt="Solaria" width="200" />
</p>

# Solaria

**An open-source platform that connects open-source programming environments to popular robotics hardware — prioritising open-source options — through a single standard protocol.**

Solaria defines the **Solaria Standard Protocol (SSP)** — a communication layer that lets educators, makers, and developers control diverse robotics hardware from supported client applications. Write once, control any robot.

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)

---

## Architecture

```text
┌─────────────────────────────────────────────────────────┐
│  Layer 3: AI / Agent (Future)                           │
│  Natural language → robot commands                      │
└───────────────────────────┬─────────────────────────────┘
                            │
┌───────────────────────────┴─────────────────────────────┐
│  Layer 2: Clients                                       │
│  App Inventor · Python · Web · Scratch™ · (your own)     │
└───────────────────────────┬─────────────────────────────┘
                            │  ← SSP messages
┌───────────────────────────┴─────────────────────────────┐
│  Layer 1: Bridge Firmware                               │
│  Translates SSP → hardware-native commands              │
└───────────────────────────┬─────────────────────────────┘
                            │
┌───────────────────────────┴─────────────────────────────┐
│  Layer 0: Hardware                                      │
│  LEGO SPIKE · micro:bit™ · OpenCat · ESP32 · …           │
└─────────────────────────────────────────────────────────┘
```

**Hybrid architecture:** Solaria supports two types of hardware:

- **TYPE 1 (Open Firmware)** — devices you can flash with SSP firmware directly (ESP32, micro:bit™, StackChan, Raspberry Pi, CyberBrick, mBot2). Once firmware is built, ALL clients work automatically.
- **TYPE 2 (Closed Firmware)** — devices with proprietary firmware (LEGO, Sony® toio™, UBTECH® uGot). A protocol translation library converts SSP ↔ proprietary commands, then each client needs a thin wrapper.

See [ARCHITECTURE.md](ARCHITECTURE.md) for the full explanation with data flow diagrams and cost implications.

---

## Supported Hardware

| Hardware Platform | Status | Repository |
| :--- | :---: | :--- |
| LEGO® SPIKE™ Prime | ✅ Available | [solaria-appinventor-spike-prime](https://github.com/edcheng1010/solaria-appinventor-spike-prime) |
| BBC micro:bit™ / MicroBlocks | 📋 Planned | — |
| StackChan (M5Stack) | 📋 Planned | — |
| Generic ESP32 "Solaria Firmware" | 💡 Proposed | — |
| Raspberry Pi | 💡 Proposed | — |
| CyberBrick | 📋 Planned | — |
| Makeblock mBot2 (CyberPi) | 💡 Proposed | — |
| LEGO Powered Up (LWP3 family) | 💡 Proposed | — |
| UBTECH® uGot | 💡 Proposed | — |
| Sony® toio™ | 💡 Proposed | — |

> **What's next?** The community decides. [Vote here →](#vote-for-the-next-integration)

---

## Client Platforms

| Client Environment | Status | Notes |
| :--- | :---: | :--- |
| MIT App Inventor | ✅ Supported | Via .aix extension |
| MicroBlocks | 📋 Planned | Open source, block-based |
| Python | 📋 Planned | Via SDK/library |
| Scratch™ | ✅ Supported | TurboWarp/PenguinMod via Web Bluetooth (Phase 4a) |
| Web (JavaScript) | 💡 Proposed | Via Web Bluetooth SDK |
| MakeCode® | 💡 Proposed | Open source, Microsoft |
| Arduino IDE | 💡 Proposed | C/C++ library for advanced users |

> Solaria focuses on **open-source** client platforms to ensure the widest possible access without licensing barriers.

---

## Quick Start

1. **Pick a bridge** — clone the bridge repo for your hardware (e.g., `solaria-appinventor-spike-prime`).
2. **Flash or run** — follow the bridge README to get SSP running on your device.
3. **Connect a client** — open your preferred environment (App Inventor, Python, etc.) and use the corresponding Solaria client library to send commands.

---

## Vote for the Next Integration

Solaria's roadmap is community-driven. We use **GitHub Discussions polls** to decide which hardware gets integrated next.

→ [Cast your vote in Discussions](#)  
→ [View the full Roadmap](ROADMAP.md)

You can also **sponsor a specific integration** to accelerate its development. See [FUNDING.md](FUNDING.md).

---

## Documentation

| Document | Description |
| :--- | :--- |
| [ROADMAP.md](ROADMAP.md) | Project phases and community voting |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Layer model and SSP overview |
| [spec/SSP-v0.8.md](spec/SSP-v0.8.md) | Full protocol specification (latest) |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to propose, build, and submit |
| [VISION.md](VISION.md) | Product vision and sustainability model |
| [FUNDING.md](FUNDING.md) | Sponsorship and "Sponsor a Bridge" |

---

## Contributing

We welcome contributions from hardware makers, app developers, educators, and students. See [CONTRIBUTING.md](CONTRIBUTING.md) for details on:

- Proposing a new hardware integration
- Building a bridge (firmware developers)
- Building a client extension (app developers)
- Improving documentation

---

## Sponsor

Solaria is an independent open-source project. Development is sustained by community support — hardware purchases, development time, and documentation all cost real resources.

→ [GitHub Sponsors](#)  
→ [Sponsor a Bridge](FUNDING.md#sponsor-a-bridge--client)

---

## License

Apache License 2.0 — see [LICENSE](LICENSE) for the full text.

Copyright © 2026 Edward Cheng. See [NOTICE](NOTICE) for attribution details.

---

**Trademark Notice:** LEGO® and SPIKE™ are trademarks of the LEGO Group. micro:bit™ is a trademark of the Micro:bit Educational Foundation. Scratch™ is a trademark of the MIT Media Lab. MakeCode® is a trademark of Microsoft Corporation. Other trademarks are property of their respective owners. This project is not affiliated with or endorsed by any trademark holder. See [NOTICE](./NOTICE) for details.
