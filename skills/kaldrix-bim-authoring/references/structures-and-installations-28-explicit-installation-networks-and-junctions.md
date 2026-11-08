## Explicit installation networks and junctions

An installation circuit may carry `network: {rootId, nodes, edges}`. Nodes have stable `id`, local
floor `position: {x,y,z}`, optional installation `pointId`, and backend-derived `canRemove`.
Linked endpoints follow installation points. Unlinked nodes are movable junctions. Edges have stable
`id`, `fromId`, `toId`, intermediate `controls`, and parallel `runs`. A bend belongs to one edge;
a junction joins edges. Read `get_installation_circuit_layout` for the physical paths, quantities,
fittings, warnings and `edgeId/fromNodeId/toNodeId`; do not infer connectivity from overlapping pixels.

Use `update_installation_circuit` with the current `expectedModelVersion` and `nodeEdit`:

- `ADD`: supply `edgeId` (manual network) or `segmentIndex` (current automatic layout), and a local
  `position` on that path. The backend captures the automatic route and splits its edge without
  changing material length. Read the updated model to obtain node and edge identifiers.
- `MOVE`: `nodeId` plus local `position` moves a shared junction and all attached edge endpoints.
- `CONNECT`: `nodeId` plus `targetPointId` reconnects that circuit endpoint to the junction. Include
  `addPointIds` if the point is not a member yet. It does not create a duplicate incoming cable.
- `EDIT_EDGE`: `edgeId` plus `controls` changes only the shape of that edge; empty controls make it straight.
- `REMOVE`: `nodeId` merges a junction with one incoming and one outgoing edge while preserving its bend.
  Reconnect branches before removing a branching node; obey `canRemove`.

`add_installation_points` accepts `circuitId` plus `nodeId` to create endpoints and their branch
atomically in one model version. Electrical and water circuits use the same network contract.
`clearRouteOverrides: true` returns the circuit to automatic routing. DRAIN accepts the same explicit
network. Its graph is oriented stack-to-fixture, opposite to liquid flow. Each leg outside the initial
vertical stack connection must rise by at least slope × horizontal distance toward the fixture.
The initial stack connection may descend to the buried pipe level; it must not fold back. Each drain
edge represents one pipe (`runs=1`); use separate branches for separate pipes.

For DRAIN, MOVE keeps the requested node height and physical fixture positions. EDIT_EDGE keeps supplied
control heights. The backend fits other free heights without changing x/z, IDs or connections.
Impossible edits return INSTALLATION_CIRCUIT_DRAIN_SLOPE with field position.y and circuit/edge references;
reread the network and adjust the intent. A full network snapshot is validated exactly, without repair.
Changing routing.slope fits free heights; levelOffset is available depth and produces a warning when exceeded.
Legacy waypoints3D on automatic DRAIN routes remain unsupported; use nodeEdit for explicit 3D heights.

Read media[].automaticRoute3D, networkEditing3D, gravityManaged and networkEditingHint from
get_installation_conduit_catalog before authoring. The same server metadata is available to web and Android.
The quantity engine, preview and persistence use the same canonical network and router.
The backend rejects loops, disconnected edges, incompatible points, invalid coordinates and stale
versions. It owns takeoff and fittings; clients, including Android, only submit intent and render results.
