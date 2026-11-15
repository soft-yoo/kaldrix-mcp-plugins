# Building outline authoring

A project contains independent buildings, each with a stable UUID, name, description, type and an outer-face `baseOutline`. Floors belong to one building and inherit its footprint unless they have an `outlineOverride`. Coordinates use project plan metres (x, y); 3D plan coordinates are (x, z), with height on y.

## Required order

project → building → outline → floors → partitions → room labels; steps: [required order](house-building-workflow-03-required-order.md). Relationships:

- Outline tools own envelope walls; `create_building_floor` creates them and the initial interior. `create_partition` subdivides the interior; rooms (with their outlines) are derived backend output, named with `label_room`.
- Openings from `add_opening_by_intent` belong to one physical wall; room references only help placement.
- `create_roof` works on the highest building floor; its footprint is the effective floor outline, not room divisions; roof heights are floor-local. Default roof and pitch inputs: [required order](house-building-workflow-03-required-order.md).
- Floors have existing shared floor and ceiling surfaces to configure; never a slab per room. `buildUps` are reusable layered construction definitions referenced by walls and slabs, not buildings or floors.
- Structures and installations are room- or floor-scoped and added with typed tools.

## Editing and ownership

- Envelope geometry: `set_building_outline` or `set_floor_outline`. Internal partition or permitted envelope properties: `update_wall`; `remove_wall` (confirmation, delete scope) recalculates room division, keeping matching room identities and descriptions where possible. Re-read IDs after splits and merges.
- There are no room creation/removal, room merge/split, wall joining, boundary repair or legacy blueprint-build tools. Shared partitions exist once. A room label does not move walls, installations or slabs; a room cannot be deleted as a physical element.
- Send the current `modelVersion` on every versioned mutation; on a conflict read the model and plan again. Inspect results for the current version, created IDs and validation feedback.

## More than one building

Repeat building metadata, footprint and floors in the same project for a garage or another house, with an explicit `buildingId`. Each roof uses the highest floor of its own building. Building-local risers and structural supports do not link independent buildings; site geometry stays outside their floor collections. Site floor, reports and removal: [more than one building](house-building-workflow-05-more-than-one-building.md).
