---
# Solaria Standard Protocol (SSP) — v0.7

**Author:** Edward Cheng  
**License:** Apache 2.0  
**Status:** Draft  
**Date:** 2026-05-26

---

## Changelog for v0.7
- Added `face_orientation` feature to `orientation` port (6-value enum).
- Added `angular_velocity` feature to `orientation` port.
- Added `orientation.set_yaw` and `orientation.reset_yaw` commands.
- Added `orientation.set_reference` command for hub mounting orientation.
- Added `touch` feature on `display` port for tap detection.
- Supported Sensor-attached LED displays as `display` ports (Distance sensor 4-LED indicator and 3×3 Color Matrix).
- Added `movement.drive` with `left_speed` and `right_speed` for tank-style control.
- Added `sound.read` command for volume retrieval.
- Added `mode` parameter ("speed" or "power") to `motor.run` command.

---

## 1. Overview

The Solaria Standard Protocol (SSP) defines a hardware-agnostic communication format for controlling robotics devices from any client application. SSP operates over BLE, WiFi, RFCOMM, or Serial transport layers without modification to the core message logic.

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

**Example Profile (SPIKE Prime 3.x over BLE):**
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

**Example Profile (EV3 over RFCOMM):**
```json
{
  "profile_id": "ev3-rfcomm",
  "transport": "rfcomm",
  "discovery": {
    "device_class": "0x804",
    "name_pattern": "EV3*",
    "sdp_uuid": "00001101-0000-1000-8000-00805F9B34FB"
  },
  "framing": {"type": "length-prefixed", "prefix": "uint16-le"},
  "ssp_payload_encoding": "json-utf8-newline"
}
```

Implementations MUST support at least one transport profile. Common generic profiles include:

| Transport | Generic Discovery Method | Connection |
|-----------|--------------------------|------------|
| BLE | Service UUID: `SOLARIA_BLE_UUID` (TBD) | GATT characteristics |
| RFCOMM | SDP UUID: `SOLARIA_RFCOMM_UUID` (TBD) | Bluetooth RFCOMM socket |
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
{"cmd": "motor.run", "port": "A", "speed": 75, "duration": 2000, "duration_unit": "ms", "request_id": "req-1"}
{"cmd": "motor.stop", "port": "A", "stop_action": "coast"}
{"cmd": "led.set", "port": "status", "color": [255, 0, 0]}
{"cmd": "led.matrix.pixel", "port": "display", "x": 2, "y": 2, "brightness": 100}
{"cmd": "sound.play", "sound": "giggle"}
{"cmd": "sensor.subscribe", "port": "C", "interval": 100, "mode": "on_change", "min_change": 5}
{"cmd": "system.subscribe", "metric": "battery", "interval": 60000}
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
{"event": "sensor", "port": "imu", "type": "gesture", "value": "shake"}
{"event": "system", "metric": "battery", "value": 87}
{"event": "system", "metric": "button.left", "value": "pressed"}
{"event": "error", "code": 101, "message": "Port not connected", "request_id": "req-1"}
```

*Note: Button events are part of the system metrics subsystem (see §6.6) and are emitted as `system` events.*

### 3.2 Binary Payload Encoding (`binary`)

For bandwidth-constrained transports, a binary payload option is defined. The binary structure consists of:
- **Opcode:** 1-byte category opcode + 1-byte action opcode
- **Port ID:** 1 byte (numeric mapping declared in capability message)
- **Parameters:** In fixed order per command, with types pre-declared
- **Request ID:** 2-byte unsigned int if present (a flag bit indicates presence)
- **Framing:** As defined by the transport profile

**Category Opcode Assignment:**
| Category | Opcode |
|----------|--------|
| motor | `0x01` |
| movement | `0x02` |
| led | `0x03` |
| sound | `0x04` |
| sensor | `0x05` |
| system | `0x06` |
| batch | `0x07` |
| orientation | `0x08` |

*Note: The full per-command binary layouts will be detailed in a supplementary specification.*

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
  "ssp_version": "0.7",
  "encodings": ["json-utf8-newline", "binary"],
  "supports_batch": true,
  "system_metrics": ["battery", "charging", "temperature", "button.left", "button.right"],
  "ports": [
    {
      "id": "A",
      "type": "motor",
      "features": ["speed", "position", "stall", "power"],
      "constraints": {
        "speed": {"type": "int", "min": -100, "max": 100},
        "position": {"type": "int", "min": 0, "max": 359, "wraps": true}
      }
    },
    {
      "id": "status",
      "type": "led",
      "features": ["set"],
      "constraints": {
        "color": {"type": "array", "length": 3, "item_type": {"type": "int", "min": 0, "max": 255}}
      }
    },
    {
      "id": "C",
      "type": "color_sensor",
      "features": ["color", "reflected", "ambient", "rgb"]
    },
    {
      "id": "display",
      "type": "display",
      "width": 5,
      "height": 5,
      "depth": "grayscale",
      "features": ["pixel", "image", "text", "brightness", "orientation", "touch"]
    },
    {
      "id": "imu",
      "type": "orientation",
      "features": ["pitch", "roll", "yaw", "gesture", "face_orientation", "angular_velocity"],
      "constraints": {
        "gesture": {"type": "enum", "values": ["shake", "tap", "double_tap", "fall", "face_up", "face_down"]},
        "face_orientation": {"type": "enum", "values": ["face_up", "face_down", "port_a_up", "port_a_down", "port_e_up", "port_e_down"]}
      }
    },
    {
      "id": "speaker",
      "type": "speaker",
      "features": ["beep", "builtin", "midi", "volume"],
      "builtin_sounds": ["giggle", "alert", "fanfare"]
    }
  ]
}
```

Clients SHOULD use this declaration to dynamically configure their interface.

### 5.1 Canonical Feature Schema

To ensure consistency, feature names MUST use the canonical names defined below based on the port type. Non-standard extensions MUST be prefixed with `x_`.

- **motor:** `speed`, `position`, `absolute_position`, `stall`, `load`, `power`
- **color_sensor:** `color`, `reflected`, `ambient`, `rgb`
- **distance_sensor:** `distance`, `presence`
- **force_sensor:** `force`, `touched`
- **display:** `pixel`, `image`, `text`, `scroll`, `rgb`, `brightness`, `orientation`, `touch` (capability MUST include `width`, `height`, and `depth`)
- **orientation:** `pitch`, `roll`, `yaw`, `acceleration`, `angular_velocity`, `gesture`, `face_orientation` (gestures emit discrete events using the standard `sensor` event format: `{"event": "sensor", "type": "gesture", "value": "shake"}`)
- **speaker:** `beep`, `builtin`, `file`, `midi`, `volume` (capability SHOULD include optional `builtin_sounds` array)

### 5.2 Parameter Constraints

To allow clients to build dynamic UIs (such as sliders with correct min/max values, dropdowns for enum values, and pre-flight validation), ports can optionally include a `constraints` object.

- **Optionality:** `constraints` is OPTIONAL. If a bridge does not declare constraints for a feature, clients should assume any value allowed by the type is acceptable.
- **Supported Constraint Types:**
  - `int`: Supports `min`, `max`, and an optional `wraps` boolean (e.g., for 360-degree rotation).
  - `float`: Supports `min`, `max`.
  - `enum`: Supports a `values` array of acceptable strings.
  - `bool`: Indicates a boolean parameter.
  - `array`: Supports `length` and `item_type` (which defines the constraint for each item).
  - `string`: Supports `pattern` (a regular expression) and `max_length`.
- **Usage:** Clients SHOULD use constraints to set slider min/max, populate dropdowns, validate inputs before sending commands, and provide helpful errors to users.

*Note: Port-level dimension fields implicitly define constraints for coordinate parameters. Bridge authors do not need to repeat these in the `constraints` object:*
- `width: 5` implies `x ∈ [0, 4]` for `led.matrix.pixel`
- `height: 5` implies `y ∈ [0, 4]` for `led.matrix.pixel`
- `depth: "grayscale"` implies `brightness ∈ [0, 100]`
- `depth: "rgb"` implies a `color: [r, g, b]` parameter (each 0-255) instead of `brightness`

#### 5.2.1 Sensor-attached LED Displays

Sensor-attached LED displays are treated as `display` ports with specific dimensions and color depths. For example:
- **Distance sensor 4-LED indicator:** Advertises as `display` with `width: 2`, `height: 2`, `depth: "grayscale"`. Controlled via `led.matrix.pixel` commands.
- **3×3 Color Matrix accessory:** Advertises as `display` with `width: 3`, `height: 3`, `depth: "rgb"`. Controlled via `led.matrix.pixel` with `color: [r, g, b]`.

---

## 6. Standard Command Categories

| Category | Actions | Description |
|----------|---------|-------------|
| `motor` | `run`, `stop`, `goto`, `reset` | DC and servo motor control |
| `movement` | `configure`, `drive`, `turn`, `stop` | Coordinated drive-base motion |
| `led` | `set`, `off`, `animate`, `matrix.*` | LED and display control |
| `sound` | `beep`, `play`, `stop`, `set_volume`, `read` | Audio output |
| `sensor` | `subscribe`, `unsubscribe`, `read` | Sensor data management |
| `system` | `ping`, `info`, `reset`, `dfu`, `subscribe`, `read` | Bridge management and metrics |
| `orientation` | `set_yaw`, `reset_yaw`, `set_reference` | Hub orientation configuration |
| `batch` | N/A | Execute multiple commands |

### 6.1 `motor` and `movement` Enhancements

- **Motor Run Modes:** `motor.run` accepts an optional `mode` parameter: `"speed"` (default, closed-loop speed control) or `"power"` (open-loop raw duty cycle control). Bridges that support power mode MUST declare the `power` feature in their capabilities.
  ```json
  {"cmd": "motor.run", "port": "A", "speed": 75, "mode": "speed"}
  {"cmd": "motor.run", "port": "A", "speed": 50, "mode": "power"}
  ```
- **Duration on Run Commands:** `motor.run` and `movement.drive` support optional `duration` (number) and `duration_unit` (`"ms"`, `"degrees"`, `"rotations"`). Bridges that do not support duration will ignore the field and run indefinitely.
  ```json
  {"cmd": "motor.run", "port": "A", "speed": 75, "duration": 360, "duration_unit": "degrees"}
  ```
- **Tank Drive:** `movement.drive` supports tank-style control via `left_speed` and `right_speed` parameters. When provided, standard `speed` and `steering` parameters are ignored. Bridges MUST declare the `tank_drive` capability.
  ```json
  {"cmd": "movement.drive", "left_speed": 50, "right_speed": -50}
  ```
- **Stop Action:** `motor.stop` and `movement.stop` support an optional `stop_action` parameter which can be `"brake"` (default), `"coast"`, or `"hold"`.
  ```json
  {"cmd": "motor.stop", "port": "A", "stop_action": "hold"}
  ```

### 6.2 `led.matrix` Category

For addressable pixel/text displays, the `led.matrix` sub-commands are available:
- `led.matrix.pixel`: Set a single pixel (e.g., `{"cmd": "led.matrix.pixel", "x": 0, "y": 0, "brightness": 100}`).
- `led.matrix.image`: Display a named image pattern.
- `led.matrix.text`: Display or scroll text.
- `led.matrix.clear`: Clear the display.
- `led.matrix.brightness`: Set the display brightness (e.g., `{"cmd": "led.matrix.brightness", "port": "display", "level": 50}`).
- `led.matrix.orientation`: Set the display orientation (e.g., `{"cmd": "led.matrix.orientation", "port": "display", "rotation": 90}`).

### 6.3 `sound` Category Enhancements

The `sound.play` command accepts different payload shapes depending on the supported features of the `speaker` port:
- **Built-in sounds:** `{"cmd": "sound.play", "sound": "giggle"}` (requires `builtin` feature).
- **File playback:** `{"cmd": "sound.play", "file": "alert.wav"}` (requires `file` feature).
- **MIDI notes:** `{"cmd": "sound.play", "notes": "C4:200 E4:200 G4:400", "tempo": 120}` (requires `midi` feature).

Volume control and retrieval are available if the port advertises the `volume` feature:
```json
{"cmd": "sound.set_volume", "level": 75}
{"cmd": "sound.read", "metric": "volume"}
```
The volume getter returns the current volume level.

### 6.4 `orientation` Category

The `orientation` port provides spatial tracking and configuration:
- `orientation.set_yaw`: Set the yaw to an arbitrary angle (e.g., `{"cmd": "orientation.set_yaw", "angle": 90}`).
- `orientation.reset_yaw`: Reset the yaw to zero (e.g., `{"cmd": "orientation.reset_yaw"}`).
- `orientation.set_reference`: Configure the hub's mounting orientation. Takes a `face` parameter matching the `face_orientation` enum values (e.g., `{"cmd": "orientation.set_reference", "face": "face_up"}`).

### 6.5 Batch Commands

The `batch` category allows clients to send multiple commands in a single message. A single `request_id` correlates to the entire batch.

- **Atomic Execution:** If `atomic` is `true`, the bridge treats the batch as all-or-nothing; on the first error, the entire batch aborts. The default is `false` (best-effort).
- **Support:** Bridges that do not support batching will emit a `4xx` error. Clients should then fall back to individual commands. Support is advertised via `"supports_batch": true` in the capability declaration.

Example:
```json
{
  "cmd": "batch",
  "request_id": "frame-42",
  "atomic": true,
  "commands": [
    {"cmd": "led.matrix.pixel", "port": "display", "x": 0, "y": 0, "brightness": 100},
    {"cmd": "led.matrix.pixel", "port": "display", "x": 0, "y": 1, "brightness": 50}
  ]
}
```

### 6.6 `sensor.subscribe` Flow-Control

The `sensor.subscribe` command includes flow-control parameters to manage bandwidth and denoise data:
- `mode`: Can be `"interval"`, `"on_change"`, or `"hybrid"`.
- `interval`: Time in milliseconds between updates (used in interval and hybrid modes).
- `min_change`: The minimum change in value required to trigger an update (used in on_change and hybrid modes).

Example:
```json
{"cmd": "sensor.subscribe", "port": "C", "mode": "hybrid", "interval": 1000, "min_change": 2.5}
```

### 6.7 `system.subscribe` and Hub-Internal Metrics

Clients can subscribe to or read hub-internal metrics using `system.subscribe` and `system.read`. Events are emitted as: `{"event": "system", "metric": "<name>", "value": <val>}`.

Standard metrics include:
| Metric | Value type | Notes |
|--------|-----------|-------|
| `battery` | 0-100 | percent |
| `charging` | bool | |
| `temperature` | float | °C |
| `button.<name>` | enum | `pressed`, `released`, `held` (namespace: `left`, `right`, `center`) |
| `connection_rssi` | int | dBm, for BLE |

### 6.8 Feature-to-Command Mapping

Clients can verify support before sending commands using the following mapping between declared features and supported commands:

| Feature | Supported Commands |
|---------|--------------------|
| `speed` | `motor.run`, `motor.stop` |
| `power` | `motor.run` (with `mode="power"`) |
| `position` / `absolute_position` | `motor.goto`, `motor.reset` |
| `stall` / `load` | Generates stall/load events |
| `color` / `reflected` / `ambient` / `rgb` | `sensor.subscribe`, `sensor.read` |
| `distance` / `presence` | `sensor.subscribe`, `sensor.read` |
| `force` / `touched` | `sensor.subscribe`, `sensor.read` |
| `pixel` / `image` / `text` / `scroll` | `led.matrix.*` |
| `brightness` | `led.matrix.brightness` |
| `orientation` | `led.matrix.orientation` |
| `touch` | `sensor.subscribe`, `sensor.read` (emits `{x, y}` coordinates) |
| `pitch` / `roll` / `yaw` / `acceleration` / `angular_velocity` | `sensor.subscribe`, `sensor.read` |
| `gesture` | `sensor.subscribe`, `sensor.read` (emits discrete values) |
| `face_orientation` | `sensor.subscribe`, `sensor.read` |
| `beep` | `sound.beep` |
| `builtin` / `file` / `midi` | `sound.play` |
| `volume` | `sound.set_volume`, `sound.read` |

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

SSP follows semantic versioning. This document defines v0.7 (draft). Breaking changes increment the major version. Bridges and clients MUST declare their supported SSP version in capability messages and connection handshakes.

---

## 9. Future Considerations (v0.8+)

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
