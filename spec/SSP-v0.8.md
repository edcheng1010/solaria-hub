
# Solaria Standard Protocol (SSP) — v0.8
---

**Author:** Edward Cheng  
**License:** Apache 2.0  
**Status:** Draft  
**Date:** 2026-06-03

---

## Changelog for v0.8

### Revision 2 (2026-06-03) — conformance alignment with the SPIKE Prime reference bridge
These are draft corrections that make v0.8 accurately describe its own reference implementation; no behavioural change is required of an existing v0.8 bridge.
- Added canonical **`timer`** command category (`timer.get`, `timer.reset`) for hub-side elapsed time (§6.9).
- Documented `sound.beep` parameters (`freq`, `duration`) and its optional `wait` blocking behaviour (§6.3).
- Added the canonical **composite orientation read** (`sensor.read type=orientation` → `{pitch, roll, yaw}`) (§6.4.1).
- Clarified that `led.set` `color` MAY be an enum string when the port constraint declares `enum`, not only an RGB array (§3.1.1, §5.1).
- Clarified that `connection_rssi` is transport-provided and MAY be unavailable to the bridge itself (§6.7).
- Relaxed the non-standard-extension rule: extensions MUST be **either** registered in a published profile-extension appendix **or** `x_`-prefixed (§5.1).
- Added **Appendix §11 — Profile Extensions (SPIKE Prime 3.x)** documenting registered bridge-specific commands and predicate sensor reads.

### Revision 1 (2026-05-26) — initial v0.8
- Added `motor.set_acceleration` and `movement.set_acceleration` commands as canonical features.
- Added optional `wait` parameter to `sound.play` to control blocking behavior, with corresponding `sound_complete` event.
- Added MIDI/music canonical schema including note formatting, chords, and instrument/drum enums.
- Added optional `mode` parameter ("absolute" or "relative") to `motor.goto`.
- Clarified `motor.run` behavior when `duration` is omitted (runs indefinitely).

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
{"cmd": "led.set", "port": "status", "color": "red"}
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
  "ssp_version": "0.8",
  "encodings": ["json-utf8-newline", "binary"],
  "supports_batch": true,
  "system_metrics": ["battery", "charging", "temperature", "button.left", "button.right"],
  "ports": [
    {
      "id": "A",
      "type": "motor",
      "features": ["speed", "position", "stall", "power", "acceleration", "goto_modes"],
      "goto_modes": ["absolute", "relative"],
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
      "builtin_sounds": ["giggle", "alert", "fanfare"],
      "sound_wait_supported": true,
      "instruments": ["piano", "guitar", "synth_lead"],
      "drums": ["kick", "snare", "hi_hat_closed"]
    }
  ]
}
```

Clients SHOULD use this declaration to dynamically configure their interface.

### 5.1 Canonical Feature Schema

To ensure consistency, feature names MUST use the canonical names defined below based on the port type. Non-standard extensions MUST be **either** (a) registered in a published profile-extension appendix (e.g. Appendix §11 for the SPIKE Prime 3.x profile), **or** (b) prefixed with `x_`. Registration in a profile appendix lets a vendor profile expose ergonomic bare-named commands while keeping them clearly scoped and discoverable; the `x_` prefix remains the path for unregistered, experimental, or one-off extensions.

- **motor:** `speed`, `position`, `absolute_position`, `stall`, `load`, `power`, `acceleration`
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

*Note on `led.set` color: the `color` parameter accepts an **RGB array** `[r, g, b]` by default, or an **enum string** (e.g. `"red"`, `"off"`) when the port declares a `color` constraint of type `enum`. Clients SHOULD inspect the constraint to decide which form to send. Discrete-colour status LEDs (such as the SPIKE Prime hub light) declare the enum form.*

#### 5.2.1 Sensor-attached LED Displays

Sensor-attached LED displays are treated as `display` ports with specific dimensions and color depths. For example:
- **Distance sensor 4-LED indicator:** Advertises as `display` with `width: 2`, `height: 2`, `depth: "grayscale"`. Controlled via `led.matrix.pixel` commands.
- **3×3 Color Matrix accessory:** Advertises as `display` with `width: 3`, `height: 3`, `depth: "rgb"`. Controlled via `led.matrix.pixel` with `color: [r, g, b]`.

---

## 6. Standard Command Categories

| Category | Actions | Description |
|----------|---------|-------------|
| `motor` | `run`, `stop`, `goto`, `reset`, `set_acceleration` | DC and servo motor control |
| `movement` | `configure`, `drive`, `turn`, `stop`, `set_acceleration` | Coordinated drive-base motion |
| `led` | `set`, `off`, `animate`, `matrix.*` | LED and display control |
| `sound` | `beep`, `play`, `stop`, `set_volume`, `read` | Audio output |
| `sensor` | `subscribe`, `unsubscribe`, `read` | Sensor data management |
| `system` | `ping`, `info`, `reset`, `dfu`, `subscribe`, `read` | Bridge management and metrics |
| `orientation` | `set_yaw`, `reset_yaw`, `set_reference` | Hub orientation configuration |
| `timer` | `get`, `reset` | Hub-side elapsed timer |
| `batch` | N/A | Execute multiple commands |

### 6.1 `motor` and `movement` Enhancements

- **Motor Run Modes:** `motor.run` accepts an optional `mode` parameter: `"speed"` (default, closed-loop speed control) or `"power"` (open-loop raw duty cycle control). Bridges that support power mode MUST declare the `power` feature in their capabilities.
  ```json
  {"cmd": "motor.run", "port": "A", "speed": 75, "mode": "speed"}
  {"cmd": "motor.run", "port": "A", "speed": 50, "mode": "power"}
  ```
- **Duration on Run Commands:** `motor.run` and `movement.drive` support optional `duration` (number) and `duration_unit` (`"ms"`, `"degrees"`, `"rotations"`). When `duration` is omitted, the motor runs indefinitely until a `motor.stop` is issued. Bridges MUST treat the absence of `duration` as indefinite. `duration: 0` and negative durations are reserved and SHOULD be treated as `motor.stop` (zero-length run). Real-world distance units (cm, inches) are a **client-side convenience** — a client converts distance to degrees using a configured wheel circumference before sending; the wire carries only `ms`, `degrees`, or `rotations`.
  ```json
  {"cmd": "motor.run", "port": "A", "speed": 75, "duration": 360, "duration_unit": "degrees"}
  ```
- **Motor Goto Modes:** `motor.goto` accepts an optional `mode` parameter: `"absolute"` (default) or `"relative"`. Absolute mode goes to a specific position, while relative mode advances by a specific amount of degrees from the current position. Bridges declare supported modes via the `goto_modes` array in the motor capability. Omitting the mode defaults to absolute, ensuring backward compatibility.
  ```json
  {"cmd": "motor.goto", "port": "A", "position": 90, "mode": "absolute"}
  {"cmd": "motor.goto", "port": "A", "position": 45, "mode": "relative"}
  ```
- **Motor Acceleration:** `motor.set_acceleration` and `movement.set_acceleration` allow configuring acceleration ramps. The `rate` parameter represents the time in milliseconds to ramp from 0 to 100% speed (constraint: int, min:0, max:10000). For `movement`, this applies to the configured motor pair. Bridges that do not support hardware-level acceleration can ignore this command or implement client-side stepped velocity.
  ```json
  {"cmd": "motor.set_acceleration", "port": "A", "rate": 500}
  {"cmd": "movement.set_acceleration", "rate": 1000}
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

**`sound.beep`** plays a single tone (requires the `beep` feature):
- `freq` (int, Hz) — tone frequency. Default 440.
- `duration` (int, ms) — tone length. If `duration` is omitted, the tone plays indefinitely until `sound.stop`.
- `wait` (bool, default `false`) — when `true`, playback blocks until the tone finishes, so successive beeps play sequentially (used for note/melody sequencing). When `false`, the call returns immediately. Bridges that cannot block MUST declare `"sound_wait_supported": false`.
```json
{"cmd": "sound.beep", "freq": 440, "duration": 500}
{"cmd": "sound.beep", "freq": 880, "duration": 250, "wait": true}
```

The `sound.play` command accepts different payload shapes depending on the supported features of the `speaker` port:
- **Built-in sounds:** `{"cmd": "sound.play", "sound": "giggle"}` (requires `builtin` feature).
- **File playback:** `{"cmd": "sound.play", "file": "alert.wav"}` (requires `file` feature).
- **MIDI notes:** `{"cmd": "sound.play", "notes": "C4:200 E4:200 G4:400", "tempo": 120}` (requires `midi` feature).

**Blocking vs Non-Blocking Playback:**
The `sound.play` command accepts an optional `wait` parameter. 
- `wait: false` (default): Playback is non-blocking.
- `wait: true`: Playback blocks until complete. When `wait: true` is provided, the bridge SHOULD emit `{"event": "sound_complete"}` when playback finishes. Clients can use the event and `request_id` correlation to chain follow-up commands.
- Bridges that cannot block must declare `"sound_wait_supported": false` in their capability declaration.

Volume control and retrieval are available if the port advertises the `volume` feature:
```json
{"cmd": "sound.set_volume", "level": 75}
{"cmd": "sound.read", "metric": "volume"}
```
The volume getter returns the current volume level.

#### 6.3.1 MIDI/Music Canonical Schema

For bridges supporting the `midi` feature, notes are specified using a canonical string format:
`<note>[<accidental>]<octave>:<duration>`
- **note:** A-G
- **accidental:** `#` (sharp) or `b` (flat)
- **octave:** integer (middle C = 4)
- **duration:** beats relative to tempo

**Special Notes:**
- Rest: `R:<duration>`

**Chords:**
Chords are represented by square brackets enclosing simultaneous notes with a single duration:
`[C4 E4 G4]:1`

**Instruments and Drums:**
Bridges that support MIDI playback can specify instruments and drums. Supported items are listed in the capability declaration under `"instruments"` and `"drums"` arrays.
- **Instrument Enum:** `piano`, `electric_piano`, `organ`, `guitar`, `electric_guitar`, `bass`, `violin`, `trumpet`, `sax`, `flute`, `marimba`, `synth_lead`, `synth_pad`
- **Drum Enum:** `kick`, `snare`, `hi_hat_closed`, `hi_hat_open`, `crash`, `ride`, `tom_low`, `tom_mid`, `tom_high`, `clap`, `cowbell`

### 6.4 `orientation` Category

The `orientation` port provides spatial tracking and configuration:
- `orientation.set_yaw`: Set the yaw to an arbitrary angle (e.g., `{"cmd": "orientation.set_yaw", "angle": 90}`).
- `orientation.reset_yaw`: Reset the yaw to zero (e.g., `{"cmd": "orientation.reset_yaw"}`).
- `orientation.set_reference`: Configure the hub's mounting orientation. Takes a `face` parameter matching the `face_orientation` enum values (e.g., `{"cmd": "orientation.set_reference", "face": "face_up"}`).

#### 6.4.1 Composite Orientation Read

In addition to reading `pitch`, `roll`, and `yaw` individually, an `orientation` port MAY support a **composite read** that returns all three Euler angles in a single event. This avoids three round-trips when a client needs a full attitude snapshot. Supported via `sensor.read` (and `sensor.subscribe`) with `type: "orientation"`:
```json
{"cmd": "sensor.read", "port": "imu", "type": "orientation"}
```
```json
{"event": "sensor", "port": "imu", "type": "orientation", "value": {"pitch": 12, "roll": -3, "yaw": 90}}
```
Angles are degrees. A bridge advertising `pitch`, `roll`, and `yaw` features MAY also accept `type: "orientation"`; clients SHOULD fall back to individual reads if a composite read returns an error.

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
| `connection_rssi` | int | dBm, for BLE. **Transport-provided:** this is the link signal strength measured by whichever peer can read it — typically the BLE central (the client side), not the bridge firmware. A bridge that cannot read its own RSSI MAY omit this metric or report it as unavailable; the client SHOULD source it from the transport layer instead. |

### 6.8 Feature-to-Command Mapping

Clients can verify support before sending commands using the following mapping between declared features and supported commands:

| Feature | Supported Commands |
|---------|--------------------|
| `speed` | `motor.run`, `motor.stop` |
| `power` | `motor.run` (with `mode="power"`) |
| `position` / `absolute_position` | `motor.goto`, `motor.reset` |
| `acceleration` | `motor.set_acceleration`, `movement.set_acceleration` |
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

### 6.9 `timer` Category

Bridges MAY provide a hub-side elapsed timer, useful when the client wants timing measured by the device rather than by the (possibly throttled or backgrounded) client clock. A bridge supporting this declares no special port; the timer is a bridge-level facility.

- `timer.get`: Request the current elapsed time. The bridge responds with a `sensor` event on the synthetic `timer` port:
  ```json
  {"cmd": "timer.get", "request_id": "t1"}
  ```
  ```json
  {"event": "sensor", "port": "timer", "type": "elapsed", "value": 42, "request_id": "t1"}
  ```
  `value` is whole seconds since the last reset (bridges MAY define a finer unit; the SPIKE Prime profile uses seconds).
- `timer.reset`: Reset the elapsed time to zero. No event is emitted.
  ```json
  {"cmd": "timer.reset"}
  ```

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

SSP follows semantic versioning. This document defines v0.8 (draft). Breaking changes increment the major version. Bridges and clients MUST declare their supported SSP version in capability messages and connection handshakes.

---

## 9. Future Considerations (v0.9+)

- Firmware update over SSP (DFU protocol extension)
- Stream multiplexing for multi-device control
- Authentication and pairing security model

---

## 10. Reference Implementations

| Component | Repository |
|-----------|-----------|
| SPIKE Prime Bridge + App Inventor Client | [solaria-appinventor-spike-prime](https://github.com/edcheng1010/solaria-appinventor-spike-prime) |
| SPIKE Prime client library (TypeScript / Web Bluetooth) | [solaria-lib-spike-prime](https://github.com/edcheng1010/solaria-lib-spike-prime) |
| Scratch (TurboWarp) client | [solaria-scratch-spike-prime](https://github.com/edcheng1010/solaria-scratch-spike-prime) |

---

## 11. Appendix — Profile Extensions (SPIKE Prime 3.x)

These commands and sensor-read types are **registered extensions** of the `spike-prime-3.x` transport profile. They are not part of the core canonical surface (§6), but are published here so that any client targeting a SPIKE Prime bridge can implement them interoperably. Per §5.1, registration in this appendix exempts them from the `x_` prefix requirement. Bridges for other hardware are not required to implement them; clients SHOULD gate their use on `device == "spike-prime"` in the capability declaration.

### 11.1 Predicate sensor reads (hub-side comparison)

Block-based environments (App Inventor, Scratch) benefit from a hub-side boolean test rather than reading a value and comparing client-side. These are issued via `sensor.read` (one-shot) or `sensor.subscribe`, and emit a standard `sensor` event whose `type` echoes the predicate. The canonical value reads (`color`, `distance`, `reflected`, etc.) remain available for clients that prefer to compare client-side.

| `type` | Extra params | Event `value` shape |
|--------|--------------|---------------------|
| `is_color` | `color` (enum string) | `{"match": <bool>, "color": "<queried>"}` |
| `is_closer` | `mm` (int) | `<bool>` |
| `is_reflected_above` | `percent` (int) | `<bool>` |
| `is_tilted` | `direction` (`forward`/`backward`/`left`/`right`/`any`) | `{"tilted": <bool>, "direction": "<dir>"}` |
| `is_orientation` | `face` (face_orientation enum) | `{"match": <bool>, "face": "<queried>"}` |
| `is_shaking` | — | `<bool>` |

```json
{"cmd": "sensor.read", "port": "C", "type": "is_color", "color": "red", "request_id": "q1"}
{"event": "sensor", "port": "C", "type": "is_color", "value": {"match": true, "color": "red"}, "request_id": "q1"}
```

Tilt threshold is ±20° on the relevant axis. `is_tilted`, `is_orientation`, and `is_shaking` are read on the `imu` port.

### 11.2 `system.read` predicate metric

| Metric | Extra params | Event |
|--------|--------------|-------|
| `is_button_pressed` | `button` (`left`/`right`/`center`) | `{"event": "system", "metric": "is_button_pressed", "value": {"button": "<name>", "pressed": <bool>}}` |

Distinct from the canonical `button.<name>` metric (§6.7), which reports the `pressed`/`released`/`held` enum. This predicate form answers a single yes/no query and is convenient for block-based polling.

### 11.3 `led.distance` — distance-sensor 4-LED indicator

Sets the four indicator LEDs on a SPIKE Prime distance sensor directly. (The §5.2.1 canonical approach models this as a `display` port driven by `led.matrix.pixel`; `led.distance` is the bridge's ergonomic shortcut.)

```json
{"cmd": "led.distance", "port": "B", "tl": 100, "tr": 100, "bl": 0, "br": 0}
```
`tl`/`tr`/`bl`/`br` are top-left/top-right/bottom-left/bottom-right brightness, 0–100.

### 11.4 `led.matrix.rotate` — incremental rotation

Rotates the 5×5 matrix by a relative amount, complementing the canonical absolute `led.matrix.orientation` (§6.2).
```json
{"cmd": "led.matrix.rotate", "degrees": 90}
```
`degrees` is added to the current orientation (mod 360).

### 11.5 `sound.rest` — blocking silence

Inserts a timed silence, used for melody/rhythm sequencing alongside `sound.beep` with `wait: true`.
```json
{"cmd": "sound.rest", "duration": 250}
```
`duration` is in milliseconds; the call blocks for that interval.

---

*This specification is a living document. Contributions and feedback welcome via GitHub Issues.*
