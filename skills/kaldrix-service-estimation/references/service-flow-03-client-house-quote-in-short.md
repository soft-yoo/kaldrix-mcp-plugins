## Client house quote in short

1. `create_quick_house` creates the house and its primary estimate priced from the model (shell, finishing,
   installations) on the default price list — no need to price stages by hand.
2. After plan changes: a draft corrects its model quantities by itself — read `get_estimate_model_changes`, then
   `review_estimate_against_model` for what is left (removed and new elements, approved or offered estimates; see
   "The estimate follows the model"), and `price_estimate_from_model` for elements not priced yet.
3. Single positions: `quick_add_estimate_position` (service or own position, with or without a quantity), then
   `preview_estimate_position_scope` / `assign_estimate_position_scope` to measure it from the model, or
   `update_estimate_sheet` for a manual quantity, price or composition; `remove_estimate_position` removes it.
4. `get_estimate_checks` → fix what it lists (`fixTools`) → offer details with `update_project_estimate` (`offer`,
   `presentationMode` SIMPLE for the client) → `generate_estimate_pdf` → `send_estimate_offer` only when the user says so.
5. Negotiation (KDX-249): the client writes remarks on the offer page; read them with `list_estimate_offer_remarks`, answer
   with `answer_estimate_offer_remark` (a message the client sees — confirm with the user), revise the estimate
   (`revoke_estimate_offer` first) and send the next version with `send_estimate_offer`; `get_estimate_offer_versions`
   shows what changed. Only the current version can be accepted.
