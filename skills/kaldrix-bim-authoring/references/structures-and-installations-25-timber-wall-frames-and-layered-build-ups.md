## Timber wall frames and layered build-ups

A build-up (`BuildUp`) is a project-level layer recipe — timber-frame wall, masonry with plasters, plasterboard
partition, RC or timber floor, pitched roof. Walls and slabs reference it by `buildUpId`; assigning sets the element
thickness to the sum of layers. Build-ups are in PROJECTS; only `get_wall_frame` and `get_slab_frame` need STRUCTURES.

1. `list_build_ups` with `includeTemplates=true` (or `kaldrix://projects/build-up-templates`) — templates such as
   `TIMBER_FRAME_WALL_15`, `TIMBER_FRAME_WALL_20`, `TIMBER_FRAME_INTERNAL_WALL_12`, `TIMBER_FRAME_PARTITION_10`,
   `CERAMIC_BLOCK_WALL_25`, `AAC_BLOCK_WALL_24`,
   `CERAMIC_BLOCK_WALL_25_EPS_15`, `GYPSUM_PARTITION_75`, `RC_SLAB_18`, `RC_SLAB_20_FLOATING`, `TERIVA_SLAB_24`,
   `TIMBER_FLOOR`, `TIMBER_JOIST_FLOOR_OSB`, `TIMBER_CABIN_GROUND_FLOOR`, `TIMBER_CABIN_CEILING_BOARDS`, `PITCHED_ROOF`
   and ceiling build-ups (target `CEILING`) `SUSPENDED_CEILING_GK_WOOL`, `TIMBER_BATTEN_BOARD_CEILING`.
2. `create_build_up_from_template` → `buildUpId` (or custom `layers` + `target` + `name`).
   Layer order: walls inside (room side) → outside; slabs and roofs top → bottom. 1..15 layers, each 1 mm–1 m
   (membranes from 0.1 mm), total ≤ 1.5 m. A `FRAME` layer (studs, joists, rafters, battens) needs `frameRole`,
   `frameWidth`, `frameSpacing` (0.2–1.2 m) and usually `frameFill` (e.g. `MINERAL_WOOL`).
3. `assign_build_up` with `elementType` `WALL` (WALL build-ups, no virtual walls) or `SLAB` (ROOF build-ups → ROOF
   slabs, SLAB → other slabs) and the element ids; omit `buildUpId` to unassign (thickness stays).
   A thicker wall can create new geometry errors — the change is then rejected (`BIM_MODEL_INVARIANT_VIOLATION`).
4. `update_build_up` with a full new layer list re-thicknesses every assigned wall/slab; `remove_build_up`
   (`confirm=true`) unassigns everywhere and keeps the thickness. Editing a wall/slab thickness by hand detaches
   the build-up.
5. Estimate: services `WALL` + `WALL_LAYER_AREA`/`WALL_LAYER_VOLUME` or `SLAB` + `SLAB_LAYER_AREA`/`SLAB_LAYER_VOLUME`
   with `elementSubtypes` = `LayerMaterial` names (e.g. `["GYPSUM_BOARD"]`, `["TIMBER_FRAME"]`, `["ROOF","ROOF_TILES"]`).
   Wall layer area = single-face net wall area (openings subtracted). The wall-stud FRAME layer (timber, role `STUD`)
   takes its timber volume from the generated wall frame (step 6), its fill = the rest of the layer; other FRAME layers
   (battens, steel CW studs) use area × thickness × width/spacing. Doors and windows are inserted as before.
6. Timber wall frame: `get_wall_frame` (`projectId`, optional `floorId` / `wallId`; `includeMembers=true` for 3D axes)
   returns the frame generated on read for every wall whose build-up has a timber FRAME layer with role `STUD`:
   `BOTTOM_PLATE` and `TOP_PLATE` (single; `doubleTopPlate=true` previews a double one) along the wall and its gable top,
   `STUD` at the layer spacing from the wall start plus end studs, a pair at polyline bends, one next to the end stud at
   L corners and a pair at T junctions; at each opening `KING_STUD` ×2, `TRIMMER` ×2 under a `HEADER` (120/160/200/240 mm
   for openings ≤ 1.0/1.8/2.4/wider m; `doubleHeader=true` previews doubles), windows a `SILL` with `CRIPPLE` studs under
   it, and `CRIPPLE` studs above every header (stud grid). Regular studs inside openings are dropped. Read `warnings`
   (opening closer than 2 studs to a wall end, no room for a header) and `notes`. Set `buildUpId` with `wallId` to preview
   another build-up before `assign_build_up`. The frame follows wall and opening edits automatically.
7. Frame estimate: `WALL` + `WALL_FRAME_ROLE_LENGTH` (`MB`) / `WALL_FRAME_ROLE_VOLUME` (`M3`) / `WALL_FRAME_ROLE_COUNT`
   (`PCS`) with `elementSubtypes` = roles (at least one; WallType names may narrow the walls), e.g. headers
   `["HEADER"]`, C24 frame without headers `["STUD","BOTTOM_PLATE","TOP_PLATE","KING_STUD","TRIMMER","SILL","CRIPPLE"]`,
   bottom plate anchors `["BOTTOM_PLATE"]` in mb. Assembly per m²: `WALL_LAYER_AREA` with `["TIMBER_FRAME"]`. Do not
   also price the frame with `WALL_LAYER_VOLUME` `["TIMBER_FRAME"]` — that is the same timber.
8. Timber floors and ceilings (slab build-ups, target `SLAB`): `get_slab_frame` (`projectId`, optional
   `floorId` / `slabId`; `includeMembers=true` for 3D axes) returns the frame generated on read for every slab whose
   build-up has a timber FRAME layer with role `JOIST`, `BATTEN` or `COUNTER_BATTEN`. Joists (`JOIST`) span the shorter
   direction (edge direction with the smallest extent; `directionDeg` previews another one) at the layer spacing, first
   line 5 cm from the edge, and bear `min(0.15 m, wall thickness)` into walls of the same floor or the floor below
   (ends without a wall stay free → warning). A slab opening (`add_slab_opening` kind STAIRWELL / HOLE, or `create_slab` holesJson) gets
   `TRIMMER` joists on both sides (`doubleTrimmers=true` previews doubles), `HEADER` members across it and shortened
   joists resting on the headers. Batten layers below run perpendicular to the layer above, interrupted at holes.
   Steel CD profiles (`STEEL_FRAME`) are not generated. Layers are placed top → bottom from the slab top
   (`baseElevation`). Set `buildUpId` with `slabId` to preview another build-up.
   Roof windows and chimneys: `add_slab_opening` on a roof slab (kind ROOF_WINDOW / CHIMNEY, centerX/centerY + width
   along the eaves × length along the slope, optional flashingKind PROFILED / FLAT). The roof framing (ROOF_RAFTERS)
   then cuts the rafters crossing the opening, adds `HEADER` members above and below it and `TRIMMER` rafters at its
   sides, interrupts battens and counter-battens and reduces the covering area. Price them with SLAB_OPENING rules
   (`SLAB_OPENING_COUNT` + elementSubtypes [ROOF_WINDOW], `SLAB_OPENING_FLASHING_COUNT`). Replacing the roof keeps
   openings that still fit the new roof planes; the others are removed and reported in `warnings`.
   Dormers: `add_slab_opening` kind DORMER on a roof plane of at least 15 degrees, with centerX/centerY, width along
   the eaves (0.6-8 m), dormerRoof GABLE or SHED, optional dormerWallHeight (0.4-3 m, default 1.2) and dormerPitch.
   The server sets the length along the slope and recomputes it when the roof pitch changes; free outlines are
   refused. Price dormers with DORMER_ROOF_AREA, DORMER_WALL_AREA and the DORMER_*_LENGTH edge rules (eaves, ridge,
   valleys, verges, flashing); elementSubtypes GABLE / SHED filter by dormer roof.
9. Floor/ceiling estimate: joists and ceiling battens are framing systems (the slab has no frame roles and no
   member rules): `FRAMING_SYSTEM` + `FRAMING_ROLE_VOLUME` (`M3`) with
   `["TIMBER_FLOOR_JOISTS","JOIST","TRIMMER","HEADER","BLOCKING"]`, anchors `FRAMING_FIXING_COUNT`, ceiling battens
   `FRAMING_MAIN_LENGTH` / `FRAMING_ROLE_LENGTH` (`MB`) `["CEILING_TIMBER","BATTEN"]` — model systems and the ones assumed from slab build-ups
   (`assumedFramingSystemIds`), other layers `SLAB` + `SLAB_LAYER_AREA` per material (`["OSB"]`, `["MINERAL_WOOL"]`,
   `["VAPOR_BARRIER"]`, `["GYPSUM_BOARD"]`, `["TIMBER_BOARDS"]`, `["EPS"]`, `["BEAM_AND_BLOCK"]` for Teriva), RC concrete
   `SLAB_LAYER_VOLUME` `["REINFORCED_CONCRETE"]`. **One source per member** with framing systems: within the outline of a
   `TIMBER_FLOOR_JOISTS` system whose joists lie in the slab, the system counts the joists (the rest of the slab — the
   timber floor assumed from the build-up; `countedInFramingSystemIds` + `countedArea` in `get_slab_frame` and in the
   layer takeoff, INFO `SLAB_FRAME_JOISTS_IN_FRAMING_SYSTEM` in `validate_project_geometry`); a
   suspended ceiling system (`CEILING_*`) hanging below the slab keeps its profiles/battens and hangers, and its boards when
   `boardLayers` > 0, so within its outline the build-up layers under the structural layer of that kind are not counted
   (INFO `SLAB_CEILING_LAYERS_IN_FRAMING_SYSTEM`) while wool and membranes stay in the build-up over the whole slab. Recipe for a GK ceiling with wool: `create_framing_system` `CEILING_GK_SINGLE`
   for the room (profiles, hangers, boards) + ceiling build-up `SUSPENDED_CEILING_GK_WOOL` on the room's CEILING slab
   (wool, vapour barrier). A roof slab with a build-up (e.g. `PITCHED_ROOF`) in a roof that has `ROOF_RAFTERS` framing keeps
   rafters, counter battens and battens in the framing only: those build-up frame layers give 0 timber (INFO
   `SLAB_ROOF_LAYERS_IN_ROOF_FRAMING`), the covering, membranes, wool between rafters and boards stay in the build-up.

Error codes: `BUILD_UP_NOT_FOUND`, `BUILD_UP_TEMPLATE_NOT_FOUND`, `BUILD_UP_LAYERS_INVALID`,
`BUILD_UP_LAYER_THICKNESS_INVALID`, `BUILD_UP_THICKNESS_INVALID`, `BUILD_UP_FRAME_INVALID`, `BUILD_UP_TARGET_MISMATCH`,
`BUILD_UP_NAME_INVALID`, `BUILD_UP_ASSIGNMENT_INVALID`, `BUILD_UP_DUPLICATE`, `BUILD_UP_LIMIT` (100 per project).
