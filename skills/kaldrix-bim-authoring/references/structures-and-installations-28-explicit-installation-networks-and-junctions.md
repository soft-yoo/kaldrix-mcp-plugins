## Explicit installation networks and junctions

A circuit may carry `network: {rootId, nodes, edges}`. Nodes: stable `id`, local floor `position: {x,y,z}`, optional
installation `pointId`, backend-derived `canRemove`; linked endpoints follow installation points, unlinked nodes are
movable junctions. Edges: stable `id`, `fromId`, `toId`, intermediate `controls`, parallel `runs`. A bend belongs to
one edge; a junction joins edges. `get_installation_circuit_layout` gives physical paths, quantities, fittings,
warnings and `edgeId/fromNodeId/toNodeId`; never infer connectivity from overlapping pixels.

`update_installation_circuit` with current `expectedModelVersion` and `nodeEdit`:
- `ADD`: `edgeId` (manual network) or `segmentIndex` (current automatic layout) plus a local `position` on that path;
  the backend captures the automatic route and splits the edge without changing material length. Read the updated
  model for node and edge identifiers.
- `MOVE`: `nodeId` + local `position` moves a shared junction and all attached edge endpoints.
- `CONNECT`: `nodeId` + `targetPointId` reconnects that endpoint to the junction (include `addPointIds` if not yet a
  member); no duplicate incoming cable.
- `EDIT_EDGE`: `edgeId` + `controls` changes only that edge's shape; empty controls make it straight.
- `REMOVE`: `nodeId` merges a junction having one incoming and one outgoing edge, keeping its bend. Reconnect branches
  before removing a branching node; obey `canRemove`.

`add_installation_points` with `circuitId` + `nodeId` creates endpoints and their branch atomically in one model
version. Electrical and water circuits share this network contract. `clearRouteOverrides: true` returns to automatic routing.

DRAIN accepts the same explicit network, oriented stack-to-fixture (opposite to liquid flow). Each leg outside the
initial vertical stack connection must rise by at least slope × horizontal distance toward the fixture; the initial
stack connection may descend to the buried pipe level but must not fold back. Each drain edge is one pipe (`runs=1`);
separate pipes need separate branches. MOVE keeps the requested node height and physical fixture positions; EDIT_EDGE
keeps supplied control heights; the backend fits other free heights without changing x/z, IDs or connections.
Impossible edits return INSTALLATION_CIRCUIT_DRAIN_SLOPE with field position.y and circuit/edge references: reread the
network and adjust the intent. A full network snapshot is validated exactly, without repair. Changing routing.slope
fits free heights; levelOffset is available depth (warning when exceeded). Legacy waypoints3D on automatic DRAIN routes
stay unsupported; use nodeEdit for explicit 3D heights.

Before authoring read media[].automaticRoute3D, networkEditing3D, gravityManaged and networkEditingHint from
get_installation_conduit_catalog (same server metadata for web and Android). Quantity engine, preview and persistence
share one canonical network and router. The backend rejects loops, disconnected edges, incompatible points, invalid
coordinates and stale versions, and owns takeoff and fittings; clients, including Android, only submit intent and render results.
