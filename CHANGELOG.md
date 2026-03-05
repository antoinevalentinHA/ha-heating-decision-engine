# Changelog

All notable changes to this project will be documented in this file.

The format follows a simplified version of **Keep a Changelog**  
and uses **Semantic Versioning**.

---

## [1.0.0] — 2026-03-05

Initial public release of the **Heating Decision Engine**.

### Added

- `heating_decision_engine` central decision script
- `heating_apply_comfort` and `heating_apply_reduced` execution scripts
- example trigger automation
- `heating_thermal_target` intent sensor
- optional helpers for observability and anti-bounce
- architecture documentation
- detailed component documentation

### Features

- deterministic priority-based heating decisions
- explicit abstention mode (`neutre`)
- anti-bounce protection
- no-op guard to prevent redundant service calls
- portable architecture independent of heating hardware

### Documentation

- project README
- decision engine documentation
- apply scripts documentation
- thermal target sensor documentation
- architecture overview
