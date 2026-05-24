# Solaria

**An open-source platform that connects any programming environment to any robotics hardware through a single standard protocol.**

Solaria defines the **Solaria Standard Protocol (SSP)** — a universal communication layer that lets educators, makers, and developers control diverse robotics hardware from any client application. Write once, control any robot.

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
│  App Inventor · Python · Web · Scratch · (your own)     │
└───────────────────────────┬─────────────────────────────┘
                            │  ← SSP messages
┌───────────────────────────┴─────────────────────────────┐
│  Layer 1: Bridge Firmware                               │
│  Translates SSP → hardware-native commands              │
└───────────────────────────┬─────────────────────────────┘
                            │
┌───────────────────────────┴─────────────────────────────┐
│  Layer 0: Hardware                                      │
│  LEGO SPIKE · micro:bit · OpenCat · ESP32 · …          │
└─────────────────────────────────────────────────────────┘
```

**You own Layer 1.** The protocol and firmware bridges are what Solaria standardises. Clients above and hardware below can be anything.

---

## Supported Hardware

| Hardware Platform | Status | Repository |
| :--- | :---: | :--- |
| LEGO SPIKE Prime | ✅ Available | [appinventor-lego-spike-prime-extension](https://github.com/edcheng1010/appinventor-lego-spike-prime-extension) |
| BBC micro:bit / MicroBlocks | 📋 Planned | — |
| StackChan (M5Stack) | 📋 Planned | — |
| Raspberry Pi | 📋 Planned | — |
| CyberBrick | 📋 Planned | — |
| UBTECH uGot | 📋 Planned | — |
| Generic ESP32 "Solaria Firmware" | 💡 Proposed | — |
| LEGO Powered Up (LWP3 family) | 💡 Proposed | — |
| Petoi OpenCat (Bittle / Nybble) | 💡 Proposed | — |
| Makeblock mBot2 | 💡 Proposed | — |
| Sony toio | 💡 Proposed | — |
| DFRobot UNIHIKER | 💡 Proposed | — |

> **What's next?** The community decides. [Vote here →](#vote-for-the-next-integration)

---

## Quick Start

1. **Pick a bridge** — clone the bridge repo for your hardware (e.g., `solaria-bridge-spike-prime`).
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
| [spec/SSP-v0.1.md](spec/SSP-v0.1.md) | Full protocol specification |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to propose, build, and submit |
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
→ [Sponsor a Bridge](FUNDING.md#sponsor-a-bridge)

---

## License

Apache License 2.0 — see [LICENSE](LICENSE) for the full text.

Copyright © 2026 Edward Cheng. See [NOTICE](NOTICE) for attribution details.
