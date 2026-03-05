Heating Control Architecture
============================

Context signals
(presence, windows, house mode, etc.)
            │
            ▼
   Automation Trigger
            │
            ▼
   Heating Decision Engine
      (single authority)
            │
            ├───────────────┐
            ▼               ▼
   Apply Comfort       Apply Reduced
    (execution)         (execution)
            │               │
            ▼               ▼
       Heating system / API / Climate entity
