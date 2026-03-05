
# Heating Decision Engine

Generic **Home Assistant script** implementing a centralized heating decision engine.

The engine evaluates multiple context signals (presence, windows, airing, vacation mode, etc.) and decides whether heating should run in **Comfort**, **Reduced**, or **Abstain** mode.

The goal is to move heating logic out of scattered automations and into a **single authoritative decision engine**.

---

## Concept

Most Home Assistant installations control heating with multiple independent automations:

```
if window open -> reduce heating
if presence -> comfort
if vacation -> eco
```

Over time this creates overlapping logic, race conditions, and unnecessary service calls.

This project implements a different pattern:

```
context signals
      │
      ▼
heating_decision_engine
      │
      ▼
desired heating mode
      │
      ▼
action scripts
```

The decision engine:

- centralizes all heating logic
- enforces strict priority rules
- prevents unnecessary state changes
- provides explicit reasoning for every decision

---

## Features

- **Centralized decision logic**
- **Strict priority hierarchy**
- **Abstention mode (`neutre`)**
- **Anti-bounce protection**
- **Safe defaults for missing entities**
- **Optional observability helpers**
- **Zero unnecessary service calls**

---

## Decision Output

The engine produces one of three results:

| Mode     | Meaning                                   |
|----------|-------------------------------------------|
| comfort  | Heating should run in Comfort mode        |
| reduced  | Heating should run in Reduced / Eco mode  |
| neutre   | No action required (explicit abstention)  |

The `neutre` mode allows the engine to **explicitly abstain** when heating is already in the correct state or when no decision is required.

---

## Priority Model

The engine evaluates contexts using a strict hierarchy.  
Higher levels always override lower ones.

```
1. Operator override
2. System restrictions
3. Major contexts
4. Opportunity contexts
```

Example evaluation flow:

```
override active        -> comfort
system not allowed     -> reduced

airing active          -> reduced
windows open           -> reduced
vacation mode          -> reduced

presence detected      -> follow thermal target
absence protection     -> comfort

default                -> reduced
```

---

## Anti‑Bounce Protection

To prevent rapid oscillations and unnecessary API calls, the engine can optionally use a timer gate.

```
decision applied
      │
      ▼
anti-bounce timer starts
      │
      ▼
no new decisions until timer returns to idle
```

This protects both the heating system and Home Assistant automations from rapid state changes.

---

## Observability

The engine can optionally publish:

- the **desired heating mode**
- the **decision reason**

Example helper entities:

```
input_text.heating_desired_mode
input_text.heating_decision_reason
```

This makes debugging and dashboard diagnostics significantly easier.

---

## Action Scripts

The decision engine **does not control heating directly**.

Instead it calls two external scripts:

```
action_comfort_script
action_reduced_script
```

These scripts should apply the actual heating change.

Possible implementations include:

- `climate.set_preset_mode`
- `select.select_option`
- vendor integrations
- custom scripts

Recommended interface for these scripts:

```
fields:
  reason:
    description: Decision reason
```

Passing the `reason` field allows the action layer to remain fully observable.

---

## Example Call

```yaml
service: script.heating_decision_engine
data:
  override_entity: input_boolean.heating_override
  system_allowed_entity: binary_sensor.heating_allowed
  windows_open_entity: binary_sensor.any_window_open
  presence_entity: binary_sensor.presence_home

  current_program_entity: sensor.heating_program
  keyword_reduced: eco
  keyword_comfort: comfort

  antibounce_timer_entity: timer.heating_antibounce

  action_comfort_script: script.heating_apply_comfort
  action_reduced_script: script.heating_apply_reduced

  publish_mode_entity: input_text.heating_desired_mode
  publish_reason_entity: input_text.heating_decision_reason

  logbook_entity: sensor.heating_program
```

---

## Recommended Architecture

```
context sensors
      │
      ▼
heating_decision_engine
      │
      ▼
action scripts
      │
      ▼
heating system
```

All heating logic stays inside the **decision engine**.

Automations simply trigger the engine when relevant context changes.

---

## Why This Pattern?

This approach provides several advantages:

- eliminates conflicting automations
- improves traceability
- reduces unnecessary service calls
- simplifies maintenance
- makes heating logic easier to reason about

Instead of dozens of independent automations, the system has **one place where decisions are made**.

---

## Compatibility

Tested with:

```
Home Assistant 2024.8+
```

No custom integrations required.

---

## License

MIT License
