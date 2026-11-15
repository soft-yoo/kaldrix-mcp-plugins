# Tool order and input sources: building model

Tool descriptions say what each tool does; this page says where inputs come from and what comes next. Live schemas,
`nextTools` and `fixTools` in results stay authoritative.

## Project, buildings and floors

- `create_project` without an initial outline → `set_building_outline` (footprint), `create_building_floor`
  (storeys), `save_project_building` (rename/add buildings), later `set_project_parts`. `PROJECT_CREATE_FORBIDDEN` is
  final; do not retry.
- `duplicate_project`: check `get_projects` first for a unique copy name.
- Floors: outline first, send the intended building id; `set_floor_outline` changes a storey's footprint. Height
  intents (`create_building_floor` / `update_floor`) and `label_room` AUTO/own ceiling:
  [geometry guide](geometry-guide.md#floor-and-room-heights). Attic: `update_roof`, `set_floor_roof_space`.
- After `set_building_outline` or `move_building` read `get_project_detail` (source of `buildingId`). Project parts:
  `get_project_part_catalog` + `get_project_detail` `activeParts` + `set_project_parts`.

## Walls, rooms, slabs and openings

- Envelope walls (axes, types) only via `set_building_outline` / `set_floor_outline`.
- `create_partition`: read `get_floor_detail` first (exact wall axes, `envelopeEdgeIndex`), reread it for current
  room ids, then `label_room`. Room division changes only with `create_partition`.
- Wall ids (`add_beam_between_walls` etc.): `get_floor_detail` or `inspect_floor_bim_model`.
- Multi-element edits: `apply_geometry_batch` (`kaldrix://projects/geometry-batch-example`). `labelRooms` =
  `label_room`, `updateWalls` = `update_wall`, `updateSlabs` = part of `update_slab`, `createPartitions` =
  `create_partition`, `createSlabs` = `create_slab`. Removals: WALL = `remove_wall`, SLAB = `remove_slab`,
  INSTALLATION_POINT = `remove_installation_points`, STRUCTURAL_MEMBER = `remove_structural_members`, FRAMING_SYSTEM
  = `remove_framing_system`, INSTALLATION_CIRCUIT = `remove_installation_circuits`.
- `add_balcony`: slab, railing and balcony door on an external wall in one write.
- Roofs via `create_roof`; roof windows, chimneys, stairwells via `add_slab_opening`; their ids for
  `update_slab_opening` / `remove_slab_openings` from `list_elements` kind SLAB_OPENING. Exclude managed FLOOR/CEILING
  surfaces with `set_floor_surface_usage`; a roof slab goes only with its roof via `remove_roof`.
- Opening styles and ranges per type: `get_opening_catalog` (for `add_opening_by_intent`, `update_opening`). Single
  lintels: `set_opening_lintel`.
- `copy_to_floor` sources: `list_installation_points` (points), `list_structures` (members), `get_floor_detail` (walls).
- Measurements: ids from `get_floor_detail` measurements or `list_elements` (kind MEASUREMENT); paths as in
  `create_measurement`. `get_element` adds references to `list_elements` fields.

## Validation

- `validate_floor_geometry` after geometry changes; finish with `validate_project_geometry`.
- Run the `execute_*_action` tool named in the action's `nextTools` with the action id and `modelVersion` from
  `validate_floor_geometry`; the tool name fixes the action kind.

## Structures and framing

- New system: `preview_framing_system` → `create_framing_system` (same intent, returned `modelVersion`). Saved
  system: `preview_framing_system` with `sourceSystemId` → `update_framing_system` (also spacing, sections, height,
  openings). Prefer it to `explode_framing_system`; explode only for single-member editing (then
  `transform_structural_members` / `update_structural_member`).
- Never delete and recreate an estimated plasterboard ceiling to bypass the estimate conversion check; show
  `suggestion.reason` and warnings.
- Ids: `systemId`, `memberIds` from `list_structures` (or `create_framing_system`); `spaceId` from `get_floor_detail`;
  `roofAssemblyId` from `create_roof` or `list_structures`. Kind defaults: `kaldrix://projects/framing-system-kinds`;
  roof recipes: `kaldrix://projects/roof-construction-recipes`; member roles:
  `kaldrix://projects/structural-member-roles`.
- Single members: `add_structural_members`; beam on two walls: `add_beam_between_walls`; post under a beam:
  `add_post_under_member` with the beam `memberId` (from `add_beam_between_walls`, or `beamId` from
  `support_purlin_with_beam`). Ceilings, joists, roof framing, timber post frames: `create_framing_system`; generated
  members change only via `update_framing_system` (never add generated rafters one by one). Post-and-girt frame
  (cabin, gazebo, carport) = kind TIMBER_POST_FRAME.
- Roof support plan: read `get_roof_support_plan` (stable purlin, post, wall-plate ids, `modelVersion`) before
  `update_roof_support_plan` or `support_purlin_with_beam`. `PURLIN_BEAM_NO_WALL`: add a post there or draw the beam
  with `add_beam_between_walls`. No wall plates: `params.wallPlates` off via `update_framing_system`.
- `update_framing_member`: ids and `modelVersion` from `get_framing_members` (never preview ids or list indices);
  reread afterwards. All physical timber incl. battens: `get_timber_schedule`; unsaved system:
  `preview_framing_system`. Removed sources show as SOURCE_REMOVED in `review_estimate_against_model`.
- Timber-frame house: `set_technology_build_ups` (or `create_build_up_from_template` + `assign_build_up`) →
  `get_wall_frame` / `get_slab_frame`. Templates: `list_build_ups` `includeTemplates=true` or
  `kaldrix://projects/build-up-templates`. Openings via `add_opening_by_intent`; the wall frame follows. Joist section,
  spacing, direction: `create_framing_system` for the floor or a room.
- Construction systems: show stairs/chimney results of `preview_construction_system` → `create_construction_system`
  with its `modelVersion`. Managed kinds (footings, ring beams, lintels): `update_construction_system`; `systemId`
  from `list_construction_systems`.
- Foundations: wall ids and footing bands from `get_foundation` (or `add_foundation_wall` result); lowest-storey wall
  ids from `get_floor_detail`. Floor validation reports `FOUNDATION_OUTSIDE_WALLS`. Ground floor build-up templates:
  `list_build_ups`.

## Installations

- Point sets: `add_installation_point_preset` (one room) or `add_installation_point_presets_for_floor` (every room);
  adjust with `update_installation_point` / `remove_installation_points`. Default mount/height per kind:
  `kaldrix://projects/installation-point-kinds`. Trade guide intents: PLACE_POINT = `add_installation_points`,
  PROPOSE_CIRCUITS = `suggest_installation_circuits`.
- Add source points (DISTRIBUTION_BOARD, WATER_METER, WATER_MANIFOLD, DRAIN_STACK, HEAT_SOURCE, UFH_MANIFOLD,
  GAS_METER…) before `suggest_installation_circuits`. `missingSources`: add a `missingSources.suggestedPointKind`
  point and suggest again; `missingSources.medium`: add that line with `update_installation_riser` or a compatible
  source. Save via `apply_installation_circuit_suggestions` with the returned `modelVersion`, same system filter and
  language.
- Manual circuits: `get_installation_circuit_selection` → its `modelVersion` in `create_installation_circuit` /
  `update_installation_circuit`; point ids from `list_installation_points`; `spec` from
  `get_installation_conduit_catalog` `allowedSpecs`. Whole floor: prefer suggest + apply. Adding points to an existing
  circuit: selection with `editingCircuitId`. Shared junctions and 3D edges: `update_installation_circuit`
  `nodeEdit`, previewed with `preview_installation_circuit_node` at the same `modelVersion`. One branch:
  `routeOverrides`. `explode_installation_circuit` makes plain lines and loses parametric behaviour. Summaries and ids:
  `list_installation_circuits`; geometry: `get_installation_circuit_layout`. Examples:
  `kaldrix://projects/installation-authoring`, `kaldrix://projects/structures-and-installations`.
- `get_installation_trade_guide` and `plan_floor_installations` take `expectedModelVersion` from `get_project_detail`.
- Risers: multi-floor points from `create_installation_riser`; route each floor to its riser points; change
  connection points with `update_installation_riser`.
- Ventilation: `suggest_ventilation` → `apply_ventilation_suggestions` with its `modelVersion` (diffusers) → add
  missing MVHR_UNIT / AIR_DISTRIBUTION_BOX / AIR_INTAKE / AIR_EXHAUST → circuit suggest + apply with
  `system=VENTILATION`. Duct lengths/warnings: `get_ventilation_route_report` with `list_installation_circuits` and
  `get_installation_circuit_layout`. Whole-project balance is in `suggest_ventilation`, not the room report.
- Underfloor heating: `preview_underfloor_heating` → `create_underfloor_heating` with the identical request (source
  UFH_MANIFOLD; never paired heating circuits). Rooms with loops: `preview_underfloor_heating_update` →
  `update_underfloor_heating`, or `reduce_underfloor_heating` (confirm=true) when the plan has `removedCircuitIds`.
  `pipeProductId` from `get_underfloor_pipe_products`.

## Scene and underlays

- Read `get_project_scene` before any scene write; its `sceneVersion` is `expectedSceneVersion` for
  `place_graphic_model`, `update_placed_model`, `remove_placed_model`, surface textures and wall texture regions.
  Reread after `SCENE_VERSION_CONFLICT`. Asset ids: `list_graphic_assets` (MODEL_3D / TEXTURE).
- Underlays: read `get_project_underlays` (no GRAPHICS module needed) or the scene first; viewpoints:
  `get_project_scene_viewpoints`. Details: `kaldrix://projects/scene-graphics`.

## Roofs, quick house and site

- `create_quick_house` does not divide rooms: continue with `create_partition`, `label_room`, openings via
  `add_opening_by_intent` (styles, sizes, ranges from `get_opening_catalog`), then `review_estimate_against_model`.
- `create_roof`: building or floor outline first; editable starting parameters in
  `kaldrix://projects/roof-construction-recipes`. Framing: `preview_framing_system` kind ROOF_RAFTERS with the
  returned `roofAssemblyId` → `create_framing_system`; keep it parametric for adjustment in the app.
- Change a roof with `update_roof`, not `remove_roof` + `create_roof` (deletes its framing). `validate_project_geometry`
  reports a covering below its minimum pitch (BIM_ROOF_PITCH_BELOW_COVERING_MINIMUM). `roofAssemblyId`: `create_roof`,
  `update_roof` or `list_structures` (`roofAssemblies[].id`).
- Site location: a point chosen by the user or returned by `find_land_parcel` / `get_land_parcel_at`; never infer
  coordinates from an incomplete address. After `set_project_location` read `get_project_detail`, then
  `get_land_parcel_at` for the saved point. Parcel boundaries are degrees; never send them to outline tools (metres).
  A building site from `list_map_features`: `targetId` is its project; continue with `get_project_detail`.
- Renovation survey: room ids from `list_survey_rooms`.
