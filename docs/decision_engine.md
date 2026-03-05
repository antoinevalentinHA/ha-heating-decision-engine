# Heating Decision Engine

This document describes the internal behavior of the
`heating_decision_engine` script.

## Role

The script evaluates context signals and determines the desired heating mode.

Possible outputs:

- comfort
- reduced
- neutre

The script never controls hardware directly.
It only delegates execution to external scripts.

## Execution flow

1. Guard checks
2. Context evaluation
3. Mode resolution
4. No-op guard
5. Apply script execution
6. Observability publish

## Decision priority

Rules are evaluated in strict order:

1. override_entity
2. system_allowed_entity
3. airing_active_entity
4. airing_block_entity
5. windows_open_entity
6. vacation_mode
7. stove_block
8. presence → thermal_target
9. absence_protection
10. fallback

First match wins.

## Abstention mode

The `neutre` value indicates that the engine evaluated the situation
but intentionally decided not to change the heating program.

This prevents unnecessary service calls.
