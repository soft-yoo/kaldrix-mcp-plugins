
## Shorter electrical circuit routes

`validate_floor_geometry` can return `OPTIMIZE_CIRCUIT_ROUTE` for an existing POWER or LIGHTING circuit: the backend
compares its complete CHAIN and TREE routes, incl. vertical drops and the unchanged reserve factor. Over MCP the
`availableActions[]` entry carries `id`, `modelVersion`, `elementIds` and `nextTools`, not the app's before/after
metres; `preview_circuit_optimization` returns full before/after routes for comparison. Bounded cable-length
heuristic, not a global minimum or price optimization: extra junction boxes can cost more than the saved cable.

Execute the exact `actionId` and `modelVersion` with `execute_optimize_circuit_route_action(confirm=true)`, then re-read
`get_installation_circuit_layout` and takeoff. Source, target point IDs, circuit ID, cable spec, protection, RCD and
routing parameters are preserved; lighting and socket circuits are never merged; a stale version is rejected instead
of silently applying a new proposal.

No CHAIN/TREE geometry action for: manual networks, branch waypoints, adjusted routes awaiting review, explicit STAR
home runs, circuits with warnings, fewer than two or more than 64 unique targets, other trades. No action = no
supported safe improvement; never clear manual routes or change protective devices to force it. Water, drainage,
heating, gas and ventilation use the separate `preview_circuit_optimization` contract below (keeps canonical
medium-specific routing constraints).
