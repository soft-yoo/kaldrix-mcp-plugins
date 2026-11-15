# Scenery: reusable house surroundings for the 3D presentation

A scenery is a **company graphics library** object: trees, shrubs, a hedge or fence, a far backdrop and landscape
(mountains, sea, river, lake) around an empty place for a building. It belongs to no project; a project only points
at it and places it relative to its building. A scenery never changes the building model, quantities, the estimate,
`modelVersion` or `sceneVersion`. Needs the GRAPHICS module and `graphics.*` scopes; only company team members change
the library.

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

- Origin: centre of the **building clear zone**. Plot and clear zone are rectangles centred on it: width along X,
  depth along Y, metres.
- `x`, `y`: plan metres on the project plan axes; `elevation`: height above ground. **The plot front is the −Y side.**
  `rotationDeg`: about the vertical axis, 0–360.
- In a project the origin sits at the anchor (centre of the bounding box of the building footprints; (0, 0) without
  a building) plus `offsetX`, `offsetY`; the whole scenery then turns about that point by `rotationDeg`.

## Versions

Every write to an existing scenery needs `expectedVersion` (`version` from `get_scenery` or the previous write's
result). A stale value fails with `SCENERY_VERSION_CONFLICT`: reread, check what changed, repeat.
`set_project_scenery` needs no version.

## Generation

`generate_scenery` needs a planting `style` and a `density`. Styles: `GARDEN` (fenced garden), `HEDGED_PLOT` (hedge
around the plot), `FOREST` (woodland beyond the plot), `ORCHARD` (trees in rows), `MEADOW` (open area, low
vegetation). Maximum generated objects: 40 `LOW`, 90 `MEDIUM`, 160 `HIGH`.

The generator keeps: nothing in the building clear zone and trees 4 m away from it; an empty 4 m wide access
corridor from the clear zone to the plot front; nothing inside a water area; the same seed, settings, available models
and kept objects reproduce the layout.

Without `sceneryId` a new scenery is created. With `sceneryId` and `expectedVersion` it is regenerated: unlocked
`GENERATED` objects are replaced, `MANUAL` and locked ones stay. Lock a generated tree with `save_scenery_objects` to
keep it; send another `seed` for a different layout.

Optional `feature` (any style): `SEATING` (outdoor seating and furniture), `VEGETABLE_GARDEN` (garden beds or
equipment), `PLAY_AREA` (play equipment), using available Kaldrix catalog models that fit on the plot; missing
equipment is skipped. Omitted `feature` keeps the saved choice on regeneration; new and older sceneries default to
`NONE`; send `NONE` explicitly to turn equipment off. The result reports `generationFeature`.

Optional landscape:

- `backdrop`: omitted = tree lines and distant houses, usually 90–220 m away (for large plots the ring moves outward,
  inner radius ≥ 10 m beyond half the plot diagonal). Others: `NONE`, `TREELINE`, `HILLS` (1.2–1.8 km),
  `MOUNTAINS` (6–8 km). A backdrop without a library model is skipped. Every variant respects the clear zone and kept
  objects.
- `water`: `POND` (inside the plot, away from the clear zone), `RIVER` (winding band beyond one plot side), `SEA`
  (very large area behind the plot, +Y side) or `NONE`. Creates at most one `GENERATED` `WATER` area. Regeneration
  replaces unlocked generated areas; manual and locked areas stay; a kept `WATER` area with the same outline and
  elevation is not duplicated.

`forProjectId` sizes the clear zone from that project's building footprint plus 2 m per side (read only);
`PROJECT_SCENERY_NO_BUILDING` when it has no building. Defaults: plot 30 × 40 m, clear zone 14 × 12 m; the plot must
reach at least 2 m past the clear zone on every side. `SCENERY_CATALOG_EMPTY`: no usable catalog models for this
generation; place a ready company model by hand with `save_scenery_objects`.

## Objects by hand

`save_scenery_objects` takes 1–100 items per write. Items without `id` add `MANUAL` objects and need `assetId`, `x`,
`y`; items with `id` change only given fields; never mix both kinds in one call. `assetId`: READY `MODEL_3D` from
`list_graphic_assets` or `list_graphic_library`. One invalid item rejects the whole write. Up to `objectLimit` objects
(`SCENERY_OBJECT_LIMIT_EXCEEDED`). Far landscape is allowed: `x`, `y` within ±10 000 m, `elevation` −500…5000 m,
`scale` 0.01…1000, so one mountain model can stand kilometres away at a large scale.

## Water and ground areas

`save_scenery_areas` adds (no `id`: `kind` and `outline` required) or changes (with `id`) flat polygons:

- `WATER`: sea, lake or river; no texture; default `elevation` −0.3; its bounding box must not overlap the building
  clear zone (`SCENERY_AREA_INVALID`).
- `GROUND`: terrain patch such as a beach or meadow; default `elevation` 0; optional `textureAssetId` (READY
  `TEXTURE`); may overlap the clear zone.

`outline`: flat list `x1, y1, x2, y2, …` of 3–200 points in scenery plan metres; do not repeat the first point; edges
must not cross. Up to `areaLimit` areas (`SCENERY_AREA_LIMIT_EXCEEDED`).

To detach a texture but keep the area and outline, send its `id` and `clearTexture: true`. Without
`clearTexture: true`, omitting `textureAssetId` or setting it to null keeps the saved texture. Detaching needs no
access to the old texture, including a withdrawn catalog item.

## Scenery in a project

`set_project_scenery` `choice`: `DEFAULT` (company default, when one exists), `SELECTED` (the `sceneryId`), `NONE`
(none, even with a company default). Omitted `offsetX`, `offsetY` and `rotationDeg` keep saved values. One house:

1. `generate_scenery` with `forProjectId`, a `style` and a `density`.
2. `set_project_scenery` with `choice=SELECTED` and the new `sceneryId`.
3. Read `get_project_scene` and report which scenery is shown.

Same surroundings for every company project: `update_scenery` `defaultForCompany=true` (one default per company).

## Deleting

`delete_scenery` and `remove_scenery_objects` need `confirm=true` and `graphics.delete`; ask the user first.
`remove_scenery_objects` takes ids (`objectIds`, `areaIds`; locked ones too) or a `scope`: `GENERATED_ONLY` keeps
`MANUAL` and locked objects and areas, `ALL` removes everything. `delete_scenery` fails with `SCENERY_IN_USE` while a
project has selected it: choose another scenery there first. Projects showing it only as company default are left
with none. A library model or texture used in a scenery cannot be deleted from the company library until removed
from the scenery.

If a catalog asset is withdrawn or its file becomes unavailable, the object or area and its ID stay in the scenery.
Remove the entry by ID, or detach a ground area's texture with `save_scenery_areas` and `clearTexture: true`. A failed
download alone does not prove a file was deleted.
