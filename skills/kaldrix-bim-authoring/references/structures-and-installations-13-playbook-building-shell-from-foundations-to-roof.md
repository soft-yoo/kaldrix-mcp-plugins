## Playbook: Building shell from foundations to roof

Shell systems (footings, ring beams, lintels, sills, plinth, gutters, stairs, railings, chimney) are construction
systems on a floor; the backend computes parts, quantities and warnings at read time. Managed kinds are created and
removed by the backend from the building technology and geometry — never create them, disable them instead.

1. `set_building_construction` — `technology` (MASONRY / TIMBER_FRAME / REINFORCED_CONCRETE), `foundation`
   (STRIP_FOOTING / RAFT / NONE; RAFT gives a managed RAFT_FOUNDATION), `groundLevel`. After the outline exists,
   `set_ground_floor_build_up` puts the matching ground-floor build-up (on grade or on raft) on the lowest floor slabs.
   Foundation outline: by default the footings / raft follow the building outline. `set_foundation_outline` sets an own
   outline (perimeter footings along its edges, raft = outline + overhang; omit the outline argument, rather than sending null, to follow the building outline again).
   Foundation wall: `update_construction_system` on STRIP_FOOTING with `params.footing.wallHeight` (from the footing
   top; default up to level 0) and `wallMaterial` (CONCRETE_BLOCK default, EXPANDED_CLAY_BLOCK — block count in pcs
   plus mortar; MONOLITHIC_CONCRETE — concrete m³ plus two-sided formwork). Summary: `wallCourses`, `blockCount`.
2. Outline, floors and partitions as in `kaldrix://projects/house-building-workflow` (`set_building_outline`,
   `create_building_floor`, openings with `add_opening_by_intent`).
3. `list_construction_systems` per floor — managed systems appear on their own; adjust them with
   `update_construction_system` (`params`, `overrides` per wall, `enabled=false`). Single lintel: `set_opening_lintel`.
4. Stairs / chimney / railings: `preview_construction_system` → `create_construction_system` (keeps `createOpenings`
   true: stairwell in the ceiling and in the floor above, chimney openings through slabs and roof planes). A warning
   with `fix = CUT_OPENINGS` means a stairwell or chimney opening is missing: add it with `add_slab_opening` (kind
   STAIRWELL or CHIMNEY). Railing around a stairwell on the
   upper floor: `placement.host = STAIRWELL`.
   Chimney: `params.chimney.construction` — SYSTEM_CERAMIC (default; block + ceramic pipe + wool), MASONRY_BRICK
   (solid brick; channels `sectionWidth` × `sectionDepth` or a STAINLESS_STEEL liner with `diameter`) or
   STEEL_TWIN_WALL (insulated pipe, smoke / flue-gas flues only — VENTILATION → `CHIMNEY_FLUE_KIND_UNSUPPORTED`);
   `flues` in row order, e.g. `[{kind: SMOKE, diameter: 180}, {kind: VENTILATION}]`. Leave `sizeX` / `sizeY` empty —
   the backend sizes the shaft (system smoke + ventilation 0.50 × 0.36 m; a smaller shaft → `CHIMNEY_TOO_SMALL`).
   Leave `heightAboveRoof` empty for the required minimum (summary `aboveRoofMin`; lower → warning
   `CHIMNEY_BELOW_MINIMUM`). Options: `aboveRoofFinish` CLINKER / PLASTER / NONE, `cap`, `plasterInside`,
   `sweepAccess` (null = on for roofs steeper than 12°), `foundationPad`. The chimney's slab openings belong to it
   (`ownerSystemId`) — move or resize the chimney, not the opening (`SLAB_OPENING_OWNED_BY_SYSTEM`; the same for a stairwell cut by
   stairs — update/remove the stairs, renaming the opening is allowed); its flashing is
   counted once per chimney (part `CHIMNEY_FLASHING`), not per roof opening.
5. Attic: `set_floor_roof_space` (USABLE / NON_USABLE, `kneeWallHeight`) **before** `create_roof`; changing it later
   rebuilds a roof created from the outline (a hand-drawn roof is refused with `ROOF_SPACE_ROOF_EXISTS`).
6. `create_roof` on the highest floor; a lower floor may get its own roof over the part not covered by a higher storey.
7. Roof framing (`preview_framing_system` → `create_framing_system` `ROOF_RAFTERS`). Warning
   `framing.warning.roofSupportHeights` → `fit_roof_to_wall_plates` (rebuilds the roof onto the wall crown; `projects.write`, `projects.delete` and user-approved `confirm=true`).
8. `validate_project_geometry` — `CONSTRUCTION_*` warnings (e.g. `CONSTRUCTION_STOREYS_WITHOUT_STAIRS`).
9. Estimate: `install_construction_starter_pack`, services with a part role (`CONSTRUCTION_*` rules), then
   `review_estimate_against_model` → `uncoveredShellSystems` lists shell systems without a position.
