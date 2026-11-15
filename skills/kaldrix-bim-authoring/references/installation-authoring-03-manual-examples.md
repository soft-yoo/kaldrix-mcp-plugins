## Manual examples

Individual gas endpoint (`add_installation_points`):

```json
{"projectId":"<project>","floorId":"<floor>","points":[{"kind":"GAS_POINT","planX":2.0,"planY":1.0,"height":0.5,"label":"Cooker"}],"idempotencyKey":"qa-installation-authoring-1","expectedModelVersion":7}
```

Heating source to a manifold (`create_installation_circuit`; obtain both point IDs first):

```json
{"projectId":"<project>","floorId":"<floor>","medium":"HEATING_SUPPLY_RETURN","sourcePointId":"<heat-source>","pointIds":["<ufh-manifold>"],"topology":"TREE","expectedModelVersion":7,"idempotencyKey":"qa-installation-authoring-2"}
```

Separate cold/hot runs to one fixture = two `create_installation_circuit` calls, each with the latest returned version:

```json
{"projectId":"<project>","floorId":"<floor>","medium":"COLD_WATER","sourcePointId":"<water-manifold>","pointIds":["<water-sink>"],"topology":"STAR","expectedModelVersion":7,"idempotencyKey":"qa-installation-authoring-3"}
```

Repeat with HOT_WATER and the new version; the two calls are not one atomic batch. For a whole floor prefer reviewed circuit suggestions.

Drain branch (`create_installation_circuit`):

```json
{"projectId":"<project>","floorId":"<floor>","medium":"DRAIN","sourcePointId":"<stack>","pointIds":["<wc-drain>"],"spec":"PVC_U_110","slope":0.02,"expectedModelVersion":7,"idempotencyKey":"qa-installation-authoring-4"}
```

Ventilation: `suggest_ventilation` → review → `apply_ventilation_suggestions` at its version; create MVHR_UNIT, distribution boxes, AIR_INTAKE and AIR_EXHAUST individually; then suggest/apply circuits with system VENTILATION. `get_ventilation_route_report` gives actual room flows and flexible duct runs (ducts are also empty for connected rigid Spiro routes; take their lengths from list_installation_circuits and get_installation_circuit_layout). Run `suggest_ventilation` again for whole-project balance; a room flow report alone does not establish it.
