
## Shorter electrical circuit routes

`validate_floor_geometry` can return `OPTIMIZE_CIRCUIT_ROUTE` for an existing
POWER or LIGHTING circuit. The backend compares its complete CHAIN and TREE routes, including vertical drops
and the unchanged reserve factor. Over MCP the action in `availableActions[]` carries its `id`, `modelVersion`,
`elementIds` and `nextTools`, not the before/after metres shown in the app; `preview_circuit_optimization` returns the
full before/after routes when the user wants to compare. This is a bounded cable-length heuristic, not a global
minimum or a price optimization: additional junction boxes can cost more than the saved cable.

Read the proposal, then execute its exact `actionId` and `modelVersion` with
`execute_optimize_circuit_route_action(confirm=true)`. Re-read `get_installation_circuit_layout` and quantity takeoff.
Source, target point IDs, circuit ID, cable spec, protection, RCD and routing parameters are preserved.
Lighting and socket circuits are never merged. A stale version is rejected instead of silently applying a new proposal.

Manual networks, branch waypoints, adjusted routes awaiting review, explicit STAR home runs, circuits with
warnings, fewer than two or more than 64 unique targets, and other trades have no CHAIN/TREE geometry action.
Absence of this action means no supported safe improvement is available; do not clear manual routes or change
protective devices to force it. For water, drainage, heating, gas and ventilation use the separate
`preview_circuit_optimization` contract below; it preserves canonical medium-specific routing constraints.
