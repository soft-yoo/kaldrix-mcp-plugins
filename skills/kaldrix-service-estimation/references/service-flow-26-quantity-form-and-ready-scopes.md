## Quantity form and ready scopes

Both `list_estimate_quantity_sources` and `get_estimate_scope_proposals` return a `form`.
Use its method availability, `sourceUnit`, `requiresMultiplier`, open minimum and closed maximum.
`modelUnavailableReason` distinguishes an unconfigured model source from a configured source
with no matching elements and from a unit that has no geometric source. Never infer these rules
from the billing unit in a client. A scoped proposal may lock the form to BIM; manual input remains
supported by the quantity planner outside that locked workflow.

Call `plan_estimate_position_quantity` with the selected source references and the current model
version. Supply an explicit `measurementMultiplier` when the form requires thickness or height.
For manual quantity, send an empty source list and `manualQuantity`; this works in every billing unit.
`quantityOverride` is an explicit corrected total: the backend distributes it across the selected
sources. An incomplete area-to-volume conversion returns rawSourceQuantity and no scopes.
Copy the returned ready scopes unchanged to `add_catalog_service_to_estimate` or
`replace_estimate_position`. This plan is read-only, requires ESTIMATES and estimates.read, and does
not create an estimate or persist rows.
