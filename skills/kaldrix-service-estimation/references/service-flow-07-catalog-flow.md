## Catalog Flow

`update_catalog_resource` replaces provided values, preserves omitted fields and clears fields only through explicit `clearFields`. Review the replacement with the user, then use `confirm=true` with `catalog.write` and `catalog.delete`.

1. Call `list_price_lists` to see the company price lists (the default first) and `get_price_list` for one of them
   (name, stored currency, saved EUR rate, default service-composition markup, catalog version, status). Send the chosen `priceListId` to
   `list_catalog_resources`, `create_catalog_resource`, `list_service_recipes`, `create_service_recipe` and
   `create_project_estimate`; catalog tools without it use the default price list. A new estimate resolves an active
   list matching its project currency; an explicit list must match too. A company without a price list can start
   from a ready-made demo catalog: `list_demo_price_lists`, then `copy_price_list` with `source=DEMO`. To prepare next
   year's prices use `copy_price_list` (`source=OWN`, the default), change prices in the copy (`adjust_price_list_prices` raises or lowers RMS prices by a
   percent, e.g. labour +8 %), `set_default_price_list`, and move draft estimates with
   `change_estimate_price_list` (rows are matched to the same services and resources of the copy).
   `compare_price_lists` shows what changed between two lists. To load many items at once use `import_price_list`
   (resources by SKU, then services with composition by SKU). `copy_service_to_price_list` copies one service.
   Deactivate an old price list with `archive_price_list` (an inactive list is locked: no edits, and estimates on it
   cannot add services or refresh prices until they move to an active list); `delete_price_list` is refused for the default price list or one used by any
   estimate. `add_service_labor_activity` adds a priced labour activity to a service composition.
   `install_construction_starter_pack` adds the ready construction services (installation points and circuits,
   suspended ceilings, joists, roof framing and battens, post frames, ring beams, build-up layers) to a price list;
   it is idempotent: existing names, prices and compositions stay; only missing scope values are completed.
2. Call `list_catalog_resources` and reuse returned resource IDs. Filter by `kind` when you specifically need
   materials, labor, or equipment. Send `nextCursor` as `cursor` while `hasMore` is true until all
   required results have been read.
3. Create a missing material, labor rate, or equipment rate with `create_catalog_resource`. Enter `priceNet` in
   the stored currency of its target price list; no conversion is performed.
   For a material bought in bags, buckets, packs or boards (unit `PACK` or `PCS`) you may set `packageContent` and
   `packageContentUnit` (e.g. `25` + `kg` for a 25 kg bag, `1.44` + `m2` for a pack of panels). It is information only:
   the estimate never converts units, so when a builder needs 36 kg you can tell them it is `ceil(36 / 25) = 2` full
   bags. `update_catalog_resource` preserves omitted package fields; `clearFields=["packageContent"]` clears both.
4. Call `list_service_recipes` before creating a duplicate recipe. Read the returned `quantityRule`, billing `unit`,
   `description`, `quantitySources` and component units. A `MANUAL` service requires a user-supplied quantity and
   is not added by `price_estimate_from_model`; its name alone does not imply a model binding.
5. Create a service with `create_service_recipe`. Supply its billing `unit`, `vatRate`, `components` as a typed
   array (every component needs a valid `resourceId` from the same price list) and an optional `markupPercent`. To
   give a service a simple price per unit, add a labour activity with `add_service_labor_activity` (it creates a
   LABOR resource in the service unit and appends it). Use `PER_SERVICE_UNIT` for consumption multiplied by the service scope, `FIXED` for an
   amount added once per service occurrence, or `MANUAL` when the amount must be entered while building the estimate.
   `create_catalog_resource` no longer needs `sku` or `category`.

**Component unit rule.** `PER_SERVICE_UNIT` (a norm per one service unit) is allowed only when the resource unit
equals the service unit, e.g. an `M2` tiling labor rate inside an `M2` tiling service. A resource in another unit
(`M3` of concrete or `PACK` of tile adhesive inside an `M2` service) does not follow from the measured quantity: give
it `MANUAL` (quantity typed when the service is added to an estimate) or `FIXED` (a set amount per occurrence). The
server rejects `PER_SERVICE_UNIT` in another unit. Older components saved that way are returned and priced as
`MANUAL` with their stored quantity, so `get_service_recipe` already shows the effective `calculationMode`.

Read `kaldrix://estimate/material-categories`, `kaldrix://estimate/units`, and `kaldrix://estimate/vat-rates` instead of
guessing material categories, units, or VAT values. Service categories are listed at
`kaldrix://estimation/catalog-categories`.
