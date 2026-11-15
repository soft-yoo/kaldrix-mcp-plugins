## Catalog Flow

`update_catalog_resource` replaces provided values, keeps omitted fields and clears fields only through
explicit `clearFields`. Review the replacement with the user, then use `confirm=true` with `catalog.write`
and `catalog.delete`.

1. `list_price_lists` (default first) and `get_price_list` (name, stored currency, saved EUR rate, default
   service-composition markup, catalog version, status). Send the chosen `priceListId` to
   `list_catalog_resources`, `create_catalog_resource`, `list_service_recipes`, `create_service_recipe` and
   `create_project_estimate`; catalog tools without it use the default price list. A new estimate resolves an
   active list matching its project currency; an explicit list must match too.
   - No price list yet: start from a ready demo catalog — `list_demo_price_lists`, then `copy_price_list`
     with `source=DEMO`.
   - Next year's prices: `copy_price_list` (`source=OWN`, the default), change prices in the copy
     (`adjust_price_list_prices` raises or lowers RMS prices by a percent, e.g. labour +8 %),
     `set_default_price_list`, then move draft estimates with `change_estimate_price_list` (rows are matched
     to the same services and resources of the copy). `compare_price_lists` shows what changed between two lists.
   - Bulk load: `import_price_list` (resources by SKU, then services with composition by SKU).
     `copy_service_to_price_list` copies one service.
   - `archive_price_list` deactivates an old list; it is locked: no edits, and its estimates cannot add
     services or refresh prices until moved to an active list. `delete_price_list` is refused for the default
     price list or one used by any estimate.
   - `add_service_labor_activity` adds a priced labour activity to a service composition.
   - `install_construction_starter_pack` adds ready construction services (installation points and circuits,
     suspended ceilings, joists, roof framing and battens, post frames, ring beams, build-up layers) to a price
     list. It is idempotent: existing names, prices and compositions stay; only missing scope values are completed.
2. `list_catalog_resources`: reuse returned resource IDs; filter by `kind` for materials, labor or equipment.
   Send `nextCursor` as `cursor` while `hasMore` is true until all required results are read.
3. `create_catalog_resource` for a missing material, labor rate or equipment rate (no `sku` or `category`
   needed). `priceNet` is in the stored currency of the target price list; no conversion is performed. A
   material bought in bags, buckets, packs or boards (unit `PACK` or `PCS`) may get `packageContent` +
   `packageContentUnit` (e.g. `25` + `kg` for a 25 kg bag, `1.44` + `m2` for a pack of panels). It is
   information only: the estimate never converts units (for 36 kg tell the builder `ceil(36 / 25) = 2` full
   bags; see "Units and packages"). `update_catalog_resource` keeps omitted package fields;
   `clearFields=["packageContent"]` clears both.
4. `list_service_recipes` before creating a duplicate recipe. Read `quantityRule`, billing `unit`,
   `description`, `quantitySources` and component units. A `MANUAL` service needs a user-supplied quantity
   and is not added by `price_estimate_from_model`; its name alone does not imply a model binding.
5. `create_service_recipe`: billing `unit`, `vatRate`, `components` as a typed array (each with a valid
   `resourceId` from the same price list) and optional `markupPercent`. For a simple price per unit add a
   labour activity with `add_service_labor_activity` (it creates a LABOR resource in the service unit and
   appends it). Modes: `PER_SERVICE_UNIT` = consumption multiplied by the service scope; `FIXED` = amount
   added once per service occurrence; `MANUAL` = amount entered while building the estimate.

**Component unit rule.** `PER_SERVICE_UNIT` (a norm per one service unit) only when the resource unit equals
the service unit, e.g. an `M2` tiling labor rate inside an `M2` tiling service. A resource in another unit
(`M3` of concrete or `PACK` of tile adhesive inside an `M2` service) does not follow from the measured
quantity: make it `MANUAL` (typed when the service is added to an estimate) or `FIXED` (a set amount per
occurrence). The server rejects `PER_SERVICE_UNIT` in another unit. Older components saved that way are
returned and priced as `MANUAL` with their stored quantity, so `get_service_recipe` shows the effective
`calculationMode`.

Never guess: material categories `kaldrix://estimate/material-categories`, units `kaldrix://estimate/units`,
VAT values `kaldrix://estimate/vat-rates`, service categories `kaldrix://estimation/catalog-categories`.
