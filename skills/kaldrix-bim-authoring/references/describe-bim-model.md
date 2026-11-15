# Building outline authoring

A project contains independent buildings, each with a stable UUID, name, description, type and an outer-face `baseOutline`. Floors belong to one building and inherit its footprint unless they have an `outlineOverride`. Coordinates use project plan metres (x, y); 3D plan coordinates are (x, z), with height on y.

## Required order

project → building → outline → floors → partitions → room labels. Steps, inputs and examples:
[required order](house-building-workflow-03-required-order.md); heights: [geometry guide](geometry-guide.md#floor-and-room-heights).

- `create_project` creates one building; keep `created.buildingId`. Outline with `set_building_outline`, then
  `create_building_floor` (`confirm=true`). Partitions once with `create_partition`; rooms are derived, named with
  `label_room`; never build room rectangles. Openings via `add_opening_by_intent` belong to their wall.
- `create_roof` on the highest building floor: `roof: {}` gives the app's default roof; `roofType` and, for a pitched
  roof, at most one of `pitchDegrees` (simplest), `ridgeHeight` or `ridgeElevation` override it. Its footprint is the
  effective floor outline; room divisions do not define it. Roof heights are floor-local.
- Configure the existing shared floor and ceiling surfaces; `buildUps` are layered definitions for walls and slabs,
  not buildings or floors. Finish with `validate_project_geometry`.

## Editing and ownership

Envelope: outline tools only; partitions: `update_wall` / `remove_wall` (confirmation, delete scope). No room create/remove/merge/split tools; a room label moves nothing and a room is not a deletable physical element. Send current `modelVersion`; on conflict reread and replan. Details: [editing and ownership](house-building-workflow-04-editing-and-ownership.md).

## More than one building

Repeat metadata, footprint and floors per building in one project with an explicit `buildingId`; each roof uses its own building's highest floor. See [more than one building](house-building-workflow-05-more-than-one-building.md).

## Quantity metrics

`totalArea`: sum of distinct `Space.area` values. `totalWallNetArea`: sum of distinct physical wall net areas.
`totalExternalWallNetArea`: net facade area, calculated from exterior faces on building envelope walls across floors,
including gables and subtracting hosted openings. A room-contained standalone wall is one floor-level wall and is counted once.
Room finish quantities use only that room's faces and portions of the shared floor and ceiling surfaces.
