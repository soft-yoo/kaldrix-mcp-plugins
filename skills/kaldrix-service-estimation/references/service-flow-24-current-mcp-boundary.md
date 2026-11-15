## Current MCP boundary

MCP covers catalog resources, service recipes, estimate creation and copying, service positions, sections,
model review and quantity refresh, catalog price refresh, estimate-level markups and PDF export. Use
`update_estimate_sheet` (1 to 200 positions in one atomic call, `confirm=true`) for multi-position edits,
composition, executed quantities, KNR codes, row order and extra sheet columns. `update_estimation_item` and
`update_estimate_sheet` change only the fields declared in their schemas; do not invent extra arguments.

### Services for a selected BIM element

After `list_service_recipes`, `list_element_service_quantities(projectId, elementId, recipeIds)` finds which
services accept the selected element (at most 200 distinct recipe IDs per call). The backend calculates the
drawing once and returns exact scopes for every service from the same `modelVersion`. An empty `sources` list
means no positive accepted quantity for this element. This read-only lookup creates no estimate or position.

A whole installation riser uses `elementType: INSTALLATION_RISER` and `floorId: null`. Its CO length already
includes supply, return and the route allowance; WLZ has its own cable allowance. Send the returned scopes
unchanged to `add_catalog_service_to_estimate`. Choose whole-riser scopes or its floor-segment scopes; both
duplicate the same physical quantities. The model review detects that overlap between positions. Within one
position, `preview_estimate_position_scope` marks overlapping physical quantities as not ready; assignment and
direct position writes reject them with `ESTIMATE_SCOPE_OVERLAP`. Choose the whole element or its parts.
