## Quantity form and ready scopes

`list_estimate_quantity_sources` and `get_estimate_scope_proposals` return a `form`: method availability,
`sourceUnit`, `requiresMultiplier`, open minimum and closed maximum. `modelUnavailableReason` distinguishes an
unconfigured model source, a configured source with no matching elements and a unit without a geometric
source. Never infer these rules from the billing unit in a client. A scoped proposal may lock the form to BIM;
outside that locked workflow the quantity planner still accepts manual input.

Call `plan_estimate_position_quantity` with the selected source references and current model version. Supply
an explicit `measurementMultiplier` when the form requires thickness or height. For a manual quantity send an
empty source list and `manualQuantity` (works in every billing unit). `quantityOverride` is an explicit
corrected total that the backend distributes across the selected sources. An incomplete area-to-volume
conversion returns rawSourceQuantity and no scopes. Copy the ready scopes unchanged to
`add_catalog_service_to_estimate` or `replace_estimate_position`. The plan is read-only, requires ESTIMATES
and estimates.read, and creates no estimate or rows.
