## Route through several rooms and edit in 3D

Circuit plan coordinates use the shared project coordinate system. Plan `planX/planY` maps to 3D `x/z`; 3D `y` is height above the floor in metres. Routes may extend below finished floor level within backend bounds. Point mounting heights have their own stricter bounds. Read stored positions after snapping; do not assume requested positions survived unchanged.

Choose CHAIN, STAR or TREE for an automatic route. Set `routeOverrides:[{targetPointId,waypoints:[{x,y}]}]` to guide an automatic branch through chosen rooms. If automaticRoute3D is true, use waypoints3D instead of waypoints. Never send both. Once a manual network exists, edit its edges and junctions; changing topology does not replace that network.

To add a real junction, read the physical layout and send nodeEdit.ADD with segmentIndex and a position on that segment (or edgeId for an existing network). Read the resulting node/edge IDs. For a preview use `preview_installation_circuit_node`; save exactly the same nodeEdit with `update_installation_circuit` at the reviewed version.

```json
{"projectId":"<project>","floorId":"<floor>","circuitId":"<circuit>","expectedModelVersion":7,"nodeEdit":{"action":"EDIT_EDGE","edgeId":"<edge>","controls":[{"x":2.0,"y":1.2,"z":1.0},{"x":2.0,"y":1.2,"z":2.0}]}}
```

- MOVE with nodeId/position moves a shared junction and every incident endpoint.
- CONNECT with nodeId/targetPointId attaches an endpoint to that junction. Add its ID in addPointIds in the same update when it is not already a member.
- `add_installation_points` with circuitId/nodeId creates a new compatible endpoint and connects it atomically.
- EDIT_EDGE replaces the full intermediate bend list; it does not move linked fixtures.
- REMOVE requires canRemove=true. A branching junction must be reconnected before removal.
- DRAIN uses the same nodeEdit operations, with backend fitting of free heights and preservation of fixture heights. Legacy waypoints3D are rejected; use nodeEdit. An impossible slope rejects the edit atomically.
- `update_installation_point` moves one fixture; routes follow it. Riser-owned points must move via `update_installation_riser`.
- `update_installation_circuit` with translation `{x:-0.30,y:0.10,z:0}` moves all member points and manual route together, keeping the source fixed. This intent is exclusive: do not combine it with other edits. Shared member points or invalid placement reject the whole move.
- clearRouteOverrides=true restores automatic routing, discarding manual controls/network; inspect the resulting layout.
- For flexible ventilation, `FLEX_RUN_TOO_LONG` measures the full physical path from the source to each endpoint,
  including every intermediate network edge. Splitting a long duct into short edges does not shorten it. The limit
  excludes reserve and parallel runs. `routeLength`, in contrast, sums segment length × `runs`; `totalLength`
  then includes reserve and the paired heating multiplier. Fittings and wall passes already include parallel runs.

Parallel displaySpans from different circuits describe shared physical corridors, not connections. A junction joins only edges of its own circuit. Explicit network runs are returned by layout; nodeEdit currently edits geometry/connectivity and does not expose an arbitrary runs setter. Flexible ventilation run counts come from airflow rules and captured routes.
