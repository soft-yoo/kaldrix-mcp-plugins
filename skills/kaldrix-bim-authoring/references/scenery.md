# Scenery: reusable house surroundings for the 3D presentation

A scenery is an object of the **company graphics library**: trees, shrubs, a hedge or fence, a far
backdrop and landscape (mountains, sea, river, lake) around an empty place for a building. It belongs to no project. A project only points
at a scenery and places it relative to its building. A scenery never changes the building model,
quantities, the estimate, `modelVersion` or `sceneVersion`.

Needs the GRAPHICS module and `graphics.*` scopes. Only company team members change the library.

## Which tool when

| Goal | Tool |
| --- | --- |
| See the company sceneries, the default one and how many projects selected each | `list_sceneries` |
| Read one scenery with its objects, areas and current `version` | `get_scenery` |
| Build surroundings automatically | `generate_scenery` |
| Start an empty scenery and place objects by hand | `create_scenery`, then `save_scenery_objects` |
| Add or reshape water and ground areas | `save_scenery_areas` |
| Rename, describe or mark the company default | `update_scenery` |
| Try a variant without touching the original | `create_scenery` with `copyFromSceneryId` |
| Change the plot or clear zone size | `generate_scenery` with `sceneryId` and the new sizes |
| Move, rotate, scale or lock objects | `save_scenery_objects` with object `id`s |
| Remove chosen objects and areas or clear many | `remove_scenery_objects` |
| Choose what a project shows and where it stands | `set_project_scenery` |
| See what a project shows now | `get_project_scene` (field `scenery`) |
| Delete a scenery | `delete_scenery` |

## Coordinates

- The scenery origin is the centre of the **building clear zone**. The plot and the clear zone are
  rectangles centred on the origin: width along X, depth along Y, metres.
- `x`, `y` are plan metres with the same axes as the project plan; `elevation` is the height above the
  ground. **The plot front is the −Y side.**
- `rotationDeg` turns an object about the vertical axis, 0–360.
- In a project the origin stands at the anchor (centre of the bounding box of the building footprints)
  plus `offsetX`, `offsetY`; then the whole scenery turns about that point by `rotationDeg`. A project
  with no building has the anchor at (0, 0).

## Versions

Every write of an existing scenery needs `expectedVersion`. Read `version` from `get_scenery` or from
the result of the previous write, and send it with the next write. A stale value is refused with
`SCENERY_VERSION_CONFLICT`: read the scenery again, check what changed, then repeat the write.
`set_project_scenery` needs no version.

## Generation

`generate_scenery` needs a planting `style` and a `density`. Choose `GARDEN` for a fenced garden,
`HEDGED_PLOT` for a hedge around the plot, or `FOREST` for woodland extending beyond the plot.
`ORCHARD` places trees in rows and `MEADOW` leaves an open area with low vegetation.
The maximum number of generated objects is 40 for `LOW`, 90 for `MEDIUM` and 160 for `HIGH`.

Rules the generator keeps:

- nothing stands in the building clear zone, and trees keep 4 m away from it;
- a 4 m wide access corridor from the clear zone to the plot front stays empty;
- nothing is generated inside a water area;
- repeating the generation with the same seed, settings, available models and kept objects reproduces the layout.

Without `sceneryId` a new scenery is created. With `sceneryId` and `expectedVersion` the scenery is
generated again: unlocked `GENERATED` objects are replaced, `MANUAL` and locked objects stay. To keep
a generated tree, lock it with `save_scenery_objects` before generating again. To get a different
layout, send another `seed`.

Use the optional `feature` with any planting style. `SEATING` requests outdoor seating and related
furniture, `VEGETABLE_GARDEN` requests garden beds or equipment, and `PLAY_AREA` requests play equipment.
The generator uses available Kaldrix catalog models that fit on the plot. Missing equipment is skipped.
Omit `feature` to keep the saved choice on regeneration. New and older sceneries default to `NONE`.
Send `NONE` explicitly to turn off the generated equipment. The result reports the choice as `generationFeature`.

Optional landscape:

- `backdrop` — omitted: tree lines and distant houses, usually 90–220 m away. For large plots the ring
  moves outward, with its inner radius at least 10 m beyond half the plot diagonal. Other choices:
  `NONE`; `TREELINE`; `HILLS` (1.2–1.8 km away); `MOUNTAINS` (6–8 km away). A backdrop with no model in
  the library is skipped. Every variant respects the building clear zone and kept objects.
- `water` — `POND` (inside the plot, away from the clear zone), `RIVER` (a winding band beyond one side
  of the plot), `SEA` (a very large area behind the plot, on the +Y side) or `NONE`. It can create one
  `GENERATED` `WATER` area. Regeneration replaces unlocked generated areas; manual and locked areas
  stay. If a kept `WATER` area already has the same outline and elevation, no duplicate is added.

`forProjectId` sizes the clear zone from that project's building footprint plus 2 m on every side. It
only reads the project. Refused with `PROJECT_SCENERY_NO_BUILDING` when the project has no building.
Default sizes: plot 30 × 40 m, clear zone 14 × 12 m. The plot must reach at least 2 m past the clear
zone on every side.

`SCENERY_CATALOG_EMPTY` means the catalog has no usable models for this generation. You can place a
ready company model by hand with `save_scenery_objects`.

## Objects by hand

`save_scenery_objects` takes 1–100 items in one write. Items without `id` add `MANUAL` objects and need
`assetId`, `x`, `y`; items with `id` change only the given fields. Do not mix both kinds in one call.
`assetId` is a READY `MODEL_3D` from `list_graphic_assets` or `list_graphic_library`. One invalid item
rejects the whole write. A scenery holds up to `objectLimit` objects (`SCENERY_OBJECT_LIMIT_EXCEEDED`
above it). Far landscape is allowed: `x`, `y` within ±10 000 m, `elevation` −500…5000 m, `scale`
0.01…1000, so one mountain model can stand kilometres away at a large scale.

## Water and ground areas

`save_scenery_areas` adds (no `id`: `kind` and `outline` required) or changes (with `id`) flat polygons:

- `WATER` — sea, lake or river; no texture; default `elevation` −0.3. Its bounding box must not overlap
  the building clear zone (`SCENERY_AREA_INVALID`).
- `GROUND` — a patch of terrain such as a beach or meadow; default `elevation` 0; optional
  `textureAssetId` (a READY `TEXTURE`); may overlap the clear zone.

`outline` is a flat list `x1, y1, x2, y2, …` of 3–200 points in scenery plan metres. Do not repeat the
first point; edges must not cross. A scenery holds up to `areaLimit` areas
(`SCENERY_AREA_LIMIT_EXCEEDED` above it).

For an existing area, send its `id` and `clearTexture: true` to detach the texture while keeping the
area and its outline. A null `textureAssetId` alone does not clear it; without `clearTexture: true`,
omitting `textureAssetId` or setting it to null keeps the saved texture.
Detaching does not require access to the old texture, including a withdrawn catalog item.

## Scenery in a project

`set_project_scenery` takes a `choice`:

- `DEFAULT` — the company default scenery, when the company has one;
- `SELECTED` — the scenery given by `sceneryId`;
- `NONE` — no scenery, even when the company has a default.

Omitted `offsetX`, `offsetY` and `rotationDeg` keep their saved values. Typical flow for one house:

1. `generate_scenery` with `forProjectId`, a `style` and a `density`.
2. `set_project_scenery` with `choice=SELECTED` and the new `sceneryId`.
3. Read `get_project_scene` and report which scenery is shown.

To give every project of the company the same surroundings, mark one scenery with
`update_scenery` `defaultForCompany=true`; a company has one default.

## Deleting

`delete_scenery` and `remove_scenery_objects` need `confirm=true` and the `graphics.delete` scope. Ask
the user first. `remove_scenery_objects` takes either ids (`objectIds`, `areaIds`; locked ones too) or a `scope`:
`GENERATED_ONLY` keeps `MANUAL` and locked objects and areas, `ALL` removes everything. `delete_scenery` is refused with `SCENERY_IN_USE` while
a project has selected the scenery: choose another scenery in those projects first. Projects that
showed it only as the company default are left with no scenery. A library model or texture used in a
scenery cannot be deleted from the company library until it is removed from the scenery.

If a catalog asset is withdrawn or its file becomes unavailable, the scenery still contains the
object or area and its ID. Remove the chosen entry by its ID, or detach a ground area's texture with
`save_scenery_areas` and `clearTexture: true`. A failed download alone does not prove a file was deleted.
