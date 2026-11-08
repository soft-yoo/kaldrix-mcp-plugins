
## Electrical circuits across rooms

Use this playbook for requests such as “two cables per room”, “continue the sockets into the next room”,
“one corridor route with branches”, or “place the junction here”. A circuit belongs to one floor;
its targets may belong to several rooms. Across floors use `create_installation_riser` and the
FEEDER workflow above. Plain installation routes in `get_floor_detail` do not contain circuit routes.

### Choose connections before drawing the route

- Keep socket supply (`POWER`) and lighting (`LIGHTING`) as separate circuits with their own IDs,
  cable specifications and protective devices. Two circuits may follow the same corridor without
  being electrically joined. Sharing `sourcePointId` or `rcdGroup` does not merge circuits.
- One room does not necessarily mean exactly two cables: dedicated appliances and LAN/TV may need
  additional circuits. Automatic suggestions group socket points by room but may group lighting
  across rooms. Inspect the actual proposal instead of promising a fixed cable count per room.
- `CHAIN` models a cable path continuing from one receiver to the next. It does **not** model loads
  electrically wired in series. The backend chooses the next point by routed distance; the order
  of `pointIds` only resolves ties. Do not promise a requested room order from the array order.
- `STAR` gives a separate source-to-point run for every receiver. Use it when separate home runs
  are intended; overlapping runs remain separate cables and retain their material quantities.
- `TREE` gives one circuit a shared route with branches. Use explicit network junctions when the
  installer specifies where branching must happen. It cannot join two different circuit IDs.
- The model describes circuit membership and physical cable geometry. It does not describe
  terminal numbers, individual cable cores or switch-to-lamp electrical wiring logic. Do not
  report a terminal-level wiring diagram merely because a switch and lamp belong to LIGHTING.

### Start with existing points and circuits

1. Read `get_project_detail`, `get_floor_detail`, `list_installation_points`,
   `list_installation_circuits` and `get_installation_conduit_catalog`. Resolve real IDs and room
   membership; preserve user-entered names. Inspect existing same-medium memberships before writes.
2. For a draft call `get_installation_circuit_selection` with the current `expectedModelVersion`,
   `sourcePointId` and `medium`. For edits also set `editingCircuitId`. Use `typicalPointIds` as
   suggestions, intersect them with the requested room IDs, and use `eligiblePointIds` for explicit
   user choices. Request POWER and LIGHTING separately; do not copy all ELECTRICAL points into both.
3. For a whole-house starting proposal, call `suggest_installation_circuits` with
   `system: "ELECTRICAL"`. Inspect routes and missing sources, then
   `apply_installation_circuit_suggestions` with that exact preview version, system and language.
   For hand-picked room groups use `create_installation_circuit` with the selection's version.
4. After every mutation use its returned `modelVersion` for the next intent. On
   `MODEL_VERSION_CONFLICT`, reread selections and routes; do not silently substitute a newer version
   for the one previously reviewed. Never invent point, circuit, node or edge UUIDs from labels.

The following calls are templates: replace angle-bracket values with IDs returned by the tools.
For sockets continuing through a bedroom and study, and a separate lighting route through those rooms:

```json
{
  "projectId": "<projectId>",
  "floorId": "<floorId>",
  "expectedModelVersion": 42,
  "name": "Gniazda — sypialnia i gabinet",
  "medium": "POWER",
  "sourcePointId": "<boardPointId>",
  "pointIds": [
    "<bedroomSocketId>",
    "<studySocketId>"
  ],
  "topology": "CHAIN",
  "idempotencyKey": "qa-structures-and-installations-1"
}
```

Submit to `create_installation_circuit`. Then reread the LIGHTING selection at the returned version:

```json
{
  "projectId": "<projectId>",
  "floorId": "<floorId>",
  "expectedModelVersion": 43,
  "name": "Oświetlenie — sypialnia i gabinet",
  "medium": "LIGHTING",
  "sourcePointId": "<boardPointId>",
  "pointIds": [
    "<bedroomLightId>",
    "<bedroomSwitchId>",
    "<studyLightId>",
    "<studySwitchId>"
  ],
  "topology": "TREE",
  "idempotencyKey": "qa-structures-and-installations-2"
}
```

Versions 42/43 are illustrative; always use the real returned version. Omitted specification/protection
uses backend defaults; explicit choices must come from the catalogue. A point already in another
same-medium circuit cannot be added directly. To regroup, detach just the requested IDs with
`update_installation_circuit(removePointIds: [...])`, then reread selection and attach them to the
chosen circuit. This preserves the physical outlets; inspect the old circuit for empty membership.

### Route through the corridor or a chosen junction

For an automatic route the backend follows wall faces and existing doors/cutouts. A room is not a
waypoint: use actual plan coordinates read from geometry when requesting a corridor path.
`routeOverrides` constrains one branch by its `targetPointId`; it does not define a junction or force
CHAIN point order. Read `fromPointId`/`toPointId` to establish that branch's actual start.
Plan `waypoints` use `{x,y}`; `waypoints3D` use `{x,y,z}` where z = plan Y and y = floor-local height.
Never use project elevation as a floor-local y value. The backend adds orthogonal legs and validates
crossings; inspect `ROUTE_CROSSES_WALL` rather than claiming a crossing creates an opening.

To control branching without converting the circuit to plain lines:

1. Read `get_installation_circuit_layout`. On an automatic route choose the zero-based
   `segments` index and a position on that physical segment. For an existing manual network use
   its `edgeId`. Do not select a shifted `displaySpans` lane.
2. Preview `nodeEdit: {action: "ADD", segmentIndex: <index>, position: {x: ..., y: ..., z: ...}}`
   with `preview_installation_circuit_node` at the current version. A position must be within 15 cm
   of the selected path and at least 1 cm from its endpoints; the backend projects it onto the path.
   Save the same intent/version with `update_installation_circuit`, then take the junction `nodeId`
   and edge IDs from the saved result's `circuits[].network`.
3. To branch an existing member from that junction, preview and save
   `nodeEdit: {action: "CONNECT", nodeId: "<junctionId>", targetPointId: "<memberPointId>"}`.
   This changes that endpoint's incoming edge; it does not add a second supply to the same endpoint.
   CONNECT selects an unlinked junction node, not the node of a socket or of the source.
4. A target outside this circuit must first be eligible. Attach it with `addPointIds`, reread the
   resulting layout/version, then preview CONNECT. The node preview accepts only nodeEdit, so it
   cannot preview a not-yet-attached target together with addPointIds. Saving both in one update
   remains supported when no preview is needed. Alternatively, `add_installation_points` with
   `circuitId` and `nodeId` places a new endpoint and connects its branch atomically.
5. Use `MOVE` for the shared junction, or `EDIT_EDGE` with the complete 3D `controls` list to shape
   one edge through successive rooms. Empty controls make that edge straight and do not request
   automatic wall routing. Preview first and inspect warnings about the edited physical path.
   Editing only `topology` does not replace a stored manual network; `clearRouteOverrides: true`
   deliberately clears the manual network and branch waypoints to restore automatic routing.

If both POWER and LIGHTING should follow the corridor, edit their routes separately. Inspect
`displaySpans.circuitIds` and `cableCount` for shared spans; these are visual grouping data, not
connections. Do not merge circuit memberships, multiply electrical runs to represent cable cores,
or halve cable quantities just because two displayed routes overlap. The backend's physical
segments and takeoff remain the quantity source.

### Check and report what was actually connected

Read `get_installation_circuit_layout` for each changed circuit, `get_distribution_board` and
`validate_floor_geometry`. Report the served rooms, source, medium, topology/manual junctions,
actual cable length including reserve, unconnected requested points and remaining warning codes.
Confirm connectivity from segment point IDs or network edges, never from two lines touching on screen.
If a manual route became adjusted after walls/points moved, review it before accepting the result.
For estimate rows use current `CIRCUIT_*` quantity sources; existing frozen estimate quantities need
an explicit update through the estimate workflow. Do not silently rewrite prices or totals.
