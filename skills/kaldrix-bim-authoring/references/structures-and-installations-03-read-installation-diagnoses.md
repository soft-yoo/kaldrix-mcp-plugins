## Read installation diagnoses

Agent guidance (`reason`, `notes`, warning `message`) is English; room, point, riser and stored circuit names stay user
data. The `language` option of circuit suggestion tools sets proposed names only; use the same value when applying.

- `suggest_installation_circuits`: `reasonDetail` holds the backend reason code, member count, source kind, point
  limit, phase requirement, airflow, flexible runs, manifold/chamber flags and RCD reasons — never parse `reason`.
  `missingSources` gives the required source kind and missing medium.
- `apply_installation_circuit_suggestions`: `missingSources` and matching English `notes` come from the saved planner
  result, also when other trades still lack sources.
- `suggest_ventilation`: `issues` lists rooms needing manual placement. Warning room/point IDs, `pointKind`,
  `roomClass`, `recommendedAirflow`, supply/extract totals, capacity and measured distance are canonical; choose the
  next operation from these fields, not from `message`.
- `list_installation_risers`: warnings keep affected floor and point IDs. Unknown future warning/issue codes get
  generic English guidance with those references; inspect the code before changing the model.
