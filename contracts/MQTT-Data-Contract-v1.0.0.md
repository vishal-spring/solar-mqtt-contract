# 1. MQTT Data Contract Specification

| Property | Value |
|----------|-------|
| Document Title | MQTT Data Contract Specification |
| Project | Commercial Solar Rooftop Hybrid System |
| Publisher | Solar Hybrid System Simulator |
| Subscriber | Energy Management System (EMS) |
| Repository | solar-mqtt-contract |
| Version | 1.0.0 |
| Status | Draft |
| Last Updated | 20 July 2026 |

---

# 2. Purpose

This document defines the MQTT communication contract between the Commercial Solar Rooftop Hybrid System Simulator (Publisher) and the Energy Management System (Subscriber).

It serves as the single source of truth for both Publisher and Subscriber implementations by defining a common MQTT communication specification to ensure interoperability between both systems.

Any modification to this contract shall be reviewed and approved by both teams before implementation.

---

# 3. Scope

It specifies:

- MQTT protocol configuration and communication requirements.
- Topic hierarchy, naming conventions and device identifiers.
- Common telemetry payload structure.
- Device-specific telemetry payload definitions.
- Common fault payload structure and fault publishing behavior.
- Device operating modes, statuses and fault code definitions.
- Message publishing behavior, offline detection and startup sequencing.
- Payload examples for all supported topics.
- Versioning and compatibility guidelines.

---

# 4. MQTT Configuration

| Property | Value |
|----------|-------|
| MQTT Protocol | MQTT 5.0 |
| Broker | Mosquitto 2.1.x |
| Transport | TCP |
| Default Port | 1883 |
| QoS | 1 (At Least Once) |
| Retain | false |
| Session Type | Persistent |
| Session Expiry | 86400 seconds (24 hours) |
| Keep Alive Interval | 60 seconds |
| Automatic Reconnect | Enabled |
| Payload Format | JSON (UTF-8) |

## 4.1 Quality of Service (QoS)

All telemetry and fault topics shall publish using **QoS 1 (At Least Once)**.

QoS 1 guarantees that every published message is delivered at least once to the Subscriber. Under certain network conditions, duplicate messages may be delivered. The Subscriber shall identify and process the latest message using the received timestamp.

Clients shall automatically reconnect after temporary network interruptions. Upon reconnection, subscriptions shall be restored using the persistent MQTT session.

## 4.2 Persistent Session

Persistent MQTT sessions shall be enabled.

```text
Clean Start = false
Session Expiry = 86400 seconds
```

This allows the MQTT broker to retain subscriptions and queue QoS 1 messages for temporarily disconnected Subscribers until the configured session expiry period elapses.

Message ordering is guaranteed only within a single MQTT topic. No ordering shall be assumed across different topics.

## 4.3 Offline Detection

A device shall be considered **OFFLINE** if no telemetry message is received within the configured offline threshold.

The offline threshold shall always be greater than the configured publish interval to accommodate temporary network delays.

Example:

```text
Publish Interval : 1 hour
Grace Period     : 20 minutes
Offline Threshold: 80 minutes
```

No explicit communication-loss message shall be published by the Publisher. Device offline detection is performed entirely by the Subscriber based on message reception.

## 4.4 Publish Behaviour

The EMS shall automatically register previously unknown Organizations, Sites and Devices upon receiving valid telemetry for the first time.

Telemetry topics shall be published periodically at the configured publish interval.

The default publish interval is:

```text
1 hour
```

The Publisher may configure the publish interval as required. The configured interval shall apply uniformly to all telemetry topics.

Fault topics are event-driven and shall be published only when:

- A fault is raised.
- A fault is cleared.

Fault topics shall not be published periodically while a fault remains active.

---

# 5. MQTT Topic Hierarchy

All MQTT topics shall follow a hierarchical structure based on the Organization, Site and Device Identifiers.

The topic hierarchy is designed to:

- Support multiple organizations using a common MQTT broker.
- Allow topic-based subscription filtering.
- Maintain a consistent naming convention across all telemetry and fault topics.

## 5.1 Topic Naming Convention

All topic names shall:

- Use lowercase English alphabets.
- Use forward slash (`/`) as the hierarchy separator.
- Use hyphen (`-`) within identifiers where required.
- Not contain spaces or special characters.
- Keep device identifiers unique only within a site.

General topic format:

```text
org/{orgId}/site/{siteId}/{deviceType}/{deviceId}
```

Fault topic format:

```text
org/{orgId}/site/{siteId}/fault
```

## 5.2 Topic Structure

| Component | Topic Format |
|----------|--------------|
| Weather Station | `org/{orgId}/site/{siteId}/weather/{weatherId}` |
| PV String | `org/{orgId}/site/{siteId}/string/{stringId}` |
| PV Aggregate | `org/{orgId}/site/{siteId}/string/string-all` |
| Battery | `org/{orgId}/site/{siteId}/battery/{batteryId}` |
| Smart Hybrid Inverter | `org/{orgId}/site/{siteId}/inverter/{inverterId}` |
| Smart Meter | `org/{orgId}/site/{siteId}/meter/{meterId}` |
| Fault | `org/{orgId}/site/{siteId}/fault` |

## 5.3 Device Identifier Convention

The following device identifiers shall be used.

| Device | Identifier |
|--------|------------|
| Weather Station | `weather-001` |
| Smart Hybrid Inverter | `inv-001` |
| Smart Meter | `meter-001` |
| PV Aggregate | `string-all` |
| PV String | `string-001`, `string-002`, ... |
| Battery | `bat-001`, `bat-002`, ... |

## 5.4 Identifier Scope

Identifier uniqueness shall follow the hierarchy below.

| Identifier | Uniqueness Scope |
|------------|------------------|
| orgId | Globally unique |
| siteId | Unique within an Organization |
| inverterId | Unique within a Site |
| weatherId | Unique within a Site |
| meterId | Unique within a Site |
| stringId | Unique within an Inverter |
| batteryId | Unique within an Inverter |

## 5.5 Fault Topic

The Fault topic is an event-driven topic used to notify the Subscriber whenever a device fault is raised or cleared.

The Fault topic shall not be published periodically.

Whenever a fault is raised:

- The corresponding device telemetry topic shall be published with its latest telemetry values and `status = FAULT`.
- The Fault topic shall be published with the corresponding fault code.

Whenever a fault is cleared:

- The corresponding device telemetry topic shall be published with its latest telemetry values and `status = ONLINE`.
- The Fault topic shall be published with `faultCode = NONE`.

The Fault topic does not replace device telemetry. It is intended solely for communicating fault state transitions.

## 5.6 PV String Fault Handling

Faults are applicable only to individual PV Strings.

The aggregate PV topic (`string-all`) shall never publish a fault code.

Whenever a fault occurs in a PV String:

- The affected PV String telemetry shall be published immediately.
- The PV Aggregate (`string-all`) telemetry shall also be published immediately to reflect the updated aggregate measurements.
- A corresponding Fault topic message shall be published.

Whenever the fault is cleared:

- The affected PV String telemetry shall be published immediately with the updated status.
- The PV Aggregate (`string-all`) telemetry shall be published again.
- A corresponding Fault topic message shall be published with `faultCode = NONE`.

---

# 6. Common Payload Structure

All telemetry topics shall follow a common payload structure to provide consistent identification and metadata across all devices.

Every device-specific payload shall include the following mandatory fields before its respective telemetry parameters.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| schemaVersion | String | Yes | Contract version used by the payload. |
| timestamp | String | Yes | Local timestamp in `yyyy-MM-dd'T'HH:mm:ss` format. |
| orgId | String | Yes | Organization identifier. |
| siteId | String | Yes | Site identifier unique within the organization. |
| deviceId | String | Yes | Device identifier unique within the site. |
| status | String | Yes | Current operational status of the device. |

### Example

```json
{
  "schemaVersion": "1.0.0",
  "timestamp": "2026-07-17T10:30:15",
  "orgId": "org-001",
  "siteId": "site-001",
  "deviceId": "inv-001",
  "status": "ONLINE"
}
```
- Device-specific telemetry parameters shall be appended after the common payload fields.

---

# 7. Common Status Definitions

The `status` field represents the current operational state of the publishing device.

| Status | Description |
|--------|-------------|
| ONLINE | Device is operating normally and publishing telemetry. |
| OFFLINE | Device telemetry has not been received within the configured offline threshold. |
| FAULT | Device is operating with an active fault condition. |
| DEGRADED | Device is operational but experiencing reduced performance. Applicable only to PV Strings and Batteries. |
| NONE | Indicates that the status is not applicable. Used only for the aggregate PV topic (`string-all`). |

## Notes

- `FAULT` indicates an active fault condition and shall be accompanied by a corresponding Fault topic message.
- `OFFLINE` is determined by the Subscriber when telemetry is not received within the configured offline threshold.
- The aggregate PV topic (`string-all`) is a logical representation of all PV Strings and therefore always publishes `status = NONE`.

---

# 8. Common Fault Payload

The Fault topic is published only when a device fault is raised or cleared.

General Topic Format:

```text
org/{orgId}/site/{siteId}/fault
```

The payload shall contain the following fields.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| schemaVersion | String | Yes | Contract version used by the payload. |
| timestamp | String | Yes | Local timestamp in `yyyy-MM-dd'T'HH:mm:ss` format. |
| orgId | String | Yes | Organization identifier. |
| siteId | String | Yes | Unique identifier of the site. |
| inverterId | String | Yes | Connected inverter identifier. |
| deviceId | String | Yes | Device identifier reporting the fault. |
| status | String | Yes | ONLINE or FAULT. |
| faultCode | String | Yes | Active fault code. `NONE` indicates that the fault has been cleared. |

## Fault Publishing Rules

- Fault messages are event-driven.
- Fault messages shall not be published periodically.
- A fault message shall be published only when a fault is raised or cleared.
- Only one active fault code shall be published for a device at any time.
- When fault is cleared, `faultCode` shall be published as `NONE` and `status` shall be set to `ONLINE`.
- The `inverterId` field shall always be present.
- For PV Strings, PV Aggregate and Batteries, it shall contain the associated inverter identifier.
- For Smart Hybrid Inverters, Smart Meters and Weather Stations, it shall be `NONE`.

---

# 9. Weather Station

## Topic

```text
org/{orgId}/site/{siteId}/weather/{weatherId}
```

## Payload

| Field | Type | Unit | Required | Description |
|------|------|------|----------|-------------|
| ambientTemperature | Number | °C | Yes | Ambient air temperature. |
| moduleTemperature | Number | °C | Yes | Solar panel surface temperature. |
| irradiance | Number | W/m² | Yes | Solar irradiance incident on the PV array. |
| windSpeed | Number | m/s | Yes | Wind speed. |
| humidity | Number | % | Yes | Relative humidity. |
| cloud_cover | Number | Index (0–1) | Yes | Fraction of the sky covered by clouds (0 = clear, 1 = overcast). |

## Status

Supported Status:

- ONLINE
- OFFLINE

## Faults

Not Applicable.

The Weather Station shall not publish any Fault topic messages.

---

# 10. PV Array

The PV Array consists of one aggregate topic (`string-all`) and one or more individual PV String topics.

Each PV String represents an independent DC generation source connected to the inverter.

The aggregate topic represents the combined output of all PV Strings connected to an inverter.

---

## 10.1 PV String

### Topic

```text
org/{orgId}/site/{siteId}/string/{stringId}
```

### Payload

| Field | Type | Unit | Required | Description |
|------|------|------|----------|-------------|
| inverterId | String | — | Yes | Connected inverter identifier. |
| dcVoltage | Number | V | Yes | DC output voltage of the string. |
| dcCurrent | Number | A | Yes | DC output current of the string. |
| dcPower | Number | kW | Yes | DC output power of the string. |

### Status

Supported Status:

- ONLINE
- DEGRADED
- OFFLINE
- FAULT

### Fault Codes

- NONE
- SHORT_CIRCUIT
- OPEN_CIRCUIT
- CELL_DEGRADATION
- DC_OVER_VOLTAGE
- DC_OVER_CURRENT

---

## 10.2 PV Aggregate

### Topic

```text
org/{orgId}/site/{siteId}/string/string-all
```

### Payload

| Field | Type | Unit | Required | Description |
|------|------|------|----------|-------------|
| inverterId | String | — | Yes | Connected inverter identifier. |
| dcVoltage | Number | V | Yes | Aggregate DC input voltage supplied to the inverter. |
| dcCurrent | Number | A | Yes | Aggregate DC current from all PV Strings. |
| dcPower | Number | kW | Yes | Aggregate DC power from all PV Strings. |

### Status

Supported Status:

- NONE

### Fault Codes

Not Applicable.

---

## Notes

- There shall be one or more PV Strings connected to an inverter.
- There shall be exactly one aggregate PV topic (`string-all`) per inverter.
- Faults apply only to individual PV Strings.
- The aggregate PV topic shall never publish fault codes.
- Whenever a PV String fault is raised or cleared:
  - The affected PV String telemetry shall be published immediately.
  - The aggregate PV telemetry (`string-all`) shall also be published immediately to reflect the updated aggregate values.
  - A corresponding Fault topic message shall be published.
 
---

# 11. Battery

Each Battery shall be associated with exactly one Smart Hybrid Inverter.

## Topic

```text
org/{orgId}/site/{siteId}/battery/{batteryId}
```

## Payload

| Field | Type | Unit | Required | Description |
|------|------|------|----------|-------------|
| inverterId | String | — | Yes | Connected inverter identifier. |
| stateOfCharge | Number | % | Yes | Current battery State of Charge (SOC). |
| stateOfHealth | Number | % | Yes | Current battery State of Health (SOH). |
| packVoltage | Number | V | Yes | Battery pack voltage. |
| packCurrent | Number | A | Yes | Battery charge/discharge current. |
| packTemperature | Number | °C | Yes | Battery pack temperature. |
| cycleCount | Integer | Count | Yes | Total charge-discharge cycles. |
| batteryState | String | — | Yes | CHARGING, DISCHARGING or IDLE. |

## Status

Supported Status:

- ONLINE
- DEGRADED
- OFFLINE
- FAULT

## Fault Codes

- NONE
- OVER_CHARGE
- OVER_DISCHARGE
- OVER_TEMPERATURE

---

# 12. Smart Hybrid Inverter

## Topic

```text
org/{orgId}/site/{siteId}/inverter/{inverterId}
```

## Payload

| Field | Type | Unit | Required | Description |
|------|------|------|----------|-------------|
| dcInputVoltage | Number | V | Yes | Aggregate DC voltage received from the PV Array. |
| dcInputCurrent | Number | A | Yes | Aggregate DC current received from the PV Array. |
| dcInputPower | Number | kW | Yes | Aggregate DC power received from the PV Array. |
| batteryVoltage | Number | V | Yes | Battery port voltage. |
| batteryCurrent | Number | A | Yes | Battery port current. |
| batteryPower | Number | kW | Yes | Battery charge/discharge power. |
| acOutputVoltage | Number | V | Yes | AC output voltage. |
| acOutputCurrent | Number | A | Yes | AC output current. |
| acOutputPower | Number | kW | Yes | AC output power. |
| gridFrequency | Number | Hz | Yes | Output frequency. |
| efficiency | Number | % | Yes | Inverter conversion efficiency. |
| operatingMode | String | — | Yes | Current inverter operating mode. |
| internalTemperature | Number | °C | Yes | Internal inverter temperature. |
| cumulativeEnergyYield | Number | kWh | Yes | Total cumulative energy generated. |

## Supported Operating Modes

It represents the current operating strategy of the inverter based on the availability of solar generation, battery state and utility grid.

| Operating Mode | Description |
|---------------|-------------|
| STANDBY | Grid available, no load demand, battery idle and no solar generation. |
| GRID_ONLY | Grid supplies the entire load. No solar generation. Battery idle. |
| SOLAR_ONLY | Solar supplies the entire load. No battery activity and no grid interaction. |
| BACKUP | Battery alone supplies the load during grid outage. |
| SOLAR_BATTERY | Solar and battery jointly supply the load during grid outage. |
| SOLAR_GRID | Solar generation is insufficient to meet the load demand. The utility grid supplies the remaining load while the battery remains idle or unavailable. |
| BATTERY_CHARGING | No load demand. Solar charges the battery. |
| SOLAR_CHARGING | Solar supplies the load and charges the battery using surplus generation. |
| GRID_TIE | Battery is fully charged. Solar generation exceeds the load demand and surplus energy is exported to the utility grid. |
| MAINTENANCE | System is under maintenance. |
| FAULT | Inverter is operating under an active fault condition. Refer to the Fault topic for the corresponding fault code. |

## Status

Supported Status:

- ONLINE
- OFFLINE
- FAULT

## Fault Codes

- NONE
- OVER_TEMPERATURE
- INVERTER_TRIP

---

# 13. Smart Meter

## Topic

```text
org/{orgId}/site/{siteId}/meter/{meterId}
```

## Payload

| Field | Type | Unit | Required | Description |
|------|------|------|----------|-------------|
| gridVoltage | Number | V | Yes | Aggregate grid voltage. |
| gridCurrent | Number | A | Yes | Aggregate grid current. |
| activePower | Number | kW | Yes | Net active power exchanged with the grid. Positive indicates import and negative indicates export. |
| reactivePower | Number | kVAr | Yes | Reactive power exchanged with the grid. |
| powerFactor | Number | - | Yes | Grid power factor. |
| importEnergy | Number | kWh | Yes | Cumulative imported energy. |
| exportEnergy | Number | kWh | Yes | Cumulative exported energy. |
| frequency | Number | Hz | Yes | Grid frequency. |

## Status

Supported Status:

- ONLINE
- OFFLINE
- FAULT

## Fault Codes

- NONE
- GRID_OUTAGE
- GRID_OVER_VOLTAGE
- GRID_OVER_CURRENT

## Notes

- The Smart Meter represents the single point of grid interconnection.
- Import and Export Energy are cumulative counters and shall continuously increase during normal operation.
- Counter reset behavior, if applicable, is implementation-specific and shall be documented by the Publisher.
- Whenever a meter fault is raised or cleared:
  - The Smart Meter telemetry shall be published immediately.
  - A corresponding Fault topic message shall be published.

---

# 14. Glossary

| Term | Description |
|------|-------------|
| EMS | Energy Management System |
| MQTT | Message Queuing Telemetry Transport 
| QoS | Quality of Service |
| PV | Photovoltaic |
| DC | Direct Current |
| AC | Alternating Current |
| SOC | State of Charge |
| SOH | State of Health |
| JSON | JavaScript Object Notation |
| UTF-8 | Unicode Transformation Format - 8-bit |
| Telemetry | Periodic operational data published by a device. |
| Fault Topic | Event-driven MQTT topic used to publish device fault notifications. |
| Publisher | The Solar Hybrid System Simulator publishing MQTT messages. |
| Subscriber | The Energy Management System receiving MQTT messages. |
| Site | A single Commercial Solar Rooftop Hybrid installation setup. |
| Device | Any telemetry-producing component within a Site. |
| Aggregate Topic | A logical MQTT topic representing cumulative measurements from multiple devices (for example, `string-all`). |

---
