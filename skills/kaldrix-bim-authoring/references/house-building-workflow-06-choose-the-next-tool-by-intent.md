## Choose the next tool by intent

| User intent | Tool sequence and IDs |
| --- | --- |
| Change width or length proportionally | `set_building_outline` with `width` and `depth` (no outline); `set_floor_outline` with `width` and `depth` for an independent storey. Heights, wall thickness and equipment sizes stay fixed. |
| Change the footprint shape | `set_building_outline` or `set_floor_outline` with polygon points; arbitrary reshaping is not proportional layout scaling. |
| Place another building | Create its metadata with a new UUID; the first outline is placed beside existing buildings with a 10 m gap by default. For exact site coordinates set `placeBesideBuildings=false`, then verify the stored outline. Use `move_building` for later placement. |
| Divide or merge the interior | `create_partition`, `update_wall` or `remove_wall`; read derived room IDs again before `label_room` or room-scoped work. |
| Floor or ceiling structure | `preview_framing_system` then `create_framing_system`; omit both `spaceId` and `outline` for the current whole-storey footprint, use `spaceId` for one room, or explicit `outline` for a custom field. Do not copy the floor polygon when the structure should follow later resizing. |
| Layered wall or slab | `list_build_ups` / `create_build_up_from_template`, then `assign_build_up` to real wall or slab IDs. Read `get_wall_frame` or `get_slab_frame`; avoid pricing the same material again as a framing system. |
| Roof covering and framing | `create_roof` creates roof geometry on the highest floor; `preview_framing_system` / `create_framing_system` with `ROOF_RAFTERS` and its `roofAssemblyId` creates framing separately. |
| Connected installations | Read `kaldrix://projects/installation-authoring`; place equipment/endpoints, read compatible conduit media and source/target selection, preview and save circuits. A standalone route is a different element. |
| Room underfloor heating | An existing `UFH_MANIFOLD` and current room IDs are required. Preview with `preview_underfloor_heating`, inspect loops and warnings, then use `create_underfloor_heating`; edit existing loops with `update_underfloor_heating`. |
| Price this building | `create_quick_house` already prices a new house. For an existing model: `create_project_estimate` with its `buildingId`, then `price_estimate_from_model`. Read `kaldrix://estimation/service-flow`. |
