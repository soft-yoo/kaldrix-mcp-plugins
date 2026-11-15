# Tool order and input sources: delivery, documents and company data

Where inputs come from and which tool comes next; model and estimate details are in the BIM and estimation skills. Live schemas, `nextTools` and `fixTools` stay authoritative.

## Projects and company

- `get_projects` → `get_project_detail` (buildings, floors). Workflow: `get_project_workflow` (company-wide: `get_company_workflow`).
- Customers: show `lookup_company_by_nip` results to the user, send confirmed data to `create_customer` / `update_customer`. A customer with history (CUSTOMER_IN_USE) is archived (`archive_customer`), not deleted; `restore_customer` takes ids from `list_customers` with `includeArchived=true`.
- Tasks: `list_tasks` gives `boardId`, `taskId` and valid `columnId` for `create_task`, `update_task`, `delete_task`.
- Budget: `lineId` and `expenseId` from `get_project_budget`.

## Contracts and annexes

- `create_project_contract`: `estimateId` from `list_project_contracts` `preparableEstimates`. Current version for edits, and the client's answer after `send_project_contract`: `get_project_contract`.
- After `return_project_contract_to_draft` and corrections, send again (new link). A signed contract's only way out is `cancel_project_contract`; then `create_project_contract` prepares a new one from the same estimate.
- Wrong tranche payment: `clear_contract_tranche_payment`, then `record_contract_tranche_payment`.
- Annexes: `positionKey` and `sectionId` from `get_contract_annex_sources`; PRICE_LIST `recipeId` from `list_service_recipes` with the sources' `priceListId`; `list_contract_annexes` shows the default VAT rate, whether a new annex is possible, the draft and later the answer. `get_estimate_annex_changes` takes `estimateId` from `list_project_estimates`.
- `contractId` from `list_project_contracts`, `annexId` from `list_contract_annexes`, `remarkId` for `answer_contract_remark` from `list_contract_remarks`; issued agreement ids from `list_issued_agreements`.

## Investor portal

- INVESTOR_PORTAL_LINK_ACTIVE on `create_investor_portal_link`: first `revoke_investor_portal_link` (softer than `delete_investor_portal_link`). `recipientKey` from `get_investor_portal`.
- `reply_investor_site_message`: `subjectId` and `recipientKey` from `list_investor_site_messages` or `get_investor_portal`.

## Realization, acceptance and warranty

- `get_project_realization`: stage source for `start_project_realization`, `completion` before `complete_project_realization` (a signed accepted FINAL protocol unlocks it).
- `sign_acceptance_protocol`: `protocolId` from `get_project_acceptance`; version and `signBlockedBy` from `get_acceptance_protocol`.
- `change_punch_item_status` (punch items and warranty defects): version from `list_punch_items` or `get_punch_item` (`kind=WARRANTY` for warranty reports). Warranty versions: `get_project_warranty`. Formality kind and checklist version: `get_project_formalities`.

## Schedule, crews and site log

- `get_project_schedule`: `stageId` and its version for `set_schedule_stage_progress`. `list_crews`: `crewId`, `crewMemberId` for `assign_crew_to_stage` and `add_time_entry`. Assignment ids/versions: `get_project_crew_assignments` or `get_work_plan`; time entries: `get_project_time_sheet`.
- `list_construction_log_entries`: stages, today's `defaultStageId` for `add_construction_log_entry`, entry ids (`entryId`, `entryIds`).

## Invoices

- `assign_invoice`: references from `list_invoices_to_assign` or `list_invoices`; without `customers.write` the project customer stays unchanged — `set_project_customer` later needs `customers.write` and `confirm=true`. `confirm_invoice_match` confirms a user-accepted suggestion from `get_project_invoicing` (with its `contractId`). `set_invoices_stage`: references from `list_invoices`, stage from `get_project_stage_settlement`; assign invoices to the project first.
- `create_manual_invoice` from a photo or PDF: upload the original first (`upload_document`, or `request_document_upload` with `invoiceAttachment=true`), then send the READY document ids; create a missing SALES customer with `create_customer`. Never invent VAT, dates or parties.

## Documents, graphics and materials

- Larger documents: `request_document_upload` → HTTP PUT → `complete_document_upload`. Link imports: `kaldrix://documents/link-import-workflow`; categories: `kaldrix://documents/categories`.
- Graphics: use a `list_graphic_library` item's `id` directly as `assetId` / `textureAssetId`; `copy_graphic_library_item` only for an own editable copy (check `copiedAssetIds` against duplicates). Own files: check `get_graphic_asset_limits`, upload via `upload_graphic_asset_file` or, above the direct limit, `request_graphic_asset_file_upload` + HTTP PUT; send the `fileId` to `create_graphic_asset`. No document uploads for catalog assets. READY MODEL_3D ids → `place_graphic_model`, or `set_opening_model` to replace a window or door's default look (all at once: `set_opening_models`); READY TEXTURE ids → `set_element_surface_texture`.
- Material lists: `itemKeys` from `get_project_material_list`. For a saved estimate read `get_estimate_purchase_resources`, not `get_project_purchase_report` (current model).
- `kaldrix://estimate/material-categories`, `kaldrix://estimate/units`, `kaldrix://estimate/vat-rates`: reference values for estimate and price-list work, not a public wholesaler catalog.
