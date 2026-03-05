# Heating Decision Engine for Home Assistant

A deterministic, single-authority heating decision engine for Home Assistant.

This project separates:

- decision logic
- execution layer
- trigger orchestration

to create a robust and predictable heating control system.

## Architecture

automation → decision engine → apply scripts

## Components

scripts/
  heating_decision_engine.yaml
  heating_apply_comfort_robust.yaml
  heating_apply_reduced_robust.yaml

examples/
  automation_trigger.yaml
  helpers.yaml

docs/
  architecture.md

## Installation

1. Copy scripts into your scripts folder
2. Create helpers (optional)
3. Install example automation
4. Adapt entities
