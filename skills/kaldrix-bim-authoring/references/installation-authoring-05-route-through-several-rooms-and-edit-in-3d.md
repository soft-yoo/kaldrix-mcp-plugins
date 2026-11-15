## Route through several rooms and edit in 3D

Circuits use the shared project coordinate system: plan `planX/planY` = 3D `x/z`; 3D `y` = height above the floor in metres. Routes may go below finished floor level within backend bounds; point mounting heights have stricter bounds. Read stored positions after snapping; requested positions may change.

Automatic route: choose CHAIN, STAR or TREE; guide a branch through rooms with `routeOverrides:[{targetPointId,waypoints:[{x,y}]}]` (waypoints3D instead when automaticRoute3D is true; never send both). Once a manual network exists, edit its edges and junctions; changing topology does not replace it.

Real junction: read the physical layout, send nodeEdit.ADD with segmentIndex and a position on that segment (or edgeId for an existing network), then read the new node/edge IDs. Preview with `preview_installation_circuit_node`; save exactly the same nodeEdit with `update_installation_circuit` at the reviewed version.

```json
{"projectId":"<project>","floorId":"<floor>","circuitId":"<circuit>","expectedModelVersion":7,"nodeEdit":{"action":"EDIT_EDGE","edgeId":"<edge>","controls":[{"x":2.0,"y":1.2,"z":1.0},{"x":2.0,"y":1.2,"z":2.0}]}}
```

- MOVE (nodeId/position) moves a shared junction and every incident endpoint.
- CONNECT (nodeId/targetPointId) attaches an endpoint; add its ID to addPointIds in the same update if not yet a member.
- `add_installation_points` with circuitId/nodeId creates a compatible endpoint and connects it atomically.
- EDIT_EDGE replaces the full intermediate bend list; linked fixtures do not move.
- REMOVE requires canRemove=true; reconnect a branching junction first.
- DRAIN uses the same nodeEdit operations; backend fits free heights and keeps fixture heights. Legacy waypoints3D are rejected. An impossible slope rejects the edit atomically.
- `update_installation_point` moves one fixture; routes follow. Riser-owned points move via `update_installation_riser`.
- `update_installation_circuit` with translation `{x:-0.30,y:0.10,z:0}` moves all member points and the manual route, source fixed. Exclusive intent: no other edits in that call. Shared member points or invalid placement reject the whole move.
- clearRouteOverrides=true restores automatic routing and discards manual controls/network; inspect the result.
- Flexible ventilation `FLEX_RUN_TOO_LONG` measures the full physical path source → each endpoint across all
  intermediate edges (splitting into short edges does not help), excluding reserve and parallel runs. `routeLength`
  sums segment length × `runs`; `totalLength` adds reserve and the paired heating multiplier. Fittings and wall passes already include parallel runs.

Parallel displaySpans from different circuits are shared physical corridors, not connections; a junction joins only edges of its own circuit. Layout returns explicit network runs; nodeEdit currently edits geometry/connectivity and does not expose an arbitrary runs setter. Flexible ventilation run counts come from airflow rules and captured routes.
