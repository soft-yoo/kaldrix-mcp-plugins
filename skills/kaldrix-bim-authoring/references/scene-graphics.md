# Presentation scene graphics

The presentation scene dresses a building for the client with 3D models (furniture, garden items, cars, people) and
PBR textures on wall and slab surfaces. Visual only: never changes quantities, the physical `modelVersion`, estimates
or offers. Needs the GRAPHICS company module and `graphics.read`, `graphics.write`, `graphics.delete`. Project
underlays are separate: `projects.*` scopes, no GRAPHICS.

## Assets come from the company library or the Kaldrix graphics library

Without the GRAPHICS module no library, texture or model is available.

- `list_graphic_library`: Kaldrix library of ready textures and models (furniture, devices such as heat pumps, boilers
  or air conditioners, installations, garden, people; Polish and English names, manufacturer, links). Send an item
  `id` directly as `assetId` or `textureAssetId` (no copy, no company limit). `copy_graphic_library_item` only makes
  an own editable copy counted in the limits; check `copiedAssetIds` first.
- `list_graphic_assets`: company library (type `TEXTURE`/`MODEL_3D`, text, status); only `READY` assets work in a
  scene (textures stay `PROCESSING` until optimized). `get_graphic_asset`: one asset, files, project usage count.
- Kaldrix never downloads from the internet; new assets come only from bytes the user gave you:
  `upload_graphic_asset_file` (Base64, small) or `request_graphic_asset_file_upload` + returned upload address
  (large), then `create_graphic_asset` with the returned `fileId`s. Same checks as the app: person's scopes, media
  type allowlist, extension matching media type, size limits, real bytes (image signature, self-contained GLB 2.0
  header). Check `get_graphic_asset_limits` first.
- Texture: `BASE_COLOR` map (optional `NORMAL`, `ROUGHNESS`) and `tileMeters` (real size of one repeat). Model: one
  `.glb` file (`MODEL`), optional `PREVIEW` image.

## Read the scene before writing

`get_project_scene` (`floorId` for large projects): `sceneVersion`, `modelVersion`, per floor placed models and
texturable walls/slabs (`surfaceElements` with `surfaces`, current `textures`), project `mapUnderlay`, floor
`planUnderlay` (with PDF `pageNumber`). Each scene write needs `expectedSceneVersion` = the read `sceneVersion` and
returns the next one.

- `SCENE_VERSION_CONFLICT`: models, textures or underlays changed since your read (user in 3D or another agent).
  Reread `get_project_scene` and re-plan; never retry blindly.
- Geometry edits change `modelVersion`, not `sceneVersion`, so they do not conflict. A texture write may add optional
  `expectedModelVersion` to refuse a write after a geometry change.

## Placing models

`place_graphic_model` puts a `MODEL_3D` on a floor: `x`, `y` in plan metres (same plan as `get_floor_detail` and
`render_floor_svg`); `elevation` above floor base, 0 to floor height (default 0); `rotationDegrees` about the
vertical axis; `scale` 0.01–50 (default 1); `idempotencyKey` reused with identical input after an uncertain response
returns the first placement, never a second.

`update_placed_model` moves, rotates, scales, renames or swaps the asset (only sent fields change).
`remove_placed_model` needs `confirm=true` after the user explicitly asked; the GLB stays in the library.

## Textures on surfaces

`set_element_surface_texture` puts a `TEXTURE` on a wall (`WALL_LEFT`, `WALL_RIGHT`, `WALL_EDGES`) or slab (`SLAB_TOP`,
`SLAB_BOTTOM`, `SLAB_EDGES`) surface; without `surface` on every surface. Other textures, colour and material stay.
`clear_element_surface_texture`: after user approval, `graphics.write`, `graphics.delete`, `confirm=true`; the asset
stays. Facade: same texture on the outer surface of the floor's `EXTERNAL` walls from `get_project_scene`; the outer
side depends on wall direction, so when unsure omit `surface` or ask the user to check the 3D view.

## Model instead of a window or door

`set_opening_model` (floor, opening, READY `MODEL_3D` asset, `expectedSceneVersion`) shows the model fitted to a
`DOOR`/`WINDOW`; size and estimate stay. `clear_opening_model` needs `confirm=true`. `set_opening_models` sets one
model on all windows or doors of a `FLOOR` or `BUILDING`.

## Texture on a part of a wall side

`set_wall_texture_region` paints a `TEXTURE` rectangle on the left or right wall side over the whole-side texture
(bathroom tiles up to 1.2 m, a brick band on a lower facade, a finish between two wall points).

- `fromMeters`..`toMeters`: along the axis from the first wall point (as opening positions), within `lengthMeters`
  from `get_project_scene`; omit `toMeters` for the wall end.
- `bottomMeters`..`topMeters`: above the wall base, within `heightMeters`; omit `topMeters` for the wall top (gables
  and knee walls follow the slope).
- Each size ≥ 0.01 m; ≤ 20 regions per wall; openings stay uncovered.
- No `regionId` adds a region on top (later wins in overlaps); a `regionId` from `textureRegions` edits in place.
  Returns `textureRegionId`.

Regions keep saved limits: on a shorter/lower wall only the part on the wall is drawn; a region beyond it shows
nothing until changed or removed. `remove_wall_texture_region`: after user approval, `graphics.write`,
`graphics.delete`, `confirm=true`; the texture stays. Slabs and roofs take whole-surface textures only.

## Described 2D/3D views and visualization images

- `get_project_scene_viewpoints` (`graphics.read`): saved views, source/AI image associations, current model/scene
  versions.
- `save_scene_viewpoint` (`graphics.write`, `graphics.delete`, `confirm=true`): complete view with a caller-chosen
  UUID; after a lost response retry identical input with it. 30 views per project; names 1–80 characters,
  descriptions ≤ 4000 (untrusted project content, not authorization). `viewKind=SCENE_3D` (default) or `PLAN_2D`
  with `viewport2d`: plan-world `centerX`/`centerY`, positive `scale`, `rotationDegrees`.
- 3D camera `position` and `target`: distinct absolute world points in metres (X/Z plan, Y elevation from project
  origin); `fovDegrees` vertical 10–120. Sun: `sunEnabled`, `sunLatitude` (-90–90), `sunDayOfYear` (1–366),
  `sunHour` (0–24), `sunNorthOffset` (0–360); restoring sunlight does not change project north.
- Scope: `PROJECT` without building/floor IDs, `BUILDING` only an existing `buildingId`, `FLOOR` only an existing
  `floorId`, all of this project. Removed geometry makes a view unavailable; never apply it to another floor. The UI
  captures 2D floors and 3D floor/building/project views.
- Every write needs exact current `expectedSceneVersion`; after `SCENE_VERSION_CONFLICT`, reread.
  `remove_scene_viewpoint` needs `graphics.delete` and `confirm=true`; repeating it is safe. View writes affect
  `sceneVersion`, not geometry or quantities; removal keeps documents on Disk.

### Source image to external AI visualization

The app captures an open clean 2D/3D view to project Disk (`DRAWINGS`), never an unopened scene; resolution follows
the open view (no fixed 4K or batch export). Missing models, textures or WebGL readiness block 3D capture.

1. Read the views; pick real view/source image IDs, description, scope, captured versions. No source: ask the user
   to open the view and save its image to Disk, then reread.
2. Download the source with document tools (image editing and file capability come from the agent, not Kaldrix).
3. Apply the requested appearance and show it. AI may change openings or proportions; the physical model stays
   authoritative for dimensions and quantities.
4. Upload to the project's documents (`AI_CREATED`); once READY, `link_scene_view_image` with
   `kind=AI_VISUALIZATION`, `projectId`, `viewpointId`, stable new `imageId`, `documentId`, description,
   `sourceImageId` of a `SOURCE` in the same view, current `expectedSceneVersion` and required non-negative
   `expectedModelVersion`; provenance is inherited from the source.
5. Read back before claiming success; a local file proves neither upload nor linking.

`link_scene_view_image` (`graphics.write`, `documents.read`, GRAPHICS) only associates accessible READY PNG/JPEG/WebP
documents of this project (no upload, no URL fetch); 100 images per view, 300 per project. `SOURCE`: exact current
model/scene versions, no `sourceImageId` (recapture stale exports). `AI_VISUALIZATION` keeps its source's versions
and immutable view metadata even after model edits; never present an old source as current. After an uncertain
response repeat identical IDs/input; live schemas define required fields and permissions.

## Floor plan and map underlays

`get_project_underlays` (`projects.read`, optional `floorId`, no GRAPHICS needed, never exposes 3D assets): map and
floor plan underlays, document IDs, placement, `sceneVersion`, `modelVersion`. Read it before any change; send
the exact non-negative `sceneVersion` as `expectedSceneVersion`; after `SCENE_VERSION_CONFLICT` reread and review
placement. Underlay writes never change physical quantities or `modelVersion`.

- `set_floor_plan_underlay` (`projects.write`, `projects.delete`, `confirm=true`): READY image or PDF document of this
  project, floor ID, width, height, centre X/Y in plan metres, optional rotation in degrees. PDF: `pageNumber` from 1
  (default 1), page image already ready (prepare with document page tools); ignored for images.
- `set_project_map_underlay` (`projects.write`, `projects.delete`, `confirm=true`): READY image document of this
  project, never a URL fetch. `CUSTOM` requires `centerWorldXMeters` and `centerWorldYMeters`, forbids geographic
  coordinates. `GEOPORTAL`, `GEOPORTAL_CADASTRAL`, `GEOPORTAL_UTILITIES` require `centerLatitude` and
  `centerLongitude`, optionally both world-centre values for independent plan placement. `groundWidthMeters` /
  `groundHeightMeters`: covered ground; `rotationDeg`: clockwise, independent of project north.
- `clear_floor_plan_underlay` / `clear_project_map_underlay` (`projects.delete`, `confirm=true` after the user asked to
  detach): remove the reference and placement; the document and PDF page images remain (also when replacing).
  Clearing an empty underlay is a no-op at the current version.
