## Project Estimate Flow (full catalog tools)

The steps below use lower-level tools from the same `workspace` catalog. For a simple position prefer
`quick_add_estimate_position`, the scope flow, `update_estimate_sheet` and `remove_estimate_position`.

For an RMS estimate choose `kind=RESOURCES` when creating it. Read `kind` and `resourceMarkups` on the estimate.
Only LABOR, MATERIAL and EQUIPMENT rows are allowed; services are refused with `ESTIMATE_KIND_SERVICES_NOT_ALLOWED`.
For a RESOURCES row send the net **purchase** price as `purchasePriceNet`; the server applies the resource markup and
returns the sale price as `unitPriceNet`. Do not add the markup yourself. Outside RESOURCES, `unitPriceNet` is the sale
price. Change estimate markup rules with `update_project_estimate` after user confirmation. Purchase prices and
resource markups are hidden without access to costs; never infer them from the sale price.

1. Call `list_project_estimates` with the project UUID.
2. Create an empty estimate with `create_project_estimate` when needed.
3. List its frozen rows with `list_estimation_items`. Both this tool and `list_estimate_sheet_rows` default to 20 compact rows. Follow `hasMore`/`nextCursor` to cover the required positions; totals cover the whole estimate. Set `includeComponents=true` for composition and `includeDetails=true` for formulas, notes and provenance. Check `componentsIncluded`/`detailsIncluded`: an omitted list is not proof of missing composition.
4. Add a service with `add_catalog_service_to_estimate`. Either send `serviceQuantity` for one manual scope, or send
   `scopes` with one entry per project element or saved measurement. A project-linked scope carries `floorId`, the
   element (`elementType` + `elementId`) or `measurementId`, the raw `sourceQuantity` with its `sourceUnit`, the
   `quantityMultiplier` and computed `serviceQuantity`. Select references from `list_estimate_quantity_sources`,
   call `plan_estimate_position_quantity`, then copy the returned scopes unchanged: navigation
   tools round quantities for display and do not expose every metric, while the server re-reads the model and rejects
   a quantity that differs from it by more than 0.0001. One `SERVICE` row is frozen per scope
   and the estimate remembers the model version it was calculated from. The same service may be added repeatedly;
   each call creates a separate occurrence.
5. Use `create_estimation_item` only for a row outside the price list. Supply its quantity before waste, unit price,
   and VAT; the saved quantity is `quantity × (1 + wastePercent / 100)`.
6. Review the replacement with the user, then correct a draft row with `update_estimation_item`, `confirm=true`,
   `estimates.write` and `estimates.delete`. `quantity` is always the amount before waste, like on create.
   A service row never changes its name or unit. A manual service row that is the only scope of its occurrence may
   change price, VAT, quantity (its composition is rescaled), and note; any other service row accepts only a note.
   To change the quantity or the sources of a service that has several scopes, resend the whole occurrence with
   `replace_estimate_position` — all of its scopes share one unit price and one one-off cost.
7. After a catalog price change, apply new prices to a DRAFT estimate with `refresh_estimation_item_prices`. Prices
   typed by hand in the estimate stay untouched unless `includeOverridden` is true (that overwrites them and needs
   `confirm=true` after the user agrees). A row counts as hand-priced
   (`priceOverridden`) only after a manual price, composition or markup change on that row, the application's
   "apply labor rate to hourly labor" estimate setting (only rows whose labor price it actually changed), or a
   component swapped for another resource when adding the service. Quantity and VAT are not prices: a quantity change,
   mandatory `componentQuantities` for `MANUAL` components and estimate-level markup rules do not mark it, and a VAT
   rate chosen in the estimate (row edit or estimate-wide VAT) is kept when prices are refreshed. Such rows are
   refreshed, and estimate markups are re-applied to the new composition price (`catalogUnitPriceNet` already
   includes them). The application's "Przelicz z całego składu" (a sheet edit with only
   `priceMode: "COMPOSITION"`) recalculates the row from its own composition and markups. It clears the hand-priced
   state only when the price list would price the row the same way: every markup inherited from the estimate or at
   its default (zero purchase/indirect costs, profit = the service margin) and the composition unchanged from the
   service (same components and resources, quantities from the service norms). A row with its own profit or an
   edited composition stays hand-priced, so a refresh never replaces its own settings.
8. Move the estimate through its lifecycle with `update_project_estimate` (omitted fields stay unchanged; every call
   needs `confirm=true` after the user approves, approval also freezes the estimate) and point at the binding one with
   `set_primary_project_estimate`. The same update tool can set an internal contingency reserve and ordered purchase,
   indirect-cost, and profit rules. Read `kaldrix://estimation/markup-rules` before replacing those rules. VAT is
   stored per row (`vatRate`); gross = net × (1 + VAT) is calculated by the server.
9. Remove a position with `remove_estimate_position` (any row id or the `serviceOccurrenceId`): a service goes with all
   of its scopes, an own position as one row. It requires explicit confirmation.

The server calculates the composition, the price, and the model version from the sources it is given. It rejects a
source that does not exist in the current project, does not match the recipe rule, or whose quantity differs from the
current model. In an editable draft the backend corrects model-measured quantities by itself after each model change
and records "was → is" entries (`get_estimate_model_changes`); prices stay frozen. Approved and offered estimates stay
frozen entirely; compare the saved and the current model version and ask the user before recalculating. Details:
"The estimate follows the model".

`review_estimate_against_model` answers which rows still do not match the project (in a draft, usually only
removed and new elements remain). It reports every project-linked row whose element changed its quantity
(`QUANTITY_CHANGED`) or disappeared (`SOURCE_REMOVED`), rows whose service is gone from the price list
(`SERVICE_REMOVED`), the net amount those differences would add or
remove, and the rooms no row points at yet — a room counts as priced when a row points at it or at one of its
walls, slabs, or openings. Each row is measured with the quantity rule it was frozen with (a saved measurement in its
frozen `sourceUnit`), even if the service's rule or unit has changed since. A row whose element has no data for its
stored rule is reported as `QUANTITY_RULE_UNKNOWN` instead of being compared in another measure;
`apply_estimate_model_changes` skips it, so re-add the service or correct its scopes.
It also lists `newElements`: positions measured from installations, installation points, structural members or
framing systems for which the model has matching elements that are not their scopes yet (e.g. a socket drawn after
pricing). New automatic stage positions search the estimate's allowed building/project scope, including ordinary
walls, slabs and rooms qualified by their original stage and work section. Older and manually scoped positions retain
the existing floor heuristic and conservative source types. A manual scope replacement or assignment, even when only
removing a scope, ends automatic stage coverage for that position. The review itself writes nothing and never refreshes
frozen amounts. After the user
approves, call `apply_estimate_model_changes`: omit both lists to apply everything (changed rows and new elements),
or set `itemIds` (changed rows) and/or `addNewElementsTo` (`serviceOccurrenceId` of positions to extend) — a list
that is omitted while the other is given means "none of these". It keeps each row's multiplier and id, adds each new
element as its own scope and reprices the whole service occurrence. When a new element deliberately does not belong
to a position (e.g. a socket priced in another position), ask the user and call `skip_estimate_new_elements`
(`positionId` = `serviceOccurrenceId`, `elementIds` from that position's `newElements`): amounts stay unchanged, the
review stops listing the element and counts it in `skippedNewElementCount`; adding it to the position later with
`replace_estimate_position` clears the skip. For rows whose source is gone, ask the user: keep them with a manual
quantity (`detach_estimate_items_from_model`) or remove them (`remove_estimate_items_with_missing_source`,
`confirm=true`); change sources with `assign_estimate_position_scope` (replace), and price a
missing room with `quick_add_estimate_position` + `assign_estimate_position_scope`. Always show the user the difference and its amount first.

The review also lists `doubleCounted` (`doubleCountedCount`): groups of positions that take the same quantity from the
same model element (same measure, e.g. `FRAMING_ROLE_VOLUME` + `RAFTER`) and share a material or component — typically
a whole-system service ("Więźba dachowa — całość (zamiast osobnych pozycji)") next to per-role services ("Krokwie —
drewno C24", "Murłata"). It is a separate finding: it does not change `changedCount` and `apply_estimate_model_changes`
does not touch it. Show the group (positions, `overlapQuantity`, `overlapNet`, `sharedResources`) and follow
`suggestedAction`: `REMOVE_ONE` — remove `coveringServiceOccurrenceId` or the others with
`remove_estimate_position` (user's approval); `NARROW_FILTER` — narrow `elementSubtypes` of one service
(`update_service_recipe`) or drop a scope (`replace_estimate_position`). When the user says both positions are
intended, call `set_estimate_double_count_accepted` (`accepted=true`; `false` restores the warning).
