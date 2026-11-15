## Building Scope

Read `get_project_detail` and select a building UUID. Set `buildingId` in `create_project_estimate`; omit
it only to price the whole project (including site floors). The estimate returns its fixed `buildingId`;
copying preserves it. Send this `estimateId` to `get_estimate_scope_proposals`,
`list_estimate_quantity_sources`, `list_element_service_quantities` and `plan_estimate_position_quantity`:
they use only the selected building's floors, roof, envelope walls and installation risers. Position writes
and model review enforce the same scope. Never substitute a whole-project source for a building source.

Building authoring: project → `save_project_building` when another building is needed →
`set_building_outline` → `create_building_floor` → `create_partition` → read derived rooms and
`label_room`. Refresh `modelVersion` after every change.
