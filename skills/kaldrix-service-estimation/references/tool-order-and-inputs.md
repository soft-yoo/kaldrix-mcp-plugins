# Tool order and input sources: estimates, price lists and offers

Where inputs come from and which tool comes next. Live schemas, `nextTools` and `fixTools` stay authoritative;
details are in the linked topics of `kaldrix://estimation/service-flow`.

## Catalog, services and price lists

- No price list (`get_price_list` fails or `list_price_lists` returns no items): `create_price_list` (omit
  currency to use the configured one). `copy_price_list` copies or converts: OWN ids from `list_price_lists`,
  DEMO ids from `list_demo_price_lists`, target currency from `list_price_lists` `currencyOptions`; change
  prices in the copy, move estimates with `change_estimate_price_list`. Archive a list used by an estimate with
  `archive_price_list` instead of deleting it.
- Resources: check `list_catalog_resources` for duplicates before `create_catalog_resource`; `priceNet` is in
  the `get_price_list` currency. Material categories `kaldrix://estimate/material-categories`, units
  `kaldrix://estimate/units`. After `deactivate_catalog_resource` review affected services with
  `list_service_recipes`; frozen rows keep old prices until `refresh_estimation_item_prices`.
- Services: resources first, `list_service_recipes` for duplicates, then `create_service_recipe`. Categories
  `kaldrix://estimation/catalog-categories`; element type/quantity rule/unit combinations and `elementSubtypes`
  filters `kaldrix://estimation/quantity-rules`.
- `install_construction_starter_pack`, then pick services with `list_service_recipes`. Never add a whole-system
  service and its per-role services to one estimate (`review_estimate_against_model` reports `doubleCounted`).

## Building an estimate

- `create_project_estimate`, then `add_catalog_service_to_estimate` (price-list services) and
  `create_estimation_item` (rows outside the price list).
- Scopes: sources from `list_estimate_quantity_sources` or `get_estimate_scope_proposals`; ready scopes from
  `plan_estimate_position_quantity`, copied unchanged to `add_catalog_service_to_estimate` or
  `replace_estimate_position`; never calculate or distribute quantities in the client.
  `list_element_service_quantities` takes `recipeIds` from `list_service_recipes` and riser ids from
  `list_installation_risers`.
- MANUAL components, ids and units: `get_service_recipe`. Sections: read `get_project_estimate`, write with
  `set_estimate_sections` (keep existing section ids); `list_estimation_items` returns `sectionTotals`.
- `quick_add_estimate_position` with `scopePending`: `preview_estimate_position_scope`, show the summary, then
  `assign_estimate_position_scope` with the same input after approval.
- Project-linked or multi-scope services change with `replace_estimate_position`, not `update_estimation_item`.
- `list_estimate_sheet_rows` right before `update_estimate_sheet` (exact row versions). Read-only review,
  filters, VAT, sections, discounts, options and `serviceOccurrenceId` for `replace_estimate_position` /
  `remove_estimate_position`: `list_estimation_items`.
- RMS (`kind=RESOURCES`): net purchase price as `purchasePriceNet`, also in `update_estimate_sheet`; returned
  `unitPriceNet` is the sale price and is rejected as price input; never add the markup yourself.
- Discounts: markup after discount in `list_estimation_items`; position discounts via `update_estimate_sheet`.
  Options: `save_estimate_option_groups`, then `set_estimate_position_option`.
- Markup rules: read `kaldrix://estimation/markup-rules` first. Header: all `PURCHASE_COSTS`, `INDIRECT_COSTS`
  and `PROFIT` stages (zero-percent too) or `[]` to clear; omitted/null keeps the list; incomplete lists and
  legacy percent patches leaving an incomplete header are rejected. Position-level overrides keep their rules.

## Estimate and model

- `price_estimate_from_model`; on ESTIMATE_STARTER_PACK_MISSING run `install_construction_starter_pack`.
- An editable draft takes model quantities itself: report `get_estimate_model_changes`, then
  `acknowledge_estimate_model_changes` once seen. The rest via `review_estimate_against_model`:
  `apply_estimate_model_changes` after approval (only `itemId` values and required `expectedModelVersion` =
  `modelVersion` of the latest review; re-run the review after a rejection). SOURCE_REMOVED / QUANTITY_ZERO:
  `detach_estimate_items_from_model` or `remove_estimate_items_with_missing_source` (`confirm=true`). For
  reported quantities prefer `apply_estimate_model_changes` over `replace_estimate_position`; price refresh is
  not for quantities. Topic: "The estimate follows the model".
- `skip_estimate_new_elements`; adding a skipped element later with `replace_estimate_position` clears the skip.
- Review summary, coverage gaps and applicability issues match `get_estimate_checks`.
- `set_primary_project_estimate`: `get_project_settlement` shows its totals, plan vs actual and alerts.

## Offer and documents

- Before sending, approval (`update_project_estimate` status APPROVED) or `generate_estimate_pdf`: read
  `review_estimate_against_model`, resolve `blockingPublicationItemIds` (warning consent cannot bypass them);
  for `hasPublicationWarnings` explain the findings and unacknowledged automatic quantity changes, then send
  `confirmModelWarnings=true` only after the user agrees, keeping
  `modelVersion` as `expectedModelVersion`; on `ESTIMATE_MODEL_REVIEW_STALE` explain the new review and get a
  fresh decision. Refusal codes: "Before sending, approving or generating a PDF" in the model-follow reference.
- Before `send_estimate_offer` check `get_estimate_offer_workflow` (`canSend`) and `get_estimate_checks`. Offer
  details and the STANDARD PDF view (`presentationMode` SIMPLE or DETAILED) are set with
  `update_project_estimate`, whose status changes never send the offer; `generate_estimate_pdf` follows the
  view. Client answer: `get_estimate_offer_workflow`. After a remark is ACCEPTED
  (`answer_estimate_offer_remark`), change the estimate and send a new version.
- OFFER_NUMBER_PATTERN_MISSING: set `numberPattern` with `update_company_offer_settings`; it clears omitted text
  fields, so read `get_company_offer_settings` first and send every field to keep.
- As-built PDF: executed quantities via `update_estimate_sheet` (`actualQuantity`), then
  `generate_estimate_as_built_pdf`.
- Excel: `export_estimate_excel` (`confirm=true`, needs `price-list.read`) stores the app's Excel-button XLSX
  with the project documents, replacing the previous export. Import: upload a project document
  (`upload_document` or `request_document_upload` + `complete_document_upload`), send its id as `documentId` to
  `import_estimate_rows` (`dryRun=true` first, `sheetName` for another sheet); same project only, reading needs
  `documents.read`, an edited Kaldrix export loads back without a mapping.
