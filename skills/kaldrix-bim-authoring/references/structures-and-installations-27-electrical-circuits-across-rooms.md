
## Electrical circuits across rooms

For requests like “two cables per room”, “continue the sockets into the next room”, “one corridor route with
branches” or “place the junction here”. A circuit belongs to one floor; its targets may span rooms. Across floors use
`create_installation_riser` and the FEEDER workflow above. Plain installation routes in `get_floor_detail` do not
contain circuit routes.

### Choose connections before drawing the route

- Socket supply (`POWER`) and lighting (`LIGHTING`) are separate circuits with own IDs, cable specs and protective
  devices. Two circuits may follow one corridor without being electrically joined; sharing `sourcePointId` or
  `rcdGroup` does not merge them.
- One room is not necessarily two cables: dedicated appliances and LAN/TV may need more circuits. Suggestions group
  socket points by room but may group lighting across rooms; inspect the proposal, never promise a fixed count per room.
- `CHAIN` = cable path continuing receiver to receiver, **not** loads wired in series. The backend picks the next
  point by routed distance; `pointIds` order only breaks ties, so never promise a room order from the array.
- `STAR` = separate source-to-point run per receiver (intended home runs); overlapping runs stay separate cables with
  their own quantities.
- `TREE` = one circuit with a shared route and branches; use explicit network junctions when the installer specifies
  where to branch. It cannot join two circuit IDs.
- The model holds circuit membership and physical cable geometry, not terminal numbers, cable cores or switch-to-lamp
  wiring logic; never report a terminal-level wiring diagram just because a switch and lamp are in LIGHTING.

### Start with existing points and circuits

1. Read `get_project_detail`, `get_floor_detail`, `list_installation_points`, `list_installation_circuits`,
   `get_installation_conduit_catalog`. Resolve real IDs and room membership; keep user-entered names; inspect existing
   same-medium memberships before writes.
2. Draft: `get_installation_circuit_selection` with current `expectedModelVersion`, `sourcePointId`, `medium` (+
   `editingCircuitId` for edits). `typicalPointIds` are suggestions — intersect with requested room IDs;
   `eligiblePointIds` cover explicit user choices. Request POWER and LIGHTING separately; never copy all ELECTRICAL
   points into both.
3. Whole-house start: `suggest_installation_circuits` `system: "ELECTRICAL"`, inspect routes and missing sources, then
   `apply_installation_circuit_suggestions` with that exact preview version, system and language. Hand-picked room
   groups: `create_installation_circuit` with the selection's version.
4. After every mutation use its returned `modelVersion`. On `MODEL_VERSION_CONFLICT` reread selections and routes;
   never silently substitute a newer version for the reviewed one. Never invent point, circuit, node or edge UUIDs from labels.

Templates (replace angle-bracket values with tool-returned IDs) — sockets through a bedroom and study, then a separate
lighting route through those rooms; send each to `create_installation_circuit`, rereading the LIGHTING selection at the
returned version before the second:

```json
{"projectId":"<projectId>","floorId":"<floorId>","expectedModelVersion":42,"name":"Gniazda — sypialnia i gabinet","medium":"POWER","sourcePointId":"<boardPointId>","pointIds":["<bedroomSocketId>","<studySocketId>"],"topology":"CHAIN","idempotencyKey":"qa-structures-and-installations-1"}
```

```json
{"projectId":"<projectId>","floorId":"<floorId>","expectedModelVersion":43,"name":"Oświetlenie — sypialnia i gabinet","medium":"LIGHTING","sourcePointId":"<boardPointId>","pointIds":["<bedroomLightId>","<bedroomSwitchId>","<studyLightId>","<studySwitchId>"],"topology":"TREE","idempotencyKey":"qa-structures-and-installations-2"}
```

Versions 42/43 are illustrative; use real returned versions. Omitted spec/protection uses backend defaults; explicit
choices must come from the catalogue. A point already in another same-medium circuit cannot be added directly. To
regroup, detach just those IDs with `update_installation_circuit(removePointIds: [...])`, reread selection and attach
them to the chosen circuit; outlets stay, inspect the old circuit for empty membership.

### Route through the corridor or a chosen junction

Automatic routes follow wall faces and existing doors/cutouts. A room is not a waypoint: use real plan coordinates
from geometry for a corridor path. `routeOverrides` constrains one branch by `targetPointId`; it defines no junction
and does not force CHAIN order — read `fromPointId`/`toPointId` for the branch's actual start. Plan `waypoints` use
`{x,y}`; `waypoints3D` use `{x,y,z}` with z = plan Y and y = floor-local height (never project elevation). The backend
adds orthogonal legs and validates crossings; inspect `ROUTE_CROSSES_WALL` rather than claiming a crossing creates an opening.

Control branching without converting to plain lines:

1. Read `get_installation_circuit_layout`. Automatic route: choose the zero-based `segments` index and a position on
   that physical segment; existing manual network: its `edgeId`. Never pick a shifted `displaySpans` lane.
2. Preview `nodeEdit: {action: "ADD", segmentIndex: <index>, position: {x: ..., y: ..., z: ...}}` with
   `preview_installation_circuit_node` at the current version. Position within 15 cm of the path and at least 1 cm from
   its endpoints (backend projects it onto the path). Save the same intent/version with `update_installation_circuit`;
   take the junction `nodeId` and edge IDs from the saved result's `circuits[].network`.
3. Branch an existing member from the junction: preview and save
   `nodeEdit: {action: "CONNECT", nodeId: "<junctionId>", targetPointId: "<memberPointId>"}`. This changes that
   endpoint's incoming edge, not a second supply. CONNECT selects an unlinked junction node, not a socket or source node.
4. A target outside the circuit must be eligible: attach with `addPointIds`, reread layout/version, then preview
   CONNECT (node preview accepts only nodeEdit, so it cannot preview an unattached target with addPointIds; saving
   both in one update works when no preview is needed). Or `add_installation_points` with `circuitId` and `nodeId`
   places a new endpoint and connects its branch atomically.
5. `MOVE` the shared junction, or `EDIT_EDGE` with the complete 3D `controls` list to shape one edge through rooms.
   Empty controls make the edge straight (no automatic wall routing). Preview first and inspect warnings. Editing only
   `topology` does not replace a stored manual network; `clearRouteOverrides: true` deliberately clears it and branch
   waypoints to restore automatic routing.

If POWER and LIGHTING should both follow the corridor, edit each route separately. `displaySpans.circuitIds` and
`cableCount` are visual grouping, not connections: never merge memberships, multiply runs to represent cores, or halve
quantities because displayed routes overlap. Backend physical segments and takeoff are the quantity source.

### Check and report what was actually connected

Read `get_installation_circuit_layout` per changed circuit, `get_distribution_board` and `validate_floor_geometry`.
Report served rooms, source, medium, topology/manual junctions, actual cable length incl. reserve, unconnected
requested points and remaining warning codes. Confirm connectivity from segment point IDs or network edges, never from
lines touching on screen. Review a manual route adjusted after walls/points moved before accepting it. Estimate rows
use current `CIRCUIT_*` quantity sources; frozen estimate quantities need an explicit update through the estimate
workflow. Never silently rewrite prices or totals.
