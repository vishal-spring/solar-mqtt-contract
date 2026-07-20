# Versioning

This repository follows Semantic Versioning (SemVer).

Version format:

```
MAJOR.MINOR.PATCH
```

## Version Components

| Component | Description |
|-----------|-------------|
| MAJOR | Breaking changes that are not backward compatible. |
| MINOR | Backward-compatible additions, enhancements or new functionality. |
| PATCH | Backward-compatible fixes, corrections, documentation updates or clarifications. |

## Compatibility Policy

- Publisher and Subscriber shall implement the same **MAJOR** version.
- **MINOR** version differences are supported only when backward compatibility is maintained.
- **PATCH** version differences are considered fully compatible.

## Payload Version

Every published message shall include:

```json
"schemaVersion": "<contract-version>"
```

The `schemaVersion` identifies the version of the contract implemented by the Publisher.

## Current Version

| Version | Status | Date |
|----------|--------|------|
| 1.0.0 | Draft | 20-Jul-2026 |
