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
| Last Updated | 15 July 2026 |

---

# 2. Purpose

This document defines the MQTT communication contract between the Commercial Solar Rooftop Hybrid System Simulator (Publisher) and the Energy Management System (Subscriber).

It establishes a common specification for MQTT communication to ensure interoperability between both systems.

---

# 3. Scope

This specification defines:

- MQTT connection configuration
- Topic hierarchy
- Topic naming conventions
- Payload schemas
- Publish intervals
- Quality of Service (QoS)
- Session configuration
- Validation rules
- Versioning policy

---

# 4. MQTT Configuration

| Property | Value |
|----------|-------|
| MQTT Protocol | MQTT 5.0 |
| Broker | Eclipse Mosquitto 2.x |
| Transport | TCP |
| Default Port | 1883 |
| QoS | 1 |
| Retain | false |
| Session Type | Persistent |
| Session Expiry | 86400 seconds (24 hours) |
| Keep Alive | 60 seconds |
| Automatic Reconnect | Enabled |
| Payload Format | JSON |
| Payload Encoding | UTF-8 |

---

# 5. Topic Hierarchy

All telemetry topics shall follow the hierarchy below.

```text
site/{siteId}/weather

site/{siteId}/array/{stringId}

site/{siteId}/inverter/{inverterId}

site/{siteId}/battery/{batteryId}

site/{siteId}/meter/{meterId}
```

### 5.1 Topic Definitions

| Topic | Description |
|--------|-------------|
| `site/{siteId}/weather` | Weather station telemetry |
| `site/{siteId}/array/{stringId}` | PV string telemetry |
| `site/{siteId}/inverter/{inverterId}` | Inverter telemetry |
| `site/{siteId}/battery/{batteryId}` | Battery telemetry |
| `site/{siteId}/meter/{meterId}` | Smart meter telemetry |

### 5.2 Example Topics

```text
site/site-001/weather

site/site-001/array/string-001

site/site-001/inverter/inv-001

site/site-001/battery/bat-001

site/site-001/meter/meter-001
```

---

# 6. Topic Naming Convention

## 6.1 General Rules

- All topic names shall use lowercase characters.
- Topic levels shall be separated using `/`.
- Topic names shall not contain spaces.
- Topic names shall use hyphens (`-`) where required.
- MQTT wildcards (`+` and `#`) shall not be used in published topic names.

---

## 6.2 Site Identifier

Format

```text
site-###
```

Examples

```text
site-001
site-002
site-105
```

---

## 6.3 Device Identifiers

| Device | Format | Example |
|---------|--------|---------|
| PV String | `string-###` | `string-001` |
| Inverter | `inv-###` | `inv-001` |
| Battery | `bat-###` | `bat-001` |
| Meter | `meter-###` | `meter-001` |

---

## 6.4 Publisher Topics

The simulator shall publish only to the defined telemetry topics.

Examples

```text
site/site-001/weather

site/site-001/array/string-001

site/site-001/inverter/inv-001

site/site-001/battery/bat-001

site/site-001/meter/meter-001
```

---

## 6.5 Subscriber Topics

The EMS shall subscribe to all telemetry topics.

```text
site/+/#
```

or individual subscriptions

```text
site/+/weather

site/+/array/+

site/+/inverter/+

site/+/battery/+

site/+/meter/+
```

---

# 7. Publish Policy

## 7.1 Publish Intervals

| Device | Publish Interval |
|---------|------------------|
| Weather | 30 seconds |
| PV String | 10 seconds |
| Inverter | 5 seconds |
| Battery | 10 seconds |
| Smart Meter | 5 seconds |

---

## 7.2 Initial Publish Delay

To avoid simultaneous publishing after startup, each simulator component shall introduce a random initial delay before its first publish.

| Device | Initial Delay |
|---------|---------------|
| Weather | 0–30 seconds |
| PV String | 0–10 seconds |
| Inverter | 0–5 seconds |
| Battery | 0–10 seconds |
| Smart Meter | 0–5 seconds |

After the initial delay, each device shall publish at its configured interval.

---

## 7.3 Timestamp

Every published message shall include a timestamp.

| Property | Value |
|----------|-------|
| Format | ISO-8601 |
| Timezone | UTC |
| Example | `2026-07-15T10:35:20` |

---

## 7.4 Publish Rules

- Each message shall contain the latest available telemetry.
- A device shall publish only to its assigned topic.
- The publisher shall continue publishing regardless of whether values have changed.

---

# 8. Common Payload Schema

Every telemetry message shall contain the following common fields.

| Field | Type | Required | Description |
|--------|------|----------|-------------|
| schemaVersion | String | Yes | Contract version |
| timestamp | String | Yes | Message generation time (ISO-8601 UTC) |
| siteId | String | Yes | Unique site identifier |
| deviceId | String | Yes | Unique device identifier |
| deviceType | String | Yes | Device category |
| status | String | Yes | Current device status |

---

## 8.1 Device Types

| Value |
|-------|
| WEATHER |
| PV_STRING |
| INVERTER |
| BATTERY |
| METER |

---

## 8.2 Device Status

| Value | Description |
|-------|-------------|
| HEALTHY | Device operating normally |
| DEGRADED | Operating with reduced performance |
| FAULT | Device has reported a fault |
| OFFLINE | Device communication lost |

---

## 8.3 Common Payload Example

```json
{
  "schemaVersion": "1.0.0",
  "timestamp": "2026-07-15T10:35:20",
  "siteId": "site-001",
  "deviceId": "inv-001",
  "deviceType": "INVERTER",
  "status": "HEALTHY"
}
```

---

# 9. Weather Payload Specification
## 9.1 Payload Fields

| Field | Type | Unit | Required | Description |
|--------|------|------|----------|-------------|
| irradiance | Double | W/m² | Yes | Solar irradiance |
| ambientTemperature | Double | °C | Yes | Ambient air temperature |
| moduleTemperature | Double | °C | Yes | Back of PV module temperature |
| humidity | Double | % | Yes | Relative humidity |
| windSpeed | Double | m/s | Yes | Wind speed |
| cloudCover | Double | % | Yes | Cloud cover percentage |

---

## 9.2 Example Payload

```json
{
  "schemaVersion": "1.0.0",
  "timestamp": "2026-07-15T10:35:20",
  "siteId": "site-001",
  "deviceId": "weather",
  "deviceType": "WEATHER",
  "status": "HEALTHY",

  "irradiance": 845.6,
  "ambientTemperature": 31.2,
  "moduleTemperature": 44.1,
  "humidity": 58.3,
  "windSpeed": 3.5,
  "cloudCover": 18.0
}
```

---

# 10. PV Array

## 10.1 Topics

### Individual PV String

```text
site/{siteId}/array/{stringId}
```

### Aggregate PV Array

```text
site/{siteId}/array/string-all
```

---

## 10.2 Payload Fields

| Field | Type | Unit | Required | Description |
|--------|------|------|----------|-------------|
| parentDeviceId | String | - | Yes* | Connected inverter identifier (mandatory for individual strings only) |
| dcVoltage | Double | V | Yes | DC voltage |
| dcCurrent | Double | A | Yes | DC current |
| dcPower | Double | kW | Yes | DC power |
| status | String | - | Yes | Current operating status |

---

## 10.3 Status Values

| Value | Description |
|-------|-------------|
| HEALTHY | Operating normally |
| DEGRADED | Operating with reduced performance |
| FAULT | Fault detected |
| OFFLINE | Communication lost |

---

## 10.4 Individual PV String Example

```json
{
  "schemaVersion": "1.0.0",
  "timestamp": "2026-07-15T10:35:20",
  "siteId": "site-001",
  "deviceId": "string-001",
  "parentDeviceId": "inv-001",
  "deviceType": "PV_STRING",
  "status": "HEALTHY",
  "dcVoltage": 642.8,
  "dcCurrent": 11.9,
  "dcPower": 7.65
}
```

---

## 10.5 Aggregate PV Array Example

```json
{
  "schemaVersion": "1.0.0",
  "timestamp": "2026-07-15T10:35:20",
  "siteId": "site-001",
  "deviceId": "string-all",
  "parentDeviceId": "inv-001",
  "deviceType": "PV_STRING",
  "status": "NONE",
  "dcVoltage": 648.5,
  "dcCurrent": 47.6,
  "dcPower": 30.84
}
```

---

## 10.6 Notes

- Every PV String shall belong to exactly one inverter.
- `parentDeviceId` shall reference the associated inverter and is mandatory for both individual and aggregate PV Array messages.
- Aggregate PV Array messages shall use `deviceId` as `string-all`.
- Individual PV String messages shall use unique device identifiers (e.g., `string-001`, `string-002`).
- `dcPower` in the aggregate message shall be the sum of the DC power of all PV Strings.
- `dcCurrent` in the aggregate message shall be the total DC input current to the inverter.
- `dcVoltage` in the aggregate message shall represent the inverter DC input voltage.
- `status` in the aggregate message shall always be `NONE`.
- Individual and aggregate PV Array messages shall be published independently.

---

# 11. Smart Hybrid Inverter
## 11.1 Payload Fields

| Field | Type | Unit | Required | Description |
|--------|------|------|----------|-------------|
| dcInputVoltage | Double | V | Yes | Total DC input voltage from PV array |
| dcInputCurrent | Double | A | Yes | Total DC input current from PV array |
| dcInputPower | Double | kW | Yes | Total DC input power from PV array |
| batteryVoltage | Double | V | Yes | Battery port voltage |
| batteryCurrent | Double | A | Yes | Battery port current |
| batteryPower | Double | kW | Yes | Battery charge/discharge power |
| acOutputVoltage | Double | V | Yes | AC output voltage |
| acOutputCurrent | Double | A | Yes | AC output current |
| acOutputPower | Double | kW | Yes | AC output power |
| frequency | Double | Hz | Yes | AC output frequency |
| efficiency | Double | % | Yes | Inverter efficiency |
| operatingMode | String | - | Yes | Current operating mode |
| internalTemperature | Double | °C | Yes | Internal inverter temperature |
| faultCode | String | - | Yes | Active fault code |
| energyYield | Double | kWh | Yes | Cumulative energy generated |

---

## 11.2 Operating Modes

| Value |
|-------|
| GRID_TIE |
| BACKUP |
| CHARGING |
| DISCHARGING |
| STANDBY |
| FAULT |

---

## 11.3 Fault Codes

| Value |
|-------|
| NONE |
| OVER_TEMP |
| DC_OVERVOLTAGE |
| GRID_FAULT |
| COMM_LOST |

---

## 11.4 Example Payload

```json
{
  "schemaVersion": "1.0.0",
  "timestamp": "2026-07-15T10:35:20",
  "siteId": "site-001",
  "deviceId": "inv-001",
  "deviceType": "INVERTER",
  "status": "HEALTHY",

  "dcInputVoltage": 648.5,
  "dcInputCurrent": 35.8,
  "dcInputPower": 23.2,

  "batteryVoltage": 51.6,
  "batteryCurrent": -18.2,
  "batteryPower": -0.94,

  "acOutputVoltage": 415.2,
  "acOutputCurrent": 31.5,
  "acOutputPower": 21.8,
  "frequency": 49.98,

  "efficiency": 93.97,

  "operatingMode": "GRID_TIE",

  "internalTemperature": 42.8,

  "faultCode": "NONE",

  "energyYield": 15423.6
}
```

---

# 12. Battery Management System (BMS)
## 12.1 Payload Fields

| Field | Type | Unit | Required | Description |
|--------|------|------|----------|-------------|
| stateOfCharge | Double | % | Yes | Battery State of Charge (SOC) |
| stateOfHealth | Double | % | Yes | Battery State of Health (SOH) |
| packVoltage | Double | V | Yes | Battery pack voltage |
| packCurrent | Double | A | Yes | Battery pack current |
| packTemperature | Double | °C | Yes | Battery pack temperature |
| cycleCount | Integer | - | Yes | Charge/discharge cycle count |
| operatingState | String | - | Yes | Current battery operating state |

---

## 12.2 Operating State Values

| Value | Description |
|-------|-------------|
| CHARGING | Battery is charging |
| DISCHARGING | Battery is discharging |
| IDLE | Battery is neither charging nor discharging |

---

## 12.3 Status Values

| Value | Description |
|-------|-------------|
| HEALTHY | Operating normally |
| DEGRADED | Reduced performance |
| FAULT | Fault detected |
| OFFLINE | Communication lost |

---

## 12.4 Example Payload

```json
{
  "schemaVersion": "1.0.0",
  "timestamp": "2026-07-15T10:35:20",
  "siteId": "site-001",
  "deviceId": "bat-001",
  "deviceType": "BATTERY",
  "status": "HEALTHY",

  "stateOfCharge": 82.4,
  "stateOfHealth": 98.7,
  "packVoltage": 51.6,
  "packCurrent": -18.3,
  "packTemperature": 31.4,
  "cycleCount": 452,
  "operatingState": "DISCHARGING"
}
```

---

# 13. Smart Meter (Grid Interconnection Point)

## 13.1 Payload Fields

| Field | Type | Unit | Required | Description |
|--------|------|------|----------|-------------|
| gridVoltage | Double | V | Yes | Grid voltage |
| gridCurrent | Double | A | Yes | Grid current |
| activePower | Double | kW | Yes | Active power exchanged with the grid |
| reactivePower | Double | kVAr | Yes | Reactive power exchanged with the grid |
| powerFactor | Double | - | Yes | Power factor |
| importEnergy | Double | kWh | Yes | Cumulative imported energy |
| exportEnergy | Double | kWh | Yes | Cumulative exported energy |
| frequency | Double | Hz | Yes | Grid frequency |

---

## 13.2 Example Payload

```json
{
  "schemaVersion": "1.0.0",
  "timestamp": "2026-07-15T10:35:20",
  "siteId": "site-001",
  "deviceId": "meter-001",
  "deviceType": "METER",
  "status": "HEALTHY",

  "gridVoltage": 415.6,
  "gridCurrent": 28.4,
  "activePower": 18.72,
  "reactivePower": 3.46,
  "powerFactor": 0.983,
  "importEnergy": 15432.75,
  "exportEnergy": 9876.42,
  "frequency": 49.98
}
```

---

## 13.3 Notes

- `gridVoltage` and `gridCurrent` shall represent the aggregate electrical measurements at the grid connection.
- `activePower` shall be positive when importing power from the grid and negative when exporting power to the grid.
- `reactivePower` shall follow the same sign convention as `activePower`.
- `powerFactor` shall be reported as a value between **0.0** and **1.0**.
- `importEnergy` shall be a cumulative counter that increases whenever energy is imported from the grid.
- `exportEnergy` shall be a cumulative counter that increases whenever energy is exported to the grid.
- Energy counters shall not reset during normal operation and shall only reset upon meter replacement or explicit system initialization.

---

# 14. MQTT Topic Summary

| Device | Topic | Publish Interval |
|--------|-------|------------------|
| Weather Station | `site/{siteId}/weather` | 30 seconds |
| PV String | `site/{siteId}/array/{stringId}` | 10 seconds |
| PV Array (Aggregate) | `site/{siteId}/array/string-all` | 10 seconds |
| Smart Hybrid Inverter | `site/{siteId}/inverter/{inverterId}` | 5 seconds |
| Battery Management System | `site/{siteId}/battery/{batteryId}` | 10 seconds |
| Smart Meter | `site/{siteId}/meter/{meterId}` | 5 seconds |

---

# 15. MQTT Communication Configuration

## 15.1 Retry Policy

- Publishers shall retry message delivery as defined by MQTT QoS 1.
- Clients shall automatically reconnect after network interruptions.
- Upon reconnection, subscriptions shall be restored using the persistent session.

---

## 15.2 Message Ordering

- Ordering is guaranteed only within a single topic.
- No ordering shall be assumed across different topics.
- The EMS shall process messages independently for each topic.

---

## 15.3 Duplicate Messages

QoS 1 may result in duplicate message delivery.

The EMS shall process duplicate messages safely.

Duplicate detection may be performed using:

- Topic
- Timestamp
- Payload comparison

---

## 15.4 Offline Behaviour

When the publisher disconnects:

- Messages published during the outage may be queued by the broker for persistent sessions.
- Upon reconnection, queued messages shall be delivered to the subscriber.
- The EMS shall process delayed messages in the order received for each topic.

---

# 16. Revision History

| Version | Date | Description | Author |
|----------|------|-------------|--------|
| 1.0.0 | 15-Jul-2026 | Initial Draft | Simulator & EMS Team |

---
