# Thermal Target Sensor

The **Thermal Target Sensor** expresses the *thermal intent* of the house as a ternary value — whether heating should run, stop, or remain unchanged.

| Value | Meaning |
|-------|---------|
| `comfort` | Heating is required |
| `reduced` | Heating should stop / remain reduced |
| `neutre` | No change required — abstain |
| `unknown` | Critical input missing — no decision |

The sensor does not control heating. It produces an intent signal consumed by the `heating_decision_engine`.

---

## Role in the Architecture

```
temperature signals
        │
        ▼
thermal_target_sensor       ← thermal intent (this sensor)
        │
        ▼
heating_decision_engine     ← context + priority logic
        │
        ▼
apply_comfort / apply_reduced scripts
        │
        ▼
heating system
```

This separation ensures deterministic decision logic, no hidden device calls, and reusable heating strategies across setups.

---

## Decision Model

The sensor evaluates three elements:

1. **Outdoor temperature** — compared to configurable thresholds
2. **Coldest indoor temperature** — compared to setpoint + hysteresis
3. **User setpoint and offsets** — define the comfort zone

The result is a **thermal authorization**, not a command.

| Condition | Output |
|-----------|--------|
| Outdoor temp ≥ `heating_outdoor_off` | `reduced` |
| Indoor temp < setpoint − offset_on | `comfort` |
| Indoor temp ≥ setpoint + offset_off | `reduced` |
| Indoor temp in hysteresis zone | `neutre` |
| Any critical sensor unavailable | `unknown` |

The `neutre` zone prevents unnecessary mode changes when temperature is stable near the setpoint.

---

## Required Entities

| Entity | Description |
|--------|-------------|
| `sensor.outdoor_temperature` | Outdoor temperature |
| `sensor.coldest_room_temperature` | Coldest indoor reference temperature |
| `input_number.heating_setpoint` | Desired comfort temperature |
| `input_number.heating_offset_on` | Lower hysteresis offset (below setpoint → comfort) |
| `input_number.heating_offset_off` | Upper hysteresis offset (above setpoint → reduced) |
| `input_number.heating_outdoor_off` | Outdoor threshold above which heating is disabled |

If any required entity is `unknown` or `unavailable`, the sensor returns `unknown`. The decision engine will abstain until inputs recover.

---

## Optional — Weather Anticipation

If future conditions indicate heating will soon become unnecessary, the sensor can neutralize a comfort request before it triggers an apply cycle.

| Entity | Description |
|--------|-------------|
| `input_boolean.heating_weather_anticipation` | Enable weather anticipation |
| `binary_sensor.weather_favorable_heating` | `on` = favorable weather incoming |

Rule:

```
comfort + anticipation_on + weather_favorable  →  neutre
```

This prevents heating from starting shortly before a natural temperature rise. If either entity is absent or `off`, the rule has no effect.

---

## Design Principles

**Pure decision layer** — the sensor performs no device control, calls no services, and produces no side effects. It only computes a value.

**Explicit abstention** — `neutre` means conditions are acceptable and no change is needed. This is intentional: the decision engine treats `neutre` as "authorize without acting", not as an error.

**Hardware independence** — the sensor does not read heating system state and does not depend on any cloud integration or device-specific entity. The same sensor works with `climate.*`, `select.*`, or any other actuator.

---

## Compatibility

Home Assistant **2024.8+** — no custom integrations required.
