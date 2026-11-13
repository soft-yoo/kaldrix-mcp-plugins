## From the accepted offer to the keys

Every project moves through phases computed by the backend: PREPARATION (address, customer, model, estimate, offer) →
AGREEMENT (contract, start) → REALIZATION (construction stages, construction log, progress, invoices) → HANDOVER (final
acceptance) → CLOSED. Read `get_project_workflow`: it returns the phase, the phase bar (`phases`) and the next step
(`nextStepId`, e.g. CONTRACT, START, SCHEDULE, HANDOVER, CLOSE). Never infer the phase, a stage state, an amount or an instalment state yourself. The process follows the
company modules: a step of a module that is off disappears and never blocks the rest (calling its tool returns
`MODULE_DISABLED`).

Editing an acceptance protocol, punch item or warranty defect requires `realization.write`,
`realization.delete` and `confirm=true` after the user approves the change. This applies to
`update_acceptance_protocol`, `update_punch_item` and `update_warranty_defect`; creation remains separate.

1. Offer accepted: the client accepts the sent offer under the link (`get_estimate_offer_workflow.latestOffer`). The
   accepted estimate becomes the primary one (unless the primary estimate is already approved — then it is a second
   scope with its own contract); its prices and contract quantities are frozen and only executed quantities change
   afterwards (`update_estimate_sheet` with `actualQuantity`).
2. Contracts (OFFERS module, optional): a project can have several contracts — one per accepted offer, for different
   works and different clients, each with its own annexes. `list_project_contracts` lists them with totals, the project
   deadline (the latest one) and `preparableEstimates` (accepted offers without a contract);
   `create_project_contract` (projectId, optional estimateId) prepares a draft from that accepted offer: parties (the
   client of that offer), scope, remuneration, dates from the schedule and the proposed instalments: 20% advance at signing, 70% after the stages from estimate sections, 10% after
   the final handover. Adjust it with `update_project_contract` and `set_project_contract_tranches` (`useProposal=true`
   restores the proposal); every later tool takes the `contractId` (`get_project_contract` reads one contract);
   `sendBlockedBy` must be empty before sending. Then either `send_project_contract`
   (confirm=true, a message on behalf of the company; the client accepts online) or, after a paper signature,
   `mark_project_contract_signed` (`contracts.write`, confirm=true, cannot be undone). A sent contract goes back for
   corrections with `return_project_contract_to_draft` (confirm=true, the link stops working); `cancel_project_contract`
   cancels it (`contracts.delete`, confirm=true).
   Construction formalities (Polish Construction Law, a checklist — not legal advice): `get_project_formalities` lists
   what to settle before, during and after the works with due dates and warnings — e.g. building control (PINB) must be
   notified of the intended start before the works start, and the completion notice comes before moving in;
   `update_project_formality` records the state of one formality (`realization.write`). The workflow shows the
   most important missing one in `get_project_workflow.notices`; it never blocks a step.
3. Start of works: `get_project_realization` shows the start options and the recommended stage source;
   `start_project_realization` records the start and creates the construction stages: FROM_ESTIMATE (one stage per
   estimate section, so contract instalments after a section find their stage), FROM_PROJECT_PARTS, KEEP_EXISTING or
   NONE. The project moves to REALIZATION. The works may also start without an accepted offer (oral agreement, own
   build), but the workflow suggests the offer first.
4. Running the works: `get_project_schedule` (SCHEDULE module) returns the stages with dates, state, progress, forecast
   and delay. A stage from an estimate section takes its progress from the executed quantities of the accepted estimate;
   otherwise set it with `set_schedule_stage_progress` (manual value, or `fromEstimate=true` to go back). Construction log
   entries belong to a stage: `list_construction_log_entries` lists the stages and the one running today
   (`defaultStageId`) and filters by stage; send that `stageId` to `add_construction_log_entry`. Record instalment payments with `record_contract_tranche_payment` (confirm=true only after the user confirms the payment)
   and undo a wrong one with `clear_contract_tranche_payment` (confirm=true). A change of scope during the works
   (additional, replacement or omitted works, a later completion date) is an annex to the concluded contract — never an
   edit of the accepted estimate: `list_contract_annexes` (state, amounts, `defaultVatRate`), `get_contract_annex_sources`
   (contract positions with the quantity in force and the agreed price, sections, the estimate price list;
   `includeModelProposal=true` compares the current model), `create_contract_annex` (ADDITIONAL / OMITTED lines:
   `CONTRACT_POSITION` with `positionKey` at the contract price, `PRICE_LIST` with `recipeId` priced by the estimate engine,
   or `MANUAL` with unit prices in minor units; optional new schedule stage, `deadlineShiftDays`, payment mode,
   `idempotencyKey`) or `create_contract_annex_from_model` after model changes (instead of editing the approved estimate),
   then `send_contract_annex` (confirm=true, the client accepts or rejects it on the investor site page or the annex
   link) or, after a paper signature, `mark_contract_annex_signed` (`contracts.write`, confirm=true). An agreed annex
   changes the remuneration, the instalments and the completion date, the stage plan of the settlement and the material
   list (read the annex layer of the estimate with `get_estimate_annex_changes`); a rejected one changes nothing.
   Materials for the stages (ESTIMATES module): `get_project_material_list` returns the materials of the primary estimate
   (quantity to order, waste, packages — the same rule as `get_estimate_purchase_resources`) grouped by construction
   stage, with the nearest unfinished stage (`nextGroupKey`, filter `nextStageOnly=true`); after buying or a delivery
   mark them with `set_material_list_items_state` (BOUGHT, DELIVERED, TO_BUY). The state survives estimate changes; a
   changed quantity after the purchase shows as `quantityChanged`. PDF and the e-mail to a wholesaler stay in the app.
   Crews and site hours: crews and assignments need the SCHEDULE module, logging and reading site hours needs
   TIME_TRACKING (works without SCHEDULE; then entries have no stage). `list_crews` shows the company crews (people,
   trades, subcontractor account);
   `assign_crew_to_stage` puts a crew or one person on a stage (dates follow the stage unless given) and returns conflicts
   — the same crew or person on another site on the same days — as warnings; a labour-hour stage takes its duration from
   the assigned headcount (`scheduleChanged` — read the schedule again). `get_project_crew_assignments` lists the site's
   assignments. Log hours with `add_time_entry` (yourself, a person, a crew member or a whole crew with headcount) and read
   the totals per stage and person with `get_project_time_sheet` (hours only, no costs).
5. Stage acceptance: `create_acceptance_protocol` with kind STAGE and the `stageId`, fill it with
   `update_acceptance_protocol` (participants, notes, result), add defects with `create_punch_item` (required for
   ACCEPTED_WITH_DEFECTS), check `signBlockedBy` in `get_acceptance_protocol`, then `sign_acceptance_protocol`
   (`realization.write`, confirm=true after the person on site agrees). A signed STAGE protocol with ACCEPTED or
   ACCEPTED_WITH_DEFECTS finishes the stage (100%, `acceptedOn`) and the contract instalment after that stage becomes
   due (see `get_project_contract`). HIDDEN_WORKS protocols (e.g. reinforcement before concreting) are signed the same
   way and do not finish a stage.
6. Punch list: `list_punch_items` (views OPEN, TO_VERIFY, OVERDUE, MINE, CLOSED), `update_punch_item` and
   `change_punch_item_status` (MARK_FIXED, then VERIFY; REOPEN; REJECT with a reason).
7. Final acceptance: `create_acceptance_protocol` with kind FINAL moves the project to HANDOVER. Sign it with the result
   ACCEPTED or ACCEPTED_WITH_DEFECTS; the instalment after the final handover becomes due. Record how the investor
   confirmed a signed protocol (paper or scan) with `confirm_acceptance_protocol_by_investor`, also later.
8. Completion: `get_project_realization.completion` lists what still blocks it; `complete_project_realization`
   (`realization.write`, confirm=true; `confirmOpenPunchItems=true` only when the user agrees to finish despite open
   defects) moves the project to CLOSED. Only a person with `realization.delete` can undo it in the app.
9. Warranty: `get_project_warranty` shows the final acceptance date, the contract warranty and the statutory warranty
   (rękojmia, 5 years for building defects by default — information, not legal advice) with days left, the retention,
   warnings, the nearest deadline and a summary of the job (duration, contract end versus completion, price with annexes
   versus payments). `update_project_warranty` changes the periods and the retention, `record_warranty_retention_return`
   marks the retention returned. A defect reported after completion is `create_warranty_defect` (report date and
   reporter; `warrantyCoverage` tells WARRANTY, STATUTORY or NONE — after both periods), changed with
   `update_warranty_defect` and moved through `change_punch_item_status` like any defect; `list_punch_items(kind=WARRANTY)`
   lists them.

### Renovation without a drawing

A renovation or a pricing without a drawing is a project with `workType=RENOVATION` (`create_project`, or
`update_project` for an existing one). The MODEL step is optional, so the next step after the address is the estimate.
Price it with manual positions and enter quantities by hand, also as formulas such as `2*3,5*2,7` that the backend
computes. Everything after the offer (contract, start with stages from estimate sections, log, acceptance, completion)
is the same as for a new construction.

### Private investor accounts

A private investor (account type INVESTOR, a PERSONAL workspace) can build or renovate their own house.
PERSONAL and ORGANIZATION client accounts use the same plan rules, but a PERSONAL account starts with Offers,
Customers, Invoices, Tasks, Time tracking and History switched off (an ORGANIZATION account starts with History off);
the owner can enable them in Subscription → Features.
Investor portal tools depend on the account kind. Use the returned entitlement, enabled modules and
person's consented scopes; do not infer access from the account label. KSeF connection management stays in the app.
For the construction budget, BUDGET must be included and enabled by the account owner. New accounts get it in
Pro or Advanced; legacy entitlements may differ. The budget tools are `get_project_budget` (limit, stages,
costs, paid and to pay, notices), `add_budget_expense` (a cost in a stage or in shared costs, optionally with the first
payment; omit the amount while the final price is unknown) and `record_budget_payment` (another payment or a refund).
Amounts are gross minor units (grosze). Generate a new UUID `requestId` for each intended write and reuse it only for a
retry of the same data. The budget itself is created and planned in the app.
