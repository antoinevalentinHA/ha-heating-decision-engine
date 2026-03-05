
# Heating Decision Engine

A generic **Home Assistant decision engine** that centralizes heating logic into a single script.

Instead of spreading heating logic across multiple automations, this engine evaluates context signals and decides between:

- `comfort`
- `reduced`
- `neutre` *(explicit abstention)*

The engine guarantees:

- **strict priority ordering**
- **zero redundant service calls**
- **fully traceable decisions**
- **portable architecture (no hard‑coded entities)**

---

# Architecture

```
presence / windows / vacation / airing / system signals
                        │
                        ▼
            heating_decision_engine
                        │
                        ▼
           apply_comfort / apply_reduced
                        │
                        ▼
                 heating system
```

All heating logic lives inside the **decision engine**.

Automations simply **trigger the engine when context signals change**.

---

# Features

- **Centralized decision logic**
- **Strict priority hierarchy**
- **Explicit abstention (`neutre`)**
- **Anti‑bounce protection**
- **No‑op guard (no redundant calls)**
- **Safe defaults for missing entities**
- **Optional observability helpers**
- **Logbook integration**

---

# Quick Start

1️⃣ Add the **decision engine script** to your Home Assistant configuration.

2️⃣ Create two action scripts:

- `script.heating_apply_comfort`
- `script.heating_apply_reduced`

3️⃣ Trigger the engine when relevant signals change.

Example:

```yaml
service: script.heating_decision_engine
data:
  action_comfort_script: script.heating_apply_comfort
  action_reduced_script: script.heating_apply_reduced

  current_program_entity: sensor.heating_program
  keyword_reduced: eco
  keyword_comfort: comfort

  presence_entity: binary_sensor.home_presence
  windows_open_entity: binary_sensor.any_window_open
```
---

## Example Package

For quick testing, an example Home Assistant package is provided.
packages/heating_decision_engine_example.yaml

This package creates a minimal working setup including:

- example helpers
- a thermal target sensor
- a trigger automation

It allows testing the decision engine with minimal configuration.

The decision engine and apply scripts must still be installed
from the repository `scripts/` directory.

---

# Output Modes

| Mode | Meaning |
|-----|--------|
| `comfort` | Request Comfort program |
| `reduced` | Request Reduced / Eco program |
| `neutre` | Explicit abstention (no action) |

The **`neutre` mode** allows the engine to declare:

> "Heating is already correct or no decision is needed."

This prevents unnecessary service calls.

---

# Decision Priority

Conditions are evaluated in strict order.

The **first matching rule wins**.

```
1. override_entity ON               → comfort   (operator_override)
2. system_allowed_entity OFF        → reduced   (system_not_allowed)
3. airing_active_entity ON          → reduced   (airing_active)
4. airing_block_entity ON           → reduced   (airing_block)
5. windows_open_entity ON           → reduced   (windows_open)
6. vacation_mode_entity == value    → reduced   (vacation_mode)
7. stove_block_entity ON            → reduced   (stove_active)
8. presence_entity ON               → thermal_target
9. absence_protection_entity ON     → comfort   (absence_protection)
10. fallback                        → reduced   (absence)
```

---

# Fields Reference

## Required

| Field | Type | Description |
|------|------|-------------|
| `action_comfort_script` | script | Script applying Comfort program |
| `action_reduced_script` | script | Script applying Reduced program |

These scripts may optionally accept:

```
reason
```

which contains the decision explanation.

---

## Program State

| Field | Type | Description |
|------|------|-------------|
| `current_program_entity` | sensor | Entity reflecting current heating mode |
| `keyword_reduced` | string | Keyword identifying Reduced program |
| `keyword_comfort` | string | Keyword identifying Comfort program |

---

## Input Signals

| Field | Type | Description |
|------|------|-------------|
| `override_entity` | input_boolean | Force Comfort |
| `system_allowed_entity` | binary_sensor | Heating allowed state |
| `airing_active_entity` | binary_sensor / input_boolean | Airing in progress |
| `airing_block_entity` | binary_sensor / input_boolean | Airing block active |
| `windows_open_entity` | binary_sensor | Window open detection |
| `vacation_mode_entity` | input_select | House mode entity |
| `vacation_mode_value` | string | Value indicating vacation |
| `stove_block_entity` | binary_sensor / input_boolean | Stove active |
| `presence_entity` | binary_sensor | Presence detection |
| `thermal_target_entity` | sensor | Target mode (`comfort`, `reduced`, `neutre`) |
| `absence_protection_entity` | binary_sensor / input_boolean | Absence protection |

All signals are **optional**.

Missing entities safely default to **false**.

---

# Anti‑Bounce Protection

To prevent rapid oscillations and API spam, the engine can optionally use a timer gate.

```
decision applied
      │
      ▼
anti‑bounce timer starts
      │
      ▼
new decisions blocked until timer returns to idle
```

Override mode bypasses the timer.

---

# Observability

Optional helper entities can record engine output.

Example:

```
input_text.heating_desired_mode
input_text.heating_decision_reason
```

Benefits:

- easier debugging
- dashboard diagnostics
- traceable decisions

---

# Guards & Safety

| Guard | Behavior |
|------|---------|
| Anti‑bounce | Blocks decisions while timer active |
| Unknown program | Stops execution if state cannot be normalized |
| No‑op guard | Prevents redundant service calls |
| Neutre abstention | Clean stop with no action |

Publish helpers still update even when execution stops.

---

# Design Principles

This project follows four architectural rules.

### 1️⃣ Decision ≠ Action

The engine decides.

External scripts apply the change.

### 2️⃣ Single Source of Truth

All heating decisions originate from **one place**.

### 3️⃣ Explicit Abstention

`neutre` means:

> The engine evaluated the context and intentionally did nothing.

### 4️⃣ Deterministic Priority

The first matching rule always wins.

No competing automations.

---

# Compatibility

- Home Assistant **2024.8+**
- Pure YAML
- No custom integrations required

---

# License

MIT License
