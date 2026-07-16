# Solar MQTT Contract

A common MQTT communication contract for the **Commercial Solar Rooftop Hybrid System**, defining the communication interface between the **Solar Hybrid System Simulator (Publisher)** and the **Energy Management System (Subscriber)**.

This repository serves as the single source of truth for MQTT topics, payload structures, publishing behavior, device specifications, and interoperability requirements shared between both systems.

---

## Contribution

As this repository defines a shared communication contract, any modification impacts both the Publisher and Subscriber.

Before merging any change:

- Review and obtain approval from both teams.
- Preserve backward compatibility whenever possible.
- Update the contract documentation accordingly.
- Record the change in `CHANGELOG.md`.
- Update the contract version when applicable.

---

## License

Licensed under the MIT License.
