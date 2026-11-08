# Tool order and input sources: estimates, price lists and offers

Tool descriptions say only what each tool does. This page says where inputs come from and which
tool comes next. Live schemas, `nextTools` and `fixTools` in results stay authoritative.

## Catalog, services and price lists

- No price list yet (`get_price_list` fails or `list_price_lists` returns no items): `create_price_list` (omit the currency to use the configured one).
  Copy or convert an existing list with `copy_price_list`: OWN source ids from `list_price_lists`, DEMO ids from
  `list_demo_price_lists`, target currency from `list_price_lists` `currencyOptions`. Then change prices in the copy and
  move estimates with `change_estimate_price_list`. A price list used by an estimate is archived with
  `archive_price_list`, not deleted.
- Resources: check `list_catalog_resources` for duplicates before `create_catalog_resource`; `priceNet` is in the
  currency of `get_price_list`. Material categories: `kaldrix://estimate/material-categories`; units: `kaldrix://estimate/units`.
  After `deactivate_catalog_resource`, review affected services with `list_service_recipes`. Frozen estimate rows keep
  old prices until `refresh_estimation_item_prices`.
- Services: create resources first, check `list_service_recipes` for duplicates, then `create_service_recipe`.
  Categories: `kaldrix://estimation/catalog-categories`; element type, quantity rule and unit combinations and
  `elementSubtypes` filters: `kaldrix://estimation/quantity-rules`.
- `install_construction_starter_pack`, then pick services with `list_service_recipes`. Never add a whole-system service
  and its per-role services to one estimate (`review_estimate_against_model` reports `doubleCounted`).

## Building an estimate

- `create_project_estimate`, then price-list services with `add_catalog_service_to_estimate` and rows outside the price
  list with `create_estimation_item`. The full order is in `kaldrix://estimation/service-flow`.
- Quantity scopes: sources from `list_estimate_quantity_sources` or `get_estimate_scope_proposals`; ready scopes from
  `plan_estimate_position_quantity`, copied unchanged to `add_catalog_service_to_estimate` or
  `replace_estimate_position`. Do not calculate or distribute quantities in the client. `list_element_service_quantities`
  takes `recipeIds` from `list_service_recipes` and riser ids from `list_installation_risers`.
- MANUAL components, their ids and units: `get_service_recipe`. Sections: read `get_project_estimate`, create with
  `set_estimate_sections` (keep existing section ids); `list_estimation_items` returns server-calculated `sectionTotals`.
- A pending position from `quick_add_estimate_position` (`scopePending`): `preview_estimate_position_scope`, show the
  summary, then `assign_estimate_position_scope` with the same input after approval.
- Project-linked or multi-scope services change with `replace_estimate_position`, not `update_estimation_item`.
- Reading rows: `list_estimate_sheet_rows` right before `update_estimate_sheet` (exact row versions). Read-only review,
  filters, VAT, sections, discounts, options and `serviceOccurrenceId` for `replace_estimate_position` /
  `remove_estimate_position`: `list_estimation_items`.
- Discounts: markup after discount is in `list_estimation_items`; position discounts are set with
  `update_estimate_sheet`. Options: `save_estimate_option_groups`, then `set_estimate_position_option`.
- Markup rules and their cost bases: read `kaldrix://estimation/markup-rules` before changing them.

## Estimate and model

- `price_estimate_from_model` prices a model; on ESTIMATE_STARTER_PACK_MISSING run
  `install_construction_starter_pack`. Later quantity changes: an editable draft takes model quantities by itself
  after each model change — report them from `get_estimate_model_changes` and acknowledge them with
  `acknowledge_estimate_model_changes` once the user has seen them. Then `review_estimate_against_model` for the rest:
  `apply_estimate_model_changes` after approval for remaining changes and new elements (re-run the review after a
  rejection; use only `itemId` values from the latest review). SOURCE_REMOVED / QUANTITY_ZERO rows: keep with
  `detach_estimate_items_from_model` or remove with `remove_estimate_items_with_missing_source` (`confirm=true`).
  Accepting reported quantities only: prefer `apply_estimate_model_changes` over `replace_estimate_position`;
  refreshing prices is not for quantity changes. Topic: "The estimate follows the model" in the service flow.
- `skip_estimate_new_elements`: adding a skipped element later with `replace_estimate_position` clears the skip.
- The review summary, coverage gaps and applicability issues match `get_estimate_checks`. VAT_RATES_MIXED /
  VAT_PROJECT_MISMATCH: `apply_project_vat_to_estimate` after confirming that no position needs another rate.
- `set_primary_project_estimate`: `get_project_settlement` shows totals, plan vs actual and alerts for it.

## Offer and documents

- Before `send_estimate_offer` check `get_estimate_offer_workflow` (`canSend`) and `get_estimate_checks`; offer details
  and the view used by the STANDARD PDF variant (`presentationMode` SIMPLE or DETAILED) are set with
  `update_project_estimate`; `generate_estimate_pdf` follows it. `update_project_estimate` status changes do not send
  the offer. The client's answer: `get_estimate_offer_workflow`. After a remark is ACCEPTED (`answer_estimate_offer_remark`),
  change the estimate and send a new version.
- OFFER_NUMBER_PATTERN_MISSING: set `numberPattern` with `update_company_offer_settings`. That tool clears omitted text
  fields, so read `get_company_offer_settings` first and send every field to keep.
- As-built PDF: enter executed quantities with `update_estimate_sheet` (`actualQuantity`) before
  `generate_estimate_as_built_pdf`.
