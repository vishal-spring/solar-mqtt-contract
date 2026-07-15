# MQTT Data Contract Specification

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

# Revision History

| Version | Date | Description | Author |
|----------|------|-------------|--------|
| 1.0.0 | 15-Jul-2026 | Initial Draft | Simulator & EMS Team |

---

# Purpose

This document defines the MQTT communication contract between the Commercial Solar Rooftop Hybrid System Simulator (Publisher) and the Energy Management System (Subscriber).

It establishes a common specification for MQTT communication to ensure interoperability between both systems.

---

# Scope

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

# MQTT Configuration

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
| Payload Format | JSON |
| Payload Encoding | UTF-8 |

---

# Topic Hierarchy

All telemetry topics shall follow the hierarchy below.

```text
site/{siteId}/weather

site/{siteId}/array/{stringId}

site/{siteId}/inverter/{inverterId}

site/{siteId}/battery/{batteryId}

site/{siteId}/meter/{meterId}
```

### Topic Definitions

| Topic | Description |
|--------|-------------|
| `site/{siteId}/weather` | Weather station telemetry |
| `site/{siteId}/array/{stringId}` | PV string telemetry |
| `site/{siteId}/inverter/{inverterId}` | Inverter telemetry |
| `site/{siteId}/battery/{batteryId}` | Battery telemetry |
| `site/{siteId}/meter/{meterId}` | Smart meter telemetry |

### Example Topics

```text
site/site-001/weather

site/site-001/array/string-001

site/site-001/inverter/inv-001

site/site-001/battery/bat-001

site/site-001/meter/meter-001
```

---

# Topic Naming Convention

## General Rules

- All topic names shall use lowercase characters.
- Topic levels shall be separated using `/`.
- Topic names shall not contain spaces.
- Topic names shall use hyphens (`-`) where required.
- MQTT wildcards (`+` and `#`) shall not be used in published topic names.

---

## Site Identifier

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

## Device Identifiers

| Device | Format | Example |
|---------|--------|---------|
| Weather | `weather-###` | `weather-001` |
| PV String | `string-###` | `string-001` |
| Inverter | `inv-###` | `inv-001` |
| Battery | `bat-###` | `bat-001` |
| Meter | `meter-###` | `meter-001` |

---

## Publisher Topics

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

## Subscriber Topics

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

# Publish Policy

## Publish Intervals

| Device | Publish Interval |
|---------|------------------|
| Weather | 30 seconds |
| PV String | 10 seconds |
| Inverter | 5 seconds |
| Battery | 10 seconds |
| Smart Meter | 5 seconds |

---

## Initial Publish Delay

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

## Timestamp

Every published message shall include a timestamp.

| Property | Value |
|----------|-------|
| Format | ISO-8601 |
| Timezone | UTC |
| Example | `2026-07-15T10:35:20` |

---

## Publish Rules

- Each message shall contain the latest available telemetry.
- A device shall publish only to its assigned topic.
- The publisher shall continue publishing regardless of whether values have changed.

---

# Common Payload Schema

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

## Device Types

| Value |
|-------|
| WEATHER |
| PV_STRING |
| INVERTER |
| BATTERY |
| METER |

---

## Device Status

| Value | Description |
|-------|-------------|
| HEALTHY | Device operating normally |
| WARNING | Device operating with warnings |
| FAULT | Device has reported a fault |
| OFFLINE | Device communication lost |

---

## Common Payload Example

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

# Weather Payload Specification
## Payload Fields

| Field | Type | Unit | Required | Description |
|--------|------|------|----------|-------------|
| irradiance | Double | W/m² | Yes | Solar irradiance |
| ambientTemperature | Double | °C | Yes | Ambient air temperature |
| moduleTemperature | Double | °C | Yes | PV module temperature |
| humidity | Double | % | Yes | Relative humidity |
| windSpeed | Double | m/s | Yes | Wind speed |
| cloudCover | Double | % | Yes | Cloud cover percentage |

---

## Example Payload

```json
{
  "schemaVersion": "1.0.0",
  "timestamp": "2026-07-15T10:35:20",
  "siteId": "site-001",
  "deviceId": "weather-001",
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

# PV String Payload Specification

## Topic

```text
site/{siteId}/array/{stringId}
```

Example

```text
site/site-001/array/string-001
```

---

## Publish Interval

10 seconds

---

## Payload Fields

| Field | Type | Unit | Required | Description |
|--------|------|------|----------|-------------|
| dcVoltage | Double | V | Yes | String DC voltage |
| dcCurrent | Double | A | Yes | String DC current |
| dcPower | Double | kW | Yes | Instantaneous DC power |

---

## Example Payload

```json
{
  "schemaVersion": "1.0.0",
  "timestamp": "2026-07-15T10:35:20",
  "siteId": "site-001",
  "deviceId": "string-001",
  "deviceType": "PV_STRING",
  "status": "HEALTHY",

  "dcVoltage": 645.2,
  "dcCurrent": 11.8,
  "dcPower": 7.61
}
```

---
