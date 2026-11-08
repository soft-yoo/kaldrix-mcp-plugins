## Read installation diagnoses

Agent guidance (`reason`, `notes`, warning `message`) is English. Room, point, riser and stored circuit names
remain user data. The `language` option of circuit suggestion tools controls proposed names only; use the same
value when applying those suggestions.

- `suggest_installation_circuits`: inspect `reasonDetail` for the backend reason code, member count, source kind,
  point limit, phase requirement, airflow, flexible runs, manifold/chamber flags and RCD reasons. Do not infer them
  by parsing `reason`. `missingSources` provides the required source kind and missing medium.
- `apply_installation_circuit_suggestions`: `missingSources` and matching English `notes` come from the same
  planner result that was saved, including when other trades still lack sources.
- `suggest_ventilation`: `issues` identifies rooms requiring manual placement. Warning room/point IDs,
  `pointKind`, `roomClass`, `recommendedAirflow`, supply/extract totals, capacity and measured distance are canonical
  backend values. Use these fields to select the next operation, not the wording of `message`.
- `list_installation_risers`: warnings keep the affected floor and point IDs. Unknown future warning/issue codes
  receive generic English guidance while retaining those references; inspect the code before changing the model.
