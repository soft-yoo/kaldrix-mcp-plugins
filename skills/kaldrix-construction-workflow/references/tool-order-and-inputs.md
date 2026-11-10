# Tool order and input sources: delivery, documents and company data

Tool descriptions say only what each tool does. This page says where inputs come from and which
tool comes next. Building-model and estimate details are in the BIM and estimation skills.
Live schemas, `nextTools` and `fixTools` in results stay authoritative.

## Projects and company

- `get_projects`, then `get_project_detail` for buildings and floors. A single project's workflow:
  `get_project_workflow` (company-wide: `get_company_workflow`).
- Customers: show `lookup_company_by_nip` results to the user, then send confirmed data to `create_customer` or
  `update_customer`. A customer with history (CUSTOMER_IN_USE) is archived with `archive_customer`, not deleted;
  `restore_customer` takes ids from `list_customers` with `includeArchived=true`.
- Tasks: `list_tasks` gives `boardId`, `taskId` and valid `columnId` values for `create_task`, `update_task` and
  `delete_task`.
- Budget: `lineId` and `expenseId` come from `get_project_budget`.

## Contracts and annexes

- `create_project_contract`: `estimateId` from `list_project_contracts` `preparableEstimates` (accepted offers without a
  contract). Current version for edits: `get_project_contract`. After `send_project_contract`, read the client's answer
  with `get_project_contract`.
- `return_project_contract_to_draft`, correct with `update_project_contract` / `set_project_contract_tranches`, then
  send again (new link). A signed contract (`mark_project_contract_signed`) cannot be undone; the only way out is
  `cancel_project_contract`, after which `create_project_contract` prepares a new one from the same estimate.
- Wrong tranche payment: `clear_contract_tranche_payment`, then `record_contract_tranche_payment`.
- Annexes: `positionKey` and `sectionId` from `get_contract_annex_sources`; PRICE_LIST `recipeId` from
  `list_service_recipes` with the `priceListId` of the sources; `list_contract_annexes` shows the default VAT rate and
  whether a new annex is possible. With `includeModelProposal=true`, turn the proposal into a draft with
  `create_contract_annex_from_model` instead of editing the approved estimate. Review the draft with
  `list_contract_annexes`, send it with `send_contract_annex`, read the answer with `list_contract_annexes`.
  `get_estimate_annex_changes` takes `estimateId` from `list_project_estimates`.
- Remarks and versions: `contractId` from `list_project_contracts`, `annexId` from `list_contract_annexes`, `remarkId`
  for `answer_contract_remark` from `list_contract_remarks`. Issued agreements: ids from `list_issued_agreements`.

## Investor portal

- INVESTOR_PORTAL_LINK_ACTIVE on `create_investor_portal_link`: revoke the old link with `revoke_investor_portal_link`
  first (the softer alternative to `delete_investor_portal_link`). `recipientKey` comes from `get_investor_portal`.
- `reply_investor_site_message`: `subjectId` and `recipientKey` from `list_investor_site_messages` or `get_investor_portal`.

## Realization, acceptance and warranty

- `start_project_realization`: the recommended stage source is in `get_project_realization`. Before
  `complete_project_realization` check `get_project_realization` `completion`; a signed accepted FINAL protocol
  unlocks it.
- Acceptance: after the person on site confirms, sign the draft with `sign_acceptance_protocol`. `protocolId` from
  `get_project_acceptance`; version and `signBlockedBy` from `get_acceptance_protocol`.
- Punch items and warranty defects: status changes go through `change_punch_item_status` (version from
  `list_punch_items` or `get_punch_item`; `kind=WARRANTY` for warranty reports). Warranty versions:
  `get_project_warranty`. Formalities: kind and checklist version from `get_project_formalities`.

## Schedule, crews and site log

- `stageId` from `get_project_schedule` (also its version for `set_schedule_stage_progress`); `crewId` and
  `crewMemberId` from `list_crews` for `assign_crew_to_stage` and `add_time_entry`. Assignment ids and versions:
  `get_project_crew_assignments` or `get_work_plan`; time entry ids and versions: `get_project_time_sheet`.
- `add_construction_log_entry`: stages and today's stage (`defaultStageId`) from `list_construction_log_entries`.
  Entry ids (`entryId`, `entryIds`) also come from `list_construction_log_entries`.

## Invoices

- `assign_invoice`: references from `list_invoices_to_assign` or `list_invoices`; without `customers.write` it leaves
  the project customer unchanged. To set it later, `set_project_customer` needs `customers.write` and `confirm=true`.
  `confirm_invoice_match` confirms a suggestion from
  `get_project_invoicing` (with its `contractId`) that the user accepted. `set_invoices_stage`: invoice references from
  `list_invoices`, stage from `get_project_stage_settlement`; invoices must be assigned to the project first.
- `create_manual_invoice` from a photo or PDF: upload the original first (`upload_document`, or
  `request_document_upload` with `invoiceAttachment=true`), then send the READY document ids. Create a missing SALES
  customer with `create_customer`. Read amounts from the source, ask about unreadable values, never invent VAT, dates
  or parties.

## Documents, graphics and materials

- Documents: larger files go `request_document_upload` → HTTP PUT → `complete_document_upload`. Link imports:
  `kaldrix://documents/link-import-workflow`; categories: `kaldrix://documents/categories`.
- Graphics: use a `list_graphic_library` item directly by its `id` as `assetId` / `textureAssetId`; copy it with
  `copy_graphic_library_item` only to keep an own, editable copy (check `copiedAssetIds` to avoid duplicates). Own files: check `get_graphic_asset_limits`, upload with
  `upload_graphic_asset_file` or, above the direct limit, `request_graphic_asset_file_upload` + HTTP PUT; send the
  returned `fileId` to `create_graphic_asset`. Do not use document uploads for catalog assets. READY MODEL_3D ids go to
  `place_graphic_model` or — to replace the default look of a window or door — `set_opening_model` (all windows or doors at once: `set_opening_models`), READY TEXTURE ids to `set_element_surface_texture`.
- Material lists: `itemKeys` from `get_project_material_list`; quantities follow `get_estimate_purchase_resources`.
  For a saved estimate read `get_estimate_purchase_resources` instead of `get_project_purchase_report` (current model).
- Estimate material categories, units and VAT rates: `kaldrix://estimate/material-categories`,
  `kaldrix://estimate/units` and `kaldrix://estimate/vat-rates`. These reference values support estimate and
  price-list work; they are not a public wholesaler catalog.
