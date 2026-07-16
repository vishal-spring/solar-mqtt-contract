# MQTT Payload Examples

This directory contains example payloads for every MQTT topic defined in the MQTT Data Contract Specification.

The examples are provided for reference only and demonstrate the expected payload structure, field names and data types.

Actual telemetry values will vary depending on the operating conditions of the system.

## Available Examples

| File | Description |
|------|-------------|
| weather.json | Weather Station telemetry |
| pv-string.json | Individual PV String telemetry |
| pv-aggregate.json | Aggregate PV Array (`string-all`) telemetry |
| battery.json | Battery telemetry |
| inverter.json | Smart Hybrid Inverter telemetry |
| meter.json | Smart Meter telemetry |
| fault.json | Event-driven Fault payload |

## Notes

- All examples conform to the current contract version.
- Example timestamps and telemetry values are illustrative only.
- Additional whitespace or JSON formatting is implementation-specific.
- Every telemetry payload includes the common payload fields defined in the contract.
- Fault payloads are published only when a fault is raised or cleared.
