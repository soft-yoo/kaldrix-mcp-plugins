# Building outline authoring

A project contains independent buildings, each with a stable UUID, name, description, type and an outer-face `baseOutline`. Floors belong to one building and inherit its footprint unless they have an `outlineOverride`. Coordinates use project plan metres (x, y); 3D plan coordinates are (x, z), with height on y.

## Required order

Canonical sequence: project → building → outline → floors → partitions → room labels. Step details,
examples and inputs: [required order](house-building-workflow-03-required-order.md).

- Create floors with `create_building_floor` (`confirm=true`, `projects.write` + `projects.delete`: an upper storey can move and rebuild an existing roof) and the height rules below.
- Draw each shared partition once with `create_partition`; rooms are derived. Name them with `label_room`.
- Call `create_roof` on the highest building floor. Its footprint is the effective floor outline; room divisions do not define it. Roof heights are floor-local.
- Configure the existing shared floor and ceiling surfaces; never create a slab per room.
- Finish with `validate_project_geometry`; `render_floor_svg` when useful.

## Floor and room heights

- Floor `height` is floor-to-floor, including the ceiling slab; `clearHeight` is the alternative clear ceiling height for a regular storey. Send one, never both: creation requires one, an update can omit both. The backend converts the intent and returns `height`, `clearHeight`, `ceilingThickness` and `ceilingThicknessSource`; read that result instead of calculating or adding slab thickness yourself. There is no MCP floor-height preview tool (the separate GraphQL preview serves the app).
- Send `ceilingThickness` only when the user explicitly changes the managed ceiling slab's thickness; omission keeps its existing raw value or assumed profile. `DEFAULT_PROFILE` is an assumption to review, not a value to copy into a write: the backend leaves raw thickness unset. `BUILD_UP` means assigned layers control thickness; edit those layers instead of overriding thickness through the floor.
- Roof-dependent attic clear height is read-only: change the storey with gross `height`, or its roof geometry with the roof/attic tools. A clear height does not uniquely determine an attic's floor-to-floor height. SITE has no ceiling; do not send `clearHeight` or `ceilingThickness`. On a backend refusal, correct the intent; do not retry it through another tool.
- A room's returned `height` is resolved to its ceiling limit; `maximumHeight` is the nominal limit including the structure, attic and suspended ceiling. Roof slopes still clip quantities separately. `heightOverride=null` means AUTO. For a deliberate lower room ceiling send `label_room.height` ≤ `maximumHeight`; `resetHeight=true` restores AUTO. Do not combine them; omit both during a name/function edit so AUTO stays automatic.
- Own room height survives a taller storey. A change that lowers the ceiling limit below an own height is refused atomically; agree a lower height or reset with the user before retrying. Interior finishes and room volume follow each room's ceiling. The shared physical partition normally reaches the structural slab underside; lowering one room's ceiling does not shorten the wall of its taller neighbour.

## Editing and ownership

Envelope: `set_building_outline` / `set_floor_outline`. Internal partition or permitted envelope properties: `update_wall`; `remove_wall` (confirmation, delete scope) removes an internal partition. Rules: [editing and ownership](house-building-workflow-04-editing-and-ownership.md).

## More than one building

Each building has its own footprint, floors and roof (on its own highest floor); keep `buildingId` explicit. Site floor, per-building reports and building removal: [more than one building](house-building-workflow-05-more-than-one-building.md).
