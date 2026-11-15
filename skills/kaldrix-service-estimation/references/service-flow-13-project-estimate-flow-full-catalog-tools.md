## Project Estimate Flow (full catalog tools)

Lower-level tools of the same `workspace` catalog. For a simple position prefer `quick_add_estimate_position`,
the scope flow, `update_estimate_sheet` and `remove_estimate_position`.

**RMS estimate** (`kind=RESOURCES`, chosen at creation; read `kind` and `resourceMarkups`): only LABOR, MATERIAL
and EQUIPMENT rows; services are refused with `ESTIMATE_KIND_SERVICES_NOT_ALLOWED`. In `create_estimation_item`,
`update_estimation_item` and `update_estimate_sheet` send the net **purchase** price as `purchasePriceNet`; the
server applies the resource markup and returns the sale price as `unitPriceNet`. Price changes through
`unitPriceNet` are rejected in RESOURCES. Never add the markup yourself or send a sale price as purchase price.
Outside RESOURCES `unitPriceNet` is the sale price; typing it keeps the stored purchase price. Change estimate
markup rules with `update_project_estimate` after user confirmation. Without access to costs, purchase prices
and resource markups are hidden; never infer them from the sale price.

1. `list_project_estimates` (project UUID); `create_project_estimate` for an empty estimate when needed.
2. `list_estimation_items` / `list_estimate_sheet_rows` default to 20 compact rows; follow
   `hasMore`/`nextCursor` to cover required positions (totals cover the whole estimate).
   `includeComponents=true` adds composition, `includeDetails=true` formulas, notes and provenance; check
   `componentsIncluded`/`detailsIncluded` — an omitted list does not prove missing composition.
3. `add_catalog_service_to_estimate`: either `serviceQuantity` (one manual scope) or `scopes`, one per project
   element or saved measurement. A linked scope has `floorId`, the element (`elementType` + `elementId`) or
   `measurementId`, raw `sourceQuantity` + `sourceUnit`, `quantityMultiplier` and computed `serviceQuantity`.
   Pick references in `list_estimate_quantity_sources`, call `plan_estimate_position_quantity` and copy its
   scopes unchanged: navigation tools round for display and hide some metrics, and the server re-reads the
   model and rejects a quantity differing by more than 0.0001. One `SERVICE` row is frozen per scope; the
   estimate remembers the model version. Each call creates a separate occurrence (repeats allowed).
4. `create_estimation_item` only for a row outside the price list: quantity before waste, unit price, VAT;
   saved quantity = `quantity × (1 + wastePercent / 100)`.
5. `update_estimation_item` corrects a draft row (`confirm=true` after reviewing the replacement with the user;
   `estimates.write` + `estimates.delete`). `quantity` is before waste, as on create. A service row never
   changes name or unit. A manual service row that is the only scope of its occurrence may change price, VAT,
   quantity (composition rescaled) and note; other service rows accept only a note. Quantity or sources of a
   multi-scope service: resend the whole occurrence with `replace_estimate_position` (its scopes share one
   unit price and one one-off cost).
6. `refresh_estimation_item_prices` applies changed catalog prices to a DRAFT. Hand-priced rows stay unless
   `includeOverridden` is true (overwrites them; `confirm=true` after the user agrees). A row is hand-priced
   (`priceOverridden`) only after a manual price, composition or markup change on it, the app's "apply labor
   rate to hourly labor" estimate setting (only rows whose labor price it changed), or a component swapped for
   another resource when adding the service. Not prices: quantity changes, mandatory `componentQuantities` for
   `MANUAL` components, estimate-level markup rules; a VAT rate chosen in the estimate (row edit or
   estimate-wide) survives refresh. Refreshed rows get estimate markups re-applied (`catalogUnitPriceNet`
   already includes them). The app's "Przelicz z całego składu" (sheet edit with only
   `priceMode: "COMPOSITION"`) recalculates the row from its own composition and markups; it clears the
   hand-priced state only when the price list would price the row the same way: every markup inherited or at
   default (zero purchase/indirect costs, profit = service margin) and the composition unchanged (same
   components and resources, quantities from service norms). Own profit or an edited composition stays
   hand-priced, so a refresh never replaces its own settings.
7. Lifecycle via `update_project_estimate` (omitted fields unchanged; every call `confirm=true` after user
   approval; approval freezes the estimate); `set_primary_project_estimate` marks the binding one. The update
   tool also sets an internal contingency reserve and ordered purchase, indirect-cost and profit rules. A
   non-empty header list must include `PURCHASE_COSTS`, `INDIRECT_COSTS` and `PROFIT` (zero-percent stages
   too); incomplete lists are rejected, `[]` clears, omitted/null keeps the list. Legacy percent patches only
   if the resulting header is complete. Position-level overrides keep their semantics. Default profit basis:
   labor, materials, equipment, purchase costs, indirect costs. Read `kaldrix://estimation/markup-rules` before
   replacing rules. VAT is per row (`vatRate`); the server calculates gross = net × (1 + VAT).
8. `remove_estimate_position` (any row id or `serviceOccurrenceId`; explicit confirmation): a service with all
   its scopes, an own position as one row.

The server derives composition, price and model version from the given sources and rejects a source missing
from the current project, not matching the recipe rule, or with a quantity differing from the current model.
A draft corrects model-measured quantities itself after each model change and records "was → is" entries
(`get_estimate_model_changes`); prices stay frozen. Approved and offered estimates stay frozen entirely:
compare saved and current model version and ask the user before recalculating ("The estimate follows the model").

### Model review

`review_estimate_against_model` writes nothing and never refreshes frozen amounts. In a draft usually only
removed and new elements or rows affected by a calculation-method update remain. It returns:

- `modelVersion`, `quantityEngineVersion`;
- linked rows with changed quantity (`QUANTITY_CHANGED`; `changeReason` `MODEL_CHANGED` = drawing changed,
  `CALCULATION_CHANGED` + `notice` = Kaldrix's calculation method changed, drawing maybe not) or a vanished
  element (`SOURCE_REMOVED`), rows whose service left the price list (`SERVICE_REMOVED`) and the net amount
  these would add or remove;
- unpriced rooms (priced = a row points at the room or one of its walls, slabs or openings);
- `newElements`: matching installations, installation points, structural members or framing systems not yet
  scopes of positions measured from them (e.g. a socket drawn after pricing). New automatic stage positions
  search the estimate's allowed building/project scope, including ordinary walls, slabs and rooms qualified by
  their original stage and work section; older and manually scoped positions keep the floor heuristic and
  conservative source types. Any manual scope replacement or assignment, even only removing a scope, ends
  automatic stage coverage for that position.

Rows are measured with their frozen quantity rule (saved measurement in its frozen `sourceUnit`), even if the
service's rule or unit changed. No data for the stored rule → `QUANTITY_RULE_UNKNOWN` (never compared in
another measure); `apply_estimate_model_changes` skips it, so re-add the service or correct its scopes.

`apply_estimate_model_changes` (after user approval; show the difference and its amount first) requires
`expectedModelVersion` = review's `modelVersion` and `expectedQuantityEngineVersion` = its
`quantityEngineVersion`; a model edit or calculation-method bump gives `ESTIMATE_MODEL_REVIEW_STALE`. Omit both
lists to apply all (changed rows and new elements), or set `itemIds` (changed rows) and/or `addNewElementsTo`
(`serviceOccurrenceId` of positions to extend); one list omitted while the other is given means "none". It
keeps each row's multiplier and id, adds each new element as its own scope and reprices the whole occurrence.
Removed sources, skipping new elements and re-scoping: "The estimate follows the model". Price a missing room
with `quick_add_estimate_position` + `assign_estimate_position_scope`.

**Double counting.** `doubleCounted` (`doubleCountedCount`) lists position groups taking the same quantity
from the same element (same measure, e.g. `FRAMING_ROLE_VOLUME` + `RAFTER`) and sharing a material or component
— typically a whole-system service ("Więźba dachowa — całość (zamiast osobnych pozycji)") next to per-role
services ("Krokwie — drewno C24", "Murłata"). Separate finding: no effect on `changedCount`, untouched by
`apply_estimate_model_changes`. Show positions, `overlapQuantity`, `overlapNet`, `sharedResources` and follow
`suggestedAction`: `REMOVE_ONE` — remove `coveringServiceOccurrenceId` or the others with
`remove_estimate_position` (user's approval); `NARROW_FILTER` — narrow `elementSubtypes` of one service
(`update_service_recipe`) or drop a scope (`replace_estimate_position`). Both intended:
`set_estimate_double_count_accepted` (`accepted=true`; `false` restores the warning).
