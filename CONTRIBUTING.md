# Contributing to Solaria

Thank you for your interest in contributing. Solaria grows through community effort — whether you're a hardware maker, app developer, educator, or student, there's a way to help.

---

## Ways to Contribute

### 1. Propose a New Hardware Integration

Have a robotics platform you'd like to see supported?

1. Check the [ROADMAP.md](ROADMAP.md) to see if it's already listed.
2. If not, open a Discussion thread with:
   - Hardware name and link
   - Communication protocol (BLE, WiFi, Serial, etc.)
   - Whether the firmware is open-source (determines TYPE 1 vs TYPE 2 — see below)
   - Whether the protocol is documented/open
   - Target audience (age range, skill level)
3. The community will vote on proposals during the next prioritisation cycle.

### 2. Build a Hardware Bridge (TYPE 1 — Open Firmware)

If the hardware allows you to **flash custom firmware** (e.g., ESP32, micro:bit, Raspberry Pi, CyberBrick, mBot2/CyberPi, StackChan):

1. Read the [SSP v0.6 specification](spec/SSP-v0.6.md).
2. Read the [Architecture guide](ARCHITECTURE.md) — specifically the TYPE 1 section.
3. Fork and create a new repo: `solaria-fw-<hardware>`.
4. Implement SSP firmware that:
   - Advertises SSP capability over BLE/WiFi
   - Publishes a capability declaration (ports, features, constraints)
   - Handles SSP JSON commands and emits sensor events
   - Implements heartbeat (§4.1) and error handling (§7)
5. Test against the reference App Inventor client (or any SSP universal client).
6. Open a PR to this hub repo to add your firmware to the README table.

**Result:** Once your firmware works, ALL existing universal clients automatically support it — zero additional work.

### 3. Build a Protocol Library (TYPE 2 — Closed Firmware)

If the hardware has **proprietary firmware you cannot replace** (e.g., LEGO SPIKE Prime, LEGO Powered Up, Sony toio, UBTECH uGot):

1. Read the [SSP v0.6 specification](spec/SSP-v0.6.md).
2. Read the [Architecture guide](ARCHITECTURE.md) — specifically the TYPE 2 section.
3. Fork and create a new repo: `solaria-lib-<hardware>`.
4. Implement a **protocol translation library** that:
   - Converts SSP commands → hardware's proprietary BLE/WiFi protocol
   - Converts hardware's sensor data → SSP sensor events
   - Exposes a clean API that client wrappers can consume
5. Write the library in a language that can be embedded into multiple clients (Java for App Inventor, Python for Python client, JS for Web, etc.) — or provide a language-agnostic spec that others can port.
6. Test against the actual hardware.
7. Open a PR to this hub repo to add your library to the README table.

**Result:** Each client platform still needs a thin wrapper to integrate the library, but the hard work (protocol reverse-engineering) is done once.

### 4. Build a Client Wrapper (TYPE 2 hardware on a specific client)

If a protocol library already exists for a TYPE 2 hardware platform:

1. Read the protocol library's API documentation.
2. Create a wrapper that integrates the library into a specific client (e.g., an .aix extension for App Inventor, a MakeCode extension, a Scratch plugin).
3. The wrapper should present the same SSP-style interface that the universal client uses for TYPE 1 hardware.
4. Test against the actual hardware.
5. Open a PR.

### 5. Build a Universal Client

You want Solaria to work in a new programming environment:

1. Read the [SSP v0.6 specification](spec/SSP-v0.6.md).
2. Read the [Architecture guide](ARCHITECTURE.md) — specifically the client section.
3. Fork and create a new repo: `solaria-client-<platform>`.
4. Implement:
   - SSP device discovery (BLE scan for SSP advertisements)
   - Capability parsing (read the device's port/feature/constraint declarations)
   - Command generation (build SSP JSON messages from user actions)
   - Sensor event handling (subscribe, receive, display)
   - Heartbeat management
5. Test against any TYPE 1 SSP firmware (e.g., Generic ESP32 Solaria Firmware).
6. Open a PR to this hub repo to add your client to the README.

**Result:** Your client automatically works with ALL TYPE 1 hardware. For TYPE 2 hardware, you'll need to integrate the relevant protocol library wrappers.

### 6. Improve Documentation

- Fix typos, clarify explanations, add examples
- Translate documentation into other languages
- Write tutorials or guides for specific use cases

### 7. Report Issues

- Found a bug in a firmware, library, or client? Open an issue in the relevant repo.
- Found a problem with the SSP spec itself? Open an issue here.

---

## Understanding the Hybrid Architecture

| | TYPE 1 (Open Firmware) | TYPE 2 (Closed Firmware) |
|---|---|---|
| **Can flash custom firmware?** | Yes | No |
| **Repo naming** | `solaria-fw-<hardware>` | `solaria-lib-<hardware>` |
| **What you build** | SSP firmware | Protocol translation library |
| **Client effort** | Zero (universal client works) | Thin wrapper per client |
| **Examples** | ESP32, micro:bit, StackChan, RPi, CyberBrick, mBot2 | LEGO SPIKE, Powered Up, Sony toio, UBTECH uGot |

See [ARCHITECTURE.md](ARCHITECTURE.md) for the full explanation with data flow diagrams.

---

## Pull Request Process

1. Fork the relevant repository.
2. Create a feature branch (`feature/my-contribution`).
3. Make your changes with clear commit messages.
4. Ensure your code follows existing style conventions.
5. Open a PR with a description of what you've done and why.
6. Respond to review feedback.

---

## Code of Conduct

All contributors are expected to follow the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). Be respectful, constructive, and welcoming.

---

## Questions?

Open a Discussion thread. We're happy to help you get started.
