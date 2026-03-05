# Apply Scripts

The decision engine does not control heating directly.

Instead, it calls two external scripts:

- heating_apply_comfort
- heating_apply_reduced

These scripts must perform the actual change on the heating system.

Example implementations are provided in `/scripts`.

They may optionally accept a parameter:

reason

which contains the explanation produced by the decision engine.
