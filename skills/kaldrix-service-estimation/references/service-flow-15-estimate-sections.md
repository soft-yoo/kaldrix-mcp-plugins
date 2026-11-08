### Estimate sections

Like a KNR estimate, a DRAFT estimate can be divided into sections, e.g. `1 Roboty ziemne`, `2 Stan surowy`,
`3 Wykończenie`. `set_estimate_sections` saves the complete ordered list: omit `id` to create a section, send the
existing `id` with a new name to rename it, and order the list to reorder. A section left out is deleted, but its
positions stay in the estimate without a section. Deleting sections is destructive: it needs the `estimates.delete`
scope and `confirm=true`; without it the call is rejected, lists the sections that would be deleted and changes
nothing. Set `sectionId` to `add_catalog_service_to_estimate` or
`create_estimation_item` to place a new position at the end of that section, and move an existing position with
`assign_estimate_position_to_section` (all scopes of a service move together; prices do not change).
`get_project_estimate` returns `sections` with their `number`; positions are numbered `section.position` (e.g. 2.3)
and positions without a section come last. `list_estimation_items` returns rows in that document order with
server-calculated `sectionTotals` — never add section subtotals up yourself.

Use `copy_project_estimate` to create an independent DRAFT copy with new item and service-occurrence IDs. A price
list is managed with `create_price_list`, `copy_price_list`, `archive_price_list` and `delete_price_list`.

Only `DRAFT` estimates accept item changes. The allowed lifecycle is `DRAFT -> PENDING_APPROVAL -> APPROVED`, with
an optional return from `PENDING_APPROVAL` to `DRAFT`. An approved estimate is immutable; copy it to continue work.
Compare `ProjectEstimate.priceListVersion` with `PriceList.catalogVersion` to detect newer catalog content. A catalog
change never overwrites a frozen estimate automatically.
