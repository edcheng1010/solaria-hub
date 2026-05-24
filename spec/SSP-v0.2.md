---
# Solaria Standard Protocol (SSP) — v0.2

**Author:** Edward Cheng  
**License:** Apache 2.0  
**Status:** Draft  
**Date:** 2026-05-24

---

## 1. Overview

The Solaria Standard Protocol (SSP) defines a hardware-agnostic communication format for controlling robotics devices from any client application. SSP operates over BLE, WiFi, or Serial transport layers without modification to the core message logic.

SSP enables:
- Any compliant client to control any compliant bridge
- Dynamic hardware discovery and capability reporting
- Real-time bidirectional communication (commands down, sensor data up)
- Flexible transport bindings via transport profiles

---

## 2. Transport Layer and Profiles

SSP is transport-agnostic. Because different vendor-locked hardware requires specific transport-level configurations (such as custom BLE UUIDs or framing), SSP defines a **Transport Profile Registry**. Each profile specifies how the SSP payload gets onto the wire, while the SSP payload remains invariant.

### 2.1 Transport Profile Registry

A transport profile defines the transport type, discovery method, framing, wrapper, and payload encoding. Bridges and clients must agree on a transport profile.

**Example Profile (SPIKE Prime 3.x):**
```json
{
  "profile_id": "spike-prime-3.x",
  "transport": "ble",
  "discovery": { "service_uuid": "0000fd02-0000-1000-8000-00805f9b34fb" },
  "framing": { "type": "cobs-xor", "delimiter": "0x02", "xor": "0x03" },
  "wrapper": { "type": "tunnel_message", "opcode": "0x32" },
  "ssp_payload_encoding": "json-utf8-newline"
}
```

Implementations MUST support at least one transport profile. Common generic profiles include:

| Transport | Generic Discovery Method | Connection |
|-----------|--------------------------|------------|
| BLE | Service UUID: `SOLARIA_BLE_UUID` (TBD) | GATT characteristics |
| WiFi | mDNS: `_solaria._tcp.local` | TCP socket (default port TBD) |
| Serial | Handshake: `SSP_HELLO` / `SSP_ACK` | UART at 115200 baud |

---

## 3. Message Format

SSP supports two payload encodings, negotiated or specified by the transport profile: `json-utf8-newline` and `binary`.

### 3.1 JSON Payload Encoding (`json-utf8-newline`)

All SSP messages are UTF-8 encoded JSON, terminated by a newline character (`\n`).

#### 3.1.1 Client → Bridge (Commands)

Commands may optionally include a `request_id` field to enable pipelined command error attribution.

```json
{"cmd": "<category>.<action>", "port": "<port_id>", "request_id": "<optional_id>", ...params}
```

**Examples:**
```json
{"cmd": "motor.run", "port": "A", "speed": 75, "request_id": "req-1"}
{"cmd": "motor.stop", "port": "A"}
{"cmd": "led.set", "port": "status", "color": [255, 0, 0]}
{"cmd": "sound.beep", "freq": 440, "duration": 500}
{"cmd": "sensor.subscribe", "port": "C", "interval": 100, "mode": "on_change", "min_change": 5}
```

#### 3.1.2 Bridge → Client (Events)

Events generated in response to a command with a `request_id` SHOULD echo that `request_id`.

```json
{"event": "<type>", "port": "<port_id>", "request_id": "<optional_id>", ...data}
```

**Examples:**
```json
{"event": "sensor", "port": "C", "type": "color", "value": "red"}
{"event": "sensor", "port": "D", "type": "distance", "value": 23.5, "unit": "cm"}
{"event": "button", "port": "left", "state": "pressed"}
{"event": "error", "code": 101, "message": "Port not connected", "request_id": "req-1"}
```

### 3.2 Binary Payload Encoding (`binary`)

For bandwidth-constrained transports, a binary payload option is reserved. The exact bit layout for standard commands will be defined in a supplementary binary specification. Profiles choose the encoding, and clients/bridges advertise their supported encodings in the capability handshake.

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
  |--- cmd: system.ping -------->| (Heartbeat)
  |<-- event: pong --------------|
  |<-- event: sensor ------------|
  |                               |
  |--- [disconnect] ------------>|
```

### 4.1 Required Heartbeat

To ensure connection stability and detect silent drops (e.g., on Android BLE), a heartbeat mechanism is required when subscriptions are active. 
- The client MUST send a `system.ping` command every 5 seconds.
- The bridge MUST respond with a `pong` event.
- The bridge MUST disconnect if it misses 2 consecutive pongs (i.e., no ping received for 10 seconds).
- The client SHOULD use the absence of `pong` events to detect a dead bridge.

---

## 5. Capability Declaration

Upon connection, the bridge MUST send a capability declaration message:

```json
{
  "type": "capability",
  "device": "<hardware_name>",
  "firmware": "<bridge_version>",
  "ssp_version": "0.2",
  "encodings": ["json-utf8-newline", "binary"],
  "ports": [
    {
      "id": "A",
      "type": "motor",
      "features": ["speed", "position", "stall"]
    },
    {
      "id": "C",
      "type": "color_sensor",
      "features": ["color", "reflected", "ambient", "rgb"]
    }
  ]
}
```

Clients SHOULD use this declaration to dynamically configure their interface.

### 5.1 Canonical Feature Schema

To ensure consistency, feature names MUST use the canonical names defined below based on the port type. Non-standard extensions MUST be prefixed with `x_`.

- **motor:** `speed`, `position`, `absolute_position`, `stall`, `load`
- **color_sensor:** `color`, `reflected`, `ambient`, `rgb`
- **distance_sensor:** `distance`, `presence`
- **force_sensor:** `force`, `touched`

---

## 6. Standard Command Categories

| Category | Actions | Description |
|----------|---------|-------------|
| `motor` | `run`, `stop`, `goto`, `reset` | DC and servo motor control |
| `movement` | `configure`, `drive`, `turn`, `stop` | Coordinated drive-base motion |
| `led` | `set`, `off`, `animate` | LED and display control |
| `sound` | `beep`, `play`, `stop` | Audio output |
| `sensor` | `subscribe`, `unsubscribe`, `read` | Sensor data management |
| `system` | `ping`, `info`, `reset`, `dfu` | Bridge management |

### 6.1 `movement` Category

The `movement` category enables coordinated drive-base motion using motor pairs. Bridges without coordinated motion support will omit this from their capabilities, and clients should fall back to issuing dual `motor.run` commands.

- `movement.configure`: Sets up the left and right motor ports (e.g., `{"cmd": "movement.configure", "left": "A", "right": "B"}`).
- `movement.drive`: Moves the base at a given speed and steering angle.
- `movement.turn`: Rotates the base by a specific angle at a given speed.
- `movement.stop`: Stops the coordinated motion.

### 6.2 `sensor.subscribe` Flow-Control

The `sensor.subscribe` command includes flow-control parameters to manage bandwidth and denoise data:
- `mode`: Can be `"interval"`, `"on_change"`, or `"hybrid"`.
- `interval`: Time in milliseconds between updates (used in interval and hybrid modes).
- `min_change`: The minimum change in value required to trigger an update (used in on_change and hybrid modes).

Example:
```json
{"cmd": "sensor.subscribe", "port": "C", "mode": "hybrid", "interval": 1000, "min_change": 2.5}
```

### 6.3 Feature-to-Command Mapping

Clients can verify support before sending commands using the following mapping between declared features and supported commands:

| Feature | Supported Commands |
|---------|--------------------|
| `speed` | `motor.run`, `motor.stop` |
| `position` / `absolute_position` | `motor.goto`, `motor.reset` |
| `stall` / `load` | Generates stall/load events |
| `color` / `reflected` / `ambient` / `rgb` | `sensor.subscribe`, `sensor.read` |
| `distance` / `presence` | `sensor.subscribe`, `sensor.read` |
| `force` / `touched` | `sensor.subscribe`, `sensor.read` |

---

## 7. Error Handling

Bridges MUST report errors as events. If the command included a `request_id`, the error event MUST echo it.

```json
{"event": "error", "code": <int>, "message": "<description>", "request_id": "<optional_id>"}
```

| Code Range | Category |
|-----------|----------|
| 100–199 | Connection errors |
| 200–299 | Command errors (invalid port, unsupported action) |
| 300–399 | Hardware errors (motor stall, sensor disconnected) |
| 400–499 | Protocol errors (malformed message, version mismatch) |

---

## 8. Versioning

SSP follows semantic versioning. This document defines v0.2 (draft). Breaking changes increment the major version. Bridges and clients MUST declare their supported SSP version in capability messages and connection handshakes.

---

## 9. Future Considerations (v0.3+)

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
