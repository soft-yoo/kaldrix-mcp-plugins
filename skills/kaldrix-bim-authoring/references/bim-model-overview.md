# Building outline authoring

A project contains independent buildings. Each building has a stable UUID, a name, a description, a type and an outer-face `baseOutline`. Floors belong to one building and inherit this footprint unless they have an `outlineOverride`. Coordinates use project plan metres (x, y); 3D plan coordinates are (x, z), with height on y.

## Required order

1. Resolve the intended project first with `get_projects` or `get_project_detail`. For a new project, call `create_project(idempotencyKey, name, description, type)`; it creates one named building. Read `projectId`, `created.buildingId` and `modelVersion` from the result. You may supply `outline: {width: 10, depth: 8}` to save its initial rectangular footprint in the same call, without creating floors.
2. Keep the returned `created.buildingId` for the house. `save_project_building` is optional: use the existing building UUID to rename or describe it, and a new UUID only for an additional building such as a garage. Do not create another building for the house by default.
3. If the building has no footprint yet, call `set_building_outline` before adding floors, with ordered outer-face polygon vertices and a positive `wallThickness`. If `create_project` already received dimensions, reuse that outline. Do not repeat the first vertex. A 10×8 m footprint is `[{x:0,y:0},{x:10,y:0},{x:10,y:8},{x:0,y:8}]`.
4. Review the change with the user, then call `create_building_floor` with this `buildingId`, a floor name, level, type, height, elevation, current version, a stable `idempotencyKey` and `confirm=true`. It requires `projects.write` and `projects.delete`: adding an upper storey can move and rebuild an existing roof. The backend creates its envelope walls and derives its initial interior. Add further floors with the same building UUID to inherit the footprint. Use `set_floor_outline` only for a differing storey; a null outline restores inheritance.
5. Read the floor with `get_floor_detail` to obtain envelope wall paths and current geometry. Then call `create_partition` with axis points, thickness and an internal type (`PARTITION`, `INTERNAL_LOAD_BEARING` or `VIRTUAL`). A partition reaching surrounding walls subdivides the existing floor interior into rooms automatically. Draw each shared physical partition once; never create adjacent room rectangles. An unfinished partition reduces usable area but need not create another room. The backend owns validation and topology.
6. Read `get_floor_detail` after each physical edit. Call `label_room` with the current derived room UUID, name and function. Room geometry is output. Never construct separate room rectangles or duplicate their shared walls.
7. Add wall-hosted openings with `add_opening_by_intent`. Prefer an explicit wall UUID for windows and external doors. Room references help placement; the physical opening belongs to its wall.
8. Call `create_roof` on the highest building floor: `roof: {}` gives the app's default roof; `roofType` and, for a pitched roof, at most one of `pitchDegrees` (simplest), `ridgeHeight` or `ridgeElevation` override it. Its footprint is the effective floor outline; room divisions do not define it. Roof heights are floor-local. Read the returned roof assembly UUID for later removal/replacement.
9. Configure the existing shared floor and ceiling surfaces. Add room- or floor-scoped structures and installations using their typed tools. `buildUps` are reusable layered construction definitions referenced by walls and slabs. They are not buildings or floors.
10. Call `validate_project_geometry` and inspect quantities and warnings before reporting completion. Render the plan with `render_floor_svg` when useful.

## Editing and ownership

Use `set_building_outline` or `set_floor_outline` for envelope geometry. Use `update_wall` for an internal partition or permitted envelope properties. Use `remove_wall` with confirmation and delete scope to remove an internal partition; room division is recalculated. Rooms retain matching identities and descriptions where possible. Re-read IDs after splits and merges.

There are no room creation/removal, room merge/split, wall joining, boundary repair or legacy blueprint-build tools. Physical shared partitions exist once. A room label does not move walls, installations or slabs. A room cannot be deleted as a physical element.

Use current `modelVersion` on every versioned mutation. Resolve a version conflict by reading the model and planning again. Inspect operation results for the current version, created IDs and validation feedback.

## More than one building

Repeat building metadata, footprint and floor creation inside the same project for a garage or another house. Keep the intended `buildingId` explicit. Each roof uses the highest floor of its own building. Building-local risers and structural supports do not link independent buildings. Site geometry remains outside their floor collections.
