# Architecture

Solaria uses a four-layer model. Each layer has a single responsibility and communicates only through well-defined interfaces.

---

## Layer Model

```
Layer 3  AI / Agent          (future — NL → commands)
         ─────────────────────────────────────────────
Layer 2  Clients             App Inventor │ Python │ Web │ Scratch
         ─────────────────────────────────────────────
Layer 1  SSP + Bridge FW     protocol parsing, transport (BLE/WiFi/Serial)
         ─────────────────────────────────────────────
Layer 0  Hardware            motors, sensors, actuators, LEDs
```

**The key insight:** Solaria owns Layer 1. Everything above and below is interchangeable.

---

## Solaria Standard Protocol (SSP)

SSP is the language spoken between Layer 2 (clients) and Layer 1 (bridges). It is transport-agnostic — the same message format works over BLE, WiFi, or Serial.

SSP defines four concerns:

### 1. Device Discovery
How a client finds and connects to a Solaria bridge.
- BLE: standardised advertising name/service UUID
- WiFi: mDNS service type `_solaria._tcp`
- Serial: handshake sequence on connection

### 2. Capability Declaration
Once connected, the bridge reports what hardware is attached:
```json
{
  "device": "spike-prime",
  "ports": [
    {"id": "A", "type": "motor", "features": ["speed", "position"]},
    {"id": "B", "type": "motor", "features": ["speed", "position"]},
    {"id": "C", "type": "color_sensor", "features": ["color", "reflected"]}
  ]
}
```
Clients use this to dynamically build their UI or block palette.

### 3. Command Format
Standardised instruction messages from client → bridge:
```json
{"cmd": "motor.run", "port": "A", "speed": 75}
{"cmd": "led.set", "port": "status", "color": [0, 255, 0]}
{"cmd": "sound.beep", "freq": 440, "duration": 500}
```

### 4. Sensor Streaming
Bridge → client, continuous or on-change:
```json
{"event": "sensor", "port": "C", "type": "color", "value": "red"}
{"event": "sensor", "port": "D", "type": "distance", "value": 23.5, "unit": "cm"}
```

> Full specification: [spec/SSP-v0.6.md](spec/SSP-v0.6.md)

---

## Separation of Concerns

| Layer | Knows about | Does NOT know about |
|-------|------------|-------------------|
| Bridge firmware | Hardware registers, SSP parsing | Which client sent the command |
| SSP protocol | Message format, transport framing | Specific hardware or specific client |
| Client extension | User-facing blocks/API, SSP generation | Hardware internals |
| AI agent | Intent parsing, goal decomposition | Transport details, hardware registers |

This means:
- A new hardware bridge requires ZERO changes to existing clients
- A new client requires ZERO changes to existing bridges
- The AI agent works with any bridge + any client combination

---

## Building a New Bridge

1. Read [spec/SSP-v0.6.md](spec/SSP-v0.6.md)
2. Create repo: `solaria-bridge-<hardware>`
3. Pick or define a transport profile (§2.1)
4. Implement transport (BLE/WiFi/Serial/RFCOMM listener)
5. Parse incoming SSP commands → map to hardware API
6. Implement capability declaration with constraints (§5)
7. Implement sensor streaming and system metrics (§6.4, §6.5)
8. Test with the reference App Inventor client
9. Submit PR to add your bridge to the hub README

---

## Building a New Client

1. Read [spec/SSP-v0.6.md](spec/SSP-v0.6.md)
2. Create repo: `solaria-client-<platform>`
3. Implement transport profiles (connect to bridge over BLE/WiFi/Serial/RFCOMM)
4. Parse capability declaration + constraints → build dynamic interface
5. Expose user-facing API or blocks that generate SSP commands
6. Handle incoming sensor events and system metrics
7. Implement heartbeat (§4.1) and batch support (§6.4)
8. Test with the reference SPIKE Prime bridge
9. Submit PR to add your client to the hub README
