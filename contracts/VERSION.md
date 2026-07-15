# Version Information

| Property | Value |
|----------|-------|
| Contract Name | Solar MQTT Communication Contract |
| Current Version | **v1.0.0** |
| Status | Initial Release |
| Release Date | 2026-07-15 |
| MQTT Version | 5.0 |
| Payload Format | JSON |
| Encoding | UTF-8 |
| QoS | 1 |
| Retained Messages | Disabled |
| Clean Start | false |

---

## Versioning Policy

This contract follows Semantic Versioning (SemVer).

### Major Version (X.0.0)

Increment when:

- MQTT topic hierarchy changes.
- Existing payload fields are removed or renamed.
- Field data types change.
- Breaking compatibility is introduced.

Examples:

- Renaming `deviceId` to `id`
- Changing topic structure
- Removing mandatory fields

---

### Minor Version (1.X.0)

Increment when:

- New optional fields are added.
- New device types are introduced.
- New MQTT topics are added.
- New status or operating mode values are introduced.

Examples:

- Adding EV Charger support
- Adding Alarm topics
- Adding optional telemetry fields

---

### Patch Version (1.0.X)

Increment when:

- Documentation corrections are made.
- Typographical errors are fixed.
- Clarifications are added without changing implementation.
- Example payloads are updated.

Examples:

- Correcting field descriptions
- Updating examples
- Improving wording

---

## Compatibility

| Version | Compatibility |
|----------|---------------|
| Patch | Fully Compatible |
| Minor | Backward Compatible |
| Major | Breaking Changes Possible |

---

## Current Release

**v1.0.0**

This release defines the initial communication contract between the Solar Simulator and EMS Receiver covering:

- MQTT communication
- Topic hierarchy
- Payload specifications
- Device telemetry
- Status definitions
- Message timing
- Communication configuration
- Validation guidelines
