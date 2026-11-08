## Manual examples

Individual gas endpoint (`add_installation_points`):

```json
{
  "projectId": "<project>",
  "floorId": "<floor>",
  "points": [
    {
      "kind": "GAS_POINT",
      "planX": 2.0,
      "planY": 1.0,
      "height": 0.5,
      "label": "Cooker"
    }
  ],
  "idempotencyKey": "qa-installation-authoring-1",
  "expectedModelVersion": 7
}
```

Heating source to a manifold (`create_installation_circuit`; obtain both point IDs first):

```json
{
  "projectId": "<project>",
  "floorId": "<floor>",
  "medium": "HEATING_SUPPLY_RETURN",
  "sourcePointId": "<heat-source>",
  "pointIds": [
    "<ufh-manifold>"
  ],
  "topology": "TREE",
  "expectedModelVersion": 7,
  "idempotencyKey": "qa-installation-authoring-2"
}
```

Create separate cold/hot runs to the same fixture with two `create_installation_circuit` calls, each using the latest returned version:

```json
{
  "projectId": "<project>",
  "floorId": "<floor>",
  "medium": "COLD_WATER",
  "sourcePointId": "<water-manifold>",
  "pointIds": [
    "<water-sink>"
  ],
  "topology": "STAR",
  "expectedModelVersion": 7,
  "idempotencyKey": "qa-installation-authoring-3"
}
```

Repeat with HOT_WATER and the new version. Do not use the two-call approach as if it were one atomic batch. For the whole floor, prefer reviewed circuit suggestions.

Drain branch (`create_installation_circuit`):

```json
{
  "projectId": "<project>",
  "floorId": "<floor>",
  "medium": "DRAIN",
  "sourcePointId": "<stack>",
  "pointIds": [
    "<wc-drain>"
  ],
  "spec": "PVC_U_110",
  "slope": 0.02,
  "expectedModelVersion": 7,
  "idempotencyKey": "qa-installation-authoring-4"
}
```

Ventilation: `suggest_ventilation` → review → `apply_ventilation_suggestions` at its version; create MVHR_UNIT, distribution boxes, AIR_INTAKE and AIR_EXHAUST individually; then suggest/apply circuits with system VENTILATION. Read `get_ventilation_route_report` for actual room flows and flexible duct runs (empty ducts also occurs with connected rigid Spiro routes; use list_installation_circuits and get_installation_circuit_layout for their lengths) and `suggest_ventilation` again for whole-project balance. A room flow report alone does not establish project balance.
