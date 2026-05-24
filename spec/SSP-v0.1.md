# Solaria Standard Protocol (SSP) — v0.1

**Author:** Edward Cheng  
**License:** Apache 2.0  
**Status:** Draft  
**Date:** 2026-05-24

---

## 1. Overview

The Solaria Standard Protocol (SSP) defines a hardware-agnostic communication format for controlling robotics devices from any client application. SSP operates over BLE, WiFi, or Serial transport layers without modification to the message format.

SSP enables:
- Any compliant client to control any compliant bridge
- Dynamic hardware discovery and capability reporting
- Real-time bidirectional communication (commands down, sensor data up)

---

## 2. Transport Layer

SSP is transport-agnostic. Implementations MUST support at least one of:

| Transport | Discovery Method | Connection |
|-----------|-----------------|------------|
| BLE | Service UUID: `SOLARIA_BLE_UUID` (TBD) | GATT characteristics |
| WiFi | mDNS: `_solaria._tcp.local` | TCP socket (default port TBD) |
| Serial | Handshake: `SSP_HELLO` / `SSP_ACK` | UART at 115200 baud |

---

## 3. Message Format

All SSP messages are UTF-8 encoded JSON, terminated by a newline character (`\n`).

### 3.1 Client → Bridge (Commands)

```json
{"cmd": "<category>.<action>", "port": "<port_id>", ...params}
```

**Examples:**
```json
{"cmd": "motor.run", "port": "A", "speed": 75}
{"cmd": "motor.stop", "port": "A"}
{"cmd": "led.set", "port": "status", "color": [255, 0, 0]}
{"cmd": "sound.beep", "freq": 440, "duration": 500}
{"cmd": "sensor.subscribe", "port": "C", "interval": 100}
```

### 3.2 Bridge → Client (Events)

```json
{"event": "<type>", "port": "<port_id>", ...data}
```

**Examples:**
```json
{"event": "sensor", "port": "C", "type": "color", "value": "red"}
{"event": "sensor", "port": "D", "type": "distance", "value": 23.5, "unit": "cm"}
{"event": "button", "port": "left", "state": "pressed"}
{"event": "error", "code": 101, "message": "Port not connected"}
```

---

## 4. Connection Lifecycle

```
Client                          Bridge
  |                               |
  |--- [connect] --------------->|
  |                               |
  |<-- capability_declaration ---|
  |                               |
  |--- cmd: motor.run ---------->|
  |--- cmd: sensor.subscribe --->|
  |                               |
  |<-- event: sensor ------------|
  |<-- event: sensor ------------|
  |                               |
  |--- [disconnect] ------------>|
```

---

## 5. Capability Declaration

Upon connection, the bridge MUST send a capability declaration message:

```json
{
  "type": "capability",
  "device": "<hardware_name>",
  "firmware": "<bridge_version>",
  "ssp_version": "0.1",
  "ports": [
    {
      "id": "A",
      "type": "motor",
      "features": ["speed", "position", "stall_detect"]
    },
    {
      "id": "C",
      "type": "color_sensor",
      "features": ["color", "reflected", "ambient"]
    }
  ]
}
```

Clients SHOULD use this declaration to dynamically configure their interface.

---

## 6. Standard Command Categories

| Category | Actions | Description |
|----------|---------|-------------|
| `motor` | `run`, `stop`, `goto`, `reset` | DC and servo motor control |
| `led` | `set`, `off`, `animate` | LED and display control |
| `sound` | `beep`, `play`, `stop` | Audio output |
| `sensor` | `subscribe`, `unsubscribe`, `read` | Sensor data management |
| `system` | `ping`, `info`, `reset`, `dfu` | Bridge management |

---

## 7. Error Handling

Bridges MUST report errors as events:

```json
{"event": "error", "code": <int>, "message": "<description>"}
```

| Code Range | Category |
|-----------|----------|
| 100–199 | Connection errors |
| 200–299 | Command errors (invalid port, unsupported action) |
| 300–399 | Hardware errors (motor stall, sensor disconnected) |
| 400–499 | Protocol errors (malformed message, version mismatch) |

---

## 8. Versioning

SSP follows semantic versioning. This document defines v0.1 (draft). Breaking changes increment the major version. Bridges and clients MUST declare their supported SSP version in capability messages and connection handshakes.

---

## 9. Future Considerations (v0.2+)

- Binary message format option for bandwidth-constrained transports
- Batch command support (multiple commands in one message)
- Firmware update over SSP (DFU protocol extension)
- Stream multiplexing for multi-device control
- Authentication and pairing security model

---

## 10. Reference Implementations

| Component | Repository |
|-----------|-----------|
| SPIKE Prime Bridge | [solaria-bridge-spike-prime](#) |
| App Inventor Client | [solaria-client-appinventor](#) |

---

*This specification is a living document. Contributions and feedback welcome via GitHub Issues.*
