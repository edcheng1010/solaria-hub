# Roadmap

This is a living document. Priorities shift based on community votes, sponsorship, and technical feasibility.

---

## Phase 1 — Foundation ✅

**Status:** Shipped

- [x] SSP v0.1 specification defined
- [x] LEGO SPIKE Prime BLE bridge released ([solaria-bridge-spike-prime](#))
- [x] App Inventor client extension (.aix) for SPIKE Prime

The first end-to-end proof: a student can open App Inventor on their phone, connect to a SPIKE Prime hub over BLE, and control motors/read sensors using SSP.

---

## Phase 2 — Ecosystem Expansion 🚧

**Status:** Community voting open

The platform is proven. Now we widen hardware support. **You decide what gets built next.**

### Candidates (vote in [Discussions](#)):

| # | Platform | Why it matters | Effort |
|---|----------|---------------|--------|
| 1 | BBC micro:bit / MicroBlocks | Millions deployed in schools globally. Near-zero integration effort (BLE extension path exists). | Low |
| 2 | Petoi OpenCat (Bittle / Nybble) | Walking robot dog — high wow factor, documented BLE protocol, 4.8k GitHub stars. | Medium |
| 3 | LEGO Powered Up (full LWP3) | Extends SPIKE work to Boost, Robot Inventor, all Powered Up hubs. Huge user base. | Low–Med |
| 4 | StackChan (M5Stack) | Desktop companion robot. ESP32-based, BLE/WiFi. Aligns with Solaria's long-term vision. | Medium |
| 5 | Generic ESP32 "Solaria Firmware" | Flash onto ANY cheap robot car from AliExpress/Amazon. Democratises access. | Medium |
| 6 | Makeblock mBot2 | Popular in schools. ESP32-based CyberPi brain, documented BLE. | Medium |
| 7 | Sony toio | Precision cubes with published BLE spec. Great for K-6. | Low–Med |
| 8 | DFRobot UNIHIKER | ESP32-S3 + Linux hybrid. Big in Asian maker community. | Medium |
| 9 | Raspberry Pi | WiFi/BLE bridge for complex Linux-based robots. | Medium |
| 10 | CyberBrick | Modular electronic building blocks. | Medium |
| 11 | UBTECH uGot | Modular AI robotics kit with servo/sensor blocks. Popular in Asian education. | Medium |

### How voting works:

1. Go to [GitHub Discussions → Polls](#)
2. Upvote the platform(s) you want
3. Optionally: [sponsor a bridge](FUNDING.md#sponsor-a-bridge) to move it up the queue

---

## Phase 3 — AI / Agent Layer 🔮

**Status:** Planned (after 3+ bridges are stable)

- Natural language → SSP command translation
- "Drive forward until you see a red object" → motor + color sensor commands
- LLM-powered, runs locally (M5Stack Module LLM) or cloud
- Works with ANY bridge — hardware-agnostic intelligence

---

## Phase 4 — Solaria Flagship Robot 🌱

**Status:** Vision

A reference hardware design that showcases the full stack:
- Custom PCB (ESP32-based)
- BLE sphere core with wireless charging
- Solarpunk aesthetic, partially 3D-printable
- Ships with SSP firmware, App Inventor + Python clients, and AI agent pre-configured
- The "iPhone of the ecosystem" — one product that demonstrates everything

---

## Changelog

| Date | Update |
|------|--------|
| 2026-05-24 | Initial roadmap published. Phase 1 shipped. Phase 2 voting open. |
