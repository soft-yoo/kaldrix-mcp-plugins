## Typical room set in one call

`add_installation_point_preset` (projectId, floorId, spaceId, optional `preset`) places a typical point set in one
room, planned from its walls, doors and windows: `KITCHEN`, `BATHROOM`, `ROOM`, `LIVING_ROOM`, `HALL`, `WC`,
`LAUNDRY`. Omit `preset` for the suggestion from room name and type (named WC or toaleta → `WC`, pralnia or laundry →
`LAUNDRY`). Every preset adds a switch beside the door and a ceiling light point; the rest depends on the preset
(worktop sockets, sink, shower, radiators under windows…). Use it first, then fine-tune.

Whole floor: `add_installation_point_presets_for_floor` (projectId, floorId, optional `presets` = `[{spaceId, preset?}]`
— rooms to fill, each with its preset or the suggested one; optional `system` for one branch). Without `presets`
every room gets its suggested preset (kitchen → `KITCHEN`, bathroom → `BATHROOM`, corridor/staircase → `HALL`, …)
in one atomic call with one model version bump — same contract as the app. Existing points are not duplicated, so
complete rooms are skipped; the result lists preset and added count per room. Over 200 points in total → call again
with `presets` for smaller room groups. Focused workflow: set `system` (ELECTRICAL, WATER, DRAINAGE, HEATING, GAS or
VENTILATION) and the rooms in `presets`; only matching point kinds are planned and saved. Presets do not guarantee
equipment for every branch in every room: use `add_installation_points` for individual equipment (e.g. gas
appliances). Then read the returned modelVersion and plan circuits from the saved source and receiver IDs.
