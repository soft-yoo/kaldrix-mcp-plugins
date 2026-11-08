## More than one building

Repeat building metadata, footprint and floor creation inside the same project for a garage or another house. Keep the intended `buildingId` explicit. Each roof uses the highest floor of its own building. Building-local risers and structural supports do not link independent buildings. Site geometry remains outside their floor collections.

When shared terrain is requested, call `create_site_floor` once for the project, without a building UUID or
outline. Send a `name`, explicit `level` (usually 0), `height` in metres (for example 2), current `expectedModelVersion`
and `idempotencyKey`. Optional `elevation` is the absolute terrain elevation in metres; when omitted the
backend uses the ground-floor elevation, or project datum if there is no ground floor. `level` is a logical
index, not an elevation. The backend rejects a second SITE. An identical retry returns the saved floor UUID
and version; do not change the input or reviewed version under the same key. Creation leaves building
storeys, roofs and project parts unchanged; it creates no plot outline, paving, fence or inter-building
connection. Read `get_floor_detail` / `get_project_detail` afterwards; use `update_floor` for its parameters.
`set_floor_outline` applies to building storeys and cannot define a site boundary.

For `get_project_purchase_report` and `get_timber_schedule`, send the building UUID from
`get_project_detail` to report only that building. Omit `buildingId` for the whole project,
including site geometry. Both responses echo `buildingId` (`null` for the whole project); a UUID
not belonging to this project is rejected with `BUILDING_NOT_FOUND`. Floor names and levels are
not identifiers: two buildings may both have a ground floor named `Parter` at level 0.

Compare raw quantities in the same unit. Each timber cut list is optimized for its own scope;
whole-project purchased bars or rounded row groups need not equal the sum of separate building
reports. The model purchase report reads the current model, not frozen estimate positions. Use
`get_estimate_purchase_resources` for resources from a specific saved estimate.

### Removing one building

Read `get_project_detail`, identify the exact building UUID and review its floors, installations,
saved views and estimate scope. Only after explicit confirmation call `remove_building` with
`confirm=true`, `projects.delete`, the reviewed positive `expectedModelVersion` and a stable
`idempotencyKey`. This permanently removes that building and its owned model elements and
building/floor views. Its roof is not transferred; other buildings, project terrain and project
views remain. Retained building estimates, including inactive ones, and active underfloor or
framing estimate references block removal. Estimates, frozen rows, PDFs and project documents
are preserved. While removal is coordinated, estimate writes can be temporarily refused.

After a timeout or unknown outcome, read the project and retry the exact same intent and key.
Changed building or reviewed version requires a newly reviewed intent. Never clear estimate
scope or recreate a deleted building UUID to bypass a refusal. Read `get_project_detail` after
success and verify both the removed scope and the preserved building/site before continuing.
