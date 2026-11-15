## Choose the next tool by intent

| User intent | Tool sequence and IDs |
| --- | --- |
| Change width or length proportionally | `set_building_outline` with `width` and `depth` (no outline); `set_floor_outline` with `width` and `depth` for an independent storey. Heights, wall thickness and equipment sizes stay fixed. |
| Change the footprint shape | `set_building_outline` or `set_floor_outline` with polygon points; reshaping is not proportional scaling. |
| Place another building | Metadata with a new UUID; its first outline goes beside existing buildings with a 10 m gap by default. For exact site coordinates set `placeBesideBuildings=false` and verify the stored outline. Later placement: `move_building`. |
| Divide or merge the interior | `create_partition`, `update_wall` or `remove_wall`; re-read derived room IDs before `label_room` or room-scoped work. |
| Floor or ceiling structure | `preview_framing_system` then `create_framing_system`; omit `spaceId` and `outline` for the current whole-storey footprint, `spaceId` for one room, explicit `outline` for a custom field. Do not copy the floor polygon when the structure should follow later resizing. |
| Layered wall or slab | `list_build_ups` / `create_build_up_from_template`, then `assign_build_up` to real wall or slab IDs. Read `get_wall_frame` or `get_slab_frame`; do not price the same material again as a framing system. |
| Roof covering and framing | `create_roof` creates roof geometry on the highest floor; framing separately with `preview_framing_system` / `create_framing_system`, `ROOF_RAFTERS` and its `roofAssemblyId`. |
| Connected installations | Read `kaldrix://projects/installation-authoring`: place equipment/endpoints, read compatible conduit media and source/target selection, preview and save circuits. A standalone route is a different element. |
| Room underfloor heating | Needs an existing `UFH_MANIFOLD` and current room IDs. `preview_underfloor_heating`, inspect loops and warnings, then `create_underfloor_heating`; edit loops with `update_underfloor_heating`. |
| Price this building | `create_quick_house` already prices a new house. Existing model: `create_project_estimate` with its `buildingId`, then `price_estimate_from_model`. Read `kaldrix://estimation/service-flow`. |
