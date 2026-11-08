## Typical room set in one call

`add_installation_point_preset` (projectId, floorId, spaceId, optional `preset`) places a typical set of points
in one room, planned by the backend from the room's walls, doors and windows: `KITCHEN`, `BATHROOM`, `ROOM`,
`LIVING_ROOM`, `HALL`, `WC`, `LAUNDRY` (omit `preset` to use the suggestion from the room name and type: a room named
WC or toaleta gets `WC`, one named pralnia or laundry gets `LAUNDRY`). Every preset adds a switch
beside the door and a ceiling light point; the rest depends on the preset (worktop sockets, sink, shower,
radiators under windows…). Use it first, then fine-tune with the tools below.

Whole floor at once: `add_installation_point_presets_for_floor` (projectId, floorId, optional `presets` =
`[{spaceId, preset?}]` — the rooms to fill, each with its preset or, without `preset`, the suggested one; optional
`system` to add only one branch). Without `presets` every room gets its suggested preset
(kitchen → `KITCHEN`, bathroom → `BATHROOM`, corridor/staircase → `HALL`, …) in one atomic call with one model
version bump — the same contract as the app. Points a room already has are not duplicated, so rooms that are already
complete are skipped; the result lists the preset and added count per room. More than 200 points in total → call again
with `presets` for smaller groups of rooms. For a focused workflow, set `system` (ELECTRICAL, WATER, DRAINAGE, HEATING,
GAS or VENTILATION) and the rooms in `presets`; only matching point kinds are planned and saved. Presets do not guarantee equipment for every
branch in every room: use `add_installation_points` for individual equipment (for example gas appliances).
Then read the returned modelVersion and plan circuits from the saved source and receiver IDs.
